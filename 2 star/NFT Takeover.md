漏洞名称：NFT 钱包接管

1. 挑战概述
漏洞类别：敏感信息泄露
挑战目标：找到泄露的 12 位 BIP39 助记词，从而接管包含官方 Soul Bound Token的加密钱包。

2. 漏洞发现线索与复现过程

在 Web3 和区块链项目中，私钥或助记词就是一切。如果开发人员不小心将助记词硬编码在前端代码、注释或是团队介绍的照片中，就会导致资产被瞬间洗劫。

1.找到相关路径

<img width="143" height="24" alt="截屏2026-11-07 16 06 29" src="https://github.com/user-attachments/assets/692c8fea-be71-46b7-b488-87e75851ca7c" />

2. 进入关于我们的页面。
3. 检查页面上公开展示的宣传图片。

<img width="613" height="258" alt="截屏2026-11-07 15 57 41" src="https://github.com/user-attachments/assets/e78a6178-d30d-45f2-9154-d358288d4225" />

4. 在图片中，毫无防备地暴露了一串 12 个英文单词：purpose betray marriage blame crunch monitor spin slide donate sport lift clutch
5. 然后对12个词进行转换为私钥得到0x5bcc3e9d38baa06e7bfaab80ae5957bbe8ef059e640311d7d6d465e6bc948e3e
<img width="695" height="702" alt="截屏2026-11-07 16 22 57" src="https://github.com/user-attachments/assets/03970df4-6c9e-48eb-85b9-bc4b938952bd" />
<img width="727" height="222" alt="截屏2026-11-07 16 23 32" src="https://github.com/user-attachments/assets/ae663ebb-db86-42af-94c5-2ad75371e542" />

6. 进入juicy-nft页面输入私钥
<img width="865" height="421" alt="截屏2026-11-07 16 12 57" src="https://github.com/user-attachments/assets/346260f0-7681-40da-9bcd-16107c49c1be" />


3. 漏洞危害与加固建议
实战危害展示：在去中心化应用中，助记词泄露是导致严重资损的头号杀手。与传统 Web2 漏洞不同，区块链交易是不可逆的。一旦黑客通过 GitHub 泄露、前端打包文件或社交媒体图片爬取到这 12 个单词，钱包内的所有 NFT 和加密货币将在几秒钟内被自动化脚本洗劫一空。

修复方案：
1. 严格的密钥管理生命周期：助记词和私钥绝对禁止以任何明文形式出现在代码仓库、前端文件或公开的图片宣传物料中。
2. 上线前机密扫描：在 CI/CD 流水线中引入硬编码机密扫描工具，在发布前强制拦截所有疑似 BIP39 单词格式、私钥或 Token 的敏感字符串。
