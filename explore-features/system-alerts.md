---
description: >-
  System alerts are automatic, pre-configured notifications that fire when your
  organization's credit balance or project/key budgets cross critical thresholds
icon: siren-on
---

# System Alerts

### Overview

Unlike custom alerts, system alerts require no setup — they are active by default for all organizations and fire when your organization's credit balance or project/key budgets cross critical thresholds. System alert notifications are sent to all connected channels org-wide.&#x20;

To manage which channels receive them, see Notification Channels.

***

### Alert Types

#### Organization Credit Balance

Fires when your organization's prepaid credit balance drops below a fixed dollar amount.

**Recipients:** All organization admins\
**Reset condition:** Balance exceeds threshold + $2.00 buffer\
**Check frequency:** Every 5 minutes

***

#### Project Budget Threshold

Fires when a project's remaining budget drops below a percentage of its configured budget cap.

| Threshold       | Severity |
| --------------- | -------- |
| < 20% remaining | Warning  |
| < 10% remaining | Warning  |
| < 5% remaining  | Critical |

**Recipients:** All project admins\
**Reset condition:** Remaining budget exceeds threshold + 2% absolute buffer\
**Check frequency:** Every 5 minutes

<figure><img src="../.gitbook/assets/Configure System Alert Thresholds.png" alt=""><figcaption><p>Configure Thresholds</p></figcaption></figure>

***

#### API Key Budget Threshold

Fires when an API key's remaining budget drops below a percentage of its configured budget cap.

| Threshold       | Severity |
| --------------- | -------- |
| < 20% remaining | Warning  |
| < 10% remaining | Warning  |
| < 5% remaining  | Critical |

**Recipients:** Key creator and all project admins\
**Reset condition:** Remaining budget exceeds threshold + 2% absolute buffer\
**Check frequency:** Every 5 minutes

***

### Notification Behavior

#### How notifications are sent

* One notification is sent per threshold crossing — notifications do not repeat until the alert resets and the threshold is crossed again.
* If multiple thresholds are crossed in a single balance drop, separate notifications are sent for each threshold crossed.
* A 15-minute grace period applies before an alert resets, preventing rapid-fire notifications when a balance fluctuates around a threshold.

### Configuring System Alerts

System alert thresholds can be toggled on or off per threshold level. You cannot add custom threshold values in addition to the defaults — the predefined thresholds above are fixed.

To configure system alerts:

1. Go to **Alerts** in the left navigation.
2. In the **System Alert Configuration** banner, click **Configure**.
3. Toggle each alert type on or off using the toggle switch.
4. Select which threshold pills are active for each alert type.
5. Click **Save configuration**.

> Turning off an alert type stops all notifications for that type until it is re-enabled. Individual threshold pills can be deselected to suppress notifications at that specific level while keeping others active.

***

### Notification Channels

System alerts are delivered to all active notification channels configured at the org level. Channels are shared across all system alert types — you cannot route different alert types to different channels.

To manage channels:

1. Go to **Alerts** in the left navigation.
2. In the **System Alert Configuration** banner, click **Manage Channels**.

<figure><img src="../.gitbook/assets/Manage System Alert Channels Including Webhook 2.png" alt=""><figcaption><p>Manage Channels</p></figcaption></figure>

#### Available channels

**Email**

Always active. Sent to all organization admins automatically. Cannot be disabled.

**Slack**

Connect your Slack workspace via OAuth. Alerts are routed to the channel you specify during connection.

To connect: **Manage Channels → Slack → Connect**

**Generic Webhook**

POST JSON payloads to any HTTPS endpoint. Supports an optional signing secret for HMAC-SHA256 payload verification.

To configure: **Manage Channels → Generic Webhook → Configure**

Payload format

```json
{
  "alert_type": "balance",
  "org_id": "...",
  "org_name": "...",
  "current_balance": 8.50,
  "threshold_value": 10.0,
  "event_type": "triggered",
  "alert_config_id": "...",
  "time": "2026-07-09 12:00:00",
  "text": "Balance $8.50 crossed below threshold $10.00"
}
```

Use **Test Payload** to send a sample delivery to your endpoint before saving.

**Flock**

Connect your Flock workspace to route alerts to a Flock channel.

To connect: **Manage Channels → Flock → Configure**

***

### Viewing System Alert History

System alert fire events appear in the **Recent Alerts** tab on the Alerts page, alongside user-configured alert events. System alert rows are marked with a **System** badge.

<figure><img src="../.gitbook/assets/Recent Alerts showing System Alerts.png" alt=""><figcaption><p>Recent System Alerts</p></figcaption></figure>

Each row shows:

* **Alert Name** — e.g. "Balance $48.38 crossed below threshold…"
* **Reason Summary** — the balance or budget value that triggered the alert, and the direction of the crossing
* **Status** — Warning or Critical
* **Triggered at** — timestamp of the alert fire event
* **Action** — View button to open the full alert detail

Clicking **View** opens the alert detail, showing:

* Current snapshot (org/project/key, current value, threshold, total triggers)
* Channels the notification was delivered to at the time of firing
* Full trigger history with per-event severity and channel delivery context

***

### FAQ

**Q: Can I add my own threshold values for system alerts?**

A: Not in the current version. System alert thresholds are predefined ($10 / $2.50 / $1 for org balance; 20% / 10% / 5% for project and key budgets). You can toggle individual thresholds on or off, but cannot add custom values.

**Q: Who receives system alerts?**

A: Org credit balance alerts go to all organization admins. Project budget alerts go to all project admins. API key budget alerts go to the key creator and all project admins. In all cases, notifications also go to any connected org-level channels (Slack, Webhook, Flock).

**Q: Can I route different system alert types to different Slack channels?**

A: Not currently. All system alert types share the same set of org-level notification channels. Per-type channel routing is not available in this version.

**Q: Why did I receive the same alert twice?**

A: This can happen if the balance dropped below a threshold, recovered above it (plus the $2 buffer), and then dropped again. Once an alert resets, it can re-trigger if the balance crosses the threshold again.

**Q: What happens if my Slack connection drops at the time an alert fires?**

A: FastRouter falls back to email delivery and records the channel delivery context in the trigger history (e.g., "Email only — Slack not connected at time").

**Q: Do system alerts count against any quota or limit?**

A: No. System alerts are infrastructure-level notifications and do not consume any credits or count against usage limits.

**Q: Can I disable system alerts entirely?**

A: You can toggle each alert type off individually in System Alert Configuration. There is no single master off-switch, as system alerts exist to protect your service continuity.

