漏洞名称：购买隐藏商品 / 漏洞组合拳与业务逻辑缺陷

1. 挑战概述
漏洞类别：业务逻辑缺陷 / 不安全的直接对象引用

挑战目标：找出一件早已在前端下架的“2014年圣诞特供商品”，并成功将其加入购物车完成结算。

2. 漏洞发现线索与复现过程

在电商业务中，开发人员下架商品时，通常只是在前端页面隐藏了它的展示，或者在数据库里打上了一个软删除标签，但“加入购物车”的底层 API 往往忘记校验商品当前的销售状态。只要我们知道隐藏商品的 ID，就能强制下单。

1. 在Database Schema，知道可以通过 SQL 注入查阅后端的 Products 表。
2. 构造payload：http://127.0.0.1:3000/rest/products/search?q=%27))%20UNION%20SELECT%20*%20FROM%20PRODUCTS%20WHERE%20deletedAt%20IS%20NOT%20NULL--
3. SQL注入后得到商品表的数据，发现2014 Christmas Special在数据库里，并且它的 ProductId 是 10。
<img width="928" height="423" alt="截屏2026-11-09 21 25 37" src="https://github.com/user-attachments/assets/e974bc6e-d744-4363-9a3c-12bc16dcd2bc" />

4. 加入一个商品进入购物车，然后在burp suite抓包，并把product ID改为10发送。
<img width="673" height="391" alt="截屏2026-11-09 21 32 32" src="https://github.com/user-attachments/assets/43b89555-5b8b-4b67-91f8-eb7db6d740fb" />

5.进行结算即可
<img width="584" height="399" alt="截屏2026-11-09 21 33 03" src="https://github.com/user-attachments/assets/32c8a1e7-c307-449a-89a0-6f167a2559bb" />

3. 实战危害与加固建议
实战危害展示：黑客可以轻松购买到企业隐藏的商品。在真实场景中，这些隐藏商品可能是“1分钱的内部测试商品”、“尚未发布的新品”或者是“定向配给 VIP 客户的超低价福利”。攻击者利用该漏洞可以大批量薅羊毛，给企业造成严重的资产流失，甚至提前泄露未发布的商业机密。

修复方案：
1. 建立健壮的商品状态机校验：在“加入购物车”和“最终订单结算”的 API 接口中，绝不能只核对 `ProductId` 是否存在。必须在服务端反查该商品的当前状态：校验 `deletedAt` 字段是否为空，或者核对 `status == 'active'`，确保下架、过期或库存异常的商品绝对无法进入结算流。
2. API 接口联动防护：在微服务架构中，订单服务在执行扣款前，必须再次向商品中心发起一次强一致性的商品可用性校验，作为订单成交前的最后一道防线。
