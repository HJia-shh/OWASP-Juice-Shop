漏洞名称：元数据地理追踪 / 隐私泄露

1. 挑战概述
漏洞类别：敏感信息泄露 / 隐私防护缺陷
挑战目标：提取平台上公开图片中的 EXIF 元数据，追踪出特定用户的物理位置，并以此作为社工线索重置该用户的密码。

2. 漏洞发现线索与复现过程

在 Web 渗透测试和开源情报收集中，未经脱敏处理的用户上传文件是挖掘受害者真实身份、住址和行踪的宝库。

1. 访问照片墙页面，寻找潜在的突破口。
2. 发现一张由用户 John 拍摄并上传的徒步照片。
<img width="223" height="300" alt="截屏2026-11-07 15 43 39" src="https://github.com/user-attachments/assets/f480db9c-b7a2-4b93-a1cb-e41ef436a2a9" />

3.确定John的重置密码问题

<img width="210" height="28" alt="截屏2026-11-07 15 44 42" src="https://github.com/user-attachments/assets/ff38bbfc-6df8-4813-a08f-6a2b1a3190c2" />

4. 使用本地命令行工具分析该照片。
5. 在解析出的 EXIF 数据中，发现系统完全没有抹除原始信息，直接暴露了照片拍摄时的精确 GPS 经纬度坐标：36°57'31.38"N 84°20'53.58"W
<img width="398" height="290" alt="截屏2026-11-07 15 45 46" src="https://github.com/user-attachments/assets/dad4e6d2-2d89-4f4b-8b25-d6056fb293b1" />
<img width="430" height="80" alt="截屏2026-11-07 15 46 23" src="https://github.com/user-attachments/assets/000ead7f-cc18-4c2a-a635-5fc73124341d" />

6. 将提取到的经纬度输入到 Google Maps中进行搜索，定位到该地点的精确名称：Daniel Boone National Forest
<img width="380" height="280" alt="截屏2026-11-07 15 49 10" src="https://github.com/user-attachments/assets/7714629d-c026-4259-9db6-01c5263624be" />

7.输入安全问题修改密码

<img width="257" height="375" alt="截屏2026-11-07 15 49 50" src="https://github.com/user-attachments/assets/3dc83d70-8f7a-41d9-aeb1-756a94286775" />


3. 漏洞危害与加固建议
实战危害展示：在护网或红蓝对抗中，这种缺陷极其致命。黑客可以通过企业官网的“团队风采”或“员工入职照”，提取出拍摄地点的 GPS 坐标甚至内网 Wi-Fi 名称。这不仅会严重威胁员工的物理人身安全，这种基于私人生活轨迹的答案还会直接击穿系统的“安全问题”找回机制，导致账号被全面接管。

修复方案：
1. 图像上传强制脱敏：在后端处理用户上传的图片文件时，绝不能直接落盘保存原始文件。必须在代码逻辑中调用图像处理库，强制剥离所有非像素数据的 EXIF信息，只保留纯净的图像内容。
2. 废除静态安全问题：基于“最喜欢的宠物”、“出生的城市”等静态信息的安全问题在现代安全标准中已彻底过时。应全面废弃此类密码重置机制，改为强制使用邮箱验证码、短信验证码或 TOTP 动态令牌进行身份验证。
3. 
