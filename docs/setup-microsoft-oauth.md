# Connecting Outlook / Hotmail to n8n (Microsoft OAuth)

A self-hosted n8n instance needs its own app registration in Microsoft Entra ID
(formerly Azure Active Directory) to read a personal Outlook.com / Hotmail
mailbox. This takes about 15 minutes and is done once.

Prerequisite: n8n is running at http://localhost:5678 (see the README).

## 1. Get a Microsoft Entra directory

Since June 2024, Microsoft only allows app registrations inside a directory
(tenant). A personal Microsoft account does not have one by default: without
it, the portal shows *"The ability to create applications outside of a
directory has been deprecated"*.

The simplest way to get one is a free Azure account:

1. Open https://azure.microsoft.com/free and sign up with your Microsoft
   account (it can be the Hotmail address you want to sort).
2. Azure may ask for a phone number and a credit card to check your identity.
   App registrations are part of Entra ID's free tier: nothing is charged for
   what this guide uses.

The sign-up creates a "Default Directory" tenant linked to your account.

## 2. Register the app

1. In n8n, open http://localhost:5678/home/credentials, create a credential
   and search for **Microsoft Outlook OAuth2 API**.
   Copy the **OAuth Redirect URL** shown at the top. It should be:
   `http://localhost:5678/rest/oauth2-credential/callback`
   Keep this tab open.
2. Open https://aka.ms/appregistrations (Entra admin center, **App
   registrations**), then **+ New registration**.
3. Name: `LLMailSorter`.
4. **Supported account types**: choose **Accounts in any organizational
   directory (Any Microsoft Entra ID tenant - Multitenant) and personal
   Microsoft accounts (e.g. Skype, Xbox)**. n8n signs in through Microsoft's
   `common` endpoint, which requires this option.
5. **Redirect URI**: platform **Web**, then paste the URL from step 1. It must
   match exactly (protocol, port, no trailing slash). Microsoft only accepts
   `http://` for `localhost` addresses.
6. **Register**. On the app's **Overview** page, copy the **Application
   (client) ID**.

## 3. Create a client secret

1. In the app, open **Certificates & secrets > Client secrets > + New client
   secret**.
2. Description: `n8n local`. Expiry: the longest offered (24 months).
3. **Add**, then copy the **Value** column right away (it is shown only once).
   Do not copy the **Secret ID**, which is a different value.

Write down the expiry date: when the secret expires, n8n can no longer refresh
its token. Create a new secret, paste it into the credential and sign in again.

## 4. Finish the credential in n8n

1. Paste the Client ID and the secret Value into the n8n credential.
2. Leave **Use Shared Inbox** off (shared inboxes are a Microsoft 365 feature).
3. Turn on **Custom Scopes** and set **Enabled Scopes** to:
   `openid offline_access Mail.ReadWrite`
   That is all LLMailSorter needs: read emails, create folders and move emails.
   n8n's default scopes would also allow sending emails and reading contacts
   and calendars.
4. Rename the credential to `Outlook - <your address>`.
5. Click **Connect my account**, pick your Hotmail / Outlook.com account and
   accept the permissions, then **Save**. The credential should show as
   connected.

The secret only lives in n8n's encrypted database. Never paste it into a file
of this repository.

You do not need to add API permissions in the portal: for delegated access,
Microsoft asks for the scopes above when you sign in.

## Troubleshooting

| Error | Cause |
|---|---|
| *The ability to create applications outside of a directory has been deprecated* | No tenant yet: see step 1. |
| `AADSTS50194` (application is not configured as a multi-tenant application) | Wrong **Supported account types**: edit it in the app's **Authentication** page (step 2.4). |
| `AADSTS50011` (redirect URI mismatch) | The redirect URI in the portal does not exactly match the one shown in n8n. |
| `AADSTS7000215` (invalid client secret) | You pasted the Secret ID instead of the Value, or the secret expired. |
| *Need admin approval* | You signed in with a work or school account: pick your personal account instead. |
