漏洞名称：额外语言包 / 隐藏敏感资产暴露

1. 挑战概述
漏洞类别：敏感数据暴露 / 隐蔽式安全缺陷

挑战目标：找出一个从未正式发布在前端界面的隐藏语言包文件，并成功访问它。

2. 漏洞发现线索与复现过程

在 Web 应用中，开发团队经常会在生产环境中遗留一些“未完成”、“测试中”或“彩蛋性质”的静态资源。

1. 切换语言，打开burp suite查看，发现了一个路劲
<img width="362" height="398" alt="截屏2026-11-11 20 21 13" src="https://github.com/user-attachments/assets/373b8436-1df7-4f4c-9460-9f9cb240e531" />

2. 然后进行代码审查，发现没有什么有价值的线索、
<img width="676" height="291" alt="截屏2026-11-11 20 36 01" src="https://github.com/user-attachments/assets/b777819d-2f65-4529-be1c-508a4b69de9b" />

3. 在谷歌上搜索有关内容发现
<img width="692" height="710" alt="截屏2026-11-11 20 37 10" src="https://github.com/user-attachments/assets/558ba481-d1b6-4f4c-a580-fd3bc94590c0" />

4. 访问该网站发现有一个语言包里没有显示的名称
<img width="794" height="366" alt="截屏2026-11-11 20 39 26" src="https://github.com/user-attachments/assets/e5180555-fb28-4678-a36b-58e316bc22c8" />

5. 找到其代码是tlh_AA
<img width="688" height="223" alt="截屏2026-11-11 20 40 54" src="https://github.com/user-attachments/assets/51b11aea-aad8-4c93-893c-2256b31750e6" />
<img width="339" height="139" alt="截屏2026-11-11 20 42 24" src="https://github.com/user-attachments/assets/69790118-af68-4456-ad7a-d4b2b58041f2" />

6. 输入改路劲即可
<img width="731" height="465" alt="截屏2026-11-11 20 43 11" src="https://github.com/user-attachments/assets/141df945-9a7a-4f11-86b7-d7e6165210a5" />

3. 实战危害与加固建议
实战危害展示：
这道题表面上只是找到了一个克林贡语彩蛋，但这种攻击手法叫做“隐藏资产挖掘”。黑客经常会在企业网站的静态目录下找到 `test.html`、`api_v1_backup.json` 甚至是包含了内网测试域名和测试账号的 `config-dev.json`。这些未下线的测试资产往往不受正常权限系统的保护。

修复方案：
1. 规范发布流程：在 DevOps 流水线的构建阶段，必须配置清理脚本。打包发布到生产环境的静态目录（如 `/assets/` 或 `/public/`）中，只能包含最终业务必需的文件，强制剔除所有 `.bak`、测试配置及未完成的模块。
2. 关闭不必要的目录遍历：确保 Web 服务器严格关闭了目录列表功能，防止攻击者直接浏览 `/assets/i18n/` 下的所有文件。
