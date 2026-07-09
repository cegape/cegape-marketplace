# cegape-marketplace

Marketplace **public** de plugins pour [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

Aucun token ni accès privé requis : n'importe qui peut ajouter ce marketplace et installer ses plugins.

## Installation

```bash
# Ajouter le marketplace
claude plugin marketplace add https://github.com/cegape/cegape-marketplace.git
claude plugin marketplace update

# Installer un plugin
claude plugin install ticket@cegape-marketplace --scope user
```

## Plugins disponibles

| Plugin | Description | Version |
|--------|-------------|---------|
| [ticket](plugins/ticket/) | Rédacteur de tickets (3 Amigos) : entretien guidé, validation, création via le MCP Atlassian ou markdown. Spécifiques (apps, environnements, clés Jira) déportés dans un fichier de config local | 0.4.0 |
| [projet-claude](plugins/projet-claude/) | Architecte de Projet Claude : entretien guidé pour créer un Projet Claude optimal (claude.ai ou Cowork) — périmètre unique, instructions au format contrat, base de connaissances sans dilution. Commandes `/projet-claude:nouveau`, `/projet-claude:instructions`, `/projet-claude:auditer` | 0.1.0 |

## Gestion des plugins

```bash
claude plugin enable  <nom-plugin>@cegape-marketplace
claude plugin disable <nom-plugin>@cegape-marketplace
claude plugin update  <nom-plugin>@cegape-marketplace
```

## Licence

MIT
