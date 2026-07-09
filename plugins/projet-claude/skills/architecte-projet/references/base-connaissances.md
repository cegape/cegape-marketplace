# La base de connaissances — RAG, formats, limites, nommage

La **base de connaissances** rassemble les fichiers que Claude utilise comme contexte de reference dans chaque chat du projet. Instructions, fichiers charges et conversation **partagent** la meme fenetre de contexte.

## Bonnes pratiques officielles

- **Televerser tout le contenu pertinent des le depart** — plus Claude a de contexte utile, mieux il aide.
- **Noms de fichiers clairs et descriptifs** — ils aident Claude a retrouver la bonne information.
- **Regrouper les documents lies** dans le meme projet pour que Claude etablisse des connexions.
- **Referencer les documents par leur nom** dans les questions pour orienter la recherche.

## RAG automatique (retrieval)

Quand la connaissance du projet approche la limite de la fenetre de contexte, Claude bascule **automatiquement** en mode RAG (recherche), ce qui etend la capacite utile **jusqu'a 10x** : il ne charge alors que les passages pertinents via un outil de recherche dans la base, au lieu de tout garder en memoire. Activation automatique, sans configuration ; disponible sur tous les plans (gratuit a Enterprise). Consequence pratique : la **granularite** et le **nommage** des fichiers pesent directement sur la qualite de la recuperation.

## Granularite et dilution

- Prefere **plusieurs fichiers thematiques** bien nommes a un unique fichier fourre-tout — meilleure recuperation RAG.
- **Fichiers de reference stables** (guide de style, specs, docs) en base ; **contenu volatil** dans le chat.
- **Dilution de l'attention** : charger des fichiers inutiles « au cas ou » degrade les reponses sur *tout* le contexte, pas seulement sur le fichier en trop. N'ajouter que ce qui est reellement reutilise.
- Anti-pattern **« fichier monstre »** : un document de centaines de pages sans titres ni decoupage nuit au RAG. Le structurer et le scinder.

## Formats et limites

- **Formats supportes** : PDF, documents texte, code, CSV, images, etc. Claude analyse le texte **et** les elements visuels des PDF de **moins de 100 pages**.
- Limites chiffrees issues de **sources tierces** (non confirmees par une page officielle Anthropic, a verifier dans l'interface qui peut afficher un compteur) : ~**30 Mo par fichier** ; jusqu'a ~**20 fichiers par conversation**. La base de connaissances d'un projet n'a **pas de plafond strict documente** sur le nombre de fichiers — la vraie contrainte est la fenetre de contexte, geree par le RAG.
- **Fenetre de contexte** : selon le modele et le plan, 200K / 500K / jusqu'a 1M tokens.
