# Privacy policy

LLMailSorter is a personal, self-hosted tool. It has no server, no account
system and no analytics run by the author.

## What data is accessed

- The tool reads emails from the Gmail account of the person who runs it,
  through the Gmail API, only when that person triggers a sort manually.
- For each email, the sender, the subject and the beginning of the body are
  sent to the LLM provider chosen by the user (for example Anthropic, OpenAI,
  Google or Mistral) to pick a category. That provider's own privacy policy
  applies to this data.

## What the tool does with it

- It applies a Gmail label and may archive the email.
- It stores a history (email id, category, sorting date) in the local n8n
  database, on the user's own machine.
- It never deletes emails on its own, never sends emails, and never shares
  data with anyone other than the chosen LLM provider.

## Where data is stored

All credentials and history stay in the n8n instance running on the user's
machine. Nothing is sent to the author of this project.

## Revoking access

Access can be revoked at any time from https://myaccount.google.com/permissions.
