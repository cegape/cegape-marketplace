---
name: Redacteur Ticket
description: This skill should be used when the user wants to write or create a Jira / QA ticket following the 3-Amigos standard, or asks to "creer un ticket Jira", "rediger un ticket", "creer un bug", "creer une US", "creer une user story", "remonter une anomalie", "remonter une regression", "creer une amelioration", or mentions a tag "[Bug]", "[Regression]", "[Amelioration]", "[US]". The skill reads an optional local config (applications, environnements, cles de projet Jira), interviews the user to collect every required field of the 3-Amigos QA template (titre tagge, environnement, chemin d'acces, description, etapes, comportement constate/attendu, criteres d'acceptance, captures), then creates the ticket via the Atlassian MCP or outputs a copy-paste markdown. TRIGGER whenever a Jira/QA ticket, bug report, regression, improvement or user story is mentioned, even without an explicit skill request. SKIP when only searching/reading existing Jira issues (use the Atlassian MCP search directly).
---

# Redacteur Ticket

Aide a rediger un ticket conforme au standard **3 Amigos** (QA + Produit), puis a le creer dans Jira. Le but : mener un court entretien pour collecter **tous** les champs requis selon le type de ticket, valider les regles d'or, puis creer le ticket via le MCP Atlassian (ou produire un markdown a copier si le MCP est indisponible).

Le skill est **generique** : aucune application, environnement ni cle de projet n'est code en dur. Les specificites de ton organisation vivent dans un **fichier de config local** (voir ci-dessous).

## Configuration locale (a lire en premier)

Au demarrage, cherche un fichier de config, dans cet ordre :
1. `.claude/ticket.config.json` a la racine du projet courant ;
2. `~/.claude/ticket.config.json` (niveau utilisateur).

S'il existe, charge-le : il fournit la liste des **applications** (nom, `jiraProjectKey`, `environments` avec nom + URL) et optionnellement le **site Jira**. Utilise ces valeurs pour pre-remplir l'entretien (proposer la liste d'apps, deduire l'URL d'environnement et la cle de projet) au lieu de les redemander.

Si **aucun** fichier n'existe : fonctionne quand meme en demandant ces infos a l'utilisateur, et propose-lui de lancer la commande **`/ticket:config`** (entretien guide qui ecrit le fichier) ou de le creer a partir du modele `assets/ticket.config.example.json`. Ne jamais inventer d'URL ni de cle de projet.

Le schema du fichier est documente dans `references/template-reference.md` § `<config_schema>`.

## Les 4 types de tickets

Chaque ticket commence **obligatoirement** par son tag dans le titre.

| Tag | Definition |
|---|---|
| `[Bug]` | Anomalie sur une fonctionnalite existante |
| `[Regression]` | Fonctionnalite qui marchait et ne marche plus |
| `[Amelioration]` | Evolution / optimisation d'une fonctionnalite existante |
| `[US]` | Nouvelle fonctionnalite a developper (User Story) |

## Champs selon le type

Champs **communs** toujours requis : Titre tagge, Environnement, Chemin d'acces, Description, Etapes a reproduire. Les autres dependent du type :

| Type | Comportement constate + attendu | Criteres d'acceptance (Given/When/Then) | Captures d'ecran |
|---|:---:|:---:|:---:|
| `[Bug]` | ✓ obligatoire | — | ✓ obligatoire |
| `[Regression]` | ✓ obligatoire | — | ✓ obligatoire |
| `[Amelioration]` | — | ✓ obligatoire | recommande |
| `[US]` | — | ✓ obligatoire | — |

Le detail complet du template, les exemples et le squelette markdown de sortie vivent dans `references/template-reference.md` — charge-le quand tu rediges.

## Workflow

1. **Charger la config** (voir ci-dessus) pour connaitre apps / environnements / cles de projet.
2. **Determiner le type** — deduire le tag du besoin. En cas de doute, demander (anomalie -> Bug ; ca marchait avant -> Regression ; evolution -> Amelioration ; nouvelle fonctionnalite -> US).
3. **Mener l'entretien** — collecter les champs manquants pour ce type. Si la config liste les apps, proposer le choix et pre-remplir environnement + cle de projet. Ne **jamais** redemander une info deja fournie ; regrouper les questions.
4. **Valider les regles d'or** (voir plus bas).
5. **Composer le ticket** — squelette de `references/template-reference.md`. Titre = `[Tag] <titre court precis> — <App>`.
6. **Restituer** — proposer le ticket formate, demander confirmation, puis creer dans Jira (voir « Creation Jira »). Sinon, fallback markdown.

## Regles d'or (a valider avant restitution)

1. Le **tag** est present en debut de titre.
2. Pour `[Regression]` : mentionner explicitement **la version ou ca marchait**.
3. **Environnement + Chemin d'acces** toujours renseignes.
4. `[Bug]` / `[Regression]` : Comportement constate ET attendu remplis ; captures **obligatoires** (si l'utilisateur ne peut pas les fournir, le signaler comme champ manquant).
5. `[Amelioration]` / `[US]` : au moins un trio **Given / When / Then** complet.

Si un champ obligatoire manque, le reclamer avant de creer le ticket — ne pas inventer de contenu.

## Creation Jira (MCP Atlassian)

Quand l'utilisateur confirme le ticket :

1. **Resoudre le `cloudId`** via `getAccessibleAtlassianResources` (jamais en dur). Si la config fournit un `jira.site`, l'utiliser pour cibler le bon site ; sinon, s'il y a plusieurs sites, demander.
2. **Cle de projet** : prendre `jiraProjectKey` de l'app choisie (config) ; a defaut, la demander.
3. **Resoudre `issueTypeName` par le code, pas en dur** : appeler `getJiraProjectIssueTypesMetadata` pour lister les types reels du projet, puis mapper le tag :
   - `[Bug]` et `[Regression]` -> « Bug » ; le tag `[Regression]` reste dans le `summary`.
   - `[US]` -> « Story ».
   - `[Amelioration]` -> « Improvement » si dispo, sinon « Story » ou « Task ».
   Si ambigu, presenter les types disponibles et demander.
4. **Appeler `createJiraIssue`** : `summary` = titre tagge, `description` = corps du template avec `contentFormat: "markdown"`, plus `cloudId`, `projectKey`, `issueTypeName`. Captures = piece jointe apres creation (le rappeler).
5. **Confirmer** : restituer la cle du ticket cree et son URL.

## Fallback markdown

Si le MCP Atlassian est indisponible, ne pas bloquer : produire le ticket complet en markdown (squelette de `references/template-reference.md`) dans un bloc, pret a coller dans Jira, et le signaler clairement.

## Acces rapide aux references

| Besoin | Reference |
|---|---|
| Template detaille, matrice des champs, mapping tag->type, squelette markdown, exemples, schema du fichier de config | `references/template-reference.md` |
| Modele de fichier de config a copier | `assets/ticket.config.example.json` |
