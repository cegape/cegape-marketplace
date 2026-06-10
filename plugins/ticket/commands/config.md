---
description: Configure le plugin ticket et cree le fichier de config local (applications, environnements, cles de projet Jira)
argument-hint: "[--user]"
allowed-tools: Read, Write, Edit, Bash(mkdir:*)
---

## Tâche

Construire, via un court entretien, le fichier de config local du plugin `ticket`, puis l'écrire sur disque. Ce fichier déporte les spécificités de l'organisation (applications, environnements, clés de projet Jira) hors du plugin — rien d'interne n'est jamais publié.

### 1. Déterminer l'emplacement cible

- Par défaut : `.claude/ticket.config.json` à la racine du projet courant.
- Si `$ARGUMENTS` contient `--user` : `~/.claude/ticket.config.json` (config valable pour tous les projets de l'utilisateur).

Le skill `redacteur-ticket` cherche ces deux emplacements dans cet ordre (projet puis utilisateur).

### 2. Ne pas écraser aveuglément

Lire d'abord le fichier cible s'il existe. S'il contient déjà une config valide :
- la présenter à l'utilisateur ;
- proposer d'**ajouter une application**, de **modifier** une valeur, ou de **repartir de zéro** ;
- ne jamais l'écraser sans accord explicite.

Le modèle de référence (schéma + valeurs d'exemple) est disponible dans
`${CLAUDE_PLUGIN_ROOT}/skills/redacteur-ticket/assets/ticket.config.example.json` — le lire pour rappeler la structure attendue.

### 3. Mener l'entretien

Collecter, en regroupant les questions :

1. **Applications** (au moins une). Pour chacune :
   - `name` : nom affiché de l'application (ex. « MonApp ») ;
   - `jiraProjectKey` : clé de projet Jira par défaut (ex. « APP ») ;
   - `environments` : une ou plusieurs entrées `{ name, url }` (ex. Recette / Préprod / Production avec leurs URLs).
2. **Site Jira** (optionnel) : `jira.site` (ex. `monorg.atlassian.net`). Si l'utilisateur ne le fournit pas, omettre la clé `jira` — le skill résoudra le site via `getAccessibleAtlassianResources`.

Ne rien inventer : si une valeur manque, la demander. Proposer de réutiliser les noms d'environnements standard (Recette, Préprod, Production) mais laisser l'utilisateur libre.

### 4. Composer et écrire le fichier

Construire le JSON selon ce schéma :

```json
{
  "applications": [
    {
      "name": "MonApp",
      "jiraProjectKey": "APP",
      "environments": [
        { "name": "Recette", "url": "https://recette.monapp.example" },
        { "name": "Production", "url": "https://monapp.example" }
      ]
    }
  ],
  "jira": { "site": "monorg.atlassian.net" }
}
```

Puis :
- créer le dossier cible si besoin (`mkdir -p` du dossier `.claude` parent) ;
- écrire le fichier à l'emplacement déterminé à l'étape 1 (JSON valide, indenté 2 espaces, sans le champ `_comment` du modèle).

### 5. Confirmer

Après écriture :
- afficher le chemin du fichier créé et un récapitulatif des applications configurées ;
- rappeler que ce fichier contient des informations internes (URLs d'environnement, site Jira) : **le garder local ou dans un dépôt privé, ne jamais le commiter dans un dépôt public** (le `.gitignore` du marketplace l'ignore déjà) ;
- indiquer que le plugin l'utilisera automatiquement : il suffit désormais de demander « crée un ticket… » pour que les applications et environnements soient pré-remplis.
