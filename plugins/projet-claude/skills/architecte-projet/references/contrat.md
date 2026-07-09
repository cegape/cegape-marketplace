# Le contrat d'instructions — format, exemples, checklist

Les **instructions du projet** s'appliquent a **toutes** les conversations du projet. Principe officiel : les garder **concises** — elles servent au contexte general, aux regles cles et au role de Claude, pas a heberger une documentation. Il n'existe **aucune limite de caracteres officielle** ; le chiffre « ~8 000 caracteres » qui circule est une estimation communautaire non sourcee, a traiter comme indicatif. La regle qui compte : *concis*.

## Le format contrat (six blocs)

1. **Role** (1 ligne) — qui Claude incarne (« Tu es l'assistant de redaction de… »).
2. **Objectif / criteres de reussite** — a quoi ressemble une bonne reponse.
3. **Contraintes** (puces) — ton, longueur, sources autorisees, perimetre.
4. **Regle d'incertitude** — « Si tu n'es pas sur, dis-le et pose une question. » Empeche l'invention.
5. **Format de sortie** — la structure attendue de chaque reponse.
6. **A ne pas faire** (puces) — interdictions explicites. Dire ce qu'il faut eviter est souvent plus efficace qu'empiler des consignes positives.

## Exemple A — « Redaction newsletter hebdo »

```
Role : Tu es l'assistant de redaction de la newsletter hebdomadaire de l'equipe Produit.

Objectif : produire un brouillon pret a relire, informatif et engageant, en 600-800 mots.

Contraintes :
- Ton direct, phrases courtes, deuxieme personne.
- T'appuyer sur le guide de style (fichier guide-style.md) et le ton des editions passees.
- Une seule idee principale par edition.

Regle d'incertitude : si une info manque (chiffre, date, nom), demande-la — n'invente jamais.

Format de sortie : Titre accrocheur + chapo (2 lignes) + 3 sections avec sous-titres + un appel a l'action final.

A ne pas faire :
- Pas de superlatifs marketing (« revolutionnaire », « incroyable »).
- Ne pas inventer de statistiques ni de citations.
- Ne pas depasser 800 mots.
```

## Exemple B — « Tri du support N1 »

```
Role : Tu es un agent de support niveau 1 pour l'application X.

Objectif : classer chaque demande entrante et proposer une premiere reponse, ou escalader.

Contraintes :
- T'appuyer sur la base de connaissances (faq.md, procedures.md) — jamais sur des suppositions.
- Rester factuel et courtois ; pas d'engagement de delai.

Regle d'incertitude : si la demande sort des procedures connues, ne devine pas — marque « A ESCALADER N2 » et resume le probleme.

Format de sortie : Categorie | Priorite (P1-P3) | Reponse proposee OU mention d'escalade.

A ne pas faire :
- Ne pas promettre de remboursement ni de correctif date.
- Ne pas citer de procedure absente de la base de connaissances.
- Ne pas repondre hors du perimetre de l'application X.
```

## Checklist qualite (sert aussi a `/projet:auditer`)

Passer chaque instruction / chaque projet au crible :

1. **Perimetre unique** — le projet couvre une seule fonction, pas un fourre-tout « Travail ».
2. **Contrat complet** — les six blocs sont presents.
3. **Concis** — aucune documentation entiere collee dans les instructions (elle devrait etre un fichier de connaissance).
4. **Sans contradiction** — pas de regles qui s'annulent (ex. « sois concis » + « detaille tout »).
5. **Pas de doublon** avec les preferences de profil (compte-wide).
6. **Nom/description non detournes** — ils ne sont pas lus par Claude ; aucune consigne ne doit y etre cachee.
7. **« A ne pas faire » present et specifique** — des interdictions concretes, pas des generalites.
8. **Base de connaissances saine** — uniquement des docs reutilises, bien nommes, granulaires ; pas de « fichier monstre », pas d'upload « au cas ou » (dilution).
9. **Connecteurs limites** au strict necessaire.
10. **Plan d'iteration** — un mecanisme pour affiner apres quelques conversations.

Pour chaque point en echec : nommer le probleme, en donner l'effet concret (dilution, regle noyee, invention), puis proposer la reecriture.
