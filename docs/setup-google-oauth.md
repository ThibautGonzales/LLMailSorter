# Connecting Gmail to n8n (Google OAuth)

A self-hosted n8n instance cannot use n8n's managed "Sign in with Google".
You need your own OAuth client in Google Cloud. This takes about 10 minutes
and is done once.

Prerequisite: n8n is running at http://localhost:5678 (see the README).

## 1. Create a Google Cloud project

1. Open https://console.cloud.google.com/projectcreate (sign in with the Gmail
   account you want to sort).
2. Project name: `LLMailSorter`. Leave the location as is, then **Create**.
3. Check that the project selector at the top shows `LLMailSorter`.

## 2. Enable the Gmail API

1. Open https://console.cloud.google.com/apis/library/gmail.googleapis.com
2. Make sure the `LLMailSorter` project is selected, then **Enable**.

## 3. Configure the consent screen

1. Go to **APIs & Services > OAuth consent screen**, then **Get started**.
2. **App information**: app name `LLMailSorter`, support email = your address.
3. **Audience**: choose **External** (a personal Gmail account has no
   Workspace organization, so Internal is not available).
4. **Contact information**: your address.
5. Accept the Google API Services User Data Policy, then **Create**.
6. In the left menu, open **Audience** > **Test users** > **Add users**, and
   add your own Gmail address. Only test users can sign in while the app is in
   Testing mode.

You do not need to add an authorized domain: Google accepts `localhost`
redirect URIs without one.

## 4. Create the OAuth client

1. In n8n, open http://localhost:5678/home/credentials (skip the "Let's build
   your first automation" welcome screen), create a credential and search for
   **Gmail OAuth2 API**.
   Copy the **OAuth Redirect URL** shown at the top. It should be:
   `http://localhost:5678/rest/oauth2-credential/callback`
   Keep this tab open.
2. In Google Cloud: **APIs & Services > Credentials > + Create credentials >
   OAuth client ID**.
3. Application type: **Web application**. Name: `n8n local`.
4. **Authorized redirect URIs > Add URI**: paste the URL from step 1. It must
   match exactly (protocol, port, no trailing slash).
5. **Create**. Copy the **Client ID** and **Client secret** from the dialog.

## 5. Finish the credential in n8n

1. Paste the Client ID and Client secret into the n8n credential.
2. Rename the credential to `Gmail - <your address>` (one credential per
   account, as planned for multi-account support).
3. Click **Sign in with Google**, pick your account.
4. Google shows "Google hasn't verified this app": click **Continue**. This is
   expected, the app is yours and is not published.
5. Grant the requested permissions, then **Save** in n8n. The credential should
   show as connected.

The secret only lives in n8n's encrypted database. Never paste it into a file
of this repository.

## Known limitation: 7-day expiry in Testing mode

While the app is in **Testing** mode, Google revokes the authorization after
7 days. n8n then fails with an authentication error, and you have to click
**Sign in with Google** again in the credential.

To avoid this, publish the app. This is done in **Google Cloud**, not in n8n:

1. Open https://console.cloud.google.com/auth/audience (check that the
   `LLMailSorter` project is selected).
2. Google first requires a complete **Branding** page. In addition to the app
   name and support email, fill in:
   - **Application home page**: `https://github.com/ThibautGonzales/LLMailSorter`
   - **Application privacy policy link**:
     `https://github.com/ThibautGonzales/LLMailSorter/blob/main/PRIVACY.md`
   - **Authorized domains**: `github.com` (required because the links above
     use it)

   Do **not** upload a logo: a logo forces Google's verification. Then
   **Save**.
3. Back in **Audience**, under **Publishing status** (currently **Testing**),
   click **Publish app**. Google shows a "Push to production?" dialog with two
   warnings that do not block personal use:
   - *"Your app will be available to any user with a Google Account"*: in
     practice, signing in also requires your Client ID and secret, which only
     exist in your local n8n. You remain the only user.
   - *"You will need to verify your app"*: Gmail scopes are classified as
     restricted, so Google flags the app as needing verification. As long as
     you do not submit it, the app keeps working as **unverified** (warning
     screen at sign-in, 100-user cap).

   Confirm. The status becomes **In production**.
4. Back in n8n, open the Gmail credential and click **Sign in with Google**
   once more, so that n8n gets a token that no longer expires after 7 days.
   On the "Google hasn't verified this app" screen, click **Advanced**, then
   **Go to LLMailSorter (unsafe)**.

For personal use you do not need Google's verification: the app stays
"unverified" and you keep the warning screen at sign-in.

## Troubleshooting

| Error | Cause |
|---|---|
| `redirect_uri_mismatch` | The redirect URI in Google does not exactly match the one shown in n8n. |
| `invalid_client` | Wrong Client ID or secret (watch for leading or trailing spaces). |
| `access_denied` | Your address is not in the test users list. |
