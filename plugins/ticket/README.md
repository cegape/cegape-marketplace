# ticket

Plugin Claude Code qui aide à rédiger des **tickets conformes au standard 3 Amigos** (QA + Produit), puis à les créer dans Jira.

Le plugin est **générique** : aucune application, environnement ni clé de projet n'est codé en dur. Les spécificités de votre organisation vivent dans un **fichier de config local** — rien de sensible n'est jamais publié.

## Pourquoi

Un ticket mal rédigé (tag manquant, pas de chemin d'accès, comportement attendu absent) fait perdre du temps à toute l'équipe. Ce plugin encode le template validé : il **mène une conversation** — une question à la fois, au fil de l'eau, jamais un formulaire à remplir — pour collecter **tous** les champs requis selon le type de ticket, valide les règles d'or, puis crée le ticket via le MCP Atlassian — ou produit un markdown prêt à coller si le MCP n'est pas disponible.

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

Pour que le skill connaisse vos applications et clés de projet Jira — et évite de vous les redemander — donnez-lui une config. **Où elle vit dépend de votre plateforme**, car Claude Cowork et Claude Code ne persistent pas les données au même endroit.

**Voie recommandée — la commande dédiée** (entretien guidé, détecte la plateforme et installe la config au bon endroit) :

```
/ticket:config            # emplacement par défaut selon la plateforme
/ticket:config --project  # (Claude Code) force le niveau projet
```

> 📖 Documentation détaillée de la commande : [docs/config-command.md](docs/config-command.md).

### Où va la config selon la plateforme

| Plateforme | Emplacement | Persistance |
|---|---|---|
| **Claude Code** | Fichier `~/.claude/ticket.config.json` (défaut) ou `.claude/ticket.config.json` (`--project`) | Le disque persiste |
| **Claude Cowork** | Un **bloc `<!-- ticket.config -->` collé dans les instructions de votre projet Cowork** | Réinjecté à chaque session |

> ⚠️ **Sous Cowork, pas de fichier.** Le sandbox Cowork est jetable : son `HOME` (`/sessions/<aléatoire>`) change à chaque session, donc aucun fichier écrit sur le disque n'y survit. Seules les **instructions du projet** persistent d'une session à l'autre — c'est là que la config doit vivre. `/ticket:config` détecte Cowork et vous fournit le bloc à coller.

Le skill lit la config **depuis le contexte** (instructions du projet, sous Cowork) ou **depuis le fichier** (projet puis utilisateur, sous Claude Code) — la première source trouvée l'emporte. Les chemins de fichiers sont **POSIX**, jamais convertis en chemin Windows.

Schéma (identique quelle que soit la source) :

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

> ⚠️ Cette config contient des informations internes (noms d'applications, clés de projet, site Jira). Gardez-la **privée** — ne la commitez jamais dans un dépôt public.

Sans config, le plugin reste pleinement fonctionnel : il demande ces informations pendant l'entretien et propose de l'installer.

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

1. **Config chargée** — apps / clés de projet lus depuis les instructions du projet (Cowork) ou le fichier local (Claude Code), s'ils existent.
2. **Type détecté** — le skill déduit le tag depuis votre demande ; en cas de doute, il demande.
3. **Conversation au fil de l'eau** — le skill part de ce que vous avez déjà dit, puis pose **une question à la fois** (ou un petit groupe naturellement lié) pour combler les champs manquants selon le type. Il reformule et avance ; jamais de formulaire vide à compléter d'un coup. Tout se collecte en **texte libre** — y compris le choix de l'appli ou du type, énoncé à l'écrit (les cartes de questions structurées de Cowork buguent, la saisie devient inaccessible). L'app est pré-remplie depuis la config, l'environnement demandé à chaque ticket.
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
