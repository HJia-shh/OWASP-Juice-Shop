漏洞名称：高级付费专区 / 密钥泄露与隐蔽式安全

1. 挑战概述
漏洞类别：密码学缺陷 / 安全配置错误

挑战目标：在不支付任何真实货币的情况下，解锁并访问 OWASP Juice Shop 的“高级专属”隐藏内容。

2. 漏洞发现线索与复现过程

根据提示3和4，我们推测：解锁高级区域的“钥匙”或“隐藏路径”，已经被开发人员遗留在了前端源码或服务器的公开目录中。这道题是“硬编码密钥”与“前端加密”结合的典型反面教材。
<img width="187" height="280" alt="截屏2026-11-12 20 37 09" src="https://github.com/user-attachments/assets/6f719c84-150a-4db8-a363-96c2e9f90f5d" />

1. 在/encryptionkeys目录下有一个premium.key文件，下载并打开里面是一串十六进制的字符串
<img width="824" height="370" alt="截屏2026-11-12 20 38 50" src="https://github.com/user-attachments/assets/0f142aef-e4f8-4ed3-9a4f-3a5f6091f656" />

2. 仔细排查HTML代码，发现一段被明显加密过的密文字符串，这段密文与“Premium”解锁相关。
<img width="408" height="198" alt="截屏2026-11-12 21 48 49" src="https://github.com/user-attachments/assets/36641f44-38fc-4e22-b57b-7f66b2565c24" />

3. 对其进行解密得到/this/page/is/hidden/behind/an/incredibly/high/paywall/that/could/only/be/unlocked/by/sending/1btc/to/us`
<img width="384" height="151" alt="截屏2026-11-12 21 50 14" src="https://github.com/user-attachments/assets/e90575ef-0b6e-4b14-9873-222281c54d19" />

4. 访问该网站http://127.0.0.1:3000/this/page/is/hidden/behind/an/incredibly/high/paywall/that/could/only/be/unlocked/by/sending/1btc/to/us
<img width="911" height="559" alt="截屏2026-11-12 21 50 59" src="https://github.com/user-attachments/assets/cc01ccbf-c738-467b-b56b-4b20cc36f9ff" />


3. 实战危害与加固建议
实战危害展示：
这是典型的“掩耳盗铃”。在真实的 Web3 或高级会员系统中，有些开发者为了省事，不在后端做严格的权限校验把高级内容放在一个“没人能猜到”的长 URL 里或者用一套加密算法把它藏在前端。殊不知，只要密钥（如 `.env` 中的 `SECRET_KEY`）暴露在公共目录，或者加密算法本身在前端执行，黑客就能轻易逆向出真实路径，实现越权白嫖。

修复方案：
1. 废弃隐蔽式安全：不要试图通过“隐藏长而复杂的 URL”来保护敏感资源。URL 一旦暴露在浏览器历史、代理日志或 Referer 报文中，将彻底失去保护。
2. 实行严格的后端访问控制：所有的付费内容必须实行强校验。当用户请求高级资源时，后端必须实时查询数据库，验证当前 Session 或 JWT 中对应的 `is_premium` 状态。
3. 保护密钥资产：类似于之前 Forged Signed JWT 的教训，绝对不允许将任何 `.key`、`.pem` 或密钥配置文件暴露在 Web 服务器的静态资源解析目录中。
