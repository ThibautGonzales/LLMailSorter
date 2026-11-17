# LLMailSorter
An email sorter, powered by LLM

## Quick start

Requirements: Docker Desktop (or Docker Engine + Compose).

```bash
cp .env.example .env          # then fill in the passwords
openssl rand -hex 32          # paste the output into N8N_ENCRYPTION_KEY
docker compose up -d
```

n8n is then available at http://localhost:5678.
Stop it with `docker compose down` (data is kept in Docker volumes).

Then:

- connect Gmail: see [docs/setup-google-oauth.md](docs/setup-google-oauth.md);
- add the Gemini API key used to classify emails: see
  [docs/llm-provider.md](docs/llm-provider.md).

## Workflows

Workflows live in [`workflows/`](workflows/) as n8n JSON exports (without
credentials or pinned data). To use one:

1. In n8n, create a new workflow, open the **...** menu (top right) and choose
   **Import from File**.
2. Open each Gmail or HTTP Request node and select the matching credential.
3. Save.

| File | Purpose |
|---|---|
| `00-test-gmail.json` | Checks the Gmail connection: a form button that reads the 5 most recent emails, without modifying anything. |
| `01-create-gmail-labels.json` | Run once: creates the `LLMailSorter` Gmail label and its 10 category sub-labels. |
| `10-classify-email.json` | Sub-workflow that picks the category of one email with Gemini Decision 1. Returns `category` and `confidence`. See [docs/llm-provider.md](docs/llm-provider.md). |
| `20-sort-gmail.json` | Sorts the Gmail inbox: every 5 minutes (or from a form button), labels and archives up to 50 unlabelled emails. See [Sorting Gmail](#sorting-gmail). |

After importing `20-sort-gmail.json`, also open its **Classify email** node and
select your `10 - Classify email` workflow: workflow ids differ between n8n
instances.

## Sorting Gmail

1. Import and run `01 - Create Gmail labels` once.
2. Publish `20 - Sort Gmail`. While n8n is running, the **Every 5 minutes**
   Schedule Trigger sorts new emails automatically. Emails received while n8n
   was stopped are caught up by the first run after it starts.
3. To sort a large backlog faster, open the workflow's form and submit it. Each
   run handles up to 50 emails; submit again until the inbox is sorted.

The interval is 5 minutes, not 1, because a run of 50 emails can take more than
a minute: a new run starting before the previous one ends could sort the same
emails twice.

Which emails are sorted: every inbox email without a user label
(`in:inbox has:nouserlabels`), read or unread. An email you labelled yourself is
left alone, and a sorted email is never sorted twice.

For each email, the workflow:

1. calls `10 - Classify email` with the sender, subject and body;
2. maps the returned category to a Gmail label (the `CATEGORY_LABELS` table in
   the **Resolve Gmail label** Code node);
3. adds that label, then archives the email (removes `INBOX`). The read/unread
   status is not changed.

| Category | Gmail label | Archived |
|---|---|---|
| `subscriptions` | `LLMailSorter/Subscriptions` | yes |
| `invoices` | `LLMailSorter/Invoices` | yes |
| `jobs` | `LLMailSorter/Jobs` | yes |
| `admin` | `LLMailSorter/Admin` | yes |
| `orders` | `LLMailSorter/Orders` | yes |
| `personal` | `LLMailSorter/Personal` | yes |
| `security` | `LLMailSorter/Security` | yes |
| `trash` | `LLMailSorter/Junk` | yes |
| `other` | `LLMailSorter/Other` | yes |
| `notSure` | `LLMailSorter/Not sure` | no, kept in the inbox for a manual check |

To change a label name or the archive setting, edit one line of
`CATEGORY_LABELS`. If you rename a label, rename it in Gmail too (or in
`01 - Create Gmail labels`, then run it again).
