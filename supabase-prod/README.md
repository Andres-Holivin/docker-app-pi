# Supabase — Dev & Prod Environments

Two fully isolated Supabase environments. They share **no** containers, networks, volumes, or secrets.

---

## Directory layout

```
docker-app-pi/
├── supabase/          ← DEV environment  (project: supabase-dev)
└── supabase-prod/     ← PROD environment (project: supabase-prod)
```

---

## DEV

**Ports**
| Service       | Port  |
|---------------|-------|
| Kong HTTP     | 8000  |
| Kong HTTPS    | 8443  |
| Postgres      | 5432  |
| Pooler (tx)   | 6543  |
| Pooler (sess) | 54322 |

**Commands**
```bash
cd ~/docker-app-pi/supabase

# Start
docker compose up -d

# Stop
docker compose down

# Destroy all data
docker compose down -v --remove-orphans
```

---

## PROD

**Ports**
| Service       | Port  |
|---------------|-------|
| Kong HTTP     | 9001  |
| Kong HTTPS    | 9444  |
| Postgres      | 5433  |
| Pooler (tx)   | 7543  |
| Pooler (sess) | 54323 |

*Postgres is exposed on host port **5433** (5432 is used by dev). Internal services still connect on 5432.

**Before first start — generate new secrets:**
```bash
cd ~/docker-app-pi/supabase-prod
sh ./utils/generate-keys.sh
# Copy the output values into .env, replacing all CHANGE_ME_* placeholders
```

**Commands**
```bash
cd ~/docker-app-pi/supabase-prod

# Start
docker compose up -d

# Stop
docker compose down

# Destroy all data
docker compose down -v --remove-orphans
```

---

## Isolation guarantees

| Factor              | Dev                   | Prod                     |
|---------------------|-----------------------|--------------------------|
| Docker project name | `supabase-dev`        | `supabase-prod`          |
| Docker network      | `supabase-dev_default`| `supabase-prod_default`  |
| Container names     | `supabase-*`          | `supabase-prod-*`        |
| Named volumes       | `supabase-dev_db-config` | `supabase-prod-db-config` |
| Data directory      | `supabase/volumes/`   | `supabase-prod/volumes/` |
| Secrets             | `supabase/.env`       | `supabase-prod/.env`     |
