漏洞名称：使用不安全的加密算法

1. 挑战概述
漏洞类别：用不安全的加密算法
挑战目标：通知商店，指出其不应该使用某种已被淘汰或极其不安全的加密算法或库。

2. 漏洞发现线索与复现过程

在安全审计和渗透测试中，识别系统底层的加密算法是非常重要的一环。现代应用中，如果使用了 MD5、SHA-1 或老旧的对称加密算法，通常会被直接判定为高危安全缺陷。

1. 在前期抓包、信息收集或前端源码审计时，我们可能会发现 Juice Shop 在处理某些敏感数据时，使用了已被业界淘汰的加密算法。
2. 从cookies提取Token：eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJzdGF0dXMiOiJzdWNjZXNzIiwiZGF0YSI6eyJpZCI6MiwidXNlcm5hbWUiOiJjb3Vwb24iLCJlbWFpbCI6ImppbUBqdWljZS1zaC5vcCIsInBhc3N3b3JkIjoiZTU0MWNhN2VjZjcyYjhkMTI4NjQ3NGZjNjEzZTVlNDUiLCJyb2xlIjoiY3VzdG9tZXIiLCJkZWx1eGVUb2tlbiI6IiIsImxhc3RMb2dpbklwIjoiIiwicHJvZmlsZUltYWdlIjoiYXNzZXRzL3B1YmxpYy9pbWFnZXMvdXBsb2Fkcy9kZWZhdWx0LnN2ZyIsInRvdHBTZWNyZXQiOiIiLCJpc0FjdGl2ZSI6dHJ1ZSwiY3JlYXRlZEF0IjoiMjAyNi0xMS0wNSAxOToyMjoxMS40NTIgKzAwOjAwIiwidXBkYXRlZEF0IjoiMjAyNi0xMS0wNSAyMDoyMjozNC41ODYgKzAwOjAwIiwiZGVsZXRlZEF0IjpudWxsfSwiaWF0IjoxNzk0MDQ2NjUyfQ.px9CYQVpiP0Ixyg4AmVwNo3HOuMMelqWtWj7Per3eG9zg1iPUdMUQEKtx2zDoSvO00a4Rx_XjeKgI_k1y2FEalEXVTJMXdlPWLHGDlZfHqmyLaETL73Ir2leSWeBvpaG2xoQ3nkkNmfDhqKTxLnPMKCInqA70nML4Yb09qlJIIw
<img width="879" height="281" alt="截屏2026-11-07 18 20 06" src="https://github.com/user-attachments/assets/8b0e9edb-881f-4826-8d09-ad42554aa5e6" />

3. 对token进行解码可以看到使用的是JWT然后密码是一段MD5算法
<img width="1040" height="738" alt="截屏2026-11-07 18 20 52" src="https://github.com/user-attachments/assets/a19d6081-84da-4dc5-ba25-7488d66a4df6" />

4. 在反馈内容输入框中，输入MD5
<img width="285" height="372" alt="截屏2026-11-07 18 24 04" src="https://github.com/user-attachments/assets/08ab114e-919d-46ce-ac23-efebb0e3e8c8" />


3. 实战危害与加固建议
实战危害展示：在真实的渗透测试中，如果系统的密码使用 MD5 进行哈希存储，一旦数据库被“脱库”，黑客可以使用彩虹表或通过 CMD5 等在线解密网站，在几秒钟内将绝大多数用户的哈希值逆向还原为明文密码，导致用户在全网的其他账号惨遭“撞库”攻击，引发大规模数据泄露。

加固方案：
1. 废弃老旧算法：在系统架构中全面淘汰 MD5、SHA-1 等容易产生哈希碰撞的算法，以及 DES、RC4 等过时的对称加密算法。
2. 采用现代哈希机制：对于用户密码的存储，绝不能使用简单的单向哈希。必须使用加盐的现代密码哈希函数，推荐使用 `Argon2`、`bcrypt` 或 `PBKDF2`。这些算法不仅不可逆，还能通过“工作量证明”机制刻意拖慢计算速度，从根本上抵御自动化暴力破解和彩虹表攻击。
