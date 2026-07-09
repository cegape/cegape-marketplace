---
description: Audite les instructions d'un projet Claude existant (dilution, contradictions, longueur) et propose une reecriture
allowed-tools: Read, Write
---

## Tâche

Critiquer les instructions (et, si fournie, la liste des fichiers de la base de connaissances) d'un projet Claude **existant**, puis proposer une réécriture.

1. Demander à l'utilisateur de **coller les instructions actuelles** de son projet (et, en option, la liste des fichiers de sa base de connaissances). Ne pas auditer à l'aveugle.
2. Lire la **checklist qualité** dans `${CLAUDE_PLUGIN_ROOT}/skills/architecte-projet/references/contrat.md` et appliquer **chacun** de ses points au contenu fourni.
3. Rendre un rapport : pour chaque point en échec, nommer le problème, son effet concret (dilution de l'attention, règle noyée ou contradictoire, invention, nom/description détournés), puis la correction.
4. Livrer une **réécriture** des instructions au format contrat (six blocs), dans un bloc à copier.
5. Si des fichiers ont été listés, signaler ceux qui provoquent de la dilution (« au cas où », fichier monstre, doublons) et ce qui devrait plutôt rester dans le chat — voir `references/base-connaissances.md`.

Ne rien inventer : si un élément est ambigu, poser une question avant de conclure.
