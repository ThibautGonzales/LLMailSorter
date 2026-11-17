# Privacy policy

LLMailSorter is a personal, self-hosted tool. It has no server, no account
system and no analytics run by the author.

## What data is accessed

- The tool reads emails from the Gmail account of the person who runs it,
  through the Gmail API, when that person triggers a sort manually or when
  the cleanup workflow they published runs on its schedule.
- For each email, the sender, the subject and the first 500 characters of the
  body are sent to the Google Gemini API (Gemini Decision 1) to pick a
  category. Google's own terms apply to this data. The API key should belong
  to a project with billing enabled: on the free tier, Google may use the
  content to improve its products.

## What the tool does with it

- It applies a Gmail label and may archive the email.
- A separate cleanup workflow, which does not use the LLM, moves emails older
  than the retention periods set by the user to the Gmail trash. It runs on a
  schedule chosen by the user, never touches starred emails, and never deletes
  emails permanently: Gmail empties its trash after 30 days.
- It never sends emails, and never shares data with anyone other than the LLM
  provider.

## Where data is stored

All credentials and workflow executions stay in the n8n instance running on the user's
machine. Nothing is sent to the author of this project.

## Revoking access

Access can be revoked at any time from https://myaccount.google.com/permissions.
