---
description: Configure le plugin ticket et cree le fichier de config local (applications, cles de projet Jira)
argument-hint: "[--user]"
allowed-tools: Read, Write, Edit
---

## Tâche

Construire, via un court entretien, le fichier de config local du plugin `ticket`, puis l'écrire sur disque. Ce fichier déporte les spécificités de l'organisation (applications, clés de projet Jira) hors du plugin — rien d'interne n'est jamais publié.

> **L'environnement n'est PAS en config.** Il dépend du contexte de chaque ticket (une même application peut être en Recette, Préprod ou Production selon le cas signalé) : son nom **et** son adresse sont demandés au moment de rédiger le ticket, jamais stockés ici.

### 1. Déterminer l'emplacement cible (selon l'OS)

Beaucoup d'utilisateurs lancent ce plugin depuis Claude Cowork sur **Windows** ou **macOS** : il faut résoudre le chemin selon le système. Détecte d'abord l'OS (le harnais fournit la plateforme : `linux`, `darwin`/macOS, ou `win32`/Windows).

| OS | Niveau projet (défaut) | Niveau utilisateur (`--user`) |
|---|---|---|
| macOS / Linux | `.claude/ticket.config.json` | `~/.claude/ticket.config.json` (= `$HOME/.claude/...`) |
| Windows | `.claude\ticket.config.json` | `%USERPROFILE%\.claude\ticket.config.json` (ex. `C:\Users\<Nom>\.claude\ticket.config.json`) |

- Sans argument : emplacement **projet**, relatif à la racine du projet courant.
- Avec `--user` dans `$ARGUMENTS` : emplacement **utilisateur**, dans le dossier personnel **résolu réellement** selon l'OS — ne jamais écrire un littéral `~` ou `%USERPROFILE%`, mais sa valeur effective (`$HOME` sous macOS/Linux, `$env:USERPROFILE` sous Windows). Utiliser le séparateur de l'OS (`/` ou `\`).

Le skill `redacteur-ticket` cherche ces deux emplacements dans le même ordre (projet puis utilisateur), avec la même résolution OS.

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
   - `jiraProjectKey` : clé de projet Jira par défaut (ex. « APP »).
   - **Ne pas demander d'environnement** : il dépend du contexte du ticket (nom + adresse saisis au moment de la rédaction), il n'a pas sa place dans une config générale.
2. **Site Jira** (optionnel) : `jira.site` (ex. `monorg.atlassian.net`). Si l'utilisateur ne le fournit pas, omettre la clé `jira` — le skill résoudra le site via `getAccessibleAtlassianResources`.

Ne rien inventer : si une valeur manque, la demander.

### 4. Composer et écrire le fichier

Construire le JSON selon ce schéma :

```json
{
  "applications": [
    {
      "name": "MonApp",
      "jiraProjectKey": "APP"
    }
  ],
  "jira": { "site": "monorg.atlassian.net" }
}
```

Puis :
- écrire le fichier avec l'outil **Write**, à l'emplacement résolu à l'étape 1 : il crée automatiquement les dossiers parents manquants (`.claude/`), quel que soit l'OS — pas besoin de `mkdir` ni de commande shell spécifique à la plateforme ;
- JSON valide, indenté 2 espaces, sans le champ `_comment` du modèle.

### 5. Confirmer

Après écriture :
- afficher le chemin du fichier créé et un récapitulatif des applications configurées ;
- rappeler que ce fichier contient des informations internes (noms d'applications, clés de projet Jira, site Jira) : **le garder local ou dans un dépôt privé, ne jamais le commiter dans un dépôt public** (le `.gitignore` du marketplace l'ignore déjà) ;
- indiquer que le plugin l'utilisera automatiquement : il suffit désormais de demander « crée un ticket… » pour que l'application et la clé de projet soient pré-remplies (l'environnement — nom et adresse — restera demandé à chaque ticket, selon son contexte).
