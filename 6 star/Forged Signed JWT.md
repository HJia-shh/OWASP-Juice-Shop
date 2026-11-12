漏洞名称：伪造签名的 JWT / 非对称向对称算法混淆攻击

1. 挑战概述
漏洞类别：密码学缺陷 / 使用含有已知漏洞的组件。

挑战目标：利用 JWT 解析库的“算法混淆”缺陷，获取服务器公开的 RSA 公钥，将加密算法从 RS256 强制降级为 HS256，并将公钥文本作为对称密钥进行签名，模拟用户 rsa_lord@juice-sh.op。

2. 漏洞发现线索与复现过程

在 JWT 规范中，RS256 是非对称加密（私钥签名，公钥验签），而 HS256 是对称加密（使用同一个密钥签名和验签）。如果没有强制指定只允许 RS256，攻击者就可以将 Header 中的算法篡改为 HS256，诱使后端将公开的 `publicKey` 字符串当成对称加密的密码来进行验签。

1. 随机提取一个用户的token进行解码，发现其 Header 采用的是 `RS256` 非对称算法。
<img width="1049" height="691" alt="截屏2026-11-12 19 01 29" src="https://github.com/user-attachments/assets/11096628-b069-48d9-8171-0bf0606d528d" />

2. 之前进行目录爆破发现有/encryptionkeys目录，里面有对外公开的 RSA 公钥文件 `jwt.pub`

<img width="900" height="313" alt="截屏2026-11-12 19 02 32" src="https://github.com/user-attachments/assets/46cf9d82-5e4f-4cbc-b914-b4486230e219" />

<img width="545" height="195" alt="截屏2026-11-12 19 02 40" src="https://github.com/user-attachments/assets/081af478-6a40-45cf-af81-2bc1f1de0718" />

3. 然后进行jwt编码并把Header的RS256篡改为对称加密算法HS256并把邮箱改为rsa_lord@juice-sh.op。
4. 再利用公钥文本进行签名

<img width="496" height="153" alt="截屏2026-11-12 19 06 12" src="https://github.com/user-attachments/assets/2fdc21b0-7620-4cbc-8773-020f68faf30e" />

<img width="1061" height="669" alt="截屏2026-11-12 19 06 26" src="https://github.com/user-attachments/assets/2db13604-4e35-4dd6-a3ec-d3b16cbde79f" />

5. 将伪造的Token替换即可，后端收到Token后读取 Header 发现是 HS256，于是直接拿服务器本地的 `jwt.pub` 文本对 Token 进行 HMAC 验签，结果匹配。
<img width="216" height="213" alt="截屏2026-11-12 19 07 52" src="https://github.com/user-attachments/assets/d0275d97-14ac-4f4c-8217-81e36029f557" />


3. 实战危害与加固建议
实战危害展示：
算法混淆是密码学在工程实现中最具代表性的逻辑灾难。服务器并没有泄露任何私钥，仅仅是按正常流程公开了用于验签的公钥，黑客就能通过篡改 Header 中的 `alg` 字段，指鹿为马，把公开的验证材料变成伪造签名的核心凭证，实现零成本的全局任意账户接管。

修复方案：
1. 显式绑定并校验加密算法：后端在调用 `verify()` 方法时，绝不能盲目采信 JWT Header 中声明的 `alg` 字段。必须在代码中硬编码白名单：`jwt.verify(token, publicKey, { algorithms: ['RS256'] })`，一旦发现传入 HS256 直接拒绝。
2. 升级底层 JWT 依赖库：现代主流版本的 `jsonwebtoken` 等组件已在底层加入类型检查，若检测到在 HMAC 算法下传入了 PEM 格式的公钥字符串，会直接抛出异常阻断执行。
