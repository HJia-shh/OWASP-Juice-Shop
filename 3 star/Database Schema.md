漏洞名称：数据库架构提取 / 联合查询 SQL 注入

1. 挑战概述
漏洞类别：注入漏洞
挑战目标：通过SQL注入获取整个数据库结构。使用UNION联合查询，提取出整个应用底层数据库的表结构定义。

2. 漏洞发现线索与复现过程

在 Web 渗透测试中，如果后端使用拼接的方式处理用户的搜索输入，攻击者就可以在搜索框中注入UNION SELECT语句，将自己想要查询的其他表数据强行拼接到原有的搜索结果中返回。

1. 在商品搜索框可能存在sql注入
2.在搜索框中输入单引号'，观察是否有报错。
<img width="497" height="457" alt="截屏2026-11-07 20 21 15" src="https://github.com/user-attachments/assets/bd9d6b35-f219-48d1-91a0-cbde7b5d8aec" />

3.在burp suite的历史中可以发现这里q=可能存在sql注入的漏洞。
<img width="856" height="110" alt="截屏2026-11-07 20 39 32" src="https://github.com/user-attachments/assets/b49598ad-51af-49a9-8f72-b95b3083e37b" />

4. 为了使用UNION联合查询，注入的查询结果列数必须与原商品查询的列数完全一致。
5. 构造Payload：q=w')) UNION SELECT 1，2，3，4，5，6，7，8，sql FROM sqlite_master--使用UNION SELECT用于追加要查询结果。
6. 输入payload：http://127.0.0.1:3000/rest/products/search?q=w%27))%20UNION%20SELECT%20sql,%202,%203,%204,%205,%206,%207,%208,%20sql%20FROM%20sqlite_schema--
<img width="889" height="519" alt="截屏2026-11-07 20 33 30" src="https://github.com/user-attachments/assets/fa94cbea-b99f-4e09-9d0b-183176dca3a7" />

3. 实战危害与加固建议
实战危害展示：能查到 Schema，就意味着数据库对攻击者彻底“单向透明”。黑客一旦掌握了表名和列名，下一步就可以再次构造 UNION 查询，将全站用户的账号和哈希密码打包拖走。这通常会导致极其严重的“暗网数据泄露”事件。

修复方案：
1. 强制参数化查询：对于搜索框等需要频繁与数据库交互的组件，必须使用预编译机制。将用户的输入严格隔离在 SQL 语法树之外，使其无法闭合引号或追加 UNION 关键字。
2. 剥离详细的错误回显：在生产环境中，绝不应该把底层的 SQL 语法错误直接抛到前端，这会极大降低攻击者猜测列数和闭合符号的难度。
3. 最小权限原则：配置数据库连接账号时，限制应用层账号对底层系统元数据表的读取权限。
