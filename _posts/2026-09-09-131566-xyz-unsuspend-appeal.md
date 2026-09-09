---
layout: post
title: "131566.xyz 解封申诉记录：从 serverHold 到恢复解析"
subtitle: "一次域名被注册局挂起后的排查、清黑与沟通全过程"
date: 2026-09-09 21:00
author: "磁贴"
header-img: "img/post-bg-digital-native.jpg"
catalog: true
tags:
    - 域名
    - serverHold
    - 申诉
    - VirusTotal
    - DNS
---

我的域名 131566.xyz 在 2026 年 8 月被 .xyz 注册局挂上了 `serverHold`，全网 NXDOMAIN，绑在 Cloudflare 上的站点一直无法激活。从 8 月中旬开始排查，到 9 月 8 日确认解封，前后二十多天，中间给注册局和十几家安全厂商都递过申诉，被驳回过，也放弃过几家。这篇文章按时间线把整个过程记下来，既是存档，也留给以后可能遇到同样问题的人一个参考。

## 域名为什么会被封

131566.xyz 注册于 2025-05-07，注册商是 Spaceship（IANA 3862），到期时间 2028-05-07，并没有过期。我很早就把 NS 改到了 Cloudflare 的 `abdullah.ns.cloudflare.com` 和 `ada.ns.cloudflare.com`，但 Cloudflare 那边一直显示 pending nameserver，怎么点 "Check nameservers now" 都不认。

排查后确认 NS 变更本身是成功的——whois 里 Name Server 已经是 Cloudflare 的两个地址。真正的问题在 `Domain Status: serverHold`：这是注册局层面的状态，不是注册商层面的。.xyz 注册局（由 CentralNic 运营）把域名从 DNS 区域里摘掉了，TLD 直接返回 NXDOMAIN，Cloudflare 查不到委托，自然永远不会激活。

至于为什么会被挂 serverHold，我只能推断：这个域名之前挂过基于 Cloudflare Workers 的反代和伪装页面，大约一年前因为 phishing 举报被处理过一次。当时我向 Spaceship 申诉过一次（原邮件已删），Spaceship 解开了注册商侧的封禁，但漏掉了注册局侧的 serverHold，域名就这么一直挂着。这只是我的推断，注册局从未给过具体的封禁理由。

2026 年 8 月中旬我再次联系 Spaceship，对方的回复很明确：域名是被 .xyz 注册局挂起的，需要直接联系注册局，并给出了官方解封入口 [gen.xyz/unsuspend](https://gen.xyz/unsuspend)。到这里，申诉对象才算真正确定下来。

## 提交前的准备：先查清黑名单

gen.xyz 的解封表单有明确的步骤要求：先确认域名是否在各黑名单里，被列入的要先申诉移除，然后描述已采取的动作并提供证据。所以正式提交之前，我先做了一圈核查。

截至 2026-08-23 的结果：Spamhaus DBL、SURBL、URIBL 均未列入，Google Safe Browsing 显示 Clean。唯一的麻烦在 VirusTotal——91 家引擎里有 10 家把域名标成 Phishing 或 Malicious，包括 Webroot、BitDefender、ESET、Sophos、Fortinet 和几家小厂。

策略因此变成两步走：先把 VT 上能清的厂商清掉，再向注册局提交解封申请。第一个突破是 Webroot：向 BrightCloud 提交变更申请后，对方把域名改判为 "Parked Domains"（发布于数据库 9.863），VT 同步变成 Clean，计数从 10/91 降到 9/91。同期我还提交了 BitDefender、ESET 的误报申诉，并用邮件向 Sophos 立了案（case #03456873）。不太顺利的是 Fortinet，首次复核在两小时内由自动流水线完成，结论是维持 Phishing。

## 正式提交：注册局工单与厂商申诉

2026-08-29，我决定不等 VT 降到理想值，先把注册局的申请递出去。在 gen.xyz/unsuspend 上 Lookup 域名，确认状态是 "Suspended for Abuse reported"，点 Request Review，过了 reCAPTCHA 后按表单填写：Spamhaus/SURBL/URIBL/Google Safe Browsing 均未列入；域名当前 NXDOMAIN、不承载任何内容；已向 ESET、VIPRE、Sophos、Bitdefender 提交误报申诉，Webroot 已改判；今后域名仅用作个人博客。提交后拿到工单 #992668，随后收到确认邮件，XYZ Anti-Abuse Team 承诺 48 小时内响应。同一天我还补了几家厂商：ESET 二次提交成功，VIPRE 通过 Zendesk 帮助台提交（需要先在邮箱里点验证链接），Forcepoint 提交了重分类申请并收到 case #01066738 的确认邮件。

这里有个明知的风险：提交时 VT 仍是 10/91（Last Analysis 停在 4 天前），而注册局的要求是从所有黑名单移除后才重新激活，这单很可能被要求继续清理后复审。事实也确实如此。

8 月 31 日是密集操作的一天。先是收到工单状态更新——#992668 从 Open 变为 Answered，但偏偏 gen.xyz 的客户端区这时数据库故障，页面只有 "Could not connect to the database"，回复正文看不了。我改用 RDAP 直接查 CentralNic：状态仍是 `server hold` + `client transfer prohibited`，last changed 停在 2026-05-05，说明注册局侧还没有任何动作。同一天我把 VT 上剩下的厂商全部覆盖了一遍：G-Data（Friendly Captcha 需要人工过验证）、Lionic、CyRadar、CRDF（拿到参考号 #20260831624599）、alphaMountain（Freshdesk 工单 #124531）。Fortinet 的二次升级申诉则因为人机验证被拦、且首次已被正式驳回，我决定放弃。

## 驳回、循环与胶着

接下来陆续回来的消息坏的多。Forcepoint 判定 "Phishing and Other Frauds"，维持原分类；CRDF 人工审核后认为不符合移除条件，维持 Malicious；加上此前的 Fortinet，十多家厂商里已有三家被正式驳回。gen.xyz 工单系统恢复后，我终于看到了 8 月 31 日凌晨的回复（Agent 543623），核心意思是：域名因违反反滥用政策被标记，必须联系被列入的黑名单逐一复审，等把所有黑名单都移除后，回复工单附上 delisting 证据，注册局才会考虑解封。也就是说，VT 上那一串厂商不清理掉，工单不会再往前走。

和 CRDF 的拉锯是整个过程里最消耗耐心的部分。它先驳回了我的申请，但提示可以在 web thread 里补充新信息。9 月 2 日我在线程里回了两轮：第一轮提供注册商账户层面的所有权证明（注册日期、Spaceship 账户、Cloudflare NS 配置），并说明可以配合添加 TXT 验证记录；第二轮按它要求的 "concrete technical changes" 路径，解释域名当前 NXDOMAIN、A/NS/TXT 均无解析，serverHold 状态下在 @131566.xyz 上设置 DNS TXT 在技术上不可行，请求对方改走注册商渠道验证所有权。结果是 CRDF 反复回同一句模板——"请提供一个具体的下一步"——明显是自动回复循环。我转而通过它的联系表单升级（参考号 175567-657283），被系统自动归类后又用 forced override 链接把消息送进了人工队列，但等来的还是同样的模板。到此我放弃了 CRDF。

同一时期，其余已提交申诉的厂商一封邮件回复都没有，局面一度胶着：已确认改判的只有 Webroot，确认 delist 的只有 G-Data（回复原话是 "The submitted URL is not reachable, it is no longer detected by us."），三家被驳回，剩下的全部状态不明。

## 转机：静默改判与解封

9 月 2 日下午，我在工单 #992668 里回复了阶段性证据：Webroot 已改判 Parked Domains、G-Data 已确认不再检测、Spamhaus/SURBL/URIBL/Google Safe Browsing 均未列入、域名当前不承载任何内容，并说明 ESET、BitDefender、VIPRE、Sophos、alphaMountain、Lionic、CyRadar、Chong Lua Dao 的误报申诉都在处理中，请求注册局移除 serverHold。工单状态随即从 Answered 变为 Customer-Reply。

真正的转机出现在 9 月 8 日。这天我复查 VirusTotal，发现计数已经从 10/91 降到了 6/89：BitDefender、ESET、VIPRE、Chong Lua Dao、Lionic 五家全部变成了 Clean——全部是静默改判，没有一家发过邮件通知。仍在标记的只剩六家：alphaMountain、CyRadar、Sophos 还在处理，CRDF、Fortinet、Forcepoint 已被驳回。

同一天我重新打开 gen.xyz 工单，才发现注册局其实早在 9 月 2 日 17:09 就回复了（Agent 654278），我之前一直没注意到。回复的关键句是 "This domain will be unsuspended and closely monitored"，同时警告如果域名继续出现滥用行为可能被无限期重新暂停，并提示 WHOIS 更新最长需要 24 小时。我立刻做了验证：RDAP 查询显示状态只剩 `client transfer prohibited`，`serverHold` 已经移除；NS 委派恢复，公共 DNS（1.1.1.1、8.8.8.8）都能解析到 Cloudflare 的两个 NS；A 记录暂时只返回 SOA，说明委派成功，就差最后一步——登录 Cloudflare 对 131566.xyz 点一次 "Check nameservers now"，站点激活后解析即可生效。

至此，这场持续二十多天的申诉算是走完了。域名从注册局挂起、全网 NXDOMAIN，恢复到正常委派状态；VT 上的标记从 10 家降到 6 家，剩下的几家已不影响解封结果。

## 经验总结

几条从这次经历里实际得到的经验：

- 先分清封禁发生在哪一层。注册商侧的限制找注册商，`serverHold` 是注册局状态，只能找注册局； Spaceship 的回复其实一开始就把方向指明白了，只是要先看懂 whois 里的状态字段。
- 注册局解封的核心是黑名单证据链，而 VirusTotal 是最直观的进度表。厂商要逐个提交误报申诉，而且很多厂商改判后不发任何通知，定期复查 VT 比等邮件靠谱——这次有五家都是静默改判的。
- 被正式驳回的厂商不必死磕。Fortinet、CRDF、Forcepoint 最终都没能移除，但注册局还是同意了"解封并密切监控"，可见它看的是整体清理进展，不要求绝对清零。当然这只是本案的结果，不代表普遍规则。
- 域名处于 NXDOMAIN 本身是最有力的论据之一：不解析、无内容，就不可能在钓鱼任何东西，几乎每一份申诉文案我都用到了这一点。
- 申诉表单的人机验证对数据中心 IP 很不友好，好几次提交都需要换到家庭宽带手动过验证码，提前有心理预期能省不少时间。
- 最后，解封是有条件的。注册局明确说了会密切监控、再犯可能无限期重封，所以恢复之后这个域名只会挂博客内容——这也是我在申诉里作出的承诺。
