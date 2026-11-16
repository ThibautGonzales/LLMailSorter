# LLM provider

LLMailSorter classifies emails with **Google Gemini Decision 1**
(`gemini-decision-1`) through the
[Gemini Decisions API](https://aistudio.google.com/docs/decisions-api).
This model does not generate text: for a "choice" question it returns a
probability for each option, which fits email sorting well (one category =
one option). It costs about $0.10 per million input tokens and nothing for
output.

## Set up the API key

1. Go to [Google AI Studio](https://aistudio.google.com/apikey) and click
   **Create API key**. Let AI Studio create a new Google Cloud project for it:
   the key does not need to live in the Gmail OAuth project.
2. Click **Set up billing** for that project. On the free tier, Google may use
   the content you send to improve its products, which is not acceptable for
   personal emails. At ~10 emails a day the cost is a few cents per year.
3. In n8n, go to **Credentials > Add credential > Header Auth**:
   - Credential name: `Gemini Decision API`
   - Name: `x-goog-api-key`
   - Value: your API key
4. Open the `classify-email` workflow, open the **Call Gemini Decision API**
   node and select this credential.

The key stays in the encrypted n8n credentials store. It is never written in
a workflow file or in this repository.

## The `classify-email` contract

Every call to the model goes through the `classify-email` sub-workflow
([`workflows/10-classify-email.json`](../workflows/10-classify-email.json)).
The sorting workflows only know this contract, not the provider.

**Input**

| Field | Description |
|---|---|
| `sender` | Sender address |
| `subject` | Email subject |
| `bodyStart` | Beginning of the body (cut to the first 500 characters) |

Any field can be empty or `null`.

**Output**

| Field | Description |
|---|---|
| `category` | One of `subscriptions`, `invoices`, `jobs`, `admin`, `orders`, `personal`, `security`, `trash`, `other`, or `notSure` |
| `confidence` | Probability of the best category, between 0 and 1 |

`other` means the model is confident that no category fits. `notSure` is not
chosen by the model: the workflow returns it when `confidence` is below
**0.6** (set in the **Extract category** node), meaning the model hesitates
between several categories.

The category ids are stable English ids. Each provider workflow (Gmail, later
Outlook) maps them to its own label or folder names.

## Change the provider

To use another model (for example TypeSafe AI's Jev, an OpenAI model, or a
classic LLM through n8n's Text Classifier node), edit only `classify-email`:

1. Replace the **Call Gemini Decision API** node with the call to the new
   provider.
2. Update **Extract category** so it still outputs `category` and
   `confidence` with the same values.

Nothing else changes: the sorting workflows keep calling `classify-email`
with the same input and reading the same output.
