# Argo CD Multi-Channel Notifications Setup Guide (Telegram & Slack)

This comprehensive technical guide details the end-to-end setup of multi-channel sync notifications for Argo CD using Telegram and Slack. By following these steps, you will configure real-time alert mechanisms for both success and failure states across your applications.

---

## 1. Architecture Overview

Argo CD Notifications operates through a central controller component within the `argocd` namespace. It evaluates application state changes against user-defined triggers, populates notifications using templates, and dispatches requests to configured communication channels.

| Component | Kubernetes Resource | Description |
| :--- | :--- | :--- |
| **Credentials Store** | `Secret/argocd-notifications-secret` | Stores sensitive API keys and tokens for Telegram and Slack integration. |
| **Engine Config** | `ConfigMap/argocd-notifications-cm` | Defines integrations, triggers (evaluation logic), and rendering templates. |
| **Subscriptions** | `Application Annotations` | Maps specific Argo CD applications to recipients (Telegram Chat ID, Slack Channel ID). |

---

## 2. Prerequisites & Credential Gathering

### 2.1 Telegram Integration Setup
- **Create Bot:** Open Telegram, message `@BotFather`, run `/newbot`, and copy the API Token (e.g., `123456789:ABCdefGhIJKlmNo...`).
- **Obtain Chat ID:** Add your bot to the target group or channel. Send a test message and get the group numeric ID using `@RawDataBot` or `@userinfobot`. *Note: Group IDs begin with `-100`.*

### 2.2 Slack Integration Setup
- **Create Slack App:** Navigate to `https://api.slack.com/apps` and select **Create New App** > **From scratch**.
- **Permissions:** Under **OAuth & Permissions**, add the `chat:write` scope under **Bot Token Scopes**.
- **Install & Authorize:** Install the app to your workspace and copy the **Bot User OAuth Token** (starts with `xoxb-`).
- **Invite Bot:** In your Slack channel, run `/invite @YourBotName`. Copy the Channel ID (e.g., `C0123456789`)[cite: 1].

---

## 3. Step-by-Step Implementation

### Step 1: Create the Kubernetes Secret

Execute the following command to store both authentication tokens securely inside the cluster[cite: 1]:

```bash
kubectl create secret generic argocd-notifications-secret \
  -n argocd \
  --from-literal=telegram-token="<YOUR_TELEGRAM_BOT_TOKEN>" \
  --from-literal=slack-token="<YOUR_SLACK_BOT_TOKEN>" \
  --dry-run=client -o yaml | kubectl apply -f -
