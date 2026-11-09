漏洞名称：URL 开放重定向白名单绕过

1. 挑战概述
漏洞类别：未验证的重定向和转发

挑战目标：利用系统中存在的 `/redirect` 路由，绕过其后端的 URL 白名单校验逻辑，成功将页面重定向到一个任意的外部网站。

2. 漏洞发现线索与复现过程

在很多 Web 应用中，为了统计点击率或统一跳转逻辑，会使用类似 `https://example.com/redirect?to=https://target.com` 的接口。为了防止被黑客恶意利用跳转到钓鱼网站，开发人员通常会维护一个“允许跳转的 URL 白名单”。但如果校验逻辑不够严谨，就会被轻易绕过。

1. 有一个 GitHub 链接，观察 URL，跳转逻辑为：http://127.0.0.1:3000/redirect?to=https://github.com/juice-shop/juice-shop
<img width="372" height="93" alt="截屏2026-11-09 21 15 56" src="https://github.com/user-attachments/assets/6b914ef1-00c1-4bff-a985-2ee0c8e16168" />

2. 将其修改为任意网址：http://127.0.0.1:3000/redirect?to=https://www.google.com系统返回 406 ，提示该 URL 不在白名单中。
<img width="912" height="472" alt="截屏2026-11-09 21 16 59" src="https://github.com/user-attachments/assets/275a67a8-521b-456f-9873-be5df66f6343" />

3. 构造 Payload：利用 URL 解析特性的差异，将合法的白名单字符串作为查询参数追加到恶意网址后面：http://127.0.0.1:3000/redirect?to=https://www.google.com?https://github.com/juice-shop/juice-shop
4. 后端代码检测到字符串中含有官方 GitHub 地址，为白名单。但浏览器在执行跳转时，会将 ? 后面的内容视为传给 google.com的查询参数，最终成功跳转到了外部网站。
<img width="802" height="329" alt="截屏2026-11-09 21 18 21" src="https://github.com/user-attachments/assets/3b86233d-d6de-45bd-9606-42462a8f6f11" />

3. 实战危害与加固建议
实战危害展示：开放重定向本身拿不到服务器权限，但黑客可以向目标企业员工发送一条带有官方域名的链接 https://trustedsite.com/redirect?to=http://fake-login.com。受害者看到域名是自己公司的，会毫无防备地点击，结果被跳转到高仿的钓鱼页面输入了账号密码。

修复方案：
1. 废弃明文 URL 传递：最优的重定向方案是不在参数中直接传递完整的 URL。而是使用映射 ID，例如 ?to=1 代表跳转到 GitHub，?to=2 代表跳转到 Twitter。后端查表获取真实地址，若输入非法 ID 则默认跳转回首页。
2. 严密的域名边界校验：如果业务必须传递 URL，绝不能使用 includes 或 indexOf 这种简单的字符串匹配。必须使用标准的 URL 解析库，准确提取出目标地址的 hostname，并与白名单进行严格的绝对相等匹配。
