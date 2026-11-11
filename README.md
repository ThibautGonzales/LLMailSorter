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
