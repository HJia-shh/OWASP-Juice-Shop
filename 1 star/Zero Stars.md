漏洞名称：零星评价

1. 挑战概述
漏洞类别：业务逻辑缺陷 / 后端输入验证缺失
挑战目标：绕过前端 UI 的星级限制，成功向系统提交一个“0 星”的客户反馈。

2. 漏洞发现线索与复现过程

在 Web 安全测试中，UI 界面（如滑动条、下拉菜单、星级评分）的限制是极其脆弱的。
这里我想到了两种方法，一种是和Repetitive Registration一样，将disable删除，第二种是通过burp suite抓包完成

第一种方法：
1.在inspect界面把disable删除即可
<img width="350" height="283" alt="截屏2026-11-06 21 53 18" src="https://github.com/user-attachments/assets/07ae2c6a-f816-489b-a2e4-8de91d97a966" />

2.这里我们可以看到提交无法点击

3.找到并删除后
<img width="630" height="331" alt="截屏2026-11-06 21 54 34" src="https://github.com/user-attachments/assets/63f0eca3-855a-4496-9dc4-55a78d3dc472" />

4.然后提交即可

<img width="385" height="397" alt="截屏2026-11-06 21 54 57" src="https://github.com/user-attachments/assets/80ab23bd-4f06-4a86-b008-ec7d2f8c35f0" />

第二种方法：
  1. 在burp suite抓包可以看到post请求
<img width="375" height="59" alt="截屏2026-11-06 21 56 15" src="https://github.com/user-attachments/assets/fba58eaa-46ec-41b8-af1a-99e3bea5f516" />

  2. 在 Burp Suite 的抓包窗口中，在请求中找到代表星级的参数："rating"

  3. 将这个参数的值直接修改为 0然后点击 Forward将篡改后的数据包放行。
  4. 数据包到达服务器后，由于后端 API 未对 `rating` 字段的边界值（1-5）进行校验，数据库直接接收了这条异常数据
<img width="187" height="175" alt="截屏2026-11-06 22 00 47" src="https://github.com/user-attachments/assets/4daa2adf-d33f-4759-b842-0103ec47f933" />
<img width="265" height="362" alt="截屏2026-11-06 22 01 24" src="https://github.com/user-attachments/assets/04af777c-5c13-486b-97a1-6317d671149f" />


3. 漏洞危害与加固建议
实战危害展示：这种前后端校验脱节的缺陷在电商或评分系统中非常普遍。如果黑客不仅能提交 0 星，还能提交负数），且后端的商品综合评分逻辑是简单的平均值计算，攻击者就可以通过自动化刷单，瞬间将竞争对手的高分商品总评分拉到负数，造成极其严重的商业声誉损害与业务混乱。

修复方案：
1. 后端强校验：不论前端 UI 做了多严密的限制，接收数据的 API 接口在处理请求时，必须第一时间校验数据类型的边界。对于评分字段，必须在代码中明确限定：if (rating < 1 || rating > 5) return 400 Bad Request。
2. 数据库层面约束：在设计数据库表结构时，为评分字段添加检查约束，例如 CONSTRAINT chk_rating CHECK (rating >= 1 AND rating <= 5)，确保异常数据在落盘前被数据库系统强制拦截。
