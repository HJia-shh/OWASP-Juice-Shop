漏洞名称：NoSQL 操纵 / 数据库查询操作符注入

1. 挑战概述
漏洞类别：注入漏洞

挑战目标：利用 NoSQL 数据库的查询操作符特性，绕过后端的业务逻辑限制，越权批量更新或操纵系统中的数据记录。

2. 漏洞原理与机制分析

与传统的 SQL 注入不同，NoSQL 注入通常发生在数据结构层面。当应用层直接接受客户端传入的 JSON 数据，并将其未经严格类型校验就传入 MongoDB 等 NoSQL 数据库执行时，攻击者可以将普通的“值”替换为“查询操作符对象”，从而篡改执行逻辑。

1. 在burp suite抓包，编辑一条评论。

<img width="372" height="259" alt="截屏2026-11-10 01 11 18" src="https://github.com/user-attachments/assets/b31def86-db8f-4bed-90df-e6af7db73d79" />


<img width="319" height="356" alt="截屏2026-11-10 01 11 40" src="https://github.com/user-attachments/assets/ac18a7d5-46f2-4b29-8235-5fb143b7a124" />

2. 我在网上搜寻到 MongoDB 的语法中，存在大量以 `$` 开头的查询操作符，如 `$ne` (不等于)、`$gt` (大于)、`$regex` (正则匹配)。

3. 构造payload，将 `"id": ""` 改为 `"id": {"$ne": -1}。

<img width="335" height="388" alt="截屏2026-11-10 01 12 29" src="https://github.com/user-attachments/assets/686dafc3-fc83-4beb-b754-219c48f496c1" />

<img width="675" height="463" alt="截屏2026-11-10 01 15 27" src="https://github.com/user-attachments/assets/3c7477d3-c538-4736-b21f-0c5175dec054" />


3. 实战危害与加固建议
实战危害展示：NoSQL 注入在采用前后端分离和 Node.js 后端的现代架构中危害极大。黑客不仅可以用 `{"$ne": null}` 绕过登录验证逻辑实现任意账户接管，还可以利用这种方式越权删除、修改全库数据，或者通过 `$where` 操作符执行服务器端 JavaScript 代码（SSJS），最终导致严重的业务瘫痪或数据泄露。

修复方案：
1. 严格的输入类型校验：这是防御 NoSQL 注入的最核心防线。在 Controller 层或网关层，必须明确约定参数的数据类型。如果 `id` 应该是一个字符串，一旦接收到Object类型的数据，应立即抛出 400 Bad Request 错误。
2. 启用 ODM/ORM 的严格模式：在使用 Mongoose 等对象文档映射库时，应严格定义 Schema 数据模型。Mongoose 默认会将非字符串的注入对象强制转换为字符串，从而使 `{"$ne": -1}` 变成无效的字面量 `"[object Object]"`，自动抵御此类操作符注入。
3. 禁用危险操作符与 SSJS：在 MongoDB 数据库层面，若业务不需要，应在配置文件中禁用执行服务器端脚本的能力，遵循最小权限原则。
