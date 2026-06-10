# Commande `/ticket:config`

Configure le plugin **ticket** en créant (ou en mettant à jour) le fichier de config local qui déporte les spécificités de votre organisation — applications, environnements, clés de projet Jira — hors du plugin. Rien d'interne n'est jamais publié : le fichier reste sur votre poste.

## Pourquoi

Le skill `redacteur-ticket` lit ce fichier pour **pré-remplir l'entretien** (proposer la liste de vos applications, déduire l'URL de l'environnement choisi et la clé de projet Jira) au lieu de vous redemander ces informations à chaque ticket. Cette commande construit ce fichier pour vous, via un court entretien guidé.

Sans config, le plugin reste fonctionnel : il demande ces informations au moment de créer un ticket.

## Utilisation

```
/ticket:config            # crée .claude/ticket.config.json (projet courant)
/ticket:config --user     # crée ~/.claude/ticket.config.json (tous vos projets)
```

| Argument | Effet |
|---|---|
| *(aucun)* | Écrit `.claude/ticket.config.json` à la racine du projet courant |
| `--user` | Écrit `~/.claude/ticket.config.json` (config valable pour tous vos projets) |

Le skill cherche ces deux emplacements dans cet ordre : **projet** puis **utilisateur**.

## Déroulé

1. **Emplacement cible** — déterminé selon `--user`.
2. **Pas d'écrasement aveugle** — si un fichier existe déjà, la commande le lit, l'affiche, et propose d'**ajouter une application**, de **modifier** une valeur ou de **repartir de zéro**. Aucune écriture sans accord.
3. **Entretien** — pour chaque application : `name`, `jiraProjectKey`, et la liste des **noms** d'`environments` (sans URL). Puis, optionnellement, le site Jira (`jira.site`). Les adresses d'environnement ne sont pas demandées ici (saisies au moment du ticket).
4. **Écriture** — crée le dossier `.claude/` si besoin et écrit un JSON valide, indenté.
5. **Confirmation** — récapitule les applications configurées et rappelle de garder le fichier local.

## Résultat

Exemple de fichier produit :

```json
{
  "applications": [
    {
      "name": "Facturation",
      "jiraProjectKey": "FACT",
      "environments": ["Recette", "Préprod", "Production"]
    }
  ],
  "jira": { "site": "monorg.atlassian.net" }
}
```

| Champ | Obligatoire | Description |
|---|:---:|---|
| `applications[].name` | ✓ | Nom affiché de l'application |
| `applications[].jiraProjectKey` | ✓ | Clé de projet Jira par défaut (ex. `FACT`) |
| `applications[].environments[]` | — | **Noms** d'environnements standard (ex. `Recette`, `Production`) — sans URL |
| `jira.site` | — | Site Atlassian (ex. `monorg.atlassian.net`). Sans lui, le site est résolu via le MCP Atlassian |

> 💡 **Pas d'URL d'environnement en config.** L'adresse change tout le temps (générée à la MR) : elle est demandée au moment de créer le ticket, jamais stockée ici. La config ne sert qu'à pré-remplir l'application, la clé de projet et le **nom** d'environnement.

## ⚠️ Sécurité

Ce fichier contient des informations internes (noms d'applications, clés de projet, site Jira). **Gardez-le local ou dans un dépôt privé — ne le commitez jamais dans un dépôt public.** Le `.gitignore` de ce marketplace ignore déjà `ticket.config.json`.

## Voir aussi

- [README du plugin](../README.md) — installation et utilisation générale
- Modèle de référence : `../skills/redacteur-ticket/assets/ticket.config.example.json`
- Schéma détaillé : `../skills/redacteur-ticket/references/template-reference.md` § `<config_schema>`
