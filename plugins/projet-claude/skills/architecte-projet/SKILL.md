---
name: Architecte de Projet Claude
description: This skill should be used when the user wants to create, structure or optimize a Claude Project (espace de travail persistant sur claude.ai ou Claude Desktop/Cowork), or asks to "creer un projet Claude", "nouveau projet Claude Desktop", "structurer un projet Claude", "rediger les instructions d'un projet", "optimiser mon projet Claude", "auditer les instructions d'un projet Claude". The skill menes a guided conversation to fix a single PERIMETRE, choose the platform (claude.ai vs Cowork), write the instructions au format CONTRAT (role + objectif + contraintes + regle d'incertitude + format + a ne pas faire), and plan a knowledge base sans DILUTION. Produit un bloc d'instructions et un plan de fichiers a coller dans le projet. SKIP for coding tasks in the current repository — a Claude Project is a UI workspace, not a code repository.
---

# Architecte de Projet Claude

Aide a creer un **Projet Claude** optimal : un espace de travail persistant qui recharge automatiquement des **instructions** et une **base de connaissances** dans chaque conversation du projet. Un projet ne se cree pas en ligne de commande — c'est un objet d'interface (claude.ai ou Claude Desktop). Le role du skill est donc de **produire les deux artefacts** que l'utilisateur collera dans l'interface : un **bloc d'instructions** au format contrat, et un **plan de base de connaissances** (quels fichiers, comment les nommer).

Le coeur est une **conversation** menee au fil de l'eau (une question a la fois, en texte libre — jamais un formulaire, jamais de cartes a options). Trois mots-guides gouvernent toutes les decisions :

- **Perimetre** — un projet = **une** fonction. Le peche capital est le fourre-tout « Travail ».
- **Contrat** — les instructions sont courtes et structurees, pas une documentation.
- **Dilution** — l'ennemi : tout fichier ou toute regle non reutilise degrade la qualite sur *tout* le reste.

## 1. Fixer le perimetre

Avant tout, cerne **un** perimetre unique par la conversation. Pars de ce que l'utilisateur veut faire de facon recurrente (ex. « rediger la newsletter », « trier le support N1 »), pas d'un domaine vague.

- Si le besoin melange plusieurs fonctions distinctes, propose de le **scinder** en projets separes — un projet flou perd son focus et sa memoire devient inutile.
- Signal decisif : si l'utilisateur se surprend a **re-expliquer le meme contexte** dans plusieurs chats, ce contexte doit vivre dans les instructions ou la base de connaissances.

Le **nom** viendra a l'etape 5 : rappelle des maintenant que nom et description **ne sont pas lus par Claude** (organisation humaine uniquement) — ils ne remplacent jamais des instructions.

## 2. Choisir la plateforme (branche)

Demande sur quelle plateforme vit le projet ; l'orientation change ensuite.

- **claude.ai** (cloud, web + apps) : base de connaissances de fichiers, partage d'equipe. Le plus polyvalent — **recommande** par defaut, surtout pour un premier projet.
- **Cowork (Claude Desktop)** : projet **local** (dossiers du disque, liens, taches planifiees, memoire dediee, pas de synchro cloud ni de partage). A choisir seulement si Claude doit lire/ecrire des fichiers locaux ou executer des taches planifiees.

Regle rapide : exploration generale sans fichiers locaux -> claude.ai. Workflows agentiques sur le disque -> Cowork. Les capacites, la memoire, l'incognito et les connecteurs different par plateforme : charge **`references/plateformes.md`** des que la plateforme est connue.

## 3. Rediger le contrat (instructions)

C'est le coeur. Redige des instructions **courtes** au format **contrat**, en six blocs :

1. **Role** (1 ligne) — qui Claude incarne.
2. **Objectif / criteres de reussite** — ce qu'une bonne reponse accomplit.
3. **Contraintes** (puces) — ton, longueur, sources, perimetre.
4. **Regle d'incertitude** — « Si tu n'es pas sur, dis-le et pose une question » (empeche les inventions).
5. **Format de sortie** — la structure attendue.
6. **A ne pas faire** (puces) — interdictions explicites ; souvent plus efficace que d'empiler des consignes positives.

Garde le contrat concis : ne **colle jamais une documentation entiere** dans les instructions (elle consomme des tokens a chaque message) — elle va en base de connaissances. Le squelette a remplir est dans `assets/instructions.template.md` ; le format detaille, deux exemples remplis et la **checklist qualite** sont dans `references/contrat.md` — charge-le pour composer.

## 4. Planifier la base de connaissances

N'ajoute que des **documents de reference reellement reutilises** (guide de style, specs, exemples). Chaque fichier inutile provoque de la **dilution**.

- **Noms de fichiers clairs et descriptifs** — Claude s'en sert pour retrouver l'info.
- **Granularite** : plusieurs fichiers thematiques bien nommes valent mieux qu'un unique « fichier monstre ».
- Reference stable -> base de connaissances ; contenu volatil -> dans le chat.

Le fonctionnement du RAG, les formats supportes et les limites (avec leurs reserves) sont dans `references/base-connaissances.md`.

## 5. Reglages du projet

- **Nom** precis (ex. « Redaction newsletter », pas « Travail ») — mais **non lu par Claude**.
- **Memoire par projet** (plans payants) pour la continuite ; **incognito** pour les echanges a ne pas memoriser.
- **Connecteurs / MCP** : limite-les au strict necessaire (ils sont couteux en tokens et elargissent la surface de risque).
- Rappelle que **le contexte n'est pas partage entre chats** d'un meme projet, sauf via la base de connaissances (ou la memoire).

Details par plateforme : `references/plateformes.md`.

## 6. Produire et iterer

Livre a l'utilisateur, dans des blocs prets a copier :

1. Le **bloc d'instructions** rempli (format contrat).
2. Le **plan de base de connaissances** : liste des fichiers a creer/uploader avec leur nom, plus ce qui doit rester dans le chat.
3. Un rappel d'**iteration** : apres 3-5 conversations, transformer les erreurs recurrentes de Claude en regles « a ne pas faire », re-uploader les fichiers mis a jour, et ne passer a l'echelle (nouveaux projets, memoire, connecteurs) que quand le besoin apparait.

## Acces rapide aux references

| Besoin | Reference |
|---|---|
| Format contrat detaille, exemples remplis, checklist qualite/audit | `references/contrat.md` |
| RAG, formats de fichiers, limites, nommage, dilution | `references/base-connaissances.md` |
| claude.ai vs Cowork, memoire, incognito, connecteurs, fenetre de contexte | `references/plateformes.md` |
| Squelette d'instructions a copier | `assets/instructions.template.md` |
