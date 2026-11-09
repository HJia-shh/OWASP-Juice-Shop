漏洞名称：多重编码隐藏路由提取 / 隐蔽式安全与敏感信息泄露

1. 挑战概述
漏洞类别：信息泄露 / 隐蔽式安全缺陷

挑战目标：通过分析暴露的文件内容，利用解码工具找出一个经过多重混淆的隐藏深层路由，并访问最终的终极彩蛋页面。

2. 漏洞发现线索与复现过程

在 Web 渗透测试的情报分析阶段，攻击者在获取到敏感文件后，通常会审查其中是否有异常的字符串。开发人员经常错误地将 Base64、URL 编码或简单的替换密码当作“加密”手段，用来隐藏后门地址或内部接口。

1.访问/ftp下载Eastere.gg。
<img width="882" height="343" alt="截屏2026-11-09 21 36 13" src="https://github.com/user-attachments/assets/8ed295aa-3be0-4b5c-b472-639a00608a4f" />

2. 这里依旧需要让URL进行两次解码，因为只能下载pdf或者zip文件，在后面加上%2500.md进行下载
<img width="617" height="90" alt="截屏2026-11-09 21 37 51" src="https://github.com/user-attachments/assets/e7ffdc51-85ab-4e17-8b87-716301e6ec73" />

3、. 发现在代码注释中隐藏着一串的 Base64 编码。
<img width="449" height="272" alt="截屏2026-11-09 21 38 33" src="https://github.com/user-attachments/assets/2f311910-bc78-42a0-aaa8-ccf68fd089e0" />

4.进行解码得到：/gur/qrif/ner/fb/shaal/gurl/uvq/na/rnfgre/rtt/jvguva/gur/rnfgre/rtt
<img width="910" height="276" alt="截屏2026-11-09 21 39 26" src="https://github.com/user-attachments/assets/6947da3b-2189-49b5-8692-1173fa18cb39" />

5. 发现是ROT 13的编码，再次进行解码得到/the/devs/are/so/funny/they/hid/an/easter/egg/within/the/easter/egg
<img width="754" height="476" alt="截屏2026-11-09 21 42 34" src="https://github.com/user-attachments/assets/f55a6fe3-1b17-4737-9340-45ad08364138" />

6. 访问该路劲即可
<img width="936" height="450" alt="截屏2026-11-09 21 44 25" src="https://github.com/user-attachments/assets/47cb08a9-c0f3-4471-93f2-63424b4ace93" />

3. 实战危害与加固建议

实战危害展示：这种多层编码隐藏信息的做法在软件分析和企业安全漏洞中非常常见。在企业业务中，如果开发人员将未授权的调试接口、SSO 跳转秘钥或临时 Token 用这种简单的编码混淆后写死在前端 JS 或公开文档中，攻击者可以利用自动化脚本轻易解码，直接拿到系统深层的后门入口。

修复方案：
1. 区分“编码”与“加密”：必须在开发团队中强调，Base64、ROT13、Hex 只是数据格式转换方案，不具备任何保密性。绝不能用它们来隐藏敏感配置、接口路由或密码凭证。
2. 剥离测试代码与彩蛋：在发布生产环境版本前，必须严格执行代码审查，利用脚本剔除所有预埋的测试后门、调试页面及无实际业务用途的“彩蛋”。
3. 严格的访问控制：不论接口路由多么隐蔽、名字多么长，只要是未开放给普通用户的端点，都必须接入全局的 IAM 鉴权中间件。确保即便黑客猜到了路由，没有合法的管理员 Token 也只会得到 401 或 403 阻断返回。
