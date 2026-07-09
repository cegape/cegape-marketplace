---
name: Redacteur Ticket
description: This skill should be used when the user wants to write or create a Jira / QA ticket following the 3-Amigos standard, or asks to "creer un ticket Jira", "rediger un ticket", "creer un bug", "creer une US", "creer une user story", "remonter une anomalie", "remonter une regression", "creer une amelioration", or mentions a tag "[Bug]", "[Regression]", "[Amelioration]", "[US]". The skill reads an optional local config (applications, cles de projet Jira), then mene une conversation guidee (un champ a la fois, jamais un formulaire) pour collecter tous les champs requis selon le type, valide les regles d'or, et cree le ticket via le MCP Atlassian ou produit un markdown a copier. TRIGGER whenever a Jira/QA ticket, bug report, regression, improvement or user story is mentioned, even without an explicit skill request. SKIP when only searching/reading existing Jira issues (use the Atlassian MCP search directly).
---

# Redacteur Ticket

Aide a rediger un ticket conforme au standard **3 Amigos** (QA + Produit), puis a le creer dans Jira. Le coeur du skill est une **conversation** : tu discutes avec l'utilisateur pour collecter, **au fil de l'eau**, tous les champs requis selon le type de ticket — jamais en lui presentant un formulaire a remplir. Une fois les champs reunis et les regles d'or validees, tu crees le ticket via le MCP Atlassian (ou produis un markdown a copier si le MCP est indisponible).

Le skill est **generique** : aucune application, environnement ni cle de projet n'est code en dur. Les specificites de l'organisation vivent dans un **fichier de config local** (voir ci-dessous).

## 1. Trouver la config

La config (liste des **applications** avec leur `jiraProjectKey`, et optionnellement le **site Jira**) peut vivre a plusieurs endroits selon l'environnement. Cherche-la dans cet ordre et **arrete-toi au premier trouve** :

1. **Dans le contexte de la conversation** — instructions / base de connaissances du **projet Claude Cowork**, bloc de config colle par l'utilisateur, ou `CLAUDE.md` deja charge. Si une liste d'`applications` (avec `jiraProjectKey`) y figure deja, utilise-la telle quelle.
2. **Fichier projet** : lis `.claude/ticket.config.json` (chemin **relatif** au projet courant) avec l'outil **Read**.
3. **Fichier utilisateur** : lis `~/.claude/ticket.config.json` avec **Read**.

**Chemins POSIX tels quels — jamais convertis en chemin Windows** (`C:\Users\...`, `%USERPROFILE%`, backslashes), meme dans Cowork : les outils de fichiers y operent en POSIX, un chemin Windows serait introuvable.

**Persistance selon l'environnement** (a connaitre pour bien orienter l'utilisateur) :
- **Claude Code** : les fichiers `.claude/…` (projet) et `~/.claude/…` (utilisateur) persistent normalement -> sources 2 et 3 fiables.
- **Claude Cowork** : le sandbox est **ephemere, propre a chaque session** ; un fichier ecrit dans `~/.claude/` **ne survit pas** d'une session a l'autre. La config doit donc vivre dans les **instructions du projet** (source 1), ou dans un `.claude/ticket.config.json` **commite dans le depot** si la session travaille dans un checkout (source 2).

La source trouvee pre-remplit la conversation (proposer la liste d'apps, deduire la cle de projet) au lieu de redemander ces infos.

**L'environnement n'est PAS en config** : il depend du contexte de chaque ticket (une meme app peut etre en Recette, Preprod ou Production selon le cas signale). Son **nom ET son adresse** se demandent pendant la conversation ; ne les stocke jamais.

Si **rien** n'est trouve : fonctionne quand meme en demandant ces infos dans la conversation. Propose alors, selon l'environnement : la commande **`/ticket:config`** (ecrit le fichier — ideal en Claude Code) **et/ou** de **coller la liste des applications dans les instructions du projet Cowork** pour qu'elle persiste entre sessions (modele : `assets/ticket.config.example.json`). Ne jamais inventer d'URL ni de cle de projet.

Le schema du fichier est documente dans `references/template-reference.md` § `<config_schema>`.

## 2. Les 4 types et leurs champs

Chaque ticket commence **obligatoirement** par son tag dans le titre. Deduire le type du besoin ; en cas de doute, le demander (anomalie -> Bug ; ca marchait avant -> Regression ; evolution -> Amelioration ; nouvelle fonctionnalite -> US).

| Tag | Definition | Champs specifiques requis |
|---|---|---|
| `[Bug]` | Anomalie sur une fonctionnalite existante | Comportement constate + attendu ; captures **obligatoires** |
| `[Regression]` | Fonctionnalite qui marchait et ne marche plus | Comportement constate + attendu ; captures **obligatoires** ; **version ou ca marchait** |
| `[Amelioration]` | Evolution / optimisation d'une fonctionnalite | Criteres d'acceptance (Given/When/Then) ; captures recommandees |
| `[US]` | Nouvelle fonctionnalite a developper (User Story) | Criteres d'acceptance (Given/When/Then) |

Champs **communs** a tous les types : Titre tagge, Environnement (nom + adresse), Chemin d'acces, Description, Etapes a reproduire.

Le detail du template, les exemples et le squelette markdown de sortie vivent dans `references/template-reference.md` — charge-le quand tu composes le ticket.

## 3. Mener la conversation (au fil de l'eau)

Tu **converses** pour reunir les champs manquants ; tu ne fais **pas** remplir un formulaire. Regles de conduite :

- **Pars de ce qui est deja dit.** Avant la moindre question, relis la demande initiale et extrais-en tous les champs deja presents (type, app, symptome, ecran concerne, version…). Ne redemande jamais une info donnee.
- **Une chose a la fois.** Pose **une seule** question par message — ou un petit groupe naturellement lie (ex. nom + adresse de l'environnement vont ensemble). N'affiche jamais le squelette vide du template en demandant de tout completer d'un coup.
- **Reformule, puis avance.** Apres chaque reponse, restitue en une phrase ce que tu as compris, puis enchaine sur le prochain champ manquant. L'utilisateur doit sentir qu'il **discute**, pas qu'il remplit des cases.
- **Tout en texte libre — evite les cartes (AskUserQuestion).** Dans Claude Cowork, les cartes de questions structurees (« visualize ») buguent : les champs de saisie deviennent inaccessibles. Collecte donc **tout** en conversation, y compris les choix fermes : pour **l'application** ou **le type** ambigu, enonce les options a l'ecrit (ex. « Sur quelle appli — PaieRH, GestionRH ? ») et laisse repondre en texte. N'utilise une carte qu'en tout dernier recours, si un choix a vraiment beaucoup d'options.
- **Ordre naturel des champs** : type -> application (cle de projet deduite) -> environnement (nom + adresse) -> chemin d'acces -> description -> etapes a reproduire -> comportement constate/attendu **ou** criteres d'acceptance (selon le type) -> captures.

**Fin de la conversation** : elle continue jusqu'a ce que **tous** les champs requis pour ce type (colonne « champs specifiques » + champs communs) soient collectes et confirmes par l'utilisateur — pas avant. Si un champ obligatoire manque, le reclamer ; ne jamais inventer de contenu.

## 4. Valider les regles d'or

Avant de composer, verifier :
1. Le **tag** est present en debut de titre.
2. Pour `[Regression]` : **la version ou ca marchait** est mentionnee.
3. **Environnement (nom + adresse) et Chemin d'acces** sont renseignes.
4. `[Bug]` / `[Regression]` : Comportement constate ET attendu remplis ; captures **obligatoires** (si l'utilisateur ne peut pas les fournir, le signaler comme champ manquant).
5. `[Amelioration]` / `[US]` : au moins un trio **Given / When / Then** complet.

## 5. Composer, confirmer, creer

1. **Composer** le ticket depuis le squelette de `references/template-reference.md`. Titre = `[Tag] <titre court precis> — <App>`. Retirer les sections conditionnelles hors sujet pour le type.
2. **Confirmer** : presenter le ticket formate a l'utilisateur et demander son accord avant creation.
3. **Creer dans Jira** (voir « Creation Jira ») ; si le MCP est indisponible, basculer sur le fallback markdown.

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
