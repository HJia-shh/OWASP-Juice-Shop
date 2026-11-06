漏洞名称：管理员登录绕过 / SQL 注入

1. 挑战概述
漏洞类别：注入漏洞 / 身份验证破坏
挑战目标：在不知道真实密码的情况下，利用 SQL 注入漏洞强行以系统超级管理员的身份登入系统。

2. 漏洞发现线索与复现过程

在真实的渗透测试中，登录框永远是安全攻防的正面战场。如果后端在处理数据库查询时直接使用了“字符串拼接”，就会给攻击者留下 SQL 注入的可乘之机。

1. 访问 Juice Shop 的用户登录页面
2. 输入admin@juice-sh.op，然后构造一段经典的“万能密码”闭合语句：' OR 1=1--或者'--
4. 在“Password”（密码）输入框中，随便输入
<img width="311" height="380" alt="截屏2026-11-06 22 59 49" src="https://github.com/user-attachments/assets/8f39f2b2-d3e8-4fb8-a86f-bfa597e322a2" />
<img width="234" height="178" alt="截屏2026-11-06 23 00 03" src="https://github.com/user-attachments/assets/cc6f530d-ef16-4ea8-9fc3-49a6fb0dd737" />


3. 漏洞危害与加固建议
实战危害展示：SQL 注入连续多年霸榜 OWASP Top 10 第一名。在这个漏洞中，后端原本的查询语句可能是 `SELECT * FROM Users WHERE email = '$email' AND password = '$password'`。当我们注入了 payload 后，语句变成了：
`SELECT * FROM Users WHERE email = '' OR 1=1--' AND password = '...'`
这里的单引号 ' 提前闭合了查询条件，`OR 1=1` 构造了一个永远为真的逻辑，而 --直接把后面的密码校验逻辑全部废掉。数据库最终默认返回了用户表中的第一条记录，导致黑客瞬间接管最高权限。

修复方案：
1. 强制使用参数化查询：这是防御 SQL 注入的唯一标准答案。无论是使用底层驱动还是 ORM 框架，都必须使用参数绑定（如?占位符），确保用户的输入在底层只被当作纯文本字面量处理，绝对不参与 SQL 语法树的编译和执行。
2. 严格的输入白名单校验：在 API 层接收 email 字段时，强制使用正则表达式校验其是否为合法邮箱格式。一旦检测到含有单引号 '、注释符 -- 或 UNION 等高危字符，直接返回 400 Bad Request阻断请求。
