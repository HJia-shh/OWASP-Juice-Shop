漏洞名称：登录 Bjoern 的账号 / 源码审计与密码生成逻辑逆向

1. 挑战概述
漏洞类别：敏感信息泄露 / 身份验证破坏

挑战目标：使用 Bjoern的 Gmail 帐户登录而不更改他的密码，使用SQL 注入或骇入他的Google帐户。

2. 漏洞发现线索与复现过程

很多开发人员在实现“第三方登录”时，为了图省事，并没有走标准的服务端授权码流程，而是在前端验证成功后，利用用户的邮箱加上一段“硬编码的盐值”通过固定算法生成一个密码，然后用这个密码去向后端发起普通登录。只要黑客审查前端代码，这个算法就会彻底暴露。

1. 在package.json.bak文件，提取邮箱：`bjoern.kimminich@googlemail.com`。
<img width="494" height="74" alt="截屏2026-11-10 00 23 20" src="https://github.com/user-attachments/assets/ea2dc4e7-d6ad-4763-b1a9-c9c497ace0f4" />

3. 在inspect进入 debugger，搜索 `oauth`
<img width="439" height="198" alt="截屏2026-11-10 00 27 02" src="https://github.com/user-attachments/assets/f4b3deb9-9998-452a-9aca-ac8dd8fe490d" />

4. 发现在 OAuth 登录回调函数中，前端向后端发送登录请求前，在本地动态生成了一个密码。它使用了一种非常脆弱的硬编码规则
<img width="317" height="107" alt="截屏2026-11-10 00 28 08" src="https://github.com/user-attachments/assets/bf2c1a1a-0a47-4de8-a5e6-137d56babd4e" />

5. 编码规则为把email切成数组后进行反转，然后进行base64编码，得到密码：bW9jLmxpYW1nQGhjaW5tbWlrLm5yZW9qYg==
<img width="283" height="181" alt="截屏2026-11-10 00 40 00" src="https://github.com/user-attachments/assets/a189ecf4-4172-4be9-a382-9eeb5b1debd8" />


3. 实战危害与加固建议
实战危害展示：这是极其典型的“硬编码机密”与“虚假 OAuth 实现”漏洞。在实战中，很多企业的 App 或小程序为了快速接入微信/QQ登录，也在前端自己“捏造”密码发给后端。黑客通过逆向 APK 或反编译前端 JS 代码，拿到这个固定算法和盐值后，就能写个脚本，只需知道任意用户的手机号或邮箱，即可瞬间计算出其密码，实现全站任意账户的批量接管。

修复方案：
1. 废除前端密码生成逻辑：真正的第三方登录绝不能在前端生成密码或维护认证状态。前端只需将第三方平台返回的授权码原封不动地传给后端。
2. 实施标准 Server-to-Server 校验：后端服务器收到 Code 后，必须携带私密的 Client Secret，在后端直接向第三方服务器发起请求换取 Access Token 和用户信息。整个验证链路必须在不可见的后端完成，彻底杜绝前端逻辑被逆向的可能。
3. 避免硬编码敏感信息：任何用于加密、签名的盐值或秘钥，绝不能明文写死在前端代码中。
