漏洞名称：终结聊天机器人 / 第三方组件动态代码注入

1. 挑战概述
漏洞类别：使用含有已知漏洞的组件 / 代码注入

挑战目标：通过审查系统依赖的开源聊天机器人源码，发现其底层的代码执行缺陷，并注入恶意代码导致进程崩溃，永久瘫痪聊天机器人。

2. 漏洞发现线索与复现过程

根据官方提示，我们需要摸清机器人 API 的底层运作机制。

1. 审查机器人的源代码并没有发现什么
<img width="586" height="309" alt="截屏2026-11-11 22 24 28" src="https://github.com/user-attachments/assets/176c4e9b-9705-481d-bc0a-4beac19e1a11" />
<img width="659" height="283" alt="截屏2026-11-11 22 27 09" src="https://github.com/user-attachments/assets/368ec8be-718c-4a43-9b8d-e242b2668466" />

2. 在package.json.bak中提到使用了juicy-chat-bot库，然后去github上搜寻他的源代码发现这个包使用了一个名为 `vm2` 的沙箱模块来动态执行 JavaScript 代码
<img width="293" height="167" alt="截屏2026-11-11 22 41 36" src="https://github.com/user-attachments/assets/5f934d92-fe70-47ed-96d3-b3ceb1e6998e" />

3。 并且开发人直接将用户输入的 `name` 变量，使用模板字符串拼接到了一段 JS 代码中，并交给 `vm2` 引擎去执行
<img width="472" height="168" alt="截屏2026-11-11 22 43 17" src="https://github.com/user-attachments/assets/72734550-62cf-4cdf-a84a-f4ab78d3463e" />

4. 构造代码注入 Payload：this.factory.run(‘users.addUser(“”, “”); process=null; users.addUser(“1234”, "malicioususer")’)
5. 在客服聊天页面机器人询问名字的时候输入payload，代码一旦执行，`process` 对象被直接置为 `null`，导致机器人的底层运行逻辑直接崩溃，内存状态被彻底破坏。
<img width="674" height="436" alt="截屏2026-11-11 22 47 34" src="https://github.com/user-attachments/assets/4606a4bd-624e-4718-aa07-8c9044e02f19" />

3. 实战危害与加固建议
实战危害展示：
这道题揭示了动态语言中极其危险的漏洞类型：当开发人员试图用 `eval()`、`setTimeout()` 或某些沙箱来动态解析业务逻辑时，如果不做严格的输入净化，黑客只需闭合字符串，就能在服务器上执行任意代码，导致DoS。黑客可以直接 `require('child_process').exec('反弹Shell')` 拿下服务器最高权限。

修复方案：
1. 杜绝动态执行机制：绝大多数业务场景根本不需要 `eval` 或 `new Function()`。应彻底重构这部分逻辑，使用安全的数据结构来处理逻辑分支，而不是把用户的输入当作代码来“运行”。
2. 绝对的输入净化：如果由于特殊原因必须拼接执行环境，用户输入必须经过极其严格的白名单过滤，确保其不包含引号、反引号、括号等任何可能引起语法截断的特殊字符。
3. 弃用不安全的沙箱：vm2 库近年来已经被爆出多次严重的沙箱逃逸 0-day 漏洞，目前已被官方宣布废弃。企业应密切关注开源组件的生命周期，及时替换高风险的基础库。

