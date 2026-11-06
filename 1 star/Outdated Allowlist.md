漏洞名称：过时的白名单 / 未验证的重定向

1. 挑战概述
漏洞类别：未验证的重定向和转发
挑战目标：利用网站的重定向机制，成功跳转到一个官方不再推广的加密货币地址。

2. 漏洞发现线索与复现过程

在 Web 渗透测试中，如果看到 URL 里有 `?to=`、`?url=`、`?redirect=` 这种参数，通常暗示系统存在跳转接口。结合前端源码审计来摸清它的跳转规则。

1. 在 Juice Shop 页面底部，点击 GitHub 图标，观察浏览器地址栏，发现它调用了跳转接口：`http://127.0.0.1:3000/redirect?to=https://github.com/juice-shop/juice-shop`。
<img width="357" height="92" alt="截屏2026-11-06 20 49 19" src="https://github.com/user-attachments/assets/cf597030-d564-4446-9378-569a4342e9e0" />

2. 尝试把 `to=` 后面的网址直接改成 `https://www.google.com` ，发现系统报错 `406 Not Acceptable`。这说明后端存在白名单校验。
<img width="813" height="412" alt="截屏2026-11-06 20 51 51" src="https://github.com/user-attachments/assets/5f800034-a726-4403-bc01-54555d2063ed" />

3. 依旧请出inspect，在 `debugger` 面板中搜索搜索关键词github.com，试图寻找前端是否硬编码了这层白名单。

4. 在源码中找到了GitHub 链接
<img width="289" height="27" alt="截屏2026-11-06 20 58 31" src="https://github.com/user-attachments/assets/c17c68d9-7390-42e2-9f72-6fbafd3476cb" />

但是没有找到其他相关地址

5. 于是不断的搜索blockchain或者Cryptocurrency

6.找到了相关路径

<img width="450" height="62" alt="截屏2026-11-06 21 01 18" src="https://github.com/user-attachments/assets/eaa29782-61dc-49cc-add9-d1ae1ff85c30" />

7.访问该路劲http://127.0.0.1:3000/redirect?to=/redirect?to=https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm
<img width="874" height="526" alt="截屏2026-11-06 21 02 18" src="https://github.com/user-attachments/assets/ea2bde92-ba07-4693-921c-8dac1c2d9dfc" />


3. 漏洞危害与加固建议
实战危害展示：这不仅仅是“代码不整洁”的问题。如果白名单中遗留了公司早期被废弃的域名，而这个废弃域名正好因为忘记续费被黑客抢注，黑客就可以利用公司官方的 `redirect` 接口，名正言顺地把用户跳转到自己的钓鱼网站上。因为链接开头是受信任的官方域名，受害者几乎必定会中招。

修复方案：
1. 定期清理废弃资产：在版本迭代时，必须将不再使用的业务代码、API 接口和白名单配置一并下线，消除僵尸代码带来的攻击面。
2. 推荐使用间接引用跳转：最佳的重定向设计是不传递完整的 URL，而是传递映射 ID（例如 `?to=1` 代表跳 GitHub，`?to=2` 代表跳 Twitter）。后端通过查询字典匹配跳转，彻底杜绝开放重定向风险。
