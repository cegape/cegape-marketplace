# ticket

Plugin Claude Code qui aide à rédiger des **tickets conformes au standard 3 Amigos** (QA + Produit), puis à les créer dans Jira.

Le plugin est **générique** : aucune application, environnement ni clé de projet n'est codé en dur. Les spécificités de votre organisation vivent dans un **fichier de config local** — rien de sensible n'est jamais publié.

## Pourquoi

Un ticket mal rédigé (tag manquant, pas de chemin d'accès, comportement attendu absent) fait perdre du temps à toute l'équipe. Ce plugin encode le template validé : il mène un court entretien pour collecter **tous** les champs requis selon le type de ticket, valide les règles d'or, puis crée le ticket via le MCP Atlassian — ou produit un markdown prêt à coller si le MCP n'est pas disponible.

## Les 4 types de tickets

Chaque ticket commence **obligatoirement** par son tag dans le titre.

| Tag | Définition |
|---|---|
| `[Bug]` | Anomalie sur une fonctionnalité existante |
| `[Régression]` | Fonctionnalité qui marchait et ne marche plus |
| `[Amélioration]` | Évolution / optimisation d'une fonctionnalité existante |
| `[US]` | Nouvelle fonctionnalité à développer (User Story) |

Les champs collectés dépendent du type : `Comportement constaté/attendu` pour Bug & Régression, `Critères d'acceptance (Given/When/Then)` pour Amélioration & US.

## Installation

```bash
claude plugin marketplace add https://github.com/cegape/cegape-marketplace.git
claude plugin marketplace update
claude plugin install ticket@cegape-marketplace --scope user
```

## Configuration (recommandé)

Pour que le skill connaisse vos applications et clés de projet Jira — et évite de vous les redemander — créez un fichier de config **local**.

**Voie recommandée — la commande dédiée** (entretien guidé, écrit le fichier pour vous) :

```
/ticket:config            # crée .claude/ticket.config.json (projet)
/ticket:config --user     # crée ~/.claude/ticket.config.json (tous vos projets)
```

> 📖 Documentation détaillée de la commande : [docs/config-command.md](docs/config-command.md).

**Voie manuelle** — copier le modèle et l'éditer :

```bash
mkdir -p .claude
cp <plugin>/skills/redacteur-ticket/assets/ticket.config.example.json \
   .claude/ticket.config.json
# …puis remplacez les valeurs d'exemple par celles de votre organisation.
```

Emplacements reconnus (ordre de recherche), résolus selon l'OS — utile depuis Claude Cowork sur Windows ou macOS :

| OS | Projet | Utilisateur |
|---|---|---|
| macOS / Linux | `.claude/ticket.config.json` | `~/.claude/ticket.config.json` |
| Windows | `.claude\ticket.config.json` | `%USERPROFILE%\.claude\ticket.config.json` |

La commande `/ticket:config` résout automatiquement le bon chemin.

Schéma :

```jsonc
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

> 💡 **L'environnement n'est pas en config** : il dépend du contexte de chaque ticket (Recette, Préprod ou Production selon le cas). Son nom **et** son adresse sont demandés au moment de créer le ticket.

> ⚠️ Ce fichier contient des informations internes (noms d'applications, clés de projet, site Jira). Gardez-le **local** ou dans un dépôt privé — ne le commitez jamais dans un dépôt public.

Sans fichier de config, le plugin reste pleinement fonctionnel : il demande ces informations pendant l'entretien et propose de créer le fichier.

### Pré-requis pour la création directe dans Jira

La création automatique s'appuie sur le **MCP Atlassian** (compte avec le scope `write:jira-work`). Sans lui, le plugin produit un markdown à copier-coller dans Jira. Vérifier la connexion avec `/mcp`.

## Utilisation

Le skill s'active **automatiquement** dès que vous parlez de rédiger un ticket. Exemples :

```
Crée un ticket Jira pour un bug sur le calcul du solde congés
Rédige une US : filtrer les individus par service
Remonte une régression : la génération des bulletins est KO depuis la v2.3
Crée une amélioration : ajouter un export PDF sur le bulletin de paie
```

### Déroulé

1. **Config chargée** — apps / clés de projet lus depuis le fichier local s'il existe.
2. **Type détecté** — le skill déduit le tag ; en cas de doute, il demande.
3. **Entretien ciblé** — il ne demande que les champs **manquants** selon le type, regroupés (jamais une info déjà fournie ; app pré-remplie depuis la config, environnement demandé à chaque ticket).
4. **Validation des règles d'or** — tag en tête de titre, environnement + chemin d'accès, version pour une régression, captures pour Bug/Régression, trio Given/When/Then pour Amélioration/US.
5. **Restitution** — après confirmation, création dans Jira (`cloudId`, clé de projet et type d'issue résolus dynamiquement) ou fallback markdown.

## Règles d'or

1. Le tag est obligatoire en début de titre.
2. `[Régression]` = mentionner la version où ça marchait.
3. Toujours renseigner environnement + chemin d'accès.
4. Bug / Régression : comportement constaté **et** attendu + captures.
5. Amélioration / US : au moins un trio Given / When / Then complet.

## Fichiers

```
ticket/
├── .claude-plugin/plugin.json
├── commands/
│   └── config.md                           # /ticket:config — crée le fichier de config local
├── docs/
│   └── config-command.md                   # doc détaillée de /ticket:config
└── skills/
    └── redacteur-ticket/
        ├── SKILL.md                            # Expertise : config, workflow, règles, mapping Jira
        ├── assets/
        │   └── ticket.config.example.json   # Modèle de config à copier (valeurs fictives)
        └── references/
            └── template-reference.md           # Template, matrice, mapping, squelette, exemples, schéma config
```

## Licence

MIT
