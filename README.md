# TP Shorty
[![ci](https://github.com/salmakacem1999/tp-shorty/actions/workflows/ci.yml/badge.svg)](https://github.com/salmakacem1999/tp-shorty/actions/workflows/ci.yml)

> **Projet de TP** du cours « Dév avec Docker » (M2 Expert Développement Full Stack). Ce n'est pas une application maintenue : elle sert de support aux exercices.

Raccourcisseur d'URL : on colle une URL, on obtient un lien court `/s/<code>` qui compte les clics.

Elle est fournie **sans aucun fichier Docker** : c'est à vous de les écrire.

| Dossier | Contenu |
|---|---|
| `api/` | API Node.js 22 + Express, port 3000 |
| `front/` | React + Vite |
| `db/init.sql` | table `links` et deux liens de démo (PostgreSQL 17) |

## API

| Route | Rôle |
|---|---|
| `GET /health` | vérifie la connexion à la base |
| `GET /api/links` | liste des liens |
| `POST /api/links` | crée un lien : `{ "url": "https://..." }` |
| `GET /s/:code` | redirige vers l'URL et compte le clic |

Variables d'environnement : `DATABASE_URL` (ex. `postgres://shorty:shorty@localhost:5432/shorty`) et `PORT` (3000 par défaut).

## Front

En dev, `npm run dev` lance Vite, qui relaie `/api` et `/s/` vers l'API (adresse dans `API_URL`, par défaut `http://localhost:3000`).

En prod, `npm run build` produit des fichiers statiques dans `dist/`. Il faut un serveur web qui les sert et relaie `/api` et `/s/` vers l'API.

## Lancer sans Docker

Il faut Node.js 22 et un PostgreSQL 17 avec une base initialisée par `db/init.sql`.

```bash
cd api && npm ci
cp .env.example .env                        # adapter DATABASE_URL
node --env-file=.env --watch src/server.js

cd front && npm ci && npm run dev           # http://localhost:5173
```

Tests de l'API (il faut la base) : `DATABASE_URL=... npm test`.


## Avec Docker

### Développement
```
copy .env.example .env        # (cp sous Linux/macOS), puis choisir un mot de passe
docker compose up -d --build
```
- Front (Vite, rechargement à chaud) : http://localhost:5173
- PostgreSQL : localhost:5432

Repartir d'une base vide (rejoue `db/init.sql`) : `docker compose down -v`

### Production (sans la surcharge)
```
docker compose -f compose.yaml up -d --build
```
Application sur http://localhost:8080

### Sécurité
## FRONT tp-shorty-front (alpine 3.24.2)
Total: 8 (HIGH: 8, CRITICAL: 0) =>  Total: 3 (HIGH: 3, CRITICAL: 0)  2026-10-21T23:23:41.2514755Z
 
Total: 12 (HIGH: 12, CRITICAL: 0) => API 2026-10-21T22:58:51.441935893Z

## Pipeline et publication des images

À chaque push sur `main` et à chaque tag `v*`, le workflow `ci.yml` :
1. lance les tests de l'API contre une vraie base PostgreSQL ;
2. construit les images `shorty-api` et `shorty-front` ;
3. les analyse avec Trivy (échec s'il reste une vulnérabilité CRITICAL corrigeable) ;
4. les publie sur Docker Hub.

                     Target                     │  Type  │ Vulnerabilities │ Secrets │
├────────────────────────────────────────────────┼────────┼─────────────────┼─────────┤
│ salmakacem199/shorty-front:tp4 (alpine 3.24.2) │ alpine │        1        │    -    │
└────────────────────────────────────────────────┴────────┴─────────────────┴─────────┘
Legend:
- '-': Not scanned
- '0': Clean (no security findings detected)


salmakacem199/shorty-front:tp4 (alpine 3.24.2)
==============================================
Total: 1 (CRITICAL: 1)

┌─────────────┬─────────────────┬──────────┬────────┬───────────────────┬───────────────┬──────────────────────────────────────────────────────┐
│   Library   │  Vulnerability  │ Severity │ Status │ Installed Version │ Fixed Version │                        Title                         │
├─────────────┼─────────────────┼──────────┼────────┼───────────────────┼───────────────┼──────────────────────────────────────────────────────┤
│ brotli-libs │ CVE-2026-111497 │ CRITICAL │ fixed  │ 1.2.0-r1          │ 1.2.1-r0      │ brotli: Out-of-Bounds Write in Google Brotli Decoder │
│             │                 │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2026-111497          │
└─────────────┴─────────────────┴──────────┴────────┴───────────────────┴───────────────┴──────────────────────────────────────────────────────┘

