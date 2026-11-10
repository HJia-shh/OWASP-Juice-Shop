漏洞名称：越权密码重置 / API 校验缺失与危险的 GET 状态变更
1. 挑战概述
漏洞类别：不当的输入验证 / 安全架构缺陷

挑战目标：在完全不知道 Bender 当前极强密码的前提下，强制修改他的账户密码。

2. 漏洞发现线索与复现过程

很多 Web 应用存在“前端防线极其严密，后端 API 却完全裸奔”的通病。开发人员往往以为只要在前端 UI 强制要求用户输入“当前密码”，系统就是安全的。同时，违背 RESTful 架构规范，使用 GET 请求来执行“修改密码”这种状态变更操作，会引发严重的安全歧义。

1. 虽然不能使用SQL注入和忘记密码重置，但是可以使用SQL注入登入Bender的账号：bender@juice-sh.op'-- 
2. 在修改密码页面，需要强制输入当前密码。但Bender的密码极强无法破解。

<img width="288" height="343" alt="截屏2026-11-10 22 20 32" src="https://github.com/user-attachments/assets/58ec30e0-6282-43ee-ac51-15f7ea03b9b5" />

3. 随便输入并使用burp suite抓包，发现是get请求，按理来讲这应该使用post，put或patch
<img width="367" height="417" alt="截屏2026-11-10 22 21 32" src="https://github.com/user-attachments/assets/f6eed56f-5ca5-44bb-8b4a-00e3a4e6aaa6" />

4. 既然提示说属于不当的输入验证，如果后端 API 没有强制校验current存在，那把current删除即可，结果修改成功
<img width="709" height="406" alt="截屏2026-11-10 22 28 57" src="https://github.com/user-attachments/assets/a15c573b-e484-40e4-8fd6-580ff8c6be23" />


3. 实战危害与加固建议
实战危害展示：
这个漏洞在实战中极其致命。黑客一旦窃取了用户的 Token，就可以直接越过“旧密码”这道最后防线，彻底锁定并接管账户。此外，将敏感的状态变更操作设计为 GET 请求，会导致这些敏感 URL 被记录在服务器的 Access Log、浏览器的历史记录以及代理服务器中，造成严重的凭证泄露风险。

修复方案：
1. 严守 HTTP 动词语义：绝对禁止使用 GET 请求执行状态变更操作。密码重置接口必须使用 POST、PUT 或 PATCH 请求，且请求数据必须放在 JSON Body 中。
2. 后端强制输入验证：永远不要信任前端的校验逻辑。后端的改密 API 必须强制要求传入 `currentPassword` 字段，并在与数据库中存储的哈希值比对完全一致后，才允许执行更新操作。
3. 引入 CSRF Token 与二次鉴权：除了比对旧密码，高价值操作还应校验 Anti-CSRF Token，或强制要求输入手机验证码，彻底切断越权改密和跨站请求伪造的可能。
