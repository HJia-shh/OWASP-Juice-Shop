漏洞名称：获取产品蓝图 / 后缀校验绕过与任意文件下载

1. 挑战概述
漏洞类别：敏感数据暴露 / 不当的输入验证
挑战目标：绕过系统对文件下载的白名单限制，成功下载一款核心产品的 3D 蓝图文件，从而“剥夺商店的收入”。

2. 漏洞发现线索与复现过程

/ftp这个目录不仅存放着合法的文档，还存放着商店的数字资产。

1. 有一款叫OWASP Juice Shop Logo 的商品。
<img width="328" height="345" alt="截屏2026-11-12 02 10 45" src="https://github.com/user-attachments/assets/512c726b-2b4d-43fb-8b59-0d2d9a16009d" />

2. 没有任何思路。。。。

3. 对他进行exif查看图片，并没有发现什么
<img width="447" height="439" alt="截屏2026-11-12 02 15 36" src="https://github.com/user-attachments/assets/f589de72-c7a8-4dd9-9caa-dfe7c690cdf3" />

未完成此挑战，不知道有什么修复建议

查看手册后发现

<img width="215" height="56" alt="截屏2026-11-12 02 20 07" src="https://github.com/user-attachments/assets/25ed63e7-2629-456b-b5f1-61db365e5732" />
