# Reference — Template de Ticket (3 Amigos)

Standard de redaction QA + Produit. Le skill est generique : les applications, environnements
et cles de projet proviennent du fichier de config local (voir `<config_schema>`), jamais d'une
valeur en dur. Charge la section utile selon ce que tu rediges.

---

<config_schema>
## Schema du fichier de config

Emplacement (ordre de recherche) : `.claude/ticket.config.json` (projet) puis
`~/.claude/ticket.config.json` (utilisateur). Modele : `assets/ticket.config.example.json`.

```jsonc
{
  "applications": [
    {
      "name": "MonApp",                 // nom affiche de l'application
      "jiraProjectKey": "APP",          // cle de projet Jira par defaut pour cette app
      "environments": [
        { "name": "Recette",    "url": "https://recette.monapp.example" },
        { "name": "Preprod",    "url": "https://preprod.monapp.example" },
        { "name": "Production", "url": "https://monapp.example" }
      ]
    }
  ],
  "jira": { "site": "monorg.atlassian.net" }   // optionnel : cible le bon site Atlassian
}
```

Regles d'usage :
- Si la config existe, proposer la liste des `applications`, pre-remplir l'URL d'environnement
  choisie et la `jiraProjectKey` correspondante.
- `jira.site` est optionnel : sans lui, resoudre le site via `getAccessibleAtlassianResources`.
- Si la config est absente, demander ces infos a l'utilisateur et proposer de creer le fichier
  depuis le modele. Ne jamais inventer d'URL ni de cle de projet.
</config_schema>

---

<template_universel>
## Template universel

Structure commune a tous les types. Les sections marquees *(conditionnel)* ne s'appliquent qu'a certains types — voir `<matrice_champs>`.

### TITRE
`[Bug]` / `[Regression]` / `[Amelioration]` / `[US]` — titre court et precis — `<App>`

### ENVIRONNEMENT
Lien URL ou nom de l'environnement (Recette / Preprod / Production) + version.
Si la config fournit l'app, reprendre l'URL de l'environnement choisi.

### CHEMIN D'ACCES
Fil d'Ariane fonctionnel. Ex : `Menu -> Sous-menu -> Ecran`.

### DESCRIPTION
Contexte metier : qui est impacte, pourquoi ce ticket existe.

### ETAPES A REPRODUIRE
Liste numerotee : 1. Aller sur... 2. Cliquer sur... 3. Observer que...
(Pour une US/Amelioration, c'est le scenario d'usage, fonctionnalite « a creer » incluse.)

### COMPORTEMENT CONSTATE *(conditionnel — Bug / Regression)*
Ce qui se passe reellement (chiffres concrets si possible).

### COMPORTEMENT ATTENDU *(conditionnel — Bug / Regression)*
Ce qui devrait se passer (regle metier exacte).

### CRITERES D'ACCEPTANCE *(conditionnel — Amelioration / US)*
Un ou plusieurs trios :
- **Given** : [contexte]
- **When** : [action]
- **Then** : [resultat attendu]

### CAPTURES D'ECRAN
Obligatoire pour Bug & Regression — recommande pour Amelioration. A joindre dans Jira apres creation.
</template_universel>

---

<matrice_champs>
## Matrice des champs requis

| Type | Constate + Attendu | Criteres d'acceptance | Captures |
|---|:---:|:---:|:---:|
| `[Bug]` | ✓ obligatoire | — | ✓ obligatoire |
| `[Regression]` | ✓ obligatoire | — | ✓ obligatoire |
| `[Amelioration]` | — | ✓ obligatoire | recommande |
| `[US]` | — | ✓ obligatoire | — |

Champs **communs obligatoires** (tous types) : Titre tagge, Environnement, Chemin d'acces, Description, Etapes a reproduire.

### Regles d'or
1. Tag obligatoire en debut de titre.
2. `[Regression]` = mentionner la version ou ca marchait.
3. Toujours renseigner environnement + chemin d'acces.
</matrice_champs>

---

<mapping_jira>
## Mapping tag -> type d'issue Jira

Le tag QA vit dans le `summary`. Le `issueTypeName` Jira doit etre **resolu via `getJiraProjectIssueTypesMetadata`**, jamais code en dur (les projets exposent des types differents).

| Tag QA | Type Jira cible (selon dispo projet) |
|---|---|
| `[Bug]` | Bug |
| `[Regression]` | Bug (tag conserve dans le titre) |
| `[Amelioration]` | Improvement, sinon Story ou Task |
| `[US]` | Story |

Si aucun type ne correspond, presenter la liste reelle des types du projet et demander.
</mapping_jira>

---

<squelette_sortie>
## Squelette markdown de sortie

A utiliser pour le `description` du `createJiraIssue` (contentFormat markdown) et pour le fallback copier-coller. Retirer les sections conditionnelles non applicables au type.

```markdown
**Titre :** [Tag] <titre court precis> — <App>

### Environnement
<URL ou nom de l'env> · <version>

### Chemin d'acces
<Menu -> Sous-menu -> Ecran>

### Description
<contexte metier : qui est impacte, pourquoi>

### Etapes a reproduire
1. <...>
2. <...>
3. <...>

<!-- Bug / Regression uniquement -->
### Comportement constate
<ce qui se passe reellement>

### Comportement attendu
<ce qui devrait se passer>

<!-- Amelioration / US uniquement -->
### Criteres d'acceptance
- **Given** <contexte>
- **When** <action>
- **Then** <resultat attendu>

### Captures d'ecran
<obligatoire Bug/Regression — a joindre dans Jira>
```
</squelette_sortie>

---

<exemple_bug>
## Exemple — [Bug]

(App et URL fictives — viennent normalement de la config.)

**Titre :** `[Bug] Calcul solde conges incorrect apres cloture mensuelle — MonApp`
**Environnement :** https://recette.monapp.example · Recette v2.3.1
**Chemin d'acces :** Gestion du Personnel -> Conges & Absences -> Solde Conges

**Description**
Apres une cloture mensuelle, le solde de conges affiche pour certains salaries est incorrect : les jours pris dans le mois cloture ne sont pas deduits.

**Etapes a reproduire**
1. Se connecter en tant que Gestionnaire RH
2. Effectuer une cloture mensuelle (mois M)
3. Aller sur Conges & Absences -> Solde Conges
4. Observer le solde d'un salarie ayant pris des conges en M

**Comportement constate**
Le solde affiche 12 jours alors que le salarie a pris 2 jours en M (devrait etre 10). Les jours pris ne sont pas deduits apres cloture.

**Comportement attendu**
Apres cloture, le solde = Solde initial − Jours pris dans le mois cloture.

Captures : obligatoire pour ce type.
</exemple_bug>

---

<exemple_regression>
## Exemple — [Regression]

**Titre :** `[Regression] Module paie KO apres maj v2.3 — MonApp`
**Environnement :** https://recette.monapp.example · Recette v2.3
**Chemin d'acces :** Paie -> Bulletins -> Generation

**Description**
La generation des bulletins fonctionnait en **v2.2** et echoue depuis la mise a jour **v2.3**.

**Etapes a reproduire**
1. Se connecter en tant que Gestionnaire Paie
2. Aller sur Paie -> Bulletins -> Generation
3. Lancer la generation d'un lot

**Comportement constate**
La generation echoue avec une erreur ; aucun bulletin produit (fonctionnait en v2.2).

**Comportement attendu**
Le lot de bulletins est genere comme en v2.2.

Captures : obligatoire. Version ou ca marchait : **v2.2**.
</exemple_regression>

---

<exemple_amelioration>
## Exemple — [Amelioration]

**Titre :** `[Amelioration] Ajouter export PDF sur le bulletin de paie — MonApp`
**Environnement :** https://recette.monapp.example · Recette v3.1.0
**Chemin d'acces :** Paie -> Bulletins -> Detail bulletin

**Description**
Les bulletins ne sont consultables qu'a l'ecran ; les gestionnaires doivent imprimer pour archiver. Un export PDF direct ferait gagner du temps.

**Criteres d'acceptance**
- **Given** un gestionnaire sur le detail d'un bulletin
- **When** il clique sur « Exporter en PDF »
- **Then** un PDF fidele du bulletin est telecharge

Captures : recommande.
</exemple_amelioration>

---

<exemple_us>
## Exemple — [US]

**Titre :** `[US] En tant que Gestionnaire RH, je veux filtrer les individus par service — MonApp`
**Environnement :** https://recette.monapp.example · Recette v3.1.0
**Chemin d'acces :** Gestion Du Personnel -> Individus -> Liste des Individus

**Description**
La liste des individus n'est pas filtrable par service ; les gestionnaires doivent faire defiler toute la liste, ce qui est chronophage.

**Scenario**
1. Se connecter en tant que Gestionnaire RH
2. Aller sur Gestion du Personnel -> Individus
3. Utiliser le filtre « Service » (a creer)
4. Selectionner un service
5. Verifier que seuls les individus du service apparaissent

**Criteres d'acceptance**
- **Given** un utilisateur connecte en tant que Gestionnaire RH
- **When** il selectionne un service dans le filtre
- **Then** seuls les individus de ce service sont affiches
</exemple_us>
