# JavaScript
[resource](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/First_steps/What_is_JavaScript)
>JavaScript 是轻量级解释型语言。浏览器接受到 JavaScript 代码，并以代码自身的文本格式运行它。技术上，几乎所有 JavaScript 转换器都运用了一种叫做即时编译（just-in-time compiling）的技术；    
 <font color="red">
当 JavaScript 源代码被执行时，它会被编译成二进制的格式，使代码运行速度更快。尽管如此，JavaScript 仍然是一门解释型语言，因为编译过程发生在代码运行中，而非之前。</font>


## 脚本加载策略，防止js先于html 和css 执行
脚本调用策略小结：

+ async 和 defer 都指示浏览器在一个单独的线程中下载脚本，而页面的其他部分（DOM 等）正在下载，因此在获取过程中页面加载不会被阻塞。
+ async 属性的脚本将在下载完成后立即执行。这将阻塞页面，并不保证任何特定的执行顺序。
+ 带有 defer 属性的脚本将按照它们的顺序加载，并且只有在所有脚本加载完毕后才会执行。
+ 如果脚本无需等待页面解析，且无依赖独立运行，那么应使用 async。
+ 如果脚本需要等待页面解析，且依赖于其他脚本，调用这些脚本时应使用 defer，将关联的脚本按所需顺序置于 HTML 的相应 \<script> 元素中。