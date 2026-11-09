漏洞名称：已知脆弱组件漏洞 / 供应链安全排查

1. 挑战概述
漏洞类别：使用含有已知漏洞的组件
挑战目标：通过信息收集获取系统的依赖库清单，对比官方的安全公告，找出系统正在使用的严重高危第三方组件，并将其报告给官方。

2. 漏洞发现线索与复现过程

在现实，黑客不需要去死磕企业自己写的复杂业务代码，而是直接寻找企业引入的第三方开源库。只要这些库存在公开的安全公告且未被修复，黑客就能直接利用现成的 EXP 轻松拿下服务器。

  1. 在/ftp目录下有`package.json.bak` 文件。
<img width="895" height="342" alt="截屏2026-11-09 18 41 48" src="https://github.com/user-attachments/assets/f2c3bd49-ffa1-4a57-9532-e291435b9981" />
<img width="885" height="360" alt="截屏2026-11-09 18 43 04" src="https://github.com/user-attachments/assets/aa8a75b6-2bfb-499f-b496-8143e0daf63b" />

  2. 发现无法打开没有权限，前面挑战也了解到只有md，pdf，zip可以被上传下载，使用我们加上%2500.md后缀即可下载
<img width="805" height="377" alt="截屏2026-11-09 18 44 59" src="https://github.com/user-attachments/assets/35f0eccf-3092-42ec-89c9-729255e7878e" />

  3.其中%25可以解码成%变成%00.md然后会再次解码变为空字符
<img width="656" height="93" alt="截屏2026-11-09 18 49 43" src="https://github.com/user-attachments/assets/862ce5df-ac75-4c3a-bc9a-af2b0f571f36" />
<img width="451" height="381" alt="截屏2026-11-09 18 51 54" src="https://github.com/user-attachments/assets/07b8990d-aaed-4da5-9a2e-3166d0da9c64" />

  4. 搜寻文件中可能存在的漏洞，发现系统存在极高危已知漏洞。
  5. 在客户反馈提交漏洞即可
<img width="273" height="128" alt="截屏2026-11-09 19 12 57" src="https://github.com/user-attachments/assets/ac7a4d63-f80f-41dc-a025-d728cf7aaad3" />


3. 实战危害与加固建议
实战危害展示：近几年的网络安全重大安全事件（如 Log4j2 核弹级漏洞、SolarWinds 供应链攻击），本质上都是第三方组件漏洞。企业通常对内部代码审计严格，但对引用的开源库缺乏管控。一旦某个底层依赖库爆出 0-day 或 N-day 漏洞的 Security Advisory，黑客就可以利用现成的自动化脚本，对全网使用该组件的企业进行无差别屠杀，直接获取服务器最高权限。

修复方案：
1. 建设 SBOM与自动化 SCA：企业必须在 CI/CD 流水线中强制集成自动化扫描工具。在代码打包发布前，实时与全球 CVE 漏洞库和安全公告进行比对，发现高危依赖立即阻断发布。
2. 持续的补丁管理生命周期：建立安全应急响应机制，一旦订阅的开源组件发布了高危 Security Advisory，必须在 24 小时内评估影响，并强制升级到安全的修补版本。
3. 很多漏洞就是校验前解一次码，读文件前又自动解一次，给了双重编码绕过的机会
4. 直接拒绝包含  \0 、 %00 、 %2500  这类空字节相关字符的输入
