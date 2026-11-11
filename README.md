# LLMailSorter
An email sorter, powered by LLM

## Démarrage rapide

Prérequis : Docker Desktop (ou Docker Engine + Compose).

```bash
cp .env.example .env          # puis remplir les mots de passe
openssl rand -hex 32          # à coller dans N8N_ENCRYPTION_KEY
docker compose up -d
```

n8n est ensuite accessible sur http://localhost:5678.
Arrêt : `docker compose down` (les données restent dans les volumes Docker).
