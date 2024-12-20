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

# 補充：Domain（域名）的組成

domain 結構由右至左來看

1. 根域（Root Domain）
   - domain 最右端，隱式表示為 `.`
   - 如 `www.example.com.` 真正 domain 全稱是 `www.example.com.root`
2. 頂級域（Top-Level Domain, TLD）
   - 位於根域左側，通常根域隱藏，因此通常 TLD 是網址最後一部分
   - 頂級網域可分為
     - 一般性頂級網域（general TLD, gTLD）：表示特定用途或類型，如 `.com`、`.net`
     - 國別頂級網域（country-code TLD, ccTLD）：表示特定國家或地區的域名，如 `.tw`、`.jp`

---

# 補充：Domain（域名）的組成

domain 結構由右至左來看

<div flex="~ gap-2" items-center >
<div class='w-2/3'>

3. 二級域名/次級域名（Second-Level Domain, SLD）
   - 位於 TLD 左側，代表網站核心標識
   - 如 `example.com` 中的 `example` 是次級域名
4. 子域（Subdomain）
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

<div class='quote'>
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
      - 若 tuple 和序列化後字串傳達資訊類似，會以序列化後字串呈現

<!-- 若兩網站有相同 origin，代表可信任彼此；反之會被隔離且受限制 -->

---

# 細究 same origin

- 依 spec，提到 same origin，可分兩種
  - same origin
  - same origin-domain
- 判斷 A 與 B origin 是否為 same origin 的演算法
  <div class='quote'>
    <p> Two origins, A and B, are said to be same origin if the following algorithm returns true: </p>
    <p> 1. If A and B are the same opaque origin, then return true. </p>
    <p> 2. If A and B are both tuple origins and their schemes, hosts, and port are identical, then return true. </p>
    <p> 3. Return false. </p>
  </div>

  - same origin 可分兩種情況
    - 都是 opaque origin
    - 都是 tuple origin，且 scheme、host、port 都相等

---

# 細究 same origin

- same origin 舉例
  - `https://huli.tw/api` origin 是 `https://huli.tw`
    - `https://huli.tw/*` 才會和它 same origin
  - `https://blog.huli.tw` origin 是 `https://blog.huli.tw`
    - 和 `https://huli.tw` host 不同，不是 same origin
- same domain 在規範的定義和前面簡單版解釋差在哪？
  - origin 定義
    - 多了 opaque origin 這種 origin
    - tuple origin 多了 domain 元素
  - same origin
    - 多了 same origin-domain

---

# 細究 same site

<div class='quote'>
  <p>HTML <a href="https://html.spec.whatwg.org/multipage/browsers.html#sites" target="_blank">spec</a>: "A site is an opaque origin or a scheme-and-host."</p>
</div>

- site 分兩種
  - opaque origin
  - scheme-and-host
- 依 spec，提到 same site，可分兩種
  - same site: 會看 scheme
  - schemelessly same site: 不看 scheme

---

# 細究 same site

- 判斷 A 與 B origin 是否為 same site<span class='text-#2f96ad text-xs ml-2'><Link to='additionalInfo' class='border-none!'>\[1\]</Link></span>
  <div class='quote'>
    <p> Two origins, A and B, are said to be same site if both of the following statements are true: </p>
    <p> 1. A and B are schemelessly same site </p>
    <p> 2. A and B are either both opaque origins, or both tuple origins with the same scheme </p>
  </div>

  - same site 可分兩種情況
    - 都是 opaque origin
    - 都是 schemelessly same site，且有相同 scheme

---

# 細究 same site

- same site 定義的發展史
  - same site 一開始不看 scheme：2016 <a href='https://datatracker.ietf.org/doc/html/draft-west-first-party-cookies-07#section-2.1' target='_blank'>RFC: Same-site Cookies</a> 中，same site 判斷不包含 scheme
  - 2019 年 6 月開始<a href='https://github.com/w3c/webappsec-fetch-metadata/issues/34' target='_blank'>討論</a>是否將 scheme 納入
  - 2019 年 9 月在此 <a href='https://github.com/whatwg/url/pull/449' target='_blank'>PR</a> 中，正式在規格中將 scheme 納入
    - same site 定義為「會看 scheme」
    - 不看 scheme 的稱為 schemelessly same site
  - 2020 年 11 月 Chrome 的文章 <a href='https://web.dev/articles/schemeful-samesite' target='_blank'>Schemeful Same-Site</a> 顯示瀏覽器當時仍把不同 scheme 視為 same site
  - 2020 年，Firefox issue <a href='https://bugzilla.mozilla.org/show_bug.cgi?id=1651119' target='_blank'>[meta] Enable cookie sameSite schemeful</a> 的 open 狀態顯示預設還沒把 scheme 納入 same site 考量
  - 2021 年 3 月 Chrome 發布 Chrome 89，將 <a href='https://chromestatus.com/feature/5096179480133632' target='_blank'>scheme 列入 same site 判斷</a>

---

# 細究 same site

- 判斷 A 與 B origin 是否為 schemelessly same site
  <div class='quote'>
    <p> Two origins, A and B, are said to be schemelessly same site if the following algorithm returns true: </p>
    <p> 1. If A and B are the same opaque origin, then return true. </p>
    <p> 2. If A and B are both tuple origins, then: </p>
    <p class='ml-4'>1. Let hostA be A's host, and let hostB be B's host. </p>
    <p class='ml-4'>2. If hostA equals hostB and hostA's registrable domain is null, then return true. </p>
    <p class='ml-4'>3. If hostA's registrable domain equals hostB's registrable domain and is non-null, then return true. </p>
    <p>3. Return false.</p>
  </div>

  - schemelessly same site 可分兩種情況
    - 都是 opaque origin
    - 都是 tuple origin，且 host 相同
      - 判斷 host 是否相同，會看 registrable domain

---

# 細究 same site

之 registrable domain 是什麼

- registrable domain 定義
  <div class='quote'>
    <p>URL <a href="https://url.spec.whatwg.org/#host-registrable-domain" target="_blank">spec</a>: "A host’s registrable domain is a domain formed by the most specific public suffix, along with the domain label immediately preceding it, if any"<span class='text-#2f96ad text-xs ml-2'><Link to='additionalInfo' class='border-none!'>[2]</Link></span></p> 
  </div>

  - registrable domain 舉例

    | Host            | Registrable Domain |
    | --------------- | ------------------ |
    | `blog.huli.tw`  | `huli.tw`          |
    | `huli.tw`       | `huli.tw`          |
    | `bob.github.io` | `bob.github.io`    |

---

# 細究 same site

之 registrable domain 是什麼

- 為何有的 registrable domain 是 `xxx.xxx.xx`、有的是 `xxx.xx` ?
  - 如果沒有「registrable domain」和「public suffix」…
    - `huli.tw` 跟 `blog.huli.tw` 被視為 same site ✅
    - `bob.github.io` 跟 `alice.github.io` 被視為 same site 🔺
      - `github.io` 是 GitHub pages 服務，每個 GitHub 使用者都有自己專屬的 subdomain
      - `bob.github.io` 和 `alice.github.io` 屬於不同使用者，希望能被視為兩個獨立網站
        - -> 「public suffix」 讓他們可被視為獨立網站

---

# 細究 same site

之 registrable domain 是什麼

- public suffix：一個人工維護的<a href='https://publicsuffix.org/list/public_suffix_list.dat' target='_blank'>清單</a>，有「不想被當作是同個網站的列表」
  - public suffix 如：`github.io`、`com.tw`、`s3.amazonaws.com`、`herokuapp.com`
  - public suffix 也稱為 eTLD（effective Top-Level-Domain）<span class='text-sm opacity-80'>(<a href='https://blog.kalan.dev/2021-11-09-url-and-samesite' target='_blank'>ref</a>)</span>
- 瀏覽器參考 public suffix 後，才決定 registrable domain 是什麼
  - registrable domain 不同，依 spec 定義，不是 same site
    | Host | Public Suffix | Registrable Domain |
    |----------------------|---------------------|---------------------|
    | `bob.github.io` | `github.io` | `github.io` |
    | `alice.github.io` | `github.io` | `github.io` |

---

# 細究 same site

之 registrable domain 是什麼

- registrable domain 與 public suffix <a href='https://url.spec.whatwg.org/#host-registrable-domain' target='_blank'>舉例表格</a>
  <img src='/images/registrable-domain-and-public-suffix-example.png' alt='registrable domain 與 public suffix 舉例表格' class='w-[650px]' />

---

# 細究 same site

再回到 same site

- same site 在規範的定義和前面簡單版解釋差在哪？
  - host 看起來像在同 parent domain，不代表是 same site
    - 兩個 host 是否 same site，要看 registrable domain，registrable domain 要看 public suffix
    - 舉例
      - `bob.github.io` 和 `alice.github.io` 的 registrable domain 不同，不是 same site
      - `blog.huli.tw`、`huli.tw` 和 `test.huli.tw` 的 registrable domain 相同，是 same site

---

# Same origin 與 same site

<div flex="~ gap-2" >
<div class='w-3/4'>

- same origin 會看：scheme、port、host
- same site 會看：scheme、host（registrable domain）
- 兩網站是 same origin，則一定是 same site

</div>
<div class='w-1/4'>
  <img src='/images/same-origin-and-same-site.png' alt='same origin 與 same site 的關係' class='w-[150px]' />
</div>
</div>

- same origin 與 same site 的差別
  | 差異項目 | Same Origin | Same Site |
  |-----------------|-------------------------------------|---------------------------------------|
  | 是否考慮 port | 是 | 否 |
  | 是否考慮 host | 是 | 不完全是（只看 registrable domain） |

---

# 神奇的 document.domain

- origin spec 中還有東西還沒提到
  - tuple origin 定義包含的 domain 屬性
  - same origin-domain
- tuple origin 定義包含的 domain 屬性
  <div class='quote'>
    <p>HTML <a href="https://html.spec.whatwg.org/multipage/browsers.html#origin" target="_blank">spec</a>: "Origins can be shared, e.g., among multiple Document objects. Furthermore, origins are generally immutable. Only the domain of a tuple origin can be changed, and only through the document.domain API."</p> 
  </div>

  - origin 除 domain 屬性外，其他都不可變(immutable)
    - domain 屬性可用 `document.domain` 改變

---

# 神奇的 document.domain

- `document.domain` demo
  - Step 1. 修改本機 `/etc/hosts`，讓兩個網址都連到 localhost
    ```
    127.0.0.1   alice.example.com
    127.0.0.1   bob.example.com
    ```
  - Step 2. 寫個 server 在 `localhost:5555` 運行
    - 頁面功能：載入 iframe、讀取 iframe 內 DOM 資料、改變 `document.domain`

<div class="ml-12">
```html {*}{maxHeight:'200px'}
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content ="width=device-width, initial-scale=1" />
  </head>
  <body>
    <h1></h1>
    <h2></h2>
    <button onclick="load('alice')">load alice iframe</button>
    <button onclick="load('bob')">load bob iframe</button>
    <button onclick="access()">access iframe content</button>
    <button onclick="update()">update domain</button>
    <br>
    <br>
  </body>
  <script>
    const name = document.domain.replace('.example.com', '')
    document.querySelector('h1').innerText = name
    document.querySelector('h2').innerText = Math.random()

    function load(name) {
      const iframe = document.createElement('iframe')
      iframe.src = 'http://' + name + '.example.com:5555'
      document.body.appendChild(iframe)
    }

    function access() {
      const win = document.querySelector('iframe').contentWindow
      alert('secret:' + win.document.querySelector('h2').innerText)
    }

    function update() {
      document.domain = 'example.com'
    }

  </script>
</html>
```
</div>

---

# 神奇的 document.domain

- `document.domain` demo
  - Step 3. 開啟 `http://alice.example.com:5555`，點「load bob iframe」載入 `http://bob.example.com:5555` iframe，再按 alice 頁面的 「access iframe content」
    - 錯誤原因：要跨越 iframe 存取 DOM，必須是 same origin
      <img src='https://aszx87410.github.io/beyond-xss/assets/images/20-04-290308aa6bce3f68805c523b6df75761.png' alt='access iframe content fail' class='w-[450px]' />

---

# 神奇的 document.domain

- `document.domain` demo
  - Step 4. 按下 alice 和 bob 頁面的「update domain」，再按 alice 頁面的「access iframe content」
    - 將 `http://alice.example.com:5555` 跟 `http://bob.example.com:5555` 從 cross origin 變成 same origin
    - 成功取得 bob 頁面資料 <br>
      -> same site 變成 same origin
      <img src='https://aszx87410.github.io/beyond-xss/assets/images/20-05-3b2e6e0865aa2cd1c254c8b23649dc9b.png' alt='access iframe content success' class='w-[300px]' />

---

# 神奇的 document.domain

- cross origin 變成 same origin 有很多限制
  - 只有 same site 網站可以
  - 設置時會檢查
    <div class='quote'>
      <p>The domain setter steps are:</p> 
      <p>1. If this's browsing context is null, then throw a "SecurityError" DOMException.</p>
      <p>2. If this's active sandboxing flag set has its sandboxed document.domain browsing context flag set, then throw a "SecurityError" DOMException.</p>
      <p>3. Let effectiveDomain be this's origin's effective domain.</p>
      <p>4. If effectiveDomain is null, then throw a "SecurityError" DOMException.</p>
      <p>5. If the given value is not a registrable domain suffix of and is not equal to effectiveDomain, then throw a "SecurityError" DOMException.</p>
      <p>6. If the surrounding agent's agent cluster's is origin-keyed is true, then return.</p>
      <p>7. Set this's origin's domain to the result of parsing the given value.</p>
    </div>
  - 如：`alice.github.io` 執行 `document.domain = 'github.io'` 會出錯
    - 錯誤原因：不符合步驟 5 registrable domain suffix 的檢查

---

# 神奇的 document.domain

- 兩頁面改 `document.domain` 後變成 same origin 的原因

  - 嚴格來說改 `document.domain` 是讓兩網站變成 same origin-domain
  - 某些檢查看 same origin-domain，非 same origin <span class='text-sm opacity-80'>(<a href='https://html.spec.whatwg.org/multipage/document-sequences.html#concept-bcc-content-document' target='_blank'>ref</a>)</span>
    <div class='quote'>
      <p>🖊️ If document's origin and container's node document's origin are not same origin-domain, then return null.</p> 
    </div>

    - 如果 document 和裡面的 node document（iframe）不是 same origin-domain，就回傳 null
    - 如果是 same origin-domain，就可存取到 iframe

---

# 神奇的 document.domain

之 same origin-domain 是什麼

- 判斷 A 與 B origin 是否為 same origin-domain<span class='text-#2f96ad text-xs ml-2'><Link to='additionalInfo' class='border-none!'>\[3\]</Link></span>
  <div class='quote'>
    <p>1. If A and B are the same opaque origin, then return true.</p> 
    <p>2. If A and B are both tuple origins:</p> 
    <p class='ml-4'>1. If A and B's schemes are identical, and their domains are identical and non-null, then return true.</p>
    <p class='ml-4'>2. Otherwise, if A and B are same origin and their domains are both null, return true.</p>
    <p>3. Return false.</p>
  </div>

  - same origin-domain 可分三種情況
    - 都是 opaque origin
    - scheme 和 domain 都相同，且<span v-mark.red='1'>domain 不是 null</span>
    - 兩個是 same origin，且 <span v-mark.red='2'>domain 都是 null</span> <br>
      (後兩者說的 domain 就是指 tuple origin 的 domain 屬性)

<!-- 觀察
兩網頁都有設置 domain 或都沒有，才有可能是 same origin-domain
如果兩網頁都有設 domain，same origin-domain 就不檢查 port -->

---

# 神奇的 document.domain

- `document.domain` 是用來改 tuple origin 的 domain 屬性
- `http://alice.example.com:5555` 跟 `http://bob.example.com:5555` 都將自己的 domain 改成 `example.com`
  - 符合「If A and B's schemes are identical, and their domains are identical and non-null, then return true.」判斷，因此是 same origin-domain

---

# document.domain 的淡出及退場

- `document.domain` 放寬 Same Origin 限制存在已久
  - 早期用途：存取 same site 但 cross origin 頁面
  - 保留原因：相容性
  - 可能問題：subdomain 有 XSS 漏洞時，影響範圍可擴大
- Chrome 對 `document.domain` 的措施
  - 2022 年<a href='https://developer.chrome.com/blog/immutable-document-domain/' target='_blank'>指出</a>最快從 Chrome 101 版開始，停止支援更改 document.domain
  - 2023 年<a href='https://developer.chrome.com/blog/document-domain-setter-deprecation' target='_blank'>宣布</a> `document.domain` 的淘汰將於 Chrome 115 生效
- document.domain 替代方案
  - `postMessage` 或 `Channel Messaging API`
  - 還是想用 `document.domain`：在 response header 帶上 `Origin-Agent-Cluster: ?0`

---

```yaml
layout: center
```

# 章節回顧

1. 如何判斷兩網站是否為 same origin?

2. 如何判斷兩網站是否為 same site?

3. 請判斷 A 和 B 是否為 same origin，並說明原因

```
A: https://blog.example.com:443/posts
B: https://blog.example.com/about
```

4. 給定以下 URL pairs，判斷它們是否為 same site，並說明原因

```
1. https://alice.github.io 與 https://bob.github.io
2. https://blog.example.com 與 http://example.com
3. https://store.example.com 與 https://blog.example.com
```

<div flex="~ gap-2" class='text-sm mt-8 opacity-80'>
<Link to='14' >same origin</Link> | <Link to='17' >same site</Link> | <Link to='19'>schemelessly same site</Link>
</div>

---

```yaml
layout: center
```

# 章節回顧

3. 請判斷 A 和 B 是否為 same origin，並說明原因

```
A: https://blog.example.com:443/posts
B: https://blog.example.com/about
```

是 same origin，兩個 scheme(https)、host(blog.example.com) 相同，雖然 B 沒有明確指定 port，但 https 預設 port 為 443，兩者 port 也相同。

---

```yaml
layout: center
```

# 章節回顧

4. 給定以下 URL pairs，判斷它們是否為 same site，並說明原因

```
1. https://alice.github.io 與 https://bob.github.io
2. https://blog.example.com 與 http://example.com
3. https://store.example.com 與 https://blog.example.com
```


1. 不是 same site，`github.io` 是 public suffix，所以 `alice.github.io` 和 `bob.github.io` 的 registrable domain 不同
2. 不是 same site，因為 scheme 不同 (https 和 http)
3. 是 same site，因為它們有相同的 scheme (https)，且 registrable domain 都是 example.com

---

# 跨來源資源共用 CORS 基本介紹

- 同源政策 same-origin policy：瀏覽器會阻止一個網站讀取另一個不同來源網站的資料
- 開發常見問題：開發時前後端可能在不同 origin，前端如何讀到後端資料？
  - 如：前端在 `huli.tw`、後端在 `api.huli.tw`
  - 解法：CORS（Cross-Origin Resource Sharing），一種可跨來源交換網站資料的機制
- 為什麼不能用 `XMLHttpRequest` 或 `fetch`（也可簡稱 AJAX）獲取跨來源資源？
  - 以 `img` 或 `script` 請求跨來源資源不會遇到問題，但 AJAX 請求卻會被阻擋

---

# 為什麼不能跨來源呼叫 API？

- 反向思考：如果跨來源請求不會被擋住，會發生什麼事？
  - 🎉 自由串 API，不受任何阻擋
    - 在自己網頁 `https://huli.tw/index.html` 用 AJAX 拿 `https://google.com` 資料
  - ⛔ 問題舉例 1
    - 情境：某公司內部網站 `http://internal.good-company.com` 只有公司員工電腦可連得到
    - 問題：在自己網頁寫 AJAX 拿內部網站資料，拿到後再傳回自己 server，就可偷到機密資料
      <img src='/images/cross-origin-problem.png' alt='cors problem' class='w-[350px] mt-2' />

---

# 為什麼不能跨來源呼叫 API？

- 反向思考：如果跨來源請求不會被擋住，會發生什麼事？
  - ⛔ 問題舉例 2
    - 情境：平常開發時會自己開 server 如 `http://localhost:3000`
    - 問題：若瀏覽器沒阻擋跨來源 API，攻擊者可拿到 localhost server 的資料
      - 資料可能是：公司機密、可分析網站漏洞的資料
  - ⛔ 問題舉例 3
    - 情境：假設跨來源請求會自動附 cookie
    - 問題：瀏覽惡意網站時，惡意網站可發 request 到 Facebook、Gmail，因為自動帶使用者 cookie，能拿到隱私資料

---

# 為什麼不能跨來源呼叫 API？

- 為什麼要擋住跨來源 AJAX？
  - 「安全性」
  - 瀏覽器若要拿網站完整內容(可完整讀取)，只能用 `XMLHttpRequest` 或 `fetch`
    - 若沒限制跨來源 AJAX，就能透過使用者瀏覽器拿任意網站內容
- 為什麼不擋圖片、CSS 或 script？
  - 屬於「網頁資源的一部分」，有其限制
  - 資源限制
    - 標籤只能拿特定類型資源
    - 取得資源<span v-mark.red='1'>無法用程式讀取</span>
      - -> 無法外傳

---

# 跨來源 AJAX 是怎麼被擋掉的？

之隨堂小測驗

- 情境：小明要用一個刪除文章 API
  - 用法：`POST` 並帶文章 id，content type 是 `application/x-www-form-urlencoded`
  - 限制：公司前後端網域不同，後端沒加 CORS header
  - 小明呼叫刪除文章 API 後，console 跳錯：「request has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource」
- 此說法正確嗎？
  - 小明認為因同源政策，AJAX 發不出請求，文章刪不掉

---

# 跨來源 AJAX 是怎麼被擋掉的？

- 誤解：跨來源請求擋住的是 request
  - ⛔ 小明認為請求無法到伺服器，資料刪不掉
- 再看一次錯誤訊息
  <div class='quote'>
    <p>request has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource</p>
  </div>
  瀏覽器已發出 request、拿到 response 才發現沒有 'Access-Control-Allow-Origin' header
<br>

#### 💡 瀏覽器擋住的不是 request，而是 response <span class='text-xs opacity-60'>(只適用於簡單請求)</span>

- request 已到伺服器，瀏覽器也收到 response，只是瀏覽器不把結果給你
- 小明的 request、文章 和 response?
  - request：已經到 server
  - 文章：已經被刪除
  - response：瀏覽器拿到了，但它不給小明

---

# 如何設置 CORS?

- 回傳 response 時，在 header 和瀏覽器說：「允許 XXX 存取這請求的 response」
  - `*` 代表允許任何 origin 讀取這 response
    ```
    Access-Control-Allow-Origin: *
    ```
  - 限制單一來源存取
    ```
    Access-Control-Allow-Origin: https://blog.huli.tw
    ```
  - `Access-Control-Allow-Origin` 目前不支援多個 origin
    - 只能依 request 動態設定不同 header

---

# 如何設置 CORS?

- 跨來源請求分「簡單請求」跟「非簡單請求」 <span class='text-sm opacity-80'>(<a href='https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS' target='_blank'>ref</a>)</span>
  - 兩者都需後端回傳 `Access-Control-Allow-Origin` header
  - 非簡單請求：會先發 preflight request，若未通過則不發正式請求 <br>
    -> 簡單請求、非簡單請求的正式請求及 preflight request 都需 `Access-Control-Allow-Origin` header
- 若要傳送自定義 header，後端要新增 `Access-Control-Allow-Headers` 才能通過 preflight <span class='text-sm opacity-80'>(<a href='https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#access-control-request-headers' target='_blank'>ref</a>)</span>
- 跨來源請求阻擋的是…
  - 簡單請求：阻擋 response
  - 非簡單請求：阻擋正式 request（因 preflight 驗證未通過，不會發正式 request）

---

# 如何設置 CORS?

之跨來源請求與 cookie

- 跨來源請求預設不會帶上 cookie
- 跨來源請求要帶上 cookie，須滿足特定條件
  | 條件 | 簡單請求 | 非簡單請求 |
  | ---------------------------------------------------------------------------- | -------- | ---------- |
  | 後端 Response header 有 `Access-Control-Allow-Credentials: true` | 不需要 | 必須 |
  | 後端 Response header 的 `Access-Control-Allow-Origin` 不能是 `*`，需明確指定 | 不需要 | 必須 |
  | 前端 fetch 加上 `credentials: 'include'` | 必須 | 必須 |

<style>
.slidev-layout table td{
  max-width: 350px;
}
</style>

---

```yaml
layout: center
```

# 章節回顧

1. 瀏覽器阻止跨來源請求的原因是什麼？CORS 在這情境下如何發揮作用？

2. 在 CORS 情境下，簡單請求和非簡單請求有何區別？瀏覽器如何處理這兩種請求？

---

```yaml
layout: center
```

# 章節回顧

1. 瀏覽器為什麼要阻止跨來源 AJAX？CORS 在這情境下如何發揮作用？

安全性，避免惡意網站存取其他來源的敏感資料；CORS 是一種可跨來源交換網站資料的機制，後端透過`Access-Control-Allow-Origin` header 告知瀏覽器哪個 origin 可存取 response

2. 在 CORS 情境下，簡單請求和非簡單請求有何區別？瀏覽器如何處理這兩種請求？

簡單請求是指符合某些標準（如：用 `GET` 或 `POST` 方法並帶有特定 header）的請求，直接傳送到伺服器。若伺服器沒有回應適當的 `Access-Control-Allow-Origin` header，瀏覽器會阻止我們用 JavaScript 存取 response。非簡單請求涉及一個 preflight request，瀏覽器會先發送 preflight request 來檢查實際請求是否安全可發送，若 preflight request 沒通過，就不會發實際請求。
<div class='text-sm opacity-80'>
簡單請求、非簡單請求的正式請求及 preflight request 都要有 <code>Access-Control-Allow-Origin</code> header 才合法
</div>


---

# 跨來源的安全性問題：CORS misconfiguration
- 若跨來源非簡單請求想帶上 cookie，`Access-Control-Allow-Origin` 就要指定單一 origin
  - 若多個 origin 都要存取 API，要動態調整 `Access-Control-Allow-Origin` 裡的 origin
- 動態調整錯誤示範 1：直接放入 request header 內的 origin
  - -> 任一 origin 都能通過 CORS
<div class='pl-6'>
```js
app.use((req, res, next) => {
  res.headers['Access-Control-Allow-Credentials'] = 'true'
  res.headers['Access-Control-Allow-Origin'] = req.headers['Origin']
})
```
</div>


---

# 跨來源的安全性問題：CORS misconfiguration

- 動態調整錯誤示範 1：直接放入 request header 內的 origin
  - 可能問題：寫個網站 `https://fake-example.com` 並讓使用者在 `example.com` 登入狀態下點這網站，可偷到使用者資料
    - 影響範圍：視網站 API 而定
    - 攻擊成立的前提
      - CORS header 錯誤設置
      - 網站用 cookie 做身份驗證，且沒設 SameSite
      - 使用者主動點擊網站且是登入狀態

<div class='pl-12'>

```js {*}{maxHeight:'150px'}
// fake-example 網站寫這段 script
// 用 api 去使用者資料，且帶上 cookie (若有設定 SameSite cookie，攻擊會失效，因 cookie 帶不上去)
fetch('https://api.example.com/me', {
  credentials: 'include'
})
  .then(res => res.text())
  .then(res => {
    // 伺服器認可 https://fake-example.com 是合格 origin，fake-example 網站也可拿到使用者資料，可傳送到自己 server
    console.log(res)
    // 把使用者導回真正的網站
    window.location = 'https://example.com'
  })
```
</div>

---

# 跨來源的安全性問題：CORS misconfiguration

- 動態調整錯誤示範 2：Regex 判斷 request origin 是否合法
  - 可以：`example.com`、`buy.example.com` ✅
  - 也可以：`fakeexample.com` 🔺

<div class='pl-12'>

```js {*}{maxHeight:'80px'}
app.use((req, res, next) => {
  res.headers['Access-Control-Allow-Credentials'] = 'true'
  const origin = req.headers['Origin']
  // 偵測是不是 example.com 結尾
  if (/example\.com$/.test(origin)) {
    res.headers['Access-Control-Allow-Origin'] = origin
  }
})
```
</div>
  
- 錯誤 CORS 設置引起的漏洞稱為 CORS misconfiguration
- 動態調整 CORS 的正確做法
  - 準備允許的 origin 清單，清單內的才通過
  - 設 sameSite cookie

<div class='pl-12'>

```js {*}{maxHeight:'100px'}
const allowOrigins = ['https://example.com', 'https://buy.example.com', 'https://social.example.com']
app.use((req, res, next) => {
  res.headers['Access-Control-Allow-Credentials'] = 'true'
  const origin = req.headers['Origin']
  if (allowOrigins.includes(origin)) {
    res.headers['Access-Control-Allow-Origin'] = origin
  }
})
```
</div>



---

# 跨來源的安全性問題：CORS misconfiguration

- 實際案例
  - 2016 年 Jordan Milne 找到的 JetBrain IDE 漏洞 <span class='text-sm opacity-80'>(<a href='https://blog.saynotolinux.com/blog/2016/08/15/jetbrains-ide-remote-code-execution-and-local-file-disclosure-vulnerability-analysis/' target='_blank'>ref</a>)</span>
    - 漏洞
      - CORS 設置不當：允許任意網站讀取 locale server 的 response
      - 路徑遍歷漏洞：攻擊者可讀取本地系統檔案
    - 攻擊方式：可讀取本地檔案，結合其他漏洞後可達成 RCE
  - 2017 年 James Kettle 在 AppSec EU 研討會分享的比特幣交易所漏洞 <span class='text-sm opacity-80'>(<a href='https://www.youtube.com/watch?v=wgkj4ZgxI4c&ab_channel=OWASP&themeRefresh=1' target='_blank'>ref</a>)</span>
    - 漏洞：API 允許任意 origin 跨來源讀取 response
    - 攻擊方式：可用 API 取得使用者 apiKey，並用它轉移比特幣
  - 2020 年 Asiayo 漏洞 <span class='text-sm opacity-80'>(<a href='https://zeroday.hitcon.org/vulnerability/ZD-2020-00829' target='_blank'>ref</a>)</span>
    - 漏洞：和上述相同，可在別的網站拿到使用者資料


---

# 其他各種 COXX 系列 header
<br>
其他以 CO(Cross-Origin) 開頭的 header，也和跨來源資料存取有關

- CORB（Cross-Origin Read Blocking）
- CORP（Cross-Origin Resource Policy）
- COEP（Cross-Origin-Embedder-Policy）
- COOP（Cross-Origin-Opener-Policy）

<div class='opacity-60 mt-40 text-right'>
  在那之前，先來看看 Meltdown 與 Spectre...
</div>


---

# 嚴重的安全漏洞：Meltdown 與 Spectre

- 2018 年 Google Project Zero 發布 CPU data cache 攻擊的介紹文章 <span class='text-sm opacity-80'>(<a href='https://googleprojectzero.blogspot.com/2018/01/reading-privileged-memory-with-side.html' target='_blank'>ref</a>)</span>
  - Spectre: bounds check bypass (CVE-2017-5753), branch target injection (CVE-2017-5715)
  - Meltdown: rogue data cache load (CVE-2017-5754)
- 攻擊嚴重性：CPU 問題，難修復
- 漏洞影響：促進瀏覽器與跨來源政策的演進

---

# 超級簡化版 Spectre 攻擊解釋
此為方便理解的簡化版，和原始攻擊有落差，但核心概念相似

- 假設一段程式碼（C 語言）
  - 宣告兩陣列 arr1（長度 16）和 arr2（長度 256）
  - 函式 `run(x)` 判斷 `x < array1_size`，若符合則執行 `array2[array1[x]]`
  - 沒超出陣列範圍，理論上沒問題


<div class='pl-6'>

```c
uint8_t arr1[16] = {1, 2, 3}; 
uint8_t arr2[256]; 
unsigned int array1_size = 16;

void run(size_t x) {
  if(x < array1_size) {
    uint8_t y = array2[array1[x]];
  }
}

size_t x = 1;
run(x);
```
</div>


---

# 超級簡化版 Spectre 攻擊解釋
之簡單介紹 CPU 機制

- CPU 機制：Branch Prediction 和 Speculative Execution
  - CPU 執行時，若碰到 `if`，會先預測結果是 `true` 或 `false`
    - 若預測結果是 `true`，就先執行 `if` 內程式碼，先算出 `if` 內的結果
  - 實際 `if` 條件執行完後
    - 若跟預測相同：皆大歡喜
    - 若跟預測不同：把計算的結果丟掉

<div class='text-sm opacity-80 pl-6 mt-4'>
Branch Prediction 是「預測」分支的走向；Speculative Execution 是「基於預測結果」執行分支中的程式碼
</div>


---

# 超級簡化版 Spectre 攻擊解釋
之簡單介紹 CPU 機制

- Branch Prediction 和 Speculative Execution 有何問題？
  - CPU 丟棄結果後我們也拿不到，但它有留下線索 🔺
    - 線索：預測執行時，結果會被放入 CPU cache
  - 如何判斷資料是否在 CPU cache 內？
    - 以存取時間判斷，讀取 CPU cache 內資料較快
  - Side-channel attack：攻擊者可利用存取時間（timing attack）來推測 CPU cache 內的資料


---

# 超級簡化版 Spectre 攻擊解釋
- 再看一次程式碼，可能會有什麼問題？
  - 跑多次 `run(10)` 後，branch prediction 預測下次也會滿足條件，提前執行 if 內程式碼
  - 當 `x` 設為 100 時，預測會執行：`uint8_t y = array2[array1[100]];`
    - 若 `array1[100]` 是 38，則執行 `y = array2[38]`
    - `array2[38]` 被放入 CPU cache
  - 實際執行發現條件不符，丟掉執行結果
  - 此時用 timing attack 讀取 array2 每個元素並計算時間，發現 `array2[38]` 讀取時間最短，回推 `array1[100]` 內容是 38 <br>
    -> 存取到其他不該存取到的記憶體

<div class='pl-6'>

```c {*}{maxHeight:'100px'}
uint8_t arr1[16] = {1, 2, 3}; 
uint8_t arr2[256]; 
unsigned int array1_size = 16;

void run(size_t x) {
  if(x < array1_size) {
    uint8_t y = array2[array1[x]];
  }
}

size_t x = 1;
run(x);
```
</div>

---

# 超級簡化版 Spectre 攻擊解釋

- Spectre 攻擊：上述攻擊原理應用於瀏覽器，可讀取同一個 process 的其他資料
  - 若同一個 process 有其他網站內容，就能讀取其他網站內容

<div class='quote ml-6 mb-10'>
在瀏覽器上，Spectre 讓你有機會讀取到其他網站的資料
</div>

- COXX 和 Spectre 的關係
  - COXX 主要目的：防止一個網站能讀取到其他網站的資料，避免惡意網站跟目標網站處在同一個 process

---

# CORB（Cross-Origin Read Blocking）
阻擋不合理的跨來源資源載入

- 其他網站的資料會如何出現？跨來源存取資源的方式如：
  - `fetch` 或 `xhr`
    - 已被 CORS 控管
    - response 在 network 相關 process，不是網站本身 process，Spectre 拿不到
  - `<img>` 或 `<script>`
    - 可載入機密資料如 `<img src="https://bank.com/secret.json">`
    - 限制：無法用 JavaScript 讀取
    - Chrome 運作機制
      - 下載檔案時，無法確定是否為符合標籤的檔案類型（e.g. 不確定它是否為圖片）
      - 下載後交由 render process 處理，發現格式錯誤才觸發載入錯誤
    - 問題：Spectre 只要在同一 process 就可存取，即使進入 render process 也能讀取

---

# CORB（Cross-Origin Read Blocking）
阻擋不合理的跨來源資源載入

- CORB 是什麼？
  - 如果想讀的資料類型不合理，就不需要進 render process，把結果丟掉即可
- 資料類型不合理是什麼意思？
  - JSON 檔的 MIME type 若是 `application/json`，代表不會是圖片
  - 用 `<script>` 載入 HTML
- CORB 主要保護的資料類型：HTML、XML 跟 JSON
  - 如何判斷這三種類型？
    - Chrome 根據內容探測（<a href='https://mimesniff.spec.whatwg.org/' target='_blank'>sniffing</a>）檔案類型，決定是否套用 CORB
      - 有誤判可能



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

```yaml
routeAlias: additionalInfo
```

# 補充

- \[1\]：2024/12 的 <a href='https://html.spec.whatwg.org/multipage/browsers.html#sites' target='_blank'>spec</a> 中，same site 演算法敘述有更改
- \[2\]：2024/12 的 <a href='https://url.spec.whatwg.org/#host-registrable-domain' target='_blank'>spec</a> 中，registrable domain 敘述有更改
- \[3\]：2024/12 的 <a href='https://html.spec.whatwg.org/multipage/browsers.html#origin' target='_blank'>spec</a> 中，same origin-domain 演算法有更改

---

# Learn More

[Documentation](https://sli.dev) · [GitHub](https://github.com/slidevjs/slidev) · [Showcases](https://sli.dev/resources/showcases)

<PoweredBySlidev mt-10 />
