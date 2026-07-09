---
description: Redige ou raffine le bloc d'instructions d'un projet (format contrat) a coller dans le projet
argument-hint: "[perimetre du projet]"
allowed-tools: Read, Write
---

## Tâche

Produire **uniquement** le bloc d'instructions d'un projet, au format **contrat**, prêt à coller dans le projet Claude.

1. Le périmètre peut être fourni dans `$ARGUMENTS`. S'il manque, ou s'il reste vague, le préciser par **une** question en texte libre — un contrat sert un seul périmètre.
2. Lire le squelette `${CLAUDE_PLUGIN_ROOT}/skills/architecte-projet/assets/instructions.template.md` et le format détaillé + exemples dans `${CLAUDE_PLUGIN_ROOT}/skills/architecte-projet/references/contrat.md`.
3. Rédiger les six blocs (Rôle, Objectif, Contraintes, Règle d'incertitude, Format de sortie, À ne pas faire). Rester **concis** : ne pas coller de documentation — elle relève de la base de connaissances.
4. Livrer le contrat dans un bloc à copier, puis le passer à la **checklist qualité** de `references/contrat.md` et signaler toute faiblesse restante.

Ne rien inventer : demander les informations manquantes plutôt que de les supposer.
