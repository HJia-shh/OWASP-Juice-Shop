漏洞名称：未签名的 JWT / 空加密算法绕过

1. 挑战概述
漏洞类别：使用含有已知漏洞的组件 / 密码学缺陷

挑战目标：利用后端 JWT 解析库的缺陷，伪造一个本质上未签名的 JWT 令牌，并成功利用该令牌模拟用户 jwtn3d@juice-sh.op 进行越权访问。

2. 漏洞发现线索与复现过程

JSON Web Token由三部分组成：Header、Payload和Signature，头部通常会声明签名所使用的算法。然而，在许多早期的 JWT 开源解析库中，存在一个极其愚蠢的逻辑漏洞：如果攻击者将算法指定为 `none`，后端解析库就会直接跳过签名验证步骤，盲目信任载荷中的所有数据。

1. 随便登入账户，在inspect里提取token
<img width="941" height="290" alt="截屏2026-11-12 02 52 14" src="https://github.com/user-attachments/assets/5a6dec1b-9808-4c97-aab9-3c61b997c53b" />

2. 解密token，这里我使用https://www.jwt.io/网站
<img width="1077" height="724" alt="截屏2026-11-12 02 53 05" src="https://github.com/user-attachments/assets/cbe77287-8143-4dda-9e72-1a51d75ad45f" />

3. 把alg改成none，把email改为jwtn3d@juice-sh.op然后编码
<img width="1059" height="630" alt="截屏2026-11-12 02 56 34" src="https://github.com/user-attachments/assets/655f790a-c56b-48b3-9ebd-5a70fbd12740" />

4. 编码完成后，把原本token改为这个即可
<img width="975" height="578" alt="截屏2026-11-12 02 58 06" src="https://github.com/user-attachments/assets/ba7bb57e-d587-4eb0-a65c-0059397873f2" />


3. 实战危害与加固建议
实战危害展示：
这是 Web 安全史上最著名的核弹级漏洞之一。如果企业的后端网关或微服务鉴权使用了脆弱的 JWT 库，黑客不需要知道服务器的任何密钥，只需要把 Header 改成 `none`，删掉 Signature，就能随意篡改 Payload 里的 `user_id` 或 `role: admin`，实现“零成本”的全局任意账号接管。

修复方案：
1. 强制校验算法：在后端的 JWT 验证逻辑中，绝对不能“根据 Token 头部的 alg 字段动态决定使用什么验证方式”。必须在代码中硬编码强制声明期望的算法，例如：`jwt.verify(token, secret, { algorithms: ['HS256'] })`。如果传入了 `none` 或其他未授权算法，直接抛出异常并拒绝服务。
2. 升级底层安全组件：立刻检查项目中使用的 JWT 库的版本。现代维护良好的 JWT 库已经在底层默认拒绝了 `none` 算法。利用 SCA 工具确保彻底消灭存在已知漏洞的老旧依赖包。
