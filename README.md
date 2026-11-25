# Snowflake CI/CD Project

Automated database deployment to Snowflake using **GitHub Actions** and **schemachange**.
## Overview

A real-time Snowflake CI/CD project that deploys database changes automatically. Write a SQL migration file, push it to GitHub, and the pipeline deploys it to Snowflake and records what was run.

| Tool | Purpose |
|------|---------|
| Git | Version control |
| GitHub | Code repository |
| GitHub Actions | Pipeline automation |
| schemachange | SQL migrations for Snowflake |
| Snowflake | Database deployment |

## How It Works

```
Write SQL  →  Git commit  →  Push to GitHub  →  GitHub Actions  →  schemachange  →  Snowflake
```

1. **Write SQL migration files** in `snowflake_objects/`
2. **Track changes with Git** (`git add`, `git commit`, `git push`)
3. **Push to GitHub** (`main` branch)
4. **GitHub Actions** runs the pipeline automatically
5. **schemachange** checks the migration history, runs only new scripts, and records each execution

## Environments

Each stage runs only after the previous one succeeds.

| Stage | Database | Job |
|-------|----------|-----|
| DEV | `CICD_DEMO_DB` | `deploy-dev` |
| PRE-PROD | `CICD_PRE_PROD_DB` | `deploy-preprod` |
| PROD | `CICD_PROD_DB` | `deploy-prod` |

## Project Structure

```
snowflake-cicd-github-actions/
├── .github/
│   └── workflows/
│       └── snowflake-deploy.yml
├── snowflake_objects/
│   ├── V001__create_customer_table.sql
│   └── V002__add_phone_number_column.sql
├── .gitignore
├── requirements.txt
└── README.md
```

## GitHub Secrets Required

Add these under **Settings → Secrets and variables → Actions**:

| Secret | Description |
|--------|-------------|
| `SNOWFLAKE_ACCOUNT` | Account identifier (`ORGNAME-ACCOUNTNAME`) |
| `SNOWFLAKE_USER` | Service user for deployments |
| `SNOWFLAKE_PRIVATE_KEY` | Private key for key-pair authentication |
| `SNOWFLAKE_ROLE` | Deployment role |
| `SNOWFLAKE_WAREHOUSE` | Warehouse used for deployments |
| `SNOWFLAKE_DATABASE` | DEV database |
| `SNOWFLAKE_SCHEMA` | Schema name |

> Never commit private keys. Keep them in GitHub secrets only.

## Migration Naming

Scripts follow `V<version>__<description>.sql` (two underscores) and run in version order, once each.

```
V001__create_customer_table.sql
V002__add_phone_number_column.sql
V003__modify_customer_name.sql
```

Never edit a script that has already been deployed. Add a new version instead.

## Deploy a Change

```bash
git add snowflake_objects/V003__modify_customer_name.sql
git commit -m "Modify customer name column"
git push
```

Then open the **Actions** tab to watch the deployment move through DEV → PRE-PROD → PROD.

## Verify Deployment

```sql
SELECT VERSION, SCRIPT, STATUS, INSTALLED_BY
FROM CICD_DEMO_DB.CICD_SCHEMA.CHANGE_HISTORY
ORDER BY VERSION;
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `404 Not Found` on login | Set `SNOWFLAKE_ACCOUNT` to the `ORGNAME-ACCOUNTNAME` format |
| Cannot query `CHANGE_HISTORY` | Grant `SELECT` on the table to your role |
| Checksum error | A deployed script was edited. Revert it and add a new version |

## Key Benefits

- Automated and faster deployments
- Version-controlled and auditable changes
- Consistent across environments
- Reduced manual effort


