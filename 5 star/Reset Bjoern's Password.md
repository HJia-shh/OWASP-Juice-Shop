漏洞名称：重置 Bjoern 的密码 / 开源情报与安全问题绕过

1. 挑战概述
漏洞类别：身份验证破坏 / 开源情报

挑战目标：通过信息收集获取 Bjoern 的个人隐私信息，回答“忘记密码”功能中的安全问题，从而强制重置他的内部账号密码。

2. 漏洞发现线索与复现过程

这是一道非常纯粹的社工与开源情报挑战。在实战中，企业高管或核心开发人员的社交媒体，往往是攻破企业内网的“零成本”突破口。靶场原作者 Bjoern Kimminich 故意将自己的真实生活轨迹留在了互联网上，供白帽子进行追踪验证。

1. 确定邮箱：bjoern@juice-sh.op，其安全密码为
<img width="216" height="50" alt="截屏2026-11-12 01 22 18" src="https://github.com/user-attachments/assets/cb01cb26-e0f8-4e0d-900d-2fa49e15e31b" />

2. 在https://gist.github.com/bkimminich中得到
<img width="261" height="298" alt="截屏2026-11-12 01 27 11" src="https://github.com/user-attachments/assets/ac449af2-ca80-4c38-94ed-f4818a4ec1a3" />

3. 在https://www.facebook.com/bjoern.kimminich中得到
<img width="474" height="580" alt="截屏2026-11-12 01 25 52" src="https://github.com/user-attachments/assets/7e512325-20e4-4e0d-b72f-76dc41069fb1" />

4. 找到了Uetersen的邮件为25436，输入后却发现邮件错误。

5. 继续搜索得知德国统一前编码不一致，将25436转换成之前代码得到，然后进行修改即可
<img width="639" height="149" alt="截屏2026-11-12 01 33 00" src="https://github.com/user-attachments/assets/95942f21-be6a-4e8f-bf35-d34dfc5c500a" />

3. 实战危害与加固建议
实战危害展示：
“静态安全问题”是现代身份验证体系中最脆弱、最荒谬的一环。在社交媒体高度发达的今天，诸如“你的宠物叫什么”、“你的高中名字”、“你第一辆车的牌子”等问题，对黑客而言相当于“半公开信息”。攻击者完全可以通过翻阅目标的 Facebook 动态、Instagram 照片或 LinkedIn 履历，利用 OSINT 轻易推导出答案，从而彻底绕过复杂的强密码策略直接接管账户。

修复方案：
1. 彻底废弃静态安全问题：现代 Web 应用应完全摒弃基于个人历史信息的“找回密码”机制（NIST 美国国家标准技术研究所已明确不推荐使用该机制）。
2. 强制采用多因素身份验证：密码找回机制必须依赖于动态验证流，例如向绑定的手机发送短信验证码、发送包含限时 Token 的重置邮件，或者使用基于时间的一次性密码。
3. 账户异常行为监控风控：对于发起找回密码请求的客户端环境进行风控分析，若判断为异地或可疑设备，应触发额外的人脸或活体校验。

