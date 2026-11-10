漏洞名称：跨站图片注入 / 组合漏洞利用与 DOM XSS

1. 挑战概述
漏洞类别：跨站脚本 / 不当的输入验证

挑战目标：通过深挖前端 Angular 源码，利用系统中隐藏的 DOM XSS 漏洞，将高级会员页面的快递箱强行替换或注入一只跨域小猫的图片。

2. 漏洞发现线索与复现过程

正如官方提示所言，开发人员在这里犯了多个愚蠢的错误。为了在这个页面动态渲染出那个带有 "JS" Logo 的 3D 快递箱，前端代码中引入了一个致命的渲染缺陷，允许我们将恶意的 HTML 标签直接“贴”在这个箱子的位置。

1. 进入高级会员页面。有一堆黄色的 JS 快递箱。进入inspect发现这个箱子不是简单的 img src="box.png"而是动态渲染的。
<img width="802" height="543" alt="截屏2026-11-10 22 54 28" src="https://github.com/user-attachments/assets/7a8a809d-1353-4a50-8241-7539ef04d448" />

2. 在debugger页面查看代码，搜索deluxe-membership，寻找有关代码。
<img width="331" height="172" alt="截屏2026-11-10 23 03 34" src="https://github.com/user-attachments/assets/16d3c2f0-ed92-41af-89a9-f6fb5ec9ef4e" />

3. 这里挑战提示给了我们小猫的链接，然后根据白名单的规则，要用到重定向挑战里的方法构造Payload：http://127.0.0.1:3000/#/deluxe-membership?testDecal=../../../../redirect?to=https://placecats.com/neo/300/200/?x=http://kimminich.de?pwned=https://github.com/juice-shop/juice-shop
4. 输入URL即可
<img width="934" height="568" alt="截屏2026-11-10 23 08 42" src="https://github.com/user-attachments/assets/51161c21-af9e-4025-859d-a9ddeebcf113" />

3. 实战危害与加固建议
实战危害展示：
这道题生动地展示了“前端渲染失控”的威力。表面上我们只是恶搞了一张小猫图片，但这种针对特定页面组件的 DOM XSS 杀伤力极大。黑客不仅可以注入 `<script>` 标签窃取 Token，还可以利用伪造的系统弹窗图片完全覆盖原有的网页组件。比如，把这个“成为会员”的模块直接替换成一个钓鱼表单。

修复方案：
1. 严格的上下文转义：前端组件在接收外部或路由参数时，必须保持框架默认的严格文本插值，确保任何意外传入的 HTML 标签都被转义为普通字符串。
2. 部署内容安全策略：在 HTTP 响应头中配置严格的 `Content-Security-Policy`，例如 `default-src 'self'`。这样即使发生了注入，浏览器也会拒绝加载不在白名单中的外部图片，形成纵深防御。
