---
description: Configure le plugin ticket (applications, cles de projet Jira) — en fichier sous Claude Code, en bloc a coller dans les instructions du projet sous Cowork
argument-hint: "[--project]"
allowed-tools: Read, Write, Edit, Bash
---

## Tâche

Construire, via un court entretien, la config du plugin `ticket` (applications + clés de projet Jira), puis la **déposer là où le skill la retrouvera**. L'emplacement dépend de la plateforme : un **fichier** sous Claude Code, un **bloc à coller dans les instructions du projet** sous Claude Cowork (dont le sandbox est jetable). Rien d'interne n'est jamais publié.

> **L'environnement n'est PAS en config.** Il dépend du contexte de chaque ticket (une même application peut être en Recette, Préprod ou Production selon le cas signalé) : son nom **et** son adresse sont demandés au moment de rédiger le ticket, jamais stockés ici.

### 1. Détecter la plateforme

Lancer, avec l'outil **Bash** :

```bash
echo "SANDBOX=$SANDBOX_RUNTIME | CC=$CLAUDECODE | HOME=$HOME"
```

- `SANDBOX_RUNTIME=1` — ou `HOME` commençant par `/sessions/` → **Claude Cowork**. Sandbox jetable : `HOME` = `/sessions/<aléatoire>` change à chaque session, donc **aucun fichier ne survit**. La config doit aller dans les **instructions du projet Cowork** → suivre l'étape 4-B.
- Sinon (`CLAUDECODE=1`, `HOME` réel) → **Claude Code**. Le disque persiste : config en **fichier** → suivre l'étape 4-A.

### 2. Retrouver une config existante (ne pas écraser aveuglément)

- **Cowork** : chercher un bloc `<!-- ticket.config -->` (ou un objet JSON avec une clé `applications`) déjà présent dans le contexte / les instructions du projet.
- **Claude Code** : Read le fichier cible — `~/.claude/ticket.config.json` par défaut, ou `.claude/ticket.config.json` si `--project` est dans `$ARGUMENTS`.

Si une config valide existe déjà : la présenter, proposer d'**ajouter une application**, de **modifier** une valeur ou de **repartir de zéro** ; ne jamais l'écraser sans accord explicite.

Modèle de référence (schéma + valeurs d'exemple) : `${CLAUDE_PLUGIN_ROOT}/skills/redacteur-ticket/assets/ticket.config.example.json`.

### 3. Mener l'entretien

En **texte libre** (pas de carte AskUserQuestion — elle bugue sous Cowork), collecter :

1. **Applications** (au moins une). Pour chacune : `name` (nom affiché, ex. « MonApp ») et `jiraProjectKey` (clé de projet Jira, ex. « APP »). Ne pas demander d'environnement (il dépend du contexte du ticket).
2. **Site Jira** (optionnel) : `jira.site` (ex. `monorg.atlassian.net`). Sans lui, omettre la clé `jira` — le skill résoudra le site via `getAccessibleAtlassianResources`.

Ne rien inventer : si une valeur manque, la demander.

### 4. Déposer la config

Construire le JSON :

```json
{
  "applications": [
    { "name": "MonApp", "jiraProjectKey": "APP" }
  ],
  "jira": { "site": "monorg.atlassian.net" }
}
```

**4-A — Claude Code (fichier).** Écrire avec l'outil **Write**, au chemin **littéral** (chemins POSIX tels quels, jamais convertis en chemin Windows `C:\Users\...`) :

| Argument | Emplacement |
|---|---|
| *(défaut)* | `~/.claude/ticket.config.json` (tous les projets) |
| `--project` | `.claude/ticket.config.json` (projet courant) |

JSON indenté 2 espaces, sans le champ `_comment`. Write crée les dossiers parents. Puis **relire** avec Read au même chemin pour confirmer que le fichier a bien atterri ; si la relecture échoue, le signaler et proposer l'autre emplacement.

**4-B — Cowork (instructions du projet).** Ne **pas** compter sur un fichier : il disparaîtrait à la prochaine session. À la place, afficher un bloc **prêt à coller**, exactement sous cette forme :

```
<!-- ticket.config -->
{
  "applications": [
    { "name": "MonApp", "jiraProjectKey": "APP" }
  ],
  "jira": { "site": "monorg.atlassian.net" }
}
```

et demander à l'utilisateur de le **coller dans les instructions de son projet Cowork** (paramètres du projet → Instructions / Knowledge) : il sera réinjecté à chaque session, et le skill le lira depuis le contexte. Tant qu'il n'y est pas, la config ne survivra pas à la session courante.

### 5. Confirmer

- afficher un récapitulatif des applications configurées ;
- **Claude Code** : rappeler le chemin du fichier écrit ; le plugin l'utilisera automatiquement au prochain « crée un ticket… ».
- **Cowork** : rappeler que la config n'est persistante **qu'une fois le bloc collé dans les instructions du projet** ;
- rappeler que ces informations sont internes (noms d'applications, clés de projet, site Jira) : **les garder privées, ne jamais les commiter dans un dépôt public** ;
- l'environnement (nom + adresse) restera demandé à chaque ticket, selon son contexte.
