漏洞名称：跨站请求伪造

1. 挑战概述
漏洞类别：跨站请求伪造
挑战目标：使用跨站点请求伪造从另一个来源更改用户的名字。

2. 漏洞发现线索与复现过程

在 Web 渗透测试中，如果某些执行敏感操作的接口仅仅依赖 Cookie 进行身份认证，而没有加入防伪造令牌，就会轻易沦为 CSRF 攻击的标靶。
这里还是有两种想法：1是通过浏览器自动发起请求，2是使用xml文件来实现

方法1:

1. 在个人页面中尝试修改用户名，并使用 Burp Suite 抓包。发现修改用户名的请求是一个发往 /profile 接口的 POST 请求，且请求中除了 Cookie 外，没有任何动态的防伪造参数。

<img width="651" height="393" alt="截屏2026-11-07 19 47 46" src="https://github.com/user-attachments/assets/106faddd-9d8c-4934-bef4-51df27371569" />

<img width="311" height="311" alt="截屏2026-11-07 19 48 10" src="https://github.com/user-attachments/assets/6551cff8-1cd6-4ac1-9850-d752b80e0461" />

2. 构造 payload：
<img width="462" height="158" alt="截屏2026-11-07 20 12 48" src="https://github.com/user-attachments/assets/dcef1bbd-ac64-4018-89e5-908abc0c5110" />

3.在https://htmledit.squarefree.com/中输入payload（这个网址在挑战中具体给了）

<img width="305" height="134" alt="截屏2026-11-07 19 53 53" src="https://github.com/user-attachments/assets/c717a4e9-77b8-43b2-ae61-946969d9f038" />

4.按理来讲页面应该自动提交表单。由于是从本地的 file:// 或其他跨域来源发起的请求，浏览器自动携带了 Juice Shop 的有效登录 Cookie。

5.但是由于浏览器限制，不得不采用其他方案。

<img width="877" height="535" alt="截屏2026-11-07 19 58 21" src="https://github.com/user-attachments/assets/f0fd0f19-1d06-4f4b-a152-64ba9683f115" />

方法2:使用hml文本
1.将payload制作成一个hml文本
<img width="450" height="383" alt="截屏2026-11-07 20 03 38" src="https://github.com/user-attachments/assets/34db9435-362f-4259-9cff-df1d135bebb0" />

2.打开这个文本，自动跳转页面并成功修改了名字
<img width="72" height="77" alt="截屏2026-11-07 20 05 01" src="https://github.com/user-attachments/assets/0ac7975a-46c6-4527-a335-b1e777328c9e" />

<img width="378" height="405" alt="截屏2026-11-07 20 05 15" src="https://github.com/user-attachments/assets/8dec90b2-041f-4306-9fa0-476feb39ed9c" />


3.实战危害与加固建议
实战危害展示：CSRF 是钓鱼攻击中最核心的利用手段之一。如果银行转账接口或管理员后台赋予权限的接口存在 CSRF 漏洞，黑客只需构造一个payload然后在邮件中发一张图片，或者一个极具诱惑力的链接，任何点击该链接且处于登录状态的系统管理员，其最高权限就会在几毫秒内被黑客利用，导致严重的资产损失或服务器失陷。
对于这个图片我记得有图片隐写技术，在后续的挑战中有相应的处理方式
修复方案：
1. 强制使用 Anti-CSRF Token：后端在生成敏感操作的表单时，应下发一个随机的、不可预测的 Token，并在提交时校验。由于同源策略的限制，处于第三方恶意网站的黑客无法通过脚本读取到这个 Token，从而无法构造出合法的伪造请求。
2. 启用 SameSite Cookie 属性：在服务器 Set-Cookie 时，将 SameSite 属性设置为 Lax 或 Strict。这能在浏览器底层直接阻断从第三方域名发起的跨站请求携带凭证，从根源上斩断 CSRF 的利用链。
