# Commande `/ticket:config`

Configure le plugin **ticket** — applications, clés de projet Jira — et dépose cette config **là où le skill la retrouvera**. L'emplacement dépend de la plateforme : un **fichier local** sous Claude Code, un **bloc à coller dans les instructions du projet** sous Claude Cowork. Rien d'interne n'est jamais publié.

## Pourquoi

Le skill `redacteur-ticket` lit cette config pour **pré-remplir l'entretien** (proposer la liste de vos applications et déduire la clé de projet Jira) au lieu de vous redemander ces informations à chaque ticket. Cette commande la construit pour vous, via un court entretien guidé, puis l'installe au bon endroit selon votre plateforme.

> **L'environnement n'est pas configuré ici.** Il dépend du contexte de chaque ticket (une même application peut être en Recette, Préprod ou Production selon le cas signalé) : son nom **et** son adresse sont demandés au moment de rédiger le ticket, jamais stockés dans la config.

Sans config, le plugin reste fonctionnel : il demande ces informations au moment de créer un ticket.

## Utilisation

```
/ticket:config            # emplacement par défaut selon la plateforme
/ticket:config --project  # (Claude Code) force le niveau projet : .claude/ticket.config.json
```

La commande **détecte la plateforme** (`SANDBOX_RUNTIME=1` → Cowork ; `CLAUDECODE=1` → Claude Code) et adapte le dépôt de la config.

### Selon la plateforme

| Plateforme | Où va la config | Persistance |
|---|---|---|
| **Claude Code** | Fichier : `~/.claude/ticket.config.json` (défaut) ou `.claude/ticket.config.json` (`--project`) | Le disque persiste |
| **Claude Cowork** | Un **bloc `<!-- ticket.config -->` à coller dans les instructions de votre projet Cowork** | Réinjecté à chaque session |

> ⚠️ **Pourquoi pas un fichier sous Cowork ?** Le sandbox Cowork est **jetable** : son `HOME` (`/sessions/<aléatoire>`) change à chaque session, donc aucun fichier écrit sur le disque n'y survit. Seules les **instructions du projet** sont réinjectées d'une session à l'autre — c'est donc là que la config doit vivre.

Les chemins de fichiers (Claude Code) sont **POSIX**, jamais convertis en chemin Windows (`C:\Users\...` ne serait pas retrouvé : les outils opèrent en POSIX).

## Déroulé

1. **Détection de plateforme** — Cowork (sandbox jetable) ou Claude Code (fichier).
2. **Config existante** — Cowork : chercher un bloc `ticket.config` déjà dans les instructions/contexte ; Claude Code : lire le fichier cible. Si une config existe, la commande l'affiche et propose d'**ajouter une application**, de **modifier** ou de **repartir de zéro**. Aucune écriture sans accord.
3. **Entretien** — pour chaque application : `name` et `jiraProjectKey`. Puis, optionnellement, le site Jira (`jira.site`). L'environnement n'est pas demandé ici.
4. **Dépôt** — Claude Code : écrit le fichier (JSON indenté) puis le relit pour confirmer. Cowork : affiche le bloc à coller dans les instructions du projet.
5. **Confirmation** — récapitule les applications ; sous Cowork, rappelle que la config n'est persistante **qu'une fois le bloc collé dans les instructions du projet**.

## Résultat

Exemple de fichier produit :

```json
{
  "applications": [
    {
      "name": "Facturation",
      "jiraProjectKey": "FACT"
    }
  ],
  "jira": { "site": "monorg.atlassian.net" }
}
```

| Champ | Obligatoire | Description |
|---|:---:|---|
| `applications[].name` | ✓ | Nom affiché de l'application |
| `applications[].jiraProjectKey` | ✓ | Clé de projet Jira par défaut (ex. `FACT`) |
| `jira.site` | — | Site Atlassian (ex. `monorg.atlassian.net`). Sans lui, le site est résolu via le MCP Atlassian |

> 💡 **L'environnement n'est pas en config.** Il dépend du contexte de chaque ticket : son nom **et** son adresse sont demandés au moment de créer le ticket, jamais stockés ici. La config ne sert qu'à pré-remplir l'application et la clé de projet.

## ⚠️ Sécurité

Ce fichier contient des informations internes (noms d'applications, clés de projet, site Jira). **Gardez-le local ou dans un dépôt privé — ne le commitez jamais dans un dépôt public.** Le `.gitignore` de ce marketplace ignore déjà `ticket.config.json`.

## Voir aussi

- [README du plugin](../README.md) — installation et utilisation générale
- Modèle de référence : `../skills/redacteur-ticket/assets/ticket.config.example.json`
- Schéma détaillé : `../skills/redacteur-ticket/references/template-reference.md` § `<config_schema>`
