漏洞名称：隐写术与敏感信息隐藏 / 隐蔽式安全缺陷

1. 挑战概述
漏洞类别：敏感数据暴露 / 隐蔽式安全缺陷

挑战目标：从前端公开的静态图片资源中，提取出被开发人员或黑客刻意隐藏在像素/文件末尾的机密信息。

2. 漏洞发现线索与复现过程

在 Web 渗透与恶意样本分析中，当常规的 Web 攻击面被排查殆尽时，我们需要将目光转向静态资源。很多时候，看似普通的图片、音频文件中，可能藏着后门密码、隐藏路由甚至完整的 WebShell。

1. 在关于我们中通过inspect发现了一张png格式的照片，将其保存下来
<img width="308" height="74" alt="截屏2026-11-10 02 13 02" src="https://github.com/user-attachments/assets/dacbec77-726f-4731-a996-01a2173f8356" />
<img width="568" height="496" alt="截屏2026-11-10 02 14 24" src="https://github.com/user-attachments/assets/bf8770a6-607b-4c59-9cda-511c86476269" />

3. 启动终端，使用常见的隐写分析工具流对图片进行分析：
4. 使用exiftool，查看开发人员是否将密码或敏感路由写在了版权、作者或备注等 EXIF 标签中。
<img width="459" height="314" alt="截屏2026-11-10 02 17 16" src="https://github.com/user-attachments/assets/23f64b46-7527-4e1d-ad47-4b07c5af0d45" />

5. 并没有发现什么，通过对隐写术的搜索，我找到了一个专门工具OpenStego
<img width="725" height="360" alt="截屏2026-11-10 02 19 23" src="https://github.com/user-attachments/assets/1b729867-5d5d-4bbe-a5bc-2fbbdb8d0a48" />

6. 对其进行分析，得到了一张图片，发现他是《瑞克和莫蒂》中的 Pickle Rick
<img width="529" height="124" alt="截屏2026-11-10 02 20 26" src="https://github.com/user-attachments/assets/113943e0-00de-4116-ace3-6a6706381e4a" />

<img width="86" height="118" alt="截屏2026-11-10 02 21 06" src="https://github.com/user-attachments/assets/8ee3b35c-9451-4867-9bdd-5a9f12f01605" />

7. 输入答案即可
<img width="370" height="372" alt="截屏2026-11-10 02 22 56" src="https://github.com/user-attachments/assets/8623534d-aef2-400a-be05-1fcbde5dfb21" />

3. 实战危害与加固建议
实战危害展示：隐写术在日常 Web 漏洞挖掘中较为少见，但在高级持续性威胁和黑灰产免杀中极其泛滥。攻击者经常将一句话木马隐藏在正常图片的 EXIF 信息或像素位中上传，以此绕过 WAF 的恶意文件检测。此外，内网的失陷主机也会将窃取的敏感数据压缩隐写在图片中，再通过正常的 HTTP 图片请求发送给外部的 C2 服务器，从而完美绕过防火墙的流量审查。

修复方案：
1. 强制图片重编码与二次渲染：企业在处理用户上传的图片，或发布内部生成的图片时，绝不能直接原样落盘。必须在后端使用 ImageMagick、GD 库等图像处理引擎，对图片进行统一的尺寸裁剪、格式转换和二次重绘。这个过程会彻底破坏原始图片中的 LSB 隐写数据和附加的文件尾。
2. 剥离所有元数据：在图片上传和分发的处理管道中，强制执行元数据剥离操作，清除不必要的 GPS 坐标、设备型号、作者备注等敏感信息。
3. 部署高级内容清洗：对于高安全级别要求的政企网络，应在网络边界或邮件网关部署“内容解除与重建”技术，对所有传入传出的富媒体文件进行物理级的净化。



