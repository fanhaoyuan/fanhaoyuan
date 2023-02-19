---
title: 【JS】属于你的 Markdown 文本渲染器
---

# 属于你的 Markdown 文本渲染器

假设现在有一段 markdown 文本，想在浏览器中正常渲染这段文本，这时我们就需要一个 markdown 文本渲染器把这段 markdown 文本转为正确的 HTML 文本来渲染。

如果我们直接使用 markdown-it 来解析，那可以直接转为 HTML 文本，为什么我们还要去做这个渲染器呢？因为想通过一些自定义组件来拓展我们的渲染器，成为一个不一样的 markdown 文本渲染器。

## 解析 markdown 文本

我们使用知名的 markdown-it 来解析 markdown 文本。我们通过 markdown-it 解析获得 MarkdownIt Tokens

```ts
import Markdown from 'markdown-it';

const tokens = new Markdown().parse(markdown, {});
```

通过上面简单的两行代码，我们就可以得到解析后的 MarkdownIt Tokens

我们先保留着这个 tokens 来备用

## 定义 Tokens

我们需要定义一个属于自己的 Token 类型，通过转换来生成 AST

首先把 Token 类型分为三种

```ts
export type MarkdownTokenType = 'element' | 'text' | 'code_block';

export interface MarkdownToken {
    type: MarkdownTokenType;
    children?: MarkdownToken[];
    attrs?: Record<string, string>;
}
```

### Element 类型

这个类型说明这个 Token 是一个元素类型，这个类型应该是可以渲染成一个 HTML 标签的

```ts
export interface MarkdownElement extends MarkdownToken {
    tag: string;
    type: 'element';
    attrs: MarkdownRendererComponentAttrs;
}
```

### Text 类型

这个类型说明这个 Token 是一个纯文本，属于在行内渲染的类型，不会生成一个 HTML 标签的

```ts
export interface MarkdownText extends MarkdownToken {
    content: string;
    type: 'text';
}
```

### CodeBlock 类型

这个类型说明这个 Token 是一个代码块，其实他是一个特殊的 Element 类型，它也会渲染一个元素，不过这个元素是固定为 pre

```ts
export interface MarkdownCodeBlock extends MarkdownToken {
    tag: 'pre';
    content: string;
    type: 'code_block';
    attrs: MarkdownRendererComponentAttrs;
}
```

转换 MarkdownIt Token 为 Markdown Token

因为 markdown-it 解析出来的 tokens 是一组一组的，所以我们要转为 AST，方便在后面渲染。

````ts
/**
 * 解析 markdown 文本为 tokens
 */
export function parse(md: string): MarkdownToken[] {
    const stack: MarkdownToken[] = [];

    function _createElement(
        tag: string,
        attrs: Record<string, string> = {},
        children?: MarkdownToken[]
    ): MarkdownElement {
        return { tag, type: 'element', attrs, children };
    }

    function _createText(content: string): MarkdownText {
        return { type: 'text', content };
    }

    function _createCodeBlock(content: string, attrs: Record<string, string> = {}): MarkdownCodeBlock {
        return { type: 'code_block', content, tag: 'pre', attrs };
    }

    function _appendChildInStackTop(...children: MarkdownToken[]) {
        const parent = stack.at(-1);

        if (parent) {
            if (parent.children) {
                parent.children.push(...children);
            } else {
                parent.children = children;
            }
        }
    }

    return (function _parse(originTokens: Token[]): MarkdownToken[] {
        return originTokens.reduce<MarkdownToken[]>((tokens, token) => {
            const { type, tag, children, content, markup, attrs, info } = token;

            const _attrs = attrs ? Object.fromEntries(attrs) : {};

            /**
             * 解析开始标签
             *
             * 有开始标签，肯定有闭合标签
             *
             * 所以应该解析完整个 group 才算结束
             */
            if (/_open$/.test(type)) {
                // 入栈 因为 group 解析还没结束
                stack.push(_createElement(tag, _attrs));
                return tokens;
            }

            /**
             * 解析关闭标签
             *
             * 因为是关闭标签，所以栈里面一定会有 group 的开始标签
             *
             * 所以最后一个元素应该出栈
             */
            if (/_close$/.test(type)) {
                const _token = stack.pop();

                // 如果栈里没有元素，则证明 originToken 有错误，抛出错误
                if (!_token) {
                    throw new Error('Markdown 解析错误');
                }

                // 如果栈是空的，证明最外层的 group 已经解析完成，推入主 token 中
                if (!stack.length) {
                    return [...tokens, _token];
                }

                // 如果栈不是空的，证明最外层的 group 仍然没解析完成，还有别的子节点，推到上层的 group 中的 children 里
                _appendChildInStackTop(_token);

                return tokens;
            }

            /**
             * 解析行内元素
             *
             * 通常是 markup 后的元素
             *
             * 主要是内容部分
             */
            if (type === 'inline') {
                // 如果有子元素，则进行深度解析
                if (children?.length) {
                    const _tokens = _parse(children);

                    // 解析后的内容一定是最后入栈的元素，因为 inline 类型是紧跟 *_open 的元素
                    _appendChildInStackTop(..._tokens);

                    return tokens;
                }

                // 解析当前节点
                return [...tokens, _createElement(tag, _attrs)];
            }

            /**
             * 解析文本类型的元素
             *
             * 如果是空文本元素，则不解析
             */
            if (type === 'text' && content) {
                const _token = _createText(content);

                // 解析后的内容一定是最后入栈的元素，因为 text 类型是在 inline 的元素中的
                _appendChildInStackTop(_token);

                return tokens;
            }

            /**
             * 解析行内代码
             */
            if (type === 'code_inline') {
                const _token = _createElement('code', {}, [_createText(content)]);

                // 解析后的内容一定是最后入栈的元素，因为 code_inline 类型是在 inline 的元素中的
                _appendChildInStackTop(_token);

                return tokens;
            }

            /**
             * 解析代码块类型的元素
             */
            if (type === 'fence' && markup === '```') {
                return [...tokens, _createCodeBlock(content, { ..._attrs, language: info })];
            }

            // 更多元素的判断

            /**
             * 其余未做判断的元素
             *
             * 则一律不解析
             */
            return tokens;
        }, []);
    })(new Markdown().parse(md, {}));
}
````

通过上面这个转换，我们可以元素出现的组生成出对应的 AST

## 局部状态

我们通过 `vc-state` 来创建一个局部状态

```tsx
import { createContext } from 'vc-state';

const defaultComponents = {};

/**
 * Markdown 渲染器 context
 */
const [MarkdownRendererContextProvider, useMarkdownRendererContext] = createContext(
    (props: MarkdownRendererContextProviderProps) => {
        const { components = {}, namespace = 'renderer' } = props;

        const dynamicComponents = Object.assign({}, defaultComponents, components);

        function createNamespace(...names: string[]) {
            return names.reduce((t, c) => `${t}-${c}`, namespace);
        }

        return {
            namespace,
            createNamespace,
            dynamicComponents,
        };
    }
);

export { MarkdownRendererContextProvider, useMarkdownRendererContext };
```

通过注册 MarkdownRendererContextProvider，我们可以在每个自定义组件中使用共享状态。

## 动态渲染组件

通过 tag 名称来渲染对应的自定义组件，并把 attrs 传到对应的渲染函数中

```tsx
const DynamicRenderer = defineComponent({
    name: 'DynamicRenderer',
    props: {
        tag: {
            type: String,
            required: true,
        },
        attrs: {
            type: Object as PropType<Record<string, string>>,
            default: () => ({}),
        },
    },
    setup(props, { slots }) {
        const { attrs } = toRefs(props);

        const { dynamicComponents } = useMarkdownRendererContext();

        const createComponent = dynamicComponents[props.tag] || dynamicComponents['p'];

        return () => h(createComponent(attrs.value), slots.default);
    },
});
```

## 自定义渲染组件

我们在上文的局部状态中共享了一个 `defaultComponents`

```tsx
const defaultComponents: MarkdownRendererComponents = {
    h1: attrs => <Heading level={1} {...attrs} />,
    h2: attrs => <Heading level={2} {...attrs} />,
    h3: attrs => <Heading level={3} {...attrs} />,
    h4: attrs => <Heading level={4} {...attrs} />,
    h5: attrs => <Heading level={5} {...attrs} />,
    h6: attrs => <Heading level={6} {...attrs} />,
    blockquote: attrs => <Blockquote {...attrs} />,
    pre: attrs => <Pre {...attrs} />,
    a: attrs => <Link {...attrs} />,
    code: attrs => <Code {...attrs} />,
    p: attrs => <Paragraph {...attrs} />,
};
```

我们在使用 MarkdownRenderer 的时候，通过覆盖自定义组件工厂函数，来动态渲染对应的自定义组件。

### useMarkdownRendererContext

在自定义组件中，可以使用 useMarkdownRendererContext 来获取渲染器的上下文

```tsx
import { defineComponent } from 'vue';
import { useMarkdownRendererContext } from 'context';

// Heading.tsx
export const Heading = defineComponent({
    name: 'Heading',
    setup(props, { slots }) {
        const context = useMarkdownRendererContext();

        return () => <h1 class={`${context.namespace}-heading`}>{slots.default?.()}</h1>;
    },
});

// App.tsx

export default defineComponent({
    name: 'App',
    setup() {
        const md = '## Hello World';

        return () => {
            return (
                <MarkdownRendererContextProvider components={{ h1: attrs => <Heading {...attrs} /> }}>
                    <MarkdownRenderer content={md} />
                </MarkdownRendererContextProvider>
            );
        };
    },
});
```

## 以 MarkdownToken 渲染

我们定义一个 MarkdownTokenRenderer， 可以通过 MarkdownToken 来渲染组件，我们可以通过修改 MarkdownToken 来局部更新组件，不需要全局更新组件。适合频繁改动的 AST。

```tsx
export const MarkdownTokenRenderer = defineComponent({
    name: 'MarkdownTokenRenderer',
    props: {
        tokens: {
            type: Array as PropType<MarkdownToken[]>,
            default: () => [],
        },
    },
    setup(props) {
        const { tokens } = toRefs(props);

        const { namespace } = useMarkdownRendererContext();

        return () => {
            const elements = (function render(t: MarkdownToken[]) {
                return t.map(item => {
                    if (isMarkdownText(item)) {
                        return item.content;
                    }

                    if (isMarkdownCodeBlock(item)) {
                        return <DynamicRenderer tag='pre' attrs={{ ...item.attrs, content: item.content }} />;
                    }

                    if (isMarkdownElement(item)) {
                        return (
                            <DynamicRenderer tag={item.tag} attrs={item.attrs}>
                                {item.children?.length && render(item.children)}
                            </DynamicRenderer>
                        );
                    }

                    return null;
                });
            })(tokens.value);

            return <div class={namespace}>{elements}</div>;
        };
    },
});
```

## 以 Markdown 文本渲染

如果不是频繁改动的场景，可以直接使用 MarkdownRenderer 来渲染

```tsx
import { defineComponent, toRefs, computed } from 'vue';
import { parse } from './parse';
import { MarkdownTokenRenderer } from './MarkdownTokenRenderer';

/**
 * Markdown 渲染结果组件
 */
export const MarkdownRenderer = defineComponent({
    name: 'MarkdownRenderer',
    props: {
        content: {
            type: String,
            default: '',
        },
    },
    setup(props) {
        const { content } = toRefs(props);

        const tokens = computed(() => parse(content.value));

        return () => <MarkdownTokenRenderer tokens={tokens.value} />;
    },
});
```

## 总结

到了这里，我们已经完成了一个简单的 Markdown 文本渲染器了，我们在文中并没有实现自定义组件，如果有兴趣，可以一起研究一下。

## 延伸阅读

-   [vc-state -- Easily to compose scoped state in Vue.js](https://github.com/fanhaoyuan/vc-state)
