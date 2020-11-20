# 9 questions que vous devriez vous poser sur vos applications Serverless

> François Bouteruche @AWS

Exemple d'un site tombé après passage à la tv.

Migration du site sur AWS Lambda pour essayer de déléguer au maximum la gestion de la charge.

Le site replante, cette fois-ci à cause des connexions à la base de données. N'était-ce pas prévisible ? N'y a-t-il pas de bonnes pratiques ?

Publication d'AWS Well-Architected Framework en 2015 et mises à jour depuis. Serverless Applications Lens publié en 2017.

Les technos vont passer, les pratiques restent (illustration avec des technos AWS mais applicable on premise ou n'importe quel CP).

## Qu'est-ce qu'une appli serverless ?

Selon AWS: Pas de serveur à gérer, mise à l'échelle flexible et automatique, paiement à la valeur délivrée, haute dispo intégrée au service.

Adopté massivement par des startups et des grandes entreprises en France et ailleurs.

## Questions à se poser:

Comment suivre l'état de santé d'une appli serverless ?
- Centralisation de l'état de tous les services (AWS: Cloud Watch)
- Tracage distribué: Suivre la requête à travers les différents services (AWS: X-Ray)

Comment régulation le nombre de requêtes entrantes ?
- Throttling
- Limite de concurrence
- Service d'ingestion de requête (passage en asyncrone)

Comment assurer la résilience de l'application ?
- Service d'ingestion de requête (passage en asyncrone)
- Dead Letter Queue pour gérer les erreurs

Comment sont optimisés les perfs de l'application ?
- Gérer le ration cout / perf avec la config

Comment les accès aux API sont contrôlés ?
- Authentification, nécessaire mais pas suffisant !
- Politiques d'accès aux ressources (groupes d'utilisateurs, adresses, etc)

Comment le périmétre de sécurité est-il géré ?
- Définir des rôles avec des permissions + associé ces rôles aux services
- Protection des données: Chiffrage des données sensibles dans tout le système

Comment la sécurité application est-elle implémentée ?
- Pas sécurisé par défaut ! Comme pour une appli "classique"
- Recommandations OWASP à appliquer (validation des données, revues de code d'un point de vue sécurité)

Comment le cycle de vie de l'application est-il géré ?
- Au delà du code métier, l'infra doit aussi être géré dans les sources (infra as code)

Comment les coûts sont optimisé ?
- Gérer la config du stockage.
- Utiliser les bons outils pour réduire les coûts.
- Optimisation du code
- Monitoring du coût de l'application pour avoir les infos nécessaires à la prise de décision
