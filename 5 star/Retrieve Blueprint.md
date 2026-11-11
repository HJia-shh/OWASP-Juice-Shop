漏洞名称：重置 Morty 的密码 / IP 欺骗与防自动化机制绕过

1. 挑战概述
漏洞类别：身份验证破坏 / 破坏防自动化

挑战目标：通过信息收集找出 Morty 混淆后的安全问题答案，并通过绕过系统的速率限制防爆破机制，成功重置他的密码。

2. 漏洞发现线索与复现过程

在实战中，即使我们拿到了目标的社工字典，很多时候也会被Web 应用防火墙或后端的“防暴力破解”机制拦截。这道题的核心考点就是如何利用 HTTP 请求头欺骗后端，绕过基于 IP 的封禁策略。

1. 确定邮箱为morty@juice-sh.op，其安全问题是
<img width="212" height="51" alt="截屏2026-11-12 01 36 36" src="https://github.com/user-attachments/assets/9a134fd3-274f-463f-a225-4bd8d410980d" />

2. 在网上搜寻发现Morty是Morty Smith，他宠物狗原名叫Snuffles，后来它给自己改名为Snowball。

3. 提示说明：答案被混淆至长度小于 10 个字符，且不包含任何特殊符号。利用 Python 脚本或现成的字典生成工具，生成基于 `snuffles` 和 `snowball` 所有 Leet Speak 变体的专属字典。

4. 在github上找到了一段python代码可以完成这个内容。其链接为https://gist.github.com/philly-vanilly/70cd34a7686e4bb75b08d3caa1f6a820
<img width="1071" height="784" alt="截屏2026-11-12 01 40 39" src="https://github.com/user-attachments/assets/31dc500c-033a-46e6-a99d-0097b2f16490" />

5. 将里面一些内容修改后运行即可自动完成，密码是5N0wb41L
<img width="303" height="145" alt="截屏2026-11-12 02 04 48" src="https://github.com/user-attachments/assets/b3ee8a58-9ac1-4b8d-898e-cc2792645b05" />


3. 实战危害与加固建议
实战危害展示：
利用 `X-Forwarded-For`、`X-Real-IP` 或 `Client-IP` 等 HTTP 头部进行 IP 欺骗，是黑客绕过企业 WAF 速率限制、刷短信验证码、进行大规模 credential stuffing的常用手段。很多后端的防刷逻辑盲目信任这些由客户端传来的 Header，导致看似严密的防暴力破解机制形同虚设。

修复方案：
1. 建立不可欺骗的限流机制：应用服务器不应盲目信任 `X-Forwarded-For` 头部来判断真实 IP。如果架构前方有 Nginx 或云 WAF 作为反向代理，必须在代理层配置重写策略，剥离客户端伪造的 XFF 头，由代理服务器强制注入真实 IP。
2. 升级防自动化策略：除了 IP 频控，更高级的做法是引入基于设备指纹、用户行为分析或隐形的综合性反爬/反自动化引擎。
3. 废弃安全问题：再次强调，安全问题极易被社工或配合针对性字典爆破击穿，应强制改用基于邮件/手机的动态一次性验证码机制。
