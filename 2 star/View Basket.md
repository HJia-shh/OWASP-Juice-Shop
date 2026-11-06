漏洞名称：水平越权访问购物车

1. 挑战概述
漏洞类别：访问控制缺陷 / 水平越权

挑战目标：在登录自己的普通用户账号时，通过篡改请求参数，成功越权查看并读取其他用户的购物车清单及商品明细。

2. 漏洞发现线索与复现过程

在 Web 渗透中，如果系统在 URL 或 API 请求中通过递增的数字 ID（如 `/basket/1`、`/basket/2`）来直接定位敏感数据，且后端没有严格校验当前用户的身份归属，就会产生水平越权。

这里还是有两种方法，一种是使用inspection的Storage，还有一种是使用burp suite
第一种方法：
1. 查看自己的购物车。
2. 开始Burp Suite抓包
3. Burp Suite 成功捕获到一个发往后端 API 的 GET 请求：`GET /rest/basket/1 HTTP/1.1`
4. 将这个抓到的数据包发送到Repeater
5. 将请求路径中的数字 1修改为其他数字
6. 服务器没有校验当前登录用户是否拥有该购物车的归属权，直接返回了 200 OK，并在响应体的 JSON 数据中吐出了其他用户的购物车商品明细。

<img width="640" height="378" alt="截屏2026-11-06 23 31 58" src="https://github.com/user-attachments/assets/f33db7a1-65d6-4922-9bc9-115ce0c65bab" />
<img width="643" height="384" alt="截屏2026-11-06 23 28 04" src="https://github.com/user-attachments/assets/c98b4c25-723b-4905-913e-10357145b437" />

第二种方法：
1。使用inspect，查看Storage
<img width="869" height="178" alt="截屏2026-11-06 23 29 44" src="https://github.com/user-attachments/assets/c713d73d-8139-4ff9-8da1-aa29c7d601bc" />
2.将bid改成其他数字刷新即可

3. 漏洞危害与加固建议
实战危害展示：水平越权是企业业务系统中最隐蔽、危害也极大的漏洞之一。由于攻击者不需要管理员权限，只需注册两个普通账号交叉对比，就能通过脚本遍历修改数字 ID，批量拖库获取全站所有用户的隐私数据、订单明细甚至收货地址。

修复方案：
1. 废弃前端可控的信任凭证：绝对不能让前端直接决定或拼接查询哪个购物车 ID。
2. 服务端强制联合校验：当后端接收到获取购物车的请求时，必须先从 HTTP 请求头的 JWT Token 中解析出当前用户的真实 `userId`，并在数据库查询时执行严格的绑定过滤：`SELECT * FROM Baskets WHERE id = ? AND UserId = ?`。如果该购物车不属于当前用户，必须直接返回 `403 Forbidden`。
