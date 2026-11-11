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

Then connect Gmail: see [docs/setup-google-oauth.md](docs/setup-google-oauth.md).

## Workflows

Workflows live in [`workflows/`](workflows/) as n8n JSON exports (without
credentials or pinned data). To use one:

1. In n8n, create a new workflow, open the **...** menu (top right) and choose
   **Import from File**.
2. Open each Gmail node and select your Gmail credential.
3. Save.

| File | Purpose |
|---|---|
| `00-test-gmail.json` | Checks the Gmail connection: a form button that reads the 5 most recent emails, without modifying anything. |
