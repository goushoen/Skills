---
name: daily-email-p0-p1-digest
description: Daily Outlook inbox triage that summarizes today's received email into only P0 and P1 items, decides whether the user needs to reply, and sends a concise multilingual action digest to Feishu/Lark. Supports Chinese, English, and Japanese delivery. Use when the user asks for a daily email summary, today's email action list, P0/P1 mail triage, messages that need reply or follow-up, or recurring evening delivery to Feishu/Lark.
---

# Daily Email P0 P1 Digest

## Overview

Use this skill to turn a daily slice of Outlook email into a short action digest and deliver it to Feishu/Lark. Keep the output practical: the user should know what needs action, whether they need to reply, and how to find the original email.

Feishu/Lark messages must support Chinese, English, and Japanese. Before first use, ask the user which language to use for the Feishu/Lark digest unless the language is already specified by the user or schedule.

中文说明：本 skill 用于每天整理当天 Outlook 邮件，只保留 P0/P1 关键事项，并推送到飞书。首次使用时请确认飞书消息语言：中文、English 或 日本語。

日本語説明：このスキルは、その日に受信した Outlook メールを P0/P1 の重要項目だけに整理し、Feishu/Lark に送信します。初回利用時は、送信メッセージの言語を中文、English、日本語から確認してください。

## Workflow

1. Confirm the Feishu/Lark message language.
   - Supported languages: Chinese, English, Japanese.
   - If the user explicitly specifies a language, use it.
   - If this is an interactive first use and no language is known, ask one short question: "飞书消息要使用哪种语言：中文、English、还是日本語？"
   - If this is a scheduled run and no language is configured, use the user's usual conversation language as the default.
   - Do not mix languages in the final digest unless the user asks.

2. Define the mail window.
   - For "today", use the user's local timezone and include mail received from 00:00 through the current run time.
   - For a scheduled evening run, summarize that calendar day.
   - If the user gives a custom date range, use it exactly.

3. Retrieve candidate emails from Outlook.
   - Prefer Outlook connector search/list actions with received-date filters and recency ordering.
   - Use list/search result fields first: subject, sender, received time, preview, read state, and links.
   - Fetch full bodies only when the preview is not enough to decide status, ownership, or reply need.

4. Classify only P0 and P1.
   - P0: The user must reply or act immediately, or the matter blocks customer, partner, delivery, money, legal, security, access, or executive work.
   - P1: The user does not need to reply right now, but should monitor because a reply, escalation, deadline, or follow-up may be needed soon.
   - Exclude P2/P3 from the digest. Newsletters, FYI, routine confirmations, and already-owned threads should not appear unless they create a P0/P1 action.

5. Decide whether the user needs to reply.
   - Mark "needs reply" only when the latest meaningful message asks the user for information, approval, decision, confirmation, or action.
   - Mark "temporarily no reply needed" when another named person is handling it, the latest message asks someone else, or the best action is to wait for feedback.
   - If uncertain, mark it as "may need confirmation" in the selected language and explain the uncertainty in one short sentence.

6. Write the digest in the selected language and keep it concise.
   - Use the matching format below.
   - Include subject, received time, and sender for fast Outlook lookup.
   - Do not add search keywords unless the user asks; subject + sender + time is enough for this workflow.

## Output Format

### Chinese

```text
每日邮件情况汇总单（YYYY-MM-DD）

P0｜立即处理
暂无。
```

If P0 has items:

```text
P0｜立即处理

1. 事项名称
邮件标题：...
邮件接收时间：H:mm 或 M/D H:mm
发件人：...

最新状态：...
是否需要你回复：需要。...
```

Then P1:

```text
P1｜重点关注 / 需要盯进展

1. 事项名称
邮件标题：...
邮件接收时间：H:mm 或 M/D H:mm
发件人：...

最新状态：...
是否需要你回复：暂不需要。...

今日建议行动
1. ...
2. ...
```

### English

```text
Daily Email Digest (YYYY-MM-DD)

P0 | Immediate Action
None.

P1 | Watch / Follow Up

1. Item name
Subject: ...
Received: H:mm or M/D H:mm
Sender: ...

Latest status: ...
Do you need to reply: Not for now. ...

Suggested actions today
1. ...
2. ...
```

### Japanese

```text
本日のメール要約（YYYY-MM-DD）

P0｜至急対応
なし。

P1｜要確認 / フォローアップ

1. 件名または項目名
メール件名：...
受信時刻：H:mm または M/D H:mm
送信者：...

最新状況：...
返信要否：現時点では不要。...

本日の推奨アクション
1. ...
2. ...
```

## Feishu/Lark Delivery

Send the finished digest to Feishu/Lark when the user asks for delivery or a scheduled run requires it.

Preferred delivery path:

1. Use the available Feishu/Lark CLI or connector already configured in the user's environment.
2. Send as bot if user-message permission is unavailable but bot delivery works.
3. Keep secrets, app IDs, chat IDs, and tokens out of the skill and out of final summaries.
4. If rich Markdown fails validation, retry once as plain text.
5. If Feishu/Lark delivery fails, show the same digest in the current thread and state the failure reason briefly.

## Scheduling Guidance

For recurring daily use, schedule one evening run after most work mail has arrived. A good default is 22:00 in the user's local timezone.

The scheduled prompt should ask for:

- Today's Outlook email only.
- P0/P1 only.
- Feishu/Lark message language: Chinese, English, or Japanese.
- The output format in this skill.
- Feishu/Lark delivery after generation.
- A fallback thread response if delivery fails.
