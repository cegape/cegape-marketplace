---
description: Configure le plugin ticket et cree le fichier de config local (applications, cles de projet Jira)
argument-hint: "[--project]"
allowed-tools: Read, Write, Edit
---

## Tâche

Construire, via un court entretien, le fichier de config local du plugin `ticket`, puis l'écrire sur disque. Ce fichier déporte les spécificités de l'organisation (applications, clés de projet Jira) hors du plugin — rien d'interne n'est jamais publié.

> **L'environnement n'est PAS en config.** Il dépend du contexte de chaque ticket (une même application peut être en Recette, Préprod ou Production selon le cas signalé) : son nom **et** son adresse sont demandés au moment de rédiger le ticket, jamais stockés ici.

### 1. Déterminer l'emplacement cible

| Argument | Emplacement écrit |
|---|---|
| *(aucun — défaut)* | `~/.claude/ticket.config.json` — dossier `.claude` **utilisateur**, valable pour tous vos projets (emplacement recommandé) |
| `--project` | `.claude/ticket.config.json` — relatif à la racine du projet courant (config spécifique à ce projet) |

Choisir selon `$ARGUMENTS` : `--project` présent → emplacement projet ; sinon → emplacement utilisateur. Passer ce chemin **tel quel** à l'outil Write (`~` est résolu par l'outil). Le skill `redacteur-ticket` lit ces deux emplacements, projet puis utilisateur (le projet, plus spécifique, l'emporte s'il existe).

### 2. Ne pas écraser aveuglément

Lire d'abord le fichier cible s'il existe. S'il contient déjà une config valide :
- la présenter à l'utilisateur ;
- proposer d'**ajouter une application**, de **modifier** une valeur, ou de **repartir de zéro** ;
- ne jamais l'écraser sans accord explicite.

Le modèle de référence (schéma + valeurs d'exemple) est disponible dans
`${CLAUDE_PLUGIN_ROOT}/skills/redacteur-ticket/assets/ticket.config.example.json` — le lire pour rappeler la structure attendue.

### 3. Mener l'entretien

Collecter :

1. **Applications** (au moins une). Pour chacune : `name` (nom affiché, ex. « MonApp ») et `jiraProjectKey` (clé de projet Jira, ex. « APP »). Ne pas demander d'environnement (il dépend du contexte du ticket).
2. **Site Jira** (optionnel) : `jira.site` (ex. `monorg.atlassian.net`). Sans lui, omettre la clé `jira` — le skill résoudra le site via `getAccessibleAtlassianResources`.

Ne rien inventer : si une valeur manque, la demander.

### 4. Composer et écrire le fichier

Construire le JSON selon ce schéma :

```json
{
  "applications": [
    { "name": "MonApp", "jiraProjectKey": "APP" }
  ],
  "jira": { "site": "monorg.atlassian.net" }
}
```

Puis écrire le fichier avec l'outil **Write**, au chemin de l'étape 1 : JSON indenté 2 espaces, sans le champ `_comment` du modèle. Write crée automatiquement les dossiers parents manquants (`.claude/`).

### 5. Vérifier puis confirmer

- **Relire le fichier** avec l'outil **Read**, au même chemin : confirmer qu'il a bien atterri là où le skill le cherchera. Si la relecture échoue, ne pas prétendre que la config est en place — signaler l'échec et proposer l'autre emplacement.
- afficher le chemin du fichier créé et un récapitulatif des applications configurées ;
- rappeler que ce fichier contient des informations internes (noms d'applications, clés de projet Jira, site Jira) : **le garder local ou dans un dépôt privé, ne jamais le commiter dans un dépôt public** (le `.gitignore` du marketplace l'ignore déjà) ;
- indiquer que le plugin l'utilisera automatiquement : il suffit désormais de demander « crée un ticket… » pour que l'application et la clé de projet soient pré-remplies (l'environnement — nom et adresse — restera demandé à chaque ticket, selon son contexte).
