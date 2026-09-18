# Gherkify — Backend RAG (gherkin-fastapi-prod)

Ce repository contient le **backend FastAPI + pgvector** de Gherkify : le service qui indexe le code (RAG) et génère des scénarios Gherkin `.feature` à partir du code source d'un microservice Java et/ou de tickets Jira.

Il est piloté par le CLI **`gherkify-cli` (`gfy`)**, un client Java (Spring Shell) qui s'exécute *depuis le projet Java à analyser* et qui parle à ce backend.

---

## 1. Prérequis

- **Docker Desktop** (avec Docker Compose v2 — `docker compose`, pas `docker-compose`)
- **Java 17+** installé localement (pour lancer le CLI `gherkify-cli-0.3.0.jar`)
- Git

---

## 2. Récupérer le projet

```bash
git clone <url-du-repo>
cd gherkin-fastapi-prod
```

## 3. Ajouter le fichier `.env`

Le fichier `.env` (clés API LLM, Gemini/Qwen, config RAG, etc.) **n'est pas versionné** (il est dans `.gitignore`) et contient des secrets. Il vous sera envoyé **séparément par Zoom**.

➡️ Placez ce fichier `.env` à la **racine du projet**, au même niveau que `docker-compose.yml`.

## 4. Lancer le backend avec Docker Compose

Depuis la racine du projet :

```bash
docker compose up -d
```

Cela démarre 3 services :

| Service | Rôle | Port |
|---|---|---|
| `fastapi` | API RAG (indexation + génération) | http://localhost:8000 |
| `postgres` (pgvector) | Stockage des embeddings | localhost:5433 |
| `pgadmin` | Interface d'admin PostgreSQL | http://localhost:5050 |

Vérifiez que tout est bien démarré :

```bash
docker compose ps
docker compose logs -f fastapi
```

L'API FastAPI doit répondre sur `http://localhost:8000/docs`.

Pour arrêter les services : `docker compose down` (ajoutez `-v` uniquement si vous voulez aussi supprimer les données Postgres).

## 5. Installer le CLI dans le microservice à analyser

Le fichier `gherkify-cli-0.3.0.jar` (présent à la racine de ce repo) est le **client** qui scanne le code d'un microservice Java et génère les `.feature`. Il doit être copié dans le projet que vous voulez analyser (pas dans ce repo) :

```bash
cp gherkify-cli-0.3.0.jar /chemin/vers/mon-microservice-java/
cd /chemin/vers/mon-microservice-java/
```

> Le CLI se connecte par défaut au backend sur `http://127.0.0.1:8000`, donc l'étape 4 (`docker compose up`) doit avoir été faite avant de l'utiliser.

## 6. Lancer le CLI et initialiser le workspace

Toujours depuis la racine du microservice Java, lancez le jar **une seule fois** :

```bash
java -jar gherkify-cli-0.3.0.jar
```

Cela démarre l'application et ouvre un **sous-shell interactif (Spring Shell)** : vous obtenez un prompt dans lequel vous tapez ensuite directement les commandes `gfy ...`, sans les préfixer par `java -jar` (pas besoin d'alias, le shell reste ouvert) :

```
shell:> gfy init
shell:> gfy help
```

`gfy init` crée le dossier `.gherkify/` (config, métadonnées, logs) nécessaire à toutes les autres commandes.

## 7. Documentation complète des commandes

Dans le même sous-shell :

```
shell:> gfy help
```

Aperçu des commandes disponibles :

| Commande | Description |
|---|---|
| `gfy init` | Initialise le workspace `.gherkify/` dans le projet Java |
| `gfy status` | État du workspace (init, dernier scan, snapshots, dernière génération) |
| `gfy scan` | Analyse la structure du code (controllers, entities, services…) |
| `gfy scan list` / `gfy scan show` | Liste / affiche le contenu des snapshots enregistrés |
| `gfy diff` | Compare le code actuel avec un snapshot embeddé (ou deux snapshots) |
| `gfy generate` | Génère des fichiers `.feature` (code RAG, Jira, ou les deux) |
| `gfy jira get <ID>` | Récupère un ticket Jira (cache local puis service Jira) |
| `gfy workspace clean` | Supprime les `.feature` générés par Gherkify (ou vide le cache) |
| `gfy workspace reset` | Réinitialise les embeddings pgvector (garde la config du workspace) |
| `gfy workspace info` | Métadonnées complètes du workspace + journal d'activité |

Pour l'aide détaillée d'une commande précise :

```
shell:> gfy help "gfy generate"
```

Exemples de génération :

```
shell:> gfy generate "focus on the crud of facteur composants"
shell:> gfy generate --jira PROJ-42
shell:> gfy generate --jira PROJ-42 --code
```

---

## Résumé du workflow complet

```bash
# 1. Backend
git clone <url-du-repo>
cd gherkin-fastapi-prod
# -> copier le .env reçu par Zoom à la racine
docker compose up -d

# 2. CLI, depuis le projet Java à analyser (une seule fois : lance le sous-shell)
cp /chemin/vers/gherkin-fastapi-prod/gherkify-cli-0.3.0.jar .
java -jar gherkify-cli-0.3.0.jar
```

Puis, dans le sous-shell qui s'ouvre :

```
shell:> gfy init
shell:> gfy help
shell:> gfy generate "focus on ..."
```
