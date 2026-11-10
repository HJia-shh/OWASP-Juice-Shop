漏洞名称：跨域邮件泄露 / JSONP 数据劫持

1. 挑战概述
漏洞类别：敏感数据暴露

挑战目标：通过跨域访问数据来获得信息泄露

2. 漏洞发现线索与复现过程

在现代 Web 架构中，同源策略会严格阻止恶意的跨域数据读取。但为了向后兼容，某些老旧 API 可能会遗留 JSONP机制。如果 API 缺乏严格的鉴权机制，攻击者就可以利用 JSONP 的 `<script>` 标签跨域特性窃取敏感数据。

1. 在burp suite 抓包分析用户状态相关的请求，发现一个 API 接口：/rest/user/whoami。
<img width="349" height="189" alt="截屏2026-11-10 23 20 30" src="https://github.com/user-attachments/assets/015b470f-91ab-4242-b887-8947e44461ad" />

2. 尝试移除 `Authorization` 头，发现该接口依然会返回当前会话的敏感信息。
<img width="716" height="381" alt="截屏2026-11-10 23 21 35" src="https://github.com/user-attachments/assets/6b1cf2ab-8d77-4a1b-8c9a-13a00996f9b5" />
<img width="664" height="395" alt="截屏2026-11-10 23 21 48" src="https://github.com/user-attachments/assets/0c61100c-c28e-481f-a5b6-44db55464c11" />

4. 为了实现跨域窃取，我们在该接口后追加 callback 参数进行试探。构造：`GET /rest/user/whoami?callback=stealData`
<img width="717" height="275" alt="截屏2026-11-10 23 25 02" src="https://github.com/user-attachments/assets/010af9d0-388f-49b0-98e3-2f27e0412b99" />

6. 输入http://127.0.0.1:3000/rest/user/whoami?callback=a，当受害者访问该恶意网站时，浏览器会无视同源策略加载该脚本，直接将其在 Juice Shop 的邮箱等敏感数据带出并发送给攻击者。
<img width="956" height="175" alt="截屏2026-11-10 23 25 54" src="https://github.com/user-attachments/assets/b1f4593e-b5b4-486e-b3cd-786d23885472" />


3. 实战危害与加固建议
实战危害展示：针对 `/whoami` 或 `/profile` 这类高频状态接口的 JSONP 劫持曾是窃取用户隐私的利器。如果受害者在登录了某大型电商平台的状态下，点击了黑客发来的钓鱼链接，恶意网页上的 `<script>` 标签就会瞬间跨域请求该平台的 JSONP 接口，直接将受害者的真实姓名、手机号甚至完整画像打包发送给黑客。

修复方案：
1. 彻底淘汰 JSONP：在现代前后端分离架构中，必须完全废弃 JSONP 这种基于脚本加载的危险跨域方式。
2. 采用严格的 CORS 策略：跨域数据交互应采用现代的 CORS 规范。后端应严格配置 `Access-Control-Allow-Origin`，绝不能配置为 `*` 或动态反射用户的 Origin。
3. 强化接口鉴权：类似 `/rest/user/whoami` 这种包含PII的高价值接口，必须进行严格的鉴权拦截，彻底阻断第三方站点的未授权跨域拉取。
