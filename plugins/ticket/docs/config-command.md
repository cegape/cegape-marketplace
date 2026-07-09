# Commande `/ticket:config`

Configure le plugin **ticket** en créant (ou en mettant à jour) le fichier de config local qui déporte les spécificités de votre organisation — applications, clés de projet Jira — hors du plugin. Rien d'interne n'est jamais publié : le fichier reste sur votre poste.

## Pourquoi

Le skill `redacteur-ticket` lit ce fichier pour **pré-remplir l'entretien** (proposer la liste de vos applications et déduire la clé de projet Jira) au lieu de vous redemander ces informations à chaque ticket. Cette commande construit ce fichier pour vous, via un court entretien guidé.

> **L'environnement n'est pas configuré ici.** Il dépend du contexte de chaque ticket (une même application peut être en Recette, Préprod ou Production selon le cas signalé) : son nom **et** son adresse sont demandés au moment de rédiger le ticket, jamais stockés dans la config.

Sans config, le plugin reste fonctionnel : il demande ces informations au moment de créer un ticket.

## Utilisation

```
/ticket:config            # crée .claude/ticket.config.json (projet courant)
/ticket:config --user     # crée ~/.claude/ticket.config.json (tous vos projets)
```

| Argument | Effet |
|---|---|
| *(aucun)* | Écrit le fichier de config **projet** à la racine du projet courant |
| `--user` | Écrit le fichier de config **utilisateur** (valable pour tous vos projets) |

### Chemins

Le fichier est écrit à un chemin **POSIX**, identique quel que soit votre système — y compris depuis Claude Cowork sous Windows, car les outils de fichiers de Claude Code opèrent dans un environnement POSIX (un chemin Windows `C:\Users\...` n'y serait pas retrouvé) :

| Argument | Emplacement |
|---|---|
| *(défaut)* | `.claude/ticket.config.json` (relatif au projet courant) |
| `--user` | `~/.claude/ticket.config.json` |

### Où le skill cherche la config (Claude Code **et** Claude Cowork)

Le skill lit la config depuis **trois sources**, dans l'ordre : (1) le **contexte de la conversation** (instructions du projet Cowork, bloc collé, `CLAUDE.md`), (2) le fichier **projet**, (3) le fichier **utilisateur**.

> ⚠️ **Persistance en Claude Cowork.** Le sandbox Cowork est **éphémère par session** : un fichier écrit par `/ticket:config` (même `--user`) **ne survit pas** à la session. Pour une config durable en Cowork, collez-la dans les **instructions de votre projet** (source 1) ou committez `.claude/ticket.config.json` dans un dépôt **privé** (source 2). En **Claude Code**, les fichiers (sources 2 et 3) persistent — rien de plus à faire. Après écriture, la commande affiche donc aussi la config dans un bloc à copier vers les instructions du projet, pour les utilisateurs Cowork.

## Déroulé

1. **Emplacement cible** — chemin POSIX déterminé selon `--user` (voir tableau ci-dessus), écrit tel quel avec l'outil Write, puis relu pour vérifier qu'il a bien atterri.
2. **Pas d'écrasement aveugle** — si un fichier existe déjà, la commande le lit, l'affiche, et propose d'**ajouter une application**, de **modifier** une valeur ou de **repartir de zéro**. Aucune écriture sans accord.
3. **Entretien** — pour chaque application : `name` et `jiraProjectKey`. Puis, optionnellement, le site Jira (`jira.site`). L'environnement n'est pas demandé ici (nom + adresse saisis au moment du ticket, selon son contexte).
4. **Écriture** — écrit un JSON valide et indenté (l'outil d'écriture crée le dossier `.claude/` parent si besoin, sur tout OS).
5. **Confirmation** — récapitule les applications configurées et rappelle de garder le fichier local.

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
