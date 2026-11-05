漏洞名称：DOM 型跨站脚本攻击 (DOM XSS)

1. 挑战概述
漏洞类别：跨站脚本攻击/前端渲染安全
挑战目标：在应用的页面中注入恶意JavaScript代码，成功触发一个包含 `alert` 的系统弹窗。

2. 漏洞发现线索与复现过程

寻找前端输入与回显点
在Web中，任何“用户输入后会在页面上原样显示”的地方都是 XSS 的高危测试点。当然Juice Shop的首页顶部提供了一个商品搜索框。
1. 点击首页右上角的搜索图标，在搜索框中输入普通的测试文字（如 `apple`），回车。
2. 观察到页面显示 `查找结果 - apple`。这说明前端代码直接读取了 URL 中的 `?q=apple` 参数，并渲染到了 HTML 页面 DOM 树中。

<img width="759" height="446" alt="截屏2026-11-06 00 54 42" src="https://github.com/user-attachments/assets/7b73e194-a231-4966-b82c-c36d6b25700c" />

根据挑战内容构造 Payload 触发弹窗
为了测试该渲染位置是否对HTML标签进行了“消毒”过滤，挑战内容里有一个带有执行JavaScript能力的恶意 iframe 标签：

<img width="277" height="28" alt="截屏2026-11-06 00 58 17" src="https://github.com/user-attachments/assets/82602cb4-c88b-4b07-bfef-797c7acc3323" />

1.我们先对<iframe src="javascript:alert(`xss`)">进行URL编码得到%3ciframe+src%3d%22javascript%3aalert(%60xss%60)%22%3e

<img width="635" height="286" alt="截屏2026-11-06 01 07 48" src="https://github.com/user-attachments/assets/486f1d1d-a16d-4029-9445-e90a5e873087" />

2. 在搜索框中输入以下 Payload 并回车：
http://127.0.0.1:3000/#/search?q=%3ciframe+src%3d%22javascript%3aalert(%60xss%60)%22%3e
完成了该挑战
<img width="339" height="106" alt="截屏2026-11-06 01 09 16" src="https://github.com/user-attachments/assets/edddf860-afba-41b0-b64f-31bb780fb4dc" />

3. 修复方案：

禁止危险渲染：严格使用纯文本属性（如 textContent 或 innerText）来回显用户数据，避免直接向 DOM 中插入 HTML。
上下文编码与消毒：如果业务确实需要渲染富文本，必须在渲染前使用成熟的消毒库（如 DOMPurify）对用户输入进行 HTML 实体编码和白名单过滤。
