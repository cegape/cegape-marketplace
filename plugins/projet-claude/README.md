# projet-claude

Plugin Claude Code qui aide à créer et structurer un **Projet Claude optimal** (sur claude.ai ou Claude Desktop/Cowork) : instructions au **format contrat**, base de connaissances sans **dilution**, un **périmètre** unique.

## Pourquoi

Un Projet Claude est un espace de travail persistant : ses **instructions** et sa **base de connaissances** sont rechargées automatiquement dans chaque conversation. La plupart des projets échouent par sur-remplissage (dilution de l'attention), instructions trop longues ou contradictoires, ou périmètre fourre-tout. Ce plugin encode les bonnes pratiques : il **mène une conversation** — une question à la fois, en texte libre — pour fixer un périmètre unique, choisir la plateforme, rédiger des instructions courtes au format contrat et planifier une base de connaissances propre. Un projet ne se créant pas en ligne de commande, le plugin **produit les artefacts** (bloc d'instructions + plan de fichiers) à coller dans l'interface.

## Trois mots-guides

- **Périmètre** — un projet = une fonction (jamais un fourre-tout « Travail »).
- **Contrat** — instructions courtes en six blocs : Rôle, Objectif, Contraintes, Règle d'incertitude, Format de sortie, À ne pas faire.
- **Dilution** — tout fichier ou toute règle non réutilisé dégrade la qualité sur *tout* le reste.

## Installation

```bash
claude plugin marketplace add https://github.com/cegape/cegape-marketplace.git
claude plugin marketplace update
claude plugin install projet-claude@cegape-marketplace --scope user
```

## Utilisation

Le skill s'active **automatiquement** dès que vous parlez de créer ou d'optimiser un Projet Claude. Trois commandes ciblent chacune un besoin :

| Commande | Rôle |
|---|---|
| `/projet-claude:nouveau` | Entretien de bout en bout → bloc d'instructions + plan de base de connaissances |
| `/projet-claude:instructions [périmètre]` | Rédige/raffine **uniquement** le contrat d'instructions |
| `/projet-claude:auditer` | On colle les instructions d'un projet existant → critique (dilution, contradictions, longueur) + réécriture |

Exemples en langage naturel :

```
Crée un projet Claude pour rédiger ma newsletter hebdo
Aide-moi à structurer un projet Claude Desktop pour trier le support
Audite les instructions de mon projet Claude, elles partent dans tous les sens
```

## À savoir sur les Projets Claude

- Le **nom** et la **description** du projet **ne sont pas lus par Claude** — organisation humaine seulement.
- Le **contexte n'est pas partagé entre chats** d'un même projet, sauf via la base de connaissances (ou la mémoire).
- **claude.ai** (cloud, recommandé par défaut) et **Cowork** (local, dossiers du disque + tâches planifiées) sont deux objets distincts, stockés séparément.
- Le **RAG** s'active automatiquement quand la base grossit — d'où l'importance de noms de fichiers clairs et d'une bonne granularité.

## Fichiers

```
projet-claude/
├── .claude-plugin/plugin.json
├── commands/
│   ├── nouveau.md                  # /projet-claude:nouveau — entretien complet
│   ├── instructions.md             # /projet-claude:instructions — le contrat seul
│   └── auditer.md                  # /projet-claude:auditer — critique + réécriture
└── skills/
    └── architecte-projet/
        ├── SKILL.md                    # Expertise : périmètre, plateforme, contrat, base, réglages, itération
        ├── assets/
        │   └── instructions.template.md   # Squelette du contrat à copier
        └── references/
            ├── contrat.md                 # Format détaillé, exemples remplis, checklist qualité/audit
            ├── base-connaissances.md       # RAG, formats, limites, nommage, dilution
            └── plateformes.md              # claude.ai vs Cowork, mémoire, incognito, connecteurs
```

## Licence

MIT
