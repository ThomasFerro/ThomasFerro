# Kubernetes enfin ultra simple & léger avec K3S

> Sébastien Moreno @Ippon

Kubernetes: Plateforme majeur pour les microservices (scheduling, scalability, self-healing). Techno basée sur une logique de cluster (ensemble de nodes).

Problématiques:
- Installation difficile
- Paramétrage complexe, 1ère expérience âpre
- Consommation de ressources assez importante

==> Difficile à mettre en place sur des env légers

k3s: Porté par Rancher, distribution Kubernetes limité à un seul binaire (40MB). Plus light, plus opti.

==> Certified Kubernetes Distribution par la CNCF

Options:
- On peut utiliser d'autres datastore que sqlite
- ...

Limitations:
- Doc un peu light
- Pas mal de choses expérimentales
- Besoin de maturité

Ecosystème d'outils pour se faciliter la vie pour l'installation de clusters.

==> k3s: Utile pour dédramatiser Kubernetes et se faire la main
