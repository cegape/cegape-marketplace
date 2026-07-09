# claude.ai vs Cowork (Desktop) — capacites, memoire, connecteurs

Deux objets differents portant le meme mot « projet ». Ils sont stockes separement ; on peut **lier** un projet claude.ai a un projet Cowork sans les fusionner.

| | **Projet claude.ai** | **Projet Cowork (Desktop)** |
|---|---|---|
| Ou vit-il | Cloud (web + apps) | **Local** sur l'ordinateur, pas de synchro cloud |
| Base de connaissances | Fichiers televerses | Dossiers locaux + liens |
| Partage d'equipe | Oui | Non |
| Taches planifiees | Non | Oui |
| Recommande pour | Exploration generale, premier projet | Workflows agentiques locaux (lire/ecrire des fichiers, taches) |

Pour un premier projet ou un usage general : **claude.ai**. Choisir **Cowork** seulement si Claude doit toucher au disque local ou executer des taches planifiees.

## Contexte, memoire, incognito

- **Le contexte n'est pas partage entre les chats** d'un meme projet — sauf ce qui est mis dans la base de connaissances (ou dans la memoire). Ce qui est dit dans le chat A n'est pas connu du chat B.
- **Memoire par projet** (plans payants) : chaque projet a sa **propre** memoire, editable ; utile pour la continuite d'un travail recurrent.
- **Incognito** : mode pour les conversations qui ne doivent pas etre enregistrees en memoire.
- Cowork a egalement une memoire **dediee** au projet local.

## Connecteurs / MCP

Le Model Context Protocol connecte Claude a des outils externes (Google Drive, Slack, etc.), geres via « Customize » > « Connectors ».

- **Limiter les connecteurs actifs** au strict necessaire : ils sont **couteux en tokens** et chaque outil actif consomme du contexte a chaque message.
- **Risque de securite** : desactiver les outils non pertinents ; se mefier de l'**injection de prompt** et des connecteurs non verifies. N'ajouter qu'un ou deux connecteurs a la fois, apres avoir evalue la confiance.

## A verifier

Disponibilites et dates par plan evoluent vite (memoire, RAG, Cowork). En cas de doute, consulter les release notes officielles avant toute decision.
