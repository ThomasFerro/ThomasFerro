# SRE, où comment on laisse la Prod voyager de ses propres ailes

> Sylvain Nieuwlandt @Sfeir

L'équipe: 2 PO un Ops et 2 devs

PO: Récupération du besoin

Dev: Implémentation + package

Ops: Déploiement + maintient

Sauf que... Les utilisateurs remontent un bug 1 déploiement / 3 => Panique, rollback, frustration.

Perte de confiance:
- Utilisateurs (stabilité du produit)
- Équipes (tensions, blame, etc)
- Vendredi, pas de déploiement
- Cercle vicieu

Dynamique négative pour les fix:
- Attente de la déclaration d'un bug des utilisateurs
- Remontés lors du point avec les utilisateurs (1 fois par semaine) ==> Logs souvent perdus
- Dév du test + part en prod

"On va tenter la SRE !" => Site reliability engineering

- PO => Nouvelles features
- Ops => Stabilité !!! (le moins de changement possible)

Le SRE = un outil ? Plutôt une mentalité !
Le SRE = un rôle dédié dans l'équipe ? Toute l'équipe doit être impliquée
Le SRE = un département spécifique dans l'entreprise ? Chaque équipe doit être responsable de son propre SRE

Mentra du SRE: Préparez-vous pour le pire.

En pratique, ça donne quoi ?

- Tout est du code (métier, infra, documentation utilisateur / API).
- Réduire les actions manuelles.
- Peur des risques = manque de confiance. Ne pas hésiter à prendre des risques !
  - Exemple, mettre en place du feature flag pour réduire le risque mais en prendre quand même
- Écrire des postmortem ! (historique, description, root cause, contournement, actions pour ne plus avoir l'incident) => C'est de la doc, à versionner donc
- On n'est pas dans une partie de Cluedo, cherche un coupable ne sert à rien

Concepts techniques du SRE:
- Objectif de fiabilité (SLO), exprimé sur tout ce qui a de la valeur dans l'appli (en % de fiabilité sur un lapse de temps)
- Service Level Agreements, des SLO contractuels
- Error budget: 100% - SLO décompté à partir du moment où une erreur apparaît jusqu'à ce que les utilisateurs ont de nouveau accès aux services

Faire adopter le SRE dans une équipe:
- Les dév: Tout doit être automatisé
- Les ops: Pareil, tout sous forme de code, versionné, review
- Les PO: L'error budget doit être respecté et n'est pas négociable

Silver bullet, véto que les PO ont le droit d'utiliser ==> impose un post mortem

Laisser la prod se réparer:
- Alerting sur du long terme et pas uniquement sur la prod, sur les env de dév et recette
- Brancher l'alerting sur des scripts de rattrapage / de gestion d'erreur
