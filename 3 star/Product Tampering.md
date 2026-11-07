漏洞名称：篡改商品信息 / 越权写入与接口裸奔

1. 挑战概述
漏洞类别：访问控制缺陷 / 不安全的直接对象引用

挑战目标：利用后端 API 权限控制的缺失，越权修改官方商品的描述内容，将其中的链接地址篡改为其他地址。

2. 漏洞发现线索与复现过程

在遵循 RESTful 架构的 Web 应用中，开发者通常会设计 `/api/Products/{id}` 这样的接口。使用 `GET` 请求获取商品信息，使用 `PUT` 或 `PATCH` 请求更新商品信息。如果后端的权限校验遗漏了对 `PUT` 方法的鉴权，就会导致普通用户甚至未登录用户也能修改全局商品库。

1. 找到O-Saft
<img width="323" height="296" alt="截屏2026-11-07 23 54 34" src="https://github.com/user-attachments/assets/b56568f9-9434-40ed-a35d-708a306f2217" />

3. 在历史里可以看到GET /api/Products/请求
<img width="897" height="140" alt="截屏2026-11-07 23 58 46" src="https://github.com/user-attachments/assets/b6bb1848-7dd4-434f-8628-43e802db43b0" />

4. 伪造篡改 Payload：先把get改成put
<img width="663" height="392" alt="截屏2026-11-08 00 04 42" src="https://github.com/user-attachments/assets/6b4e1f95-7493-4d3c-9287-87e17f0e9a83" />

5. 可以发现里面无内容，然后将其发送以后得到一串内容。

6. 然后粘贴到Request中，把description改为https://owasp.slack.com即可，后端 API 接收到 `PUT` 请求后，根本没有去校验当前用户是否具备 `Admin`（管理员）角色，直接根据 URL 中的 ID `9`，将数据库中 O-Saft 商品的描述内容更新为了我们注入的恶意链接。
<img width="341" height="403" alt="截屏2026-11-08 00 20 26" src="https://github.com/user-attachments/assets/65a746c6-aa77-46e8-a737-8b22d743fd18" />

3. 实战危害与加固建议
实战危害展示：篡改商品信息属于极高危的业务逻辑与越权漏洞。黑客一旦掌握该漏洞，可以发起“水坑攻击”将爆款商品的官方使用说明书、驱动下载链接篡改为带有木马的钓鱼链接。由于链接挂在官方域名下，用户会毫无防备地点击下载，直接导致大规模的用户设备沦陷，对企业声誉造成毁灭性打击。

修复方案：
1. 实施严格的基于角色的访问控制：在 API 网关或后端框架的路由层，必须针对 HTTP 方法进行细粒度的权限划分。对于 `/api/Products/` 接口，`GET` 方法可以对所有用户开放；但 `POST`、`PUT/PATCH`、`DELETE`方法，必须强制经过 `RequireAdminRole` 等高权限鉴权中间件。
2. 零信任架构下的接口审计：绝不能因为前端 UI 隐藏了“编辑按钮”，就认为后端接口是安全的。必须在上线前使用自动化黑盒扫描工具对所有的 RESTful 端点进行 HTTP Method 遍历测试，确保没有任何写操作接口处于“裸奔”状态。
