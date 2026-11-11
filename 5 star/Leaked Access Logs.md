漏洞名称：泄露的访问日志

1. 挑战概述
漏洞类别：敏感数据暴露 / 身份验证破坏

挑战目标：通过在知名开发者社区和代码分享平台进行开源情报挖掘，找出一份泄露的 Web 访问日志，从中提取明文密码，并接管账户。

2. 漏洞发现线索与复现过程

很多严重的数据泄露并非源于系统被黑，而是因为开发人员在网上求助时，没有对粘贴的日志进行脱敏，从而将机密信息拱手送人。

1. 提示提到了开发者常用求助平台，这里是StackOverflow。然后有一个用户发了一个有关链接，访问他得到
<img width="808" height="680" alt="截屏2026-11-12 00 58 16" src="https://github.com/user-attachments/assets/304634cf-eaf3-435e-b33d-946a3937b5f0" />

2. 日志中，发现了一条 GET 请求为161.194.17.103 - - [27/Jan/2019:11:18:35 +0000] "GET /rest/user/change-password?current=0Y8rMnww$*9VFYE%C2%A759-!Fg1L6t&6lB&new=sjss22%@%E2%82%AC55jaJasj!.k&repeat=sjss22%@%E2%82%AC55jaJasj!.k8 HTTP/1.1" 401 39 "http://localhost:3000/" "Mozilla/5.0 (Linux; Android 8.1.0; Nexus 5X) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.99 Mobile Safari/537.36"
<img width="801" height="212" alt="截屏2026-11-12 01 01 31" src="https://github.com/user-attachments/assets/c264b9e7-70f4-4a5e-ab71-a07c37d03df4" />

3. 确定密码为0Y8rMnww$*9VFYE%C2%A759-!Fg1L6t&6lB
4. 但却不能确定账号，在之前/administration路径能看到所有人的邮件
<img width="362" height="508" alt="截屏2026-11-12 01 05 26" src="https://github.com/user-attachments/assets/bc01dff1-c5e6-4dde-8b6e-44b8f9835f29" />

5. 使用 burp suite抓包登入页面然后转发至intruder
<img width="490" height="439" alt="截屏2026-11-12 01 07 25" src="https://github.com/user-attachments/assets/055b324a-6ad5-4df2-a249-4ff764e81f48" />
<img width="968" height="554" alt="截屏2026-11-12 01 09 29" src="https://github.com/user-attachments/assets/46e49d2c-e76d-4375-b71e-1d2dc5300e8c" />

6. 成功登入
<img width="202" height="175" alt="截屏2026-11-12 01 11 10" src="https://github.com/user-attachments/assets/a18e21bd-ded9-4a5c-a14b-38a034f389b7" />

3. 实战危害与加固建议
实战危害展示：
一旦开发人员在 StackOverflow、CSDN 或 GitHub Issues 中不慎贴出了带有敏感参数的 Access Log，攻击者就能直接接管账户。进一步结合密码喷洒技术，由于每个账号只尝试了一次密码，这种攻击完美绕过了传统的“同一账号连续失败 5 次即锁定”的安全策略。

修复方案：
1. 严禁 GET 请求传输敏感数据：所有的身份验证、密码重置、Token 传递，必须使用 POST/PUT 请求，并将机密数据放置在 Request Body 中。
2. 日志脱敏与 DLP：在应用层的日志组件或网关层配置脱敏规则，强制过滤掉 URL 甚至 Header 中的 `password`, `token`, `secret` 等敏感关键字，确保落盘的日志是绝对安全的。
3. 严格的代码与日志外发纪律：企业应加强安全宣贯，严禁开发和运维人员将未经审核的内网配置、堆栈报错和访问日志上传至公共技术社区。同时应部署 DLP系统，对流向外部的代码片段进行自动拦截。

