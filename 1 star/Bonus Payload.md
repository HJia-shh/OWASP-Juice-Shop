漏洞名称：DOM XSS 进阶利用 (Bonus Payload)

1. 挑战概述
漏洞类别：跨站脚本攻击/iframe 劫持
挑战目标：使用开发者在前端代码中预留的特定“Bonus Payload”，在搜索框中触发复杂的 iframe 注入，而不仅仅是简单的弹窗。

2. 漏洞发现线索与复现过程

这个挑战的介绍中已经给了我一段Payload
<iframe width="100%" height="166" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/771984076&color=%23ff5500&auto_play=true&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true"></iframe>
再加上已经完成过dom xss所以很自然的可以对他进行URL编码得到
%3ciframe+width%3d%22100%25%22+height%3d%22166%22+scrolling%3d%22no%22+frameborder%3d%22no%22+allow%3d%22autoplay%22+src%3d%22https%3a%2f%2fw.soundcloud.com%2fplayer%2f%3furl%3dhttps%253A%2f%2fapi.soundcloud.com%2ftracks%2f771984076%26color%3d%2523ff5500%26auto_play%3dtrue%26hide_related%3dfalse%26show_comments%3dtrue%26show_user%3dtrue%26show_reposts%3dfalse%26show_teaser%3dtrue%22%3e%3c%2fiframe%3e
<img width="788" height="437" alt="截屏2026-11-06 01 20 26" src="https://github.com/user-attachments/assets/8e522891-498f-4935-b059-98e45fc52d16" />
然后对他进行payload
<img width="810" height="262" alt="截屏2026-11-06 01 21 39" src="https://github.com/user-attachments/assets/cea70ef8-2791-49bf-96e9-e391a7a778a9" />

3. 漏洞危害与加固建议
实战危害展示：这道题直观地展示了 XSS 的真实危害。黑客可以将上述的音乐播放器 iframe 替换成 “伪造的系统登录框” 或 “诱导下载木马的钓鱼页面”。受害者看到网址是完全合法的（确实在当前网站域名下），就会放松警惕输入密码，从而导致账号被盗。

修复方案：
严格使用 textContent 替代 innerHTML。
若必须渲染 iframe 等富媒体元素，必须配置严格的 CSP ，例如设置 frame-src 'self'，直接从浏览器底层阻断加载任何非白名单内的外部域名。
