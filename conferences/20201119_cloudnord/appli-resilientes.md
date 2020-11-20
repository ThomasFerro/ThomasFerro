# Et si ca tourne mal ? Recettes pour des applications résilientes

> Christopher Maneu @Microsoft https://aka.ms/cloudnord-scuba

Plongée sous-marine // appli résilientes

> **Appli résiliente**: Capable de gérer des problèmes (fault) de manière à retomber sur ces pattes et ne pas perdre de données

## Préparer sa plongée

Plannifier ce qui peut mal se passer. Technique des "what if":

1. Identifier les dangers potentiel
2. Évaluer les risques
3. Préparer une solution pour chaque problème
4. Répéter le process

Monde de l'info ==> Failure mode analysis

Autre technique: Checklist à relire en entière pour tout valider

Dépassement de la zone de confort: **ne changer qu'un paramètre à la fois !**

Définir la disponibilité qu'on vise (SLA) en fonction du besoin.

## On se jette à l'eau !

Est-ce qu'on est capable de faire un petit schéam avec tous les flux, toutes les dépendances, etc ?

Quand quelque chose se passe mal, qu'est-ce qu'on fait ?

- Garder la tête froide et gérer les priorités les unes après les autres;
- Framework TDODAR 
  - T: Temps de réflexion (ou team)
  - D: Diagnostique
  - O: Options disponibles
  - D: Décision
  - A: Action / assignement
  - R: Review
- Brownout strategy: Quelles sont les briques de l'appli qu'on peut désactiver / réduire pour nous donner un peu d'air

## Une fois l'orage passé

> On sait que c'était une mauvaise plongée lorsque personne ne se parle sur le bateau.

Prendre le temps de discuter calmement des problèmes et mettre à plat ce qui a besoin de l'être. Prévoir sur le long terme.

Postmortem:
- Pas là pour accuser une personne ou une équipe;
- Tout ce qui doit être dit doit l'être (inclusivité de la parole);
- Utiliser plusieurs angles (technique, technologique, psychologique)

Amélioration continue basée sur les problèmes.

Article de Gauthier: How we built our infrastructure fail-over checklist
