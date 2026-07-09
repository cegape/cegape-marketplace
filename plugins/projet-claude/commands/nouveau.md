---
description: Cree un Projet Claude optimal de bout en bout (perimetre, plateforme, instructions au format contrat, base de connaissances)
allowed-tools: Read, Write
---

## Tâche

Mener l'entretien complet du skill **architecte-projet** pour produire un Projet Claude optimal, puis livrer les artefacts prêts à coller.

Déroule les six étapes du skill, **une question à la fois, en texte libre** (jamais un formulaire, jamais de cartes) :

1. **Périmètre** — fixer une seule fonction ; refuser le fourre-tout, proposer de scinder si le besoin est multiple.
2. **Plateforme** — claude.ai (recommandé) ou Cowork ; charger `${CLAUDE_PLUGIN_ROOT}/skills/architecte-projet/references/plateformes.md`.
3. **Contrat** — rédiger les instructions (six blocs) à partir de `${CLAUDE_PLUGIN_ROOT}/skills/architecte-projet/assets/instructions.template.md` ; format détaillé et exemples dans `references/contrat.md`.
4. **Base de connaissances** — planifier les fichiers réutilisés et bien nommés ; voir `references/base-connaissances.md`.
5. **Réglages** — nom (non lu par Claude), mémoire/incognito, connecteurs au minimum.
6. **Produire et itérer** — livrer le bloc d'instructions + le plan de fichiers dans des blocs à copier, et rappeler le cycle d'itération.

Ne rien inventer : si une information manque, la demander.
