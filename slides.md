---
highlighter: shiki
css: unocss
colorSchema: dark
transition: fade-out
mdc: true
layout: center
glowSeed: 4
title: 《Beyond XSS：探索網頁前端資安宇宙》 ch4-1~4-3
info: |
  ## 《Beyond XSS：探索網頁前端資安宇宙》 讀書會導讀：ch4-1~4-3
  - speaker：Monica
  - 《Beyond XSS：探索網頁前端資安宇宙》 讀書會：[Beyond-XSS-Book-Club](https://github.com/Tech-Book-Community/Beyond-XSS-Book-Club)
author: Monica
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
fonts:
  # basically the text
  sans: Robot Noto Sans
  # use with `font-serif` css class from UnoCSS
  serif: Robot Noto Serif
  # for code blocks, inline code, etc.
  mono: Fira Code
# slidev style reference: https://github.com/antfu/talks/tree/main/2024-06-13
---

# 《Beyond XSS：探索網頁前端資安宇宙》 ch4-1~4-3

## Same-origin policy、site、CORS 與跨來源資源

<div class='mt-10 opacity-60'>
<p>speaker：Monica</p>
<p>2024.12.26 @Tech-Book-Community</p>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

<style>
  h1{
    @apply font-bold
  }
  h2{
    @apply text-light-700;
  }
  .slidev-layout p{
    margin-top: 0px;
    margin-bottom: 0.5rem;
  }
</style>

---

```yaml
layout: intro
glowSeed: 15
glowOpacity: 0.3
```

# Hi, I'm Monica

<div class="opacity-80">

- 一年多經驗的前端工程師 <br>
- 常用技術：React、Next.js <br>
- 興趣：聽音樂、看漫畫、看小說 <br>
- 想學的很多，但學得很慢...

</div>

<!-- <div my-10 w-min flex="~ gap-1" items-center justify-center>
  <mdi:medium op50 ma text-xl/>
  <div><a href="https://medium.com/@linyawun031" target="_blank" class="border-none! font-300">Monica</a></div>
  <mdi:github op50 ma text-xl ml-4/>
  <div><a href="https://github.com/linyawun" target="_blank" class="border-none! font-300">Monica</a></div>
</div> -->

<!--
You can have `style` tag in markdown to override the style for the current page.
Learn more: https://sli.dev/features/slide-scope-style
-->

<!--
Here is another comment.
-->

---

```yaml
layout: center
glow: bottom
```

# 前次回顧：CSS injection 與只有 HTML 的攻擊方式

1. CSS injection 是什麼?
<div v-click='1' opacity-80>
攻擊者在網頁插入 style 標籤，放入惡意 CSS，就可偷到一些有敏感屬性的資料
</div>

---

```yaml
layout: center
```

# 前次回顧：CSS injection 與只有 HTML 的攻擊方式

2. CSS injection 怎麼達到攻擊?

<div v-click='1' opacity-80>

- 簡單版： CSS selector 選特定元素，搭配 background 屬性發請求
- 進階版
  - Unicode Range
  - 字體高度 + first-line + scrollbar
  - 連字 + scrollbar

</div>

---

```yaml
layout: center
```

# 前次回顧：CSS injection 與只有 HTML 的攻擊方式

3. 只有 HTML 的攻擊方式有哪些?
<div v-click='1' opacity-80>

- 反向標籤劫持(Reverse tabnabbing)
- meta 標籤重新導向
- 透過 iframe 的攻擊
- 透過表單的攻擊
- 懸掛標籤注入(Dangling Markup injection)
</div>

---

```yaml
layout: center
```

# 本次導讀 Ch4-1~4-3

## Same-origin policy、site、CORS 與跨來源資源

<p class="opacity-80">筆記工：Lois</p>

---

```yaml
layout: center
```

# 前言

- 瀏覽器安全模型： <span v-mark.red="1">不同網站</span>間無法互相存取
- Ch4 開始的攻擊方式：「想攻擊 A 網站但無法直接攻擊，需先找到 B 網站漏洞再攻擊 A 網站」

---

# 補充：URL 的組成

- URL 結構：`scheme://username:password@host:port/path?query#fragment`
  - scheme：指定瀏覽器要用哪種 protocol 請求資源，如 `http`、`ftp`
  - host：資源的主要地址，通常是 domain 或 IP 地址
    - domain：host 常見形式，最終會解析為 IP 地址，如 `www.example.com`
    - IP 地址： 代表伺服器實際位置，如 `192.168.1.1`
  - port：指定伺服器接受請求的埠號，沒顯示則使用協議預設 port
    - `http` 默認是 `80`，`https` 默認是 `443`
    - 如 `http://www.example.com` 使用默認 port `80`
  - path：資源在伺服器上的路徑，如 `/index.html`
  <div class='bg-light-800 rounded-lg p-2 mt-4'>
    <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/d/d6/URI_syntax_diagram.svg/2136px-URI_syntax_diagram.svg.png" alt="URL 結構" />
  </div>
  <div class='text-right'><a class='text-xs opacity-50 border-none!' href="https://en.wikipedia.org/wiki/URL" target="_blank">圖片來源</a></div>

---

# 補充：Domain(域名)的組成

domain 結構由右至左來看

1. 根域 (Root Domain)
   - domain 最右端，隱式表示為 `.`
   - 如 `www.example.com.` 真正 domain 全稱是 `www.example.com.root`
2. 頂級域 (Top-Level Domain, TLD)
   - 位於根域左側，但通常根域隱藏，所以通常 TLD 是網址最後一部分
   - 頂級網域可分為
     - 一般性頂級網域（general TLD, gTLD）：表示特定用途或類型，如 `.com`、`.net`
     - 國別頂級網域（country-code TLD, ccTLD）：表示特定國家或地區的域名，如 `.tw`、`.jp`

---

# 補充：Domain(域名)的組成

domain 結構由右至左來看

<div flex="~ gap-2" items-center >
<div class='w-2/3'>

3. 二級域名/次級域名 (Second-Level Domain, SLD)
   - 位於 TLD 左側，代表網站核心標識
   - 如 `example.com` 中的 `example` 是次級域名
4. 子域 (Subdomain)
   - 位於次級域左側，由域名所有者自定義，用來劃分不同功能
   - 如 `www.example.com` 中的 `www` 是 subdomain

</div>
<div class='w-1/3'>
  <div class='bg-light-800 rounded-lg p-2'>
    <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/d/d2/DNS_schema.svg/700px-DNS_schema.svg.png" alt="域名範例" />
  </div>
  <div class='text-right'><a class='text-xs opacity-50 border-none!' href="https://en.wikipedia.org/wiki/Domain_name" target="_blank">圖片來源</a></div>
</div>
</div>

<div class='text-sm opacity-60 mt-30'>
  Public suffix、Registrable domain、eTLD、eTLD+1 後面會提
</div>

---

# Origin 到底是什麼？

簡單但可能有些許錯誤的 origin 解釋

- origin 會看：scheme、host、port
- 若有個 URL 是 `https://huli.tw/abc`
  - scheme：`https`
  - host：`huli.tw`
  - port：`443`
- same origin：兩網站的 scheme、host、port 都相同

| A                     | B                          | Scheme 相同 | Port 相同 | Host 相同 | Same Origin |
| --------------------- | -------------------------- | ----------- | --------- | --------- | ----------- |
| `https://huli.tw/abc` | `https://huli.tw/hello/yo` | ✅          | ✅        | ✅        | ✅          |
| `https://huli.tw`     | `https://blog.huli.tw`     | ✅          | ✅        | ❌        | ❌          |

---

# Site 到底是什麼？

簡單但可能有些許錯誤的 site 解釋

- site 會看：scheme、host
- same site：兩網站 scheme、host 都相同
  - host 是否相同，會看 registrable domain

| A                     | B                          | Scheme 相同 | Host 相同 / 同 Registrable Domain | Same Site |
| --------------------- | -------------------------- | ----------- | --------------------------------- | --------- |
| `https://huli.tw/abc` | `https://huli.tw/hello/yo` | ✅          | ✅                                | ✅        |
| `https://huli.tw`     | `http://huli.tw`           | ❌          | ✅                                | ❌        |
| `http://huli.tw`      | `http://huli.tw:8080`      | ✅          | ✅                                | ✅        |
| `https://abc.huli.tw` | `https://blog.huli.tw`     | ✅          | 同 domain (`huli.tw`)             | ✅        |

---

# 細究 same origin

<div class='opacity-60 text-sm'>
  <p>HTML <a href="https://html.spec.whatwg.org/multipage/browsers.html#origin" target="_blank">spec</a>: "Origins are the fundamental currency of the web's security model. Two actors in the web platform that share an origin are assumed to trust each other and to have the same authority. Actors with differing origins are considered potentially hostile versus each other, and are isolated from each other to varying degrees."</p>
</div>

- origin 分兩種
  - An opaque origin：特殊狀況才會出現
    - 開本機網頁（`file:///...`）發 request 時，origin 會是 opaque origin
  - A tuple origin：主要關注的 origin
    - tuple origin 包含以下元素
      - scheme (an ASCII string)
      - host (a host)
      - port (null or a 16-bit unsigned integer)
      - domain (null or a domain). Null unless stated otherwise
    - tuple 型態如 `(https, huli.tw, null, null)`，可被序列化為字串 `https://huli.tw`

<!-- 若兩網站有相同 origin，代表可信任彼此；反之會被隔離且受限制 -->

<style>
  .slidev-layout p{ 
    line-height: 1.25rem;
  }
</style>

---

# 細究 same origin

- 依 spec，提到 same origin，可分兩種
  - same origin
  - same origin-domain
- 判斷 A 與 B origin 是否為 same origin 的演算法
  <div class='opacity-60 text-sm '>
    <p> Two origins, A and B, are said to be same origin if the following algorithm returns true: </p>
    <p> 1. If A and B are the same opaque origin, then return true. </p>
    <p> 2. If A and B are both tuple origins and their schemes, hosts, and port are identical, then return true. </p>
    <p> 3. Return false. </p>
  </div>

  - 兩種情況會是 same origin
    - 兩個都是 opaque origin
    - 兩個都是 tuple origin，且 scheme、host、port 都相等

<style>
  .slidev-layout p{ 
    margin-top: 0px;
    margin-bottom: 0.1rem;
    line-height: 1.25rem;
  }
</style>

---

# 細究 same origin

- same origin 舉例

---

# Code

Use code snippets and get the highlighting directly, and even types hover!

```ts {all|5|7|7-8|10|all} twoslash
// TwoSlash enables TypeScript hover information
// and errors in markdown code blocks
// More at https://shiki.style/packages/twoslash

import { computed, ref } from 'vue';

const count = ref(0);
const doubled = computed(() => count.value * 2);

doubled.value = 2;
```

<arrow v-click="[4, 5]" x1="350" y1="310" x2="195" y2="334" color="#953" width="2" arrowSize="1" />

<!-- This allow you to embed external code blocks -->

<<< @/snippets/external.ts#snippet

<!-- Footer -->

[Learn more](https://sli.dev/features/line-highlighting)

<!-- Inline style -->
<style>
.footnotes-sep {
  @apply mt-5 opacity-10;
}
.footnotes {
  @apply text-sm opacity-75;
}
.footnote-backref {
  display: none;
}
</style>

<!--
Notes can also sync with clicks

[click] This will be highlighted after the first click

[click] Highlighted with `count = ref(0)`

[click:3] Last click (skip two clicks)
-->

---

## level: 2

# Shiki Magic Move

Powered by [shiki-magic-move](https://shiki-magic-move.netlify.app/), Slidev supports animations across multiple code snippets.

Add multiple code blocks and wrap them with <code>````md magic-move</code> (four backticks) to enable the magic move. For example:

````md magic-move {lines: true}
```ts {*|2|*}
// step 1
const author = reactive({
  name: 'John Doe',
  books: [
    'Vue 2 - Advanced Guide',
    'Vue 3 - Basic Guide',
    'Vue 4 - The Mystery',
  ],
});
```

```ts {*|1-2|3-4|3-4,8}
// step 2
export default {
  data() {
    return {
      author: {
        name: 'John Doe',
        books: [
          'Vue 2 - Advanced Guide',
          'Vue 3 - Basic Guide',
          'Vue 4 - The Mystery',
        ],
      },
    };
  },
};
```

```ts
// step 3
export default {
  data: () => ({
    author: {
      name: 'John Doe',
      books: [
        'Vue 2 - Advanced Guide',
        'Vue 3 - Basic Guide',
        'Vue 4 - The Mystery',
      ],
    },
  }),
};
```

Non-code blocks are ignored.

```vue
<!-- step 4 -->
<script setup>
const author = {
  name: 'John Doe',
  books: [
    'Vue 2 - Advanced Guide',
    'Vue 3 - Basic Guide',
    'Vue 4 - The Mystery',
  ],
};
</script>
```
````

---

# Components

<div grid="~ cols-2 gap-4">
<div>

You can use Vue components directly inside your slides.

We have provided a few built-in components like `<Tweet/>` and `<Youtube/>` that you can use directly. And adding your custom components is also super easy.

```html
<Counter :count="10" />
```

<!-- ./components/Counter.vue -->
<Counter :count="10" m="t-4" />

Check out [the guides](https://sli.dev/builtin/components.html) for more.

</div>
<div>

```html
<Tweet id="1390115482657726468" />
```

<Tweet id="1390115482657726468" scale="0.65" />

</div>
</div>

<!--
Presenter note with **bold**, *italic*, and ~~striked~~ text.

Also, HTML elements are valid:
<div class="flex w-full">
  <span style="flex-grow: 1;">Left content</span>
  <span>Right content</span>
</div>
-->

---

## class: px-20

# Themes

Slidev comes with powerful theming support. Themes can provide styles, layouts, components, or even configurations for tools. Switching between themes by just **one edit** in your frontmatter:

<div grid="~ cols-2 gap-2" m="t-2">

```yaml
---
theme: default
---
```

```yaml
---
theme: seriph
---
```

<img border="rounded" src="https://github.com/slidevjs/themes/blob/main/screenshots/theme-default/01.png?raw=true" alt="">

<img border="rounded" src="https://github.com/slidevjs/themes/blob/main/screenshots/theme-seriph/01.png?raw=true" alt="">

</div>

Read more about [How to use a theme](https://sli.dev/guide/theme-addon#use-theme) and
check out the [Awesome Themes Gallery](https://sli.dev/resources/theme-gallery).

---

# Clicks Animations

You can add `v-click` to elements to add a click animation.

<div v-click>

This shows up when you click the slide:

```html
<div v-click>This shows up when you click the slide.</div>
```

</div>

<br>

<v-click>

The <span v-mark.red="3"><code>v-mark</code> directive</span>
also allows you to add
<span v-mark.circle.orange="4">inline marks</span>
, powered by [Rough Notation](https://roughnotation.com/):

```html
<span v-mark.underline.orange>inline markers</span>
```

</v-click>

<div mt-20 v-click>

[Learn more](https://sli.dev/guide/animations#click-animation)

</div>

---

# Motions

Motion animations are powered by [@vueuse/motion](https://motion.vueuse.org/), triggered by `v-motion` directive.

```html
<div
  v-motion
  :initial="{ x: -80 }"
  :enter="{ x: 0 }"
  :click-3="{ x: 80 }"
  :leave="{ x: 1000 }"
>
  Slidev
</div>
```

<div class="w-60 relative">
  <div class="relative w-40 h-40">
    <img
      v-motion
      :initial="{ x: 800, y: -100, scale: 1.5, rotate: -50 }"
      :enter="final"
      class="absolute inset-0"
      src="https://sli.dev/logo-square.png"
      alt=""
    />
    <img
      v-motion
      :initial="{ y: 500, x: -100, scale: 2 }"
      :enter="final"
      class="absolute inset-0"
      src="https://sli.dev/logo-circle.png"
      alt=""
    />
    <img
      v-motion
      :initial="{ x: 600, y: 400, scale: 2, rotate: 100 }"
      :enter="final"
      class="absolute inset-0"
      src="https://sli.dev/logo-triangle.png"
      alt=""
    />
  </div>

  <div
    class="text-5xl absolute top-14 left-40 text-[#2B90B6] -z-1"
    v-motion
    :initial="{ x: -80, opacity: 0}"
    :enter="{ x: 0, opacity: 1, transition: { delay: 2000, duration: 1000 } }">
    Slidev
  </div>
</div>

<!-- vue script setup scripts can be directly used in markdown, and will only affects current page -->
<script setup lang="ts">
const final = {
  x: 0,
  y: 0,
  rotate: 0,
  scale: 1,
  transition: {
    type: 'spring',
    damping: 10,
    stiffness: 20,
    mass: 2
  }
}
</script>

<div
  v-motion
  :initial="{ x:35, y: 30, opacity: 0}"
  :enter="{ y: 0, opacity: 1, transition: { delay: 3500 } }">

[Learn more](https://sli.dev/guide/animations.html#motion)

</div>

---

# LaTeX

LaTeX is supported out-of-box. Powered by [KaTeX](https://katex.org/).

<div h-3 />

Inline $\sqrt{3x-1}+(1+x)^2$

Block

$$
{1|3|all}
\begin{aligned}
\nabla \cdot \vec{E} &= \frac{\rho}{\varepsilon_0} \\
\nabla \cdot \vec{B} &= 0 \\
\nabla \times \vec{E} &= -\frac{\partial\vec{B}}{\partial t} \\
\nabla \times \vec{B} &= \mu_0\vec{J} + \mu_0\varepsilon_0\frac{\partial\vec{E}}{\partial t}
\end{aligned}
$$

[Learn more](https://sli.dev/features/latex)

---

# Diagrams

You can create diagrams / graphs from textual descriptions, directly in your Markdown.

<div class="grid grid-cols-4 gap-5 pt-4 -mb-6">

```mermaid {scale: 0.5, alt: 'A simple sequence diagram'}
sequenceDiagram
    Alice->John: Hello John, how are you?
    Note over Alice,John: A typical interaction
```

```mermaid {theme: 'neutral', scale: 0.8}
graph TD
B[Text] --> C{Decision}
C -->|One| D[Result 1]
C -->|Two| E[Result 2]
```

```mermaid
mindmap
  root((mindmap))
    Origins
      Long history
      ::icon(fa fa-book)
      Popularisation
        British popular psychology author Tony Buzan
    Research
      On effectiveness<br/>and features
      On Automatic creation
        Uses
            Creative techniques
            Strategic planning
            Argument mapping
    Tools
      Pen and paper
      Mermaid
```

```plantuml {scale: 0.7}
@startuml

package "Some Group" {
  HTTP - [First Component]
  [Another Component]
}

node "Other Groups" {
  FTP - [Second Component]
  [First Component] --> FTP
}

cloud {
  [Example 1]
}

database "MySql" {
  folder "This is my folder" {
    [Folder 3]
  }
  frame "Foo" {
    [Frame 4]
  }
}

[Another Component] --> [Example 1]
[Example 1] --> [Folder 3]
[Folder 3] --> [Frame 4]

@enduml
```

</div>

Learn more: [Mermaid Diagrams](https://sli.dev/features/mermaid) and [PlantUML Diagrams](https://sli.dev/features/plantuml)

---

foo: bar
dragPos:
square: 691,32,167,\_,-16

---

dragPos:
square: -106,0,0,0

---

dragPos:
square: -106,0,0,0

---

dragPos:
square: -106,0,0,0

---

dragPos:
square: -106,0,0,0

---

dragPos:
square: -106,0,0,0

---

dragPos:
square: -106,0,0,0

---

dragPos:
square: -106,0,0,0

---

dragPos:
square: -106,0,0,0

---

dragPos:
square: -106,0,0,0

---

dragPos:
square: -106,0,0,0

---

dragPos:
square: -106,0,0,0

---

dragPos:
square: -106,0,0,0

---

dragPos:
square: -106,0,0,0

---

# Draggable Elements

Double-click on the draggable elements to edit their positions.

<br>

###### Directive Usage

```md
<img v-drag="'square'" src="https://sli.dev/logo.png">
```

<br>

###### Component Usage

```md
<v-drag text-3xl>
  <div class="i-carbon:arrow-up" />
  Use the `v-drag` component to have a draggable container!
</v-drag>
```

<v-drag pos="663,206,261,_,-15">
  <div text-center text-3xl border border-main rounded>
    Double-click me!
  </div>
</v-drag>

<img v-drag="'square'" src="https://sli.dev/logo.png">

###### Draggable Arrow

```md
<v-drag-arrow two-way />
```

<v-drag-arrow pos="67,452,253,46" two-way op70 />

---

src: ./pages/imported-slides.md
hide: false

---

---

# Monaco Editor

Slidev provides built-in Monaco Editor support.

Add `{monaco}` to the code block to turn it into an editor:

```ts {monaco}
import { ref } from 'vue';
import { emptyArray } from './external';

const arr = ref(emptyArray(10));
```

Use `{monaco-run}` to create an editor that can execute the code directly in the slide:

```ts {monaco-run}
import { version } from 'vue';
import { emptyArray, sayHello } from './external';

sayHello();
console.log(`vue ${version}`);
console.log(
  emptyArray<number>(10).reduce(
    (fib) => [...fib, fib.at(-1)! + fib.at(-2)!],
    [1, 1]
  )
);
```

---

layout: center
class: text-center

---

# Learn More

[Documentation](https://sli.dev) · [GitHub](https://github.com/slidevjs/slidev) · [Showcases](https://sli.dev/resources/showcases)

<PoweredBySlidev mt-10 />
