# TOMAR web

Application de préparation des missions d'audit du pôle Audit de RSM France, sur la plateforme
myRSM IA : transposition du classeur TOMAR_IMMO_ORB V5.1 (dossier de conception v0.1 du 29 octobre 2026).

> L'application aide à documenter ; elle ne remplace ni le dossier CaseWare, qui reste le dossier
> de référence (décision D6), ni le jugement de l'auditeur. Toute valeur proposée porte le marqueur
> « pré-rempli – à valider » et sa provenance.

**Statut : lot 1 en développement, sur données entièrement fictives.** Aucune donnée client ne doit
être utilisée hors de l'environnement de production validé (DSI, RSSI, DPO).

## Contenu du dépôt

| Dossier | Contenu |
|---|---|
| `spec/` | Fichiers de spécification v0.1 (api.json, modèle de données, questionnaire, matrice, tables de rubriques, registre des écarts, table Mapping), avec leurs empreintes SHA-256 |
| `backend/` | API FastAPI (Python 3.12+), modèles SQLAlchemy générés depuis `spec/modele_donnees.json`, migrations Alembic, tests pytest |
| `frontend/` | Interface React + TypeScript (Vite), identité visuelle RSM France, accessibilité RGAA AA |
| `docs/` | Conventions de développement, architecture (dont les décisions d'intégration), points à valider du lot 1 (arbitrages, confirmations DSI, RSSI et DPO, évolutions v0.2) |

## Démarrage local (développement)

Prérequis : Python 3.12+, Node.js 20+, PostgreSQL 16 (connexion locale par socket, sans mot de passe).

```bash
cd backend && pip install -e ".[dev]"
```

```bash
createdb tomar_dev && cd backend && alembic upgrade head && python -m app.referentiels.chargement
```

```bash
cd backend && TOMAR_AUTH_CLE_DEV=$(openssl rand -hex 32) uvicorn app.main:app --no-access-log --port 8000
```

```bash
cd frontend && npm install && npm run dev
```

- En développement, l'assertion signée du portail est remplacée par un jeton HS256 signé avec une clé
  éphémère (`TOMAR_AUTH_CLE_DEV`) ; cette clé est **refusée** en recette et en production.
- `--no-access-log` est obligatoire : le journal d'accès d'uvicorn consignerait les chemins et chaînes
  de requête, ce qu'interdit la règle de journalisation sans donnée client.

## Tests

```bash
cd backend && pytest
```

```bash
cd frontend && npm test
```

Les tests backend créent une base PostgreSQL éphémère par exécution et la suppriment à la fin. Un test
(SAR-T60, libellés du SARA) est sauté sans l'extraction masquée du gabarit (variable
`TOMAR_EXTRACTION_MASQUEE`), conservée hors du dépôt.

## Configuration (variables d'environnement `TOMAR_…`)

Principales variables ci-dessous ; liste complète et règles (aucune valeur par défaut en recette et en
production) : `docs/ARCHITECTURE.md`, § « Paramètres d'exploitation ».

| Variable | Rôle |
|---|---|
| `TOMAR_ENVIRONNEMENT` | `dev`, `test`, `recette` ou `production` |
| `TOMAR_URL_BASE` | Connexion PostgreSQL (injectée depuis le Key Vault en production) [À CONFIRMER DSI] |
| `TOMAR_AUTH_EMETTEUR`, `TOMAR_AUTH_AUDIENCE`, `TOMAR_AUTH_URL_JWKS` | Vérification de l'assertion signée du portail [À CONFIRMER DSI] |
| `TOMAR_IMPORT_TAILLE_MAX_OCTETS`, `TOMAR_IMPORT_LIGNES_MAX` | Limites d'import [À CONFIRMER DSI] |
| `TOMAR_CLE_PSEUDONYMISATION` | Pseudonymisation de l'utilisateur dans les journaux techniques [À VALIDER – DPO] |
| `TOMAR_PURGE_DELAI_JOURS_DEFAUT` | Délai de purge après le dernier export (règle H.08) [À VALIDER – DPO] |
