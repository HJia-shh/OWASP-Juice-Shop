漏洞名称：用户凭证泄露 / 联合查询 SQL 注入脱库 (User Credentials)

1. 挑战概述
漏洞类别：注入漏洞

挑战目标：利用系统中存在的 SQL 注入漏洞，使用联合查询手法，成功提取出数据库中所有用户的登录凭证。

2. 漏洞发现线索与复现过程

在 Web 渗透测试中，搜索框往往是产生 SQL 注入的高发地带。通过前期的黑盒测试与报错注入，我们已经知道系统的商品搜索接口存在基于 SQLite 的 SQL 注入漏洞，并在之前的挑战中掌握了后台包含一个名为 `Users` 的表，且知道其包含 `email` 和 `password` 字段。

1. 前台商品搜索框，对应的底层 API 是 /rest/products/search?q=。
2. 构造Payload：aaa')) UNION SELECT email,username,createdAt,password,role,isActive,deletedAt,8, 9 FROM USERS--
3. 输入即可
<img width="934" height="521" alt="截屏2026-11-10 02 55 24" src="https://github.com/user-attachments/assets/7e6b03f0-4166-4889-990c-af85ab352d6b" />


3. 实战危害与加固建议
实战危害展示：获取用户凭证是导致企业发生数据泄露灾难的最直接原因。一旦 `Users` 表被脱库，即便密码是哈希存储的，黑客依然可以通过彩虹表、Hashcat 暴力破解，或者利用“撞库攻击”去接管这些用户在其他平台的账号，给企业带来毁灭性的公关和合规危机。

修复方案：
1. 全局弃用字符串拼接：任何涉及到数据库交互的接口，绝不能使用 `SELECT * FROM Products WHERE name LIKE '%${q}%'` 这种原生拼接写法。
2. 强制使用参数化查询：无论底层使用的是 SQLite、MySQL 还是 PostgreSQL，必须使用预编译语句。参数化查询会在数据库引擎中将用户的输入严格当作“纯数据”进行处理，使得单引号、`UNION` 等 SQL 保留字彻底失去语义执行能力。
3. 密码高强度加密存储：即使发生脱库，也应提高攻击者的破解成本。废弃 MD5、SHA-1 等容易被彩虹表击穿的哈希算法，必须使用带有随机盐值的 bcrypt、Argon2 或 PBKDF2 慢速哈希算法。
