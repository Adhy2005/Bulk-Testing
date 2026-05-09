# GitHub Webhook Integration — ERP Setup Guide

This guide walks through linking a GitHub repository to the Site2Success ERP system using webhooks, so that push and pull request events are automatically received by the backend.

---

## Step 1: Create a Project in the ERP

Before creating a repository, create a corresponding project inside the ERP system.

1. Log in to the ERP at [s2serp.vercel.app](https://s2serp.vercel.app) using a **super_admin** or **manager** account.
2. Navigate to the **Projects** section from the sidebar.
3. Click **Create Project** and fill in the project details.
4. Note the project name — you will use the exact same name when creating the GitHub repository.

---

## Step 2: Create a GitHub Repository with the Same Name

1. Go to [github.com](https://github.com) and sign in.
2. Click the **+** icon in the top-right corner and select **New repository**.
3. Set the **Repository name** to match the project name created in Step 1 exactly (case-sensitive).
4. Choose visibility (Public or Private) as appropriate.
5. Click **Create repository**.

---

## Step 3: Open Webhook Settings in GitHub

1. Inside your newly created repository, click on the **Settings** tab.
2. In the left sidebar, scroll down to the **Code and automation** section.
3. Click on **Webhooks**.
4. Click the **Add webhook** button.

---

## Step 4: Configure the Webhook

You will now fill in the webhook configuration form.

### Payload URL

Enter the ERP backend webhook endpoint:

```
https://ldhanushraj-erp-backend.hf.space/webhooks/github
```

> This endpoint receives GitHub push and pull request events. It is documented in the backend Swagger UI at `/docs` under the **Webhooks - GitHub** section as `POST /webhooks/github`.

### Content Type

Set this to:

```
application/json
```

### Secret

Enter the webhook secret that matches the value configured in your backend environment variables. This is used to verify that incoming payloads are genuinely from GitHub.

Secret Key:
```
902657f4395fbe2064ee2902be6d0bbb6e027e60f9afc5515f6fef0a30c25e2b
```

### SSL Verification

Leave this set to **Enable SSL verification** (the default). Do not disable SSL verification in production.

---

## Step 5: Select Events to Trigger the Webhook

Under **Which events would you like to trigger this webhook?**, select **Let me select individual events**, then check the following two events only:

| Event | Description |
|---|---|
| ✅ **Pushes** | Fires on every `git push` to the repository |
| ✅ **Pull requests** | Fires on PR opened, closed, merged, synchronized, and other PR actions |

Leave all other events unchecked. The ERP backend is built to handle only `push` and `pull_request` event types.

---

## Step 6: Activate and Save

1. Ensure the **Active** checkbox at the bottom is checked — this tells GitHub to start delivering events immediately.
2. Click the green **Add webhook** button to save.

GitHub will send a test `ping` event to your payload URL. If the backend is reachable, the delivery will show a green tick in the webhook's **Recent Deliveries** tab.

---

## Step 7: Verify in the ERP

1. Make a test commit or open a pull request in the repository.
2. Log in to the ERP as a super_admin or manager.
3. Navigate to the **Dashboard** — the activity feed should display the push or PR event.
4. You can also verify via the API directly:

```
GET https://ldhanushraj-erp-backend.hf.space/webhooks/
```

This returns the latest GitHub webhook activity and is restricted to super_admin and manager roles.

---

## API Endpoints Reference

The following webhook endpoints are available in the backend (visible in Swagger at `/docs`):

| Method | Endpoint | Description | Access |
|---|---|---|---|
| POST | `/webhooks/github` | Receive push and pull_request events | Public (GitHub only) |
| GET | `/webhooks/` | Get latest webhook activity | super_admin, manager |
| DELETE | `/webhooks/{event_id}` | Delete a specific webhook event | super_admin, manager |

---

## Troubleshooting

**Webhook delivers but nothing appears in the ERP dashboard**
- Confirm the repository name in GitHub matches the project name in the ERP exactly.
- Check that the backend is running and the `/webhooks/github` route is reachable.

**GitHub shows a failed delivery (red cross)**
- Verify the Payload URL is correct: `https://ldhanushraj-erp-backend.hf.space/webhooks/github`
- Ensure SSL verification is enabled and the backend has a valid certificate.
- Check that the secret value in GitHub matches the `GITHUB_WEBHOOK_SECRET` environment variable on the backend.

**Events not showing for employees**
- Only super_admin and manager roles can view webhook activity via `GET /webhooks/`. This is by design.