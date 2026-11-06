漏洞名称：静态资源解析异常

1. 挑战概述
漏洞类别：输入验证不当 / 前端逻辑缺陷

挑战目标：在网站中找回一张由于命名包含特殊字符而导致无法正常加载的猫咪照片。

2. 漏洞发现线索与复现过程

在 Web 渗透与日常排查中，网页上出现“破损图片（Broken Image）”或静态资源 404，往往暗示着资源路径在传输过程中发生了解析截断。

1. 浏览 Juice Shop 的照片墙页面，发现列表中有一张图片无法正常加载，显示为碎裂的图标。
<img width="722" height="348" alt="截屏2026-11-06 20 34 01" src="https://github.com/user-attachments/assets/72c2bba8-64ff-4732-809c-0c4355ed18c2" />

2. 在Inspect中，查看该图片的 HTML 代码。
<img width="345" height="33" alt="截屏2026-11-06 20 35 23" src="https://github.com/user-attachments/assets/a316d7cb-210f-4df5-97bf-e9d99db7e80d" />

3. 根据 HTTP 协议和 URL 规范，#是特殊保留字符。当浏览器向服务器发起请求时，会自动截断‘#’及其后面的所有内容。因此，自然返回 404 Not Found。
4. 将 URL 进行编码得到assets%2fpublic%2fimages%2fuploads%2f%e1%93%9a%e1%98%8f%e1%97%a2-%23zatschi-%23whoneedsfourlegs-1572600969477.jpg
   <img width="669" height="371" alt="截屏2026-11-06 20 42 07" src="https://github.com/user-attachments/assets/026b9537-7608-4337-a3e9-4c1b884d6231" />
5. 将编码修改完后即可看见小猫
   <img width="737" height="357" alt="截屏2026-11-06 20 43 15" src="https://github.com/user-attachments/assets/498006f4-adfb-4f17-bf1d-7f32f6c4d048" />
<img width="323" height="40" alt="截屏2026-11-06 20 42 58" src="https://github.com/user-attachments/assets/4483c636-d4fc-458f-8460-28cfead035fd" />

3. 漏洞危害与加固建议
实战危害展示：表面上看这是一个简单的图片 Bug，但在真实的漏洞挖掘中，“未对参数进行 URL 编码”往往是致命的。例如在“重置密码链接”或“OAuth 授权认证”中，如果混入了未转码的 & = # ? 等控制字符，攻击者就能借此提前截断参数、篡改业务走向，引发极其严重的账户劫持或越权漏洞。

修复方案：
1. 强制前端转码：在前端 JavaScript 拼接 URL 或 API 请求参数时，必须严格使用 `encodeURIComponent()` 函数对包含特殊字符的变量进行安全包裹与转码。
2. 重塑文件上传命名规范：后端在接收用户上传的文件时，绝不能直接使用用户原始的文件名作为物理存储路径。应当在落盘时将其重命名为安全的纯字母数字组合（如 UUID + 时间戳），从根源上消除 Emoji、空格及特殊符号带来的系统级路径解析隐患。
