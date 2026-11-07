漏洞名称：GDPR 数据擦除

1. 挑战概述
漏洞类别：身份验证破坏 / 业务逻辑缺陷

挑战目标：使用已删除的Chris用户帐户登录。

2. 漏洞发现线索与复现过程

在现代 Web 应用中，为了满足隐私合规要求，系统通常会提供“请求擦除数据”的功能。如果该高危功能的身份校验与系统的入口登录共享同一套脆弱的信任体系，就会被攻击者武器化。

1.在前面的数据注入时能查看到已删除的数据
2.可以重新构造一个payload为q=test')) UNION SELECT id,email,password,'4','5','6','7','8','9' FROM Users--

3.输入payload账号为chris.pike@juice-sh.op

4.通过sql注入登入他的账户即可

<img width="208" height="229" alt="截屏2026-11-07 21 55 00" src="https://github.com/user-attachments/assets/ca645f61-276a-499d-b14b-1a38bacb275e" />

3. 实战危害与加固建议
实战危害展示：攻击者无需勒索软件，只需利用越权或 SQL 注入拿到目标会话

修复方案：
与database schema类似
