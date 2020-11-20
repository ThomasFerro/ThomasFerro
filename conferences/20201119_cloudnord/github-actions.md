# Github Actions : enfin des pipelines accessibles aux développeurs

> Antoine Meausoone @Sfeir

"J'aime pas faire deux fois la même chose" => forcément de l'xp sur la CI/CD

Expériences:
- Hudson
- Jenkins
- Travis
- Concourse
- GitlabCI
- Github Actions

## Petit historique des pipelines

Jenkins: Premier outil qui permettait l'automatisation du build, des TU.

Apparition des plugins Jenkins (rundeck, ansible, docker swarm)

On souhaitait ajouter des conditions, compliqué à faire avec des formulaires.

Pipeline as code: Jenkinsfile, .travis.yml, .gitlab-ci.yaml ==> conditions complèxes dans la pipeline

Ajout d'un peu de shell.... rend le tout difficilement modifiable par les dev !

Extraction du pipeline dans une lib ! Réplicable, mais toujours pas opti... Par exemple, lorsque 10 projets utilisent le pipeline.. Besoin d'isolation !

Pipelines executées dans des conteneurs Docker 

Github Actions: souahite faciliter les choses, les rendre plus accessibles

## Github actions

Arrivé assez tardivement dans le milieu. Permet de récupérer la connaissance à partir des erreurs / bonnes idées des concurrents.

Comment ça marche ?

Chaque pipeline démarre dans une VM fraiche.

Chaque step s'execute dans la VM (possibilité de l'exé dans une image Docker).

Définition des pipelines dans .github/workflows. Un pipeline par fichier.

Description du fichier:
- L'event qui trigger le pipeline. (`on: [ <events> ]`) Possible aussi d'utiliser une syntaxe plus complexe, avec une configuration plus fine
- Description du pipeline (`jobs`)
  - Conditions sur les étapes ou le job complet.
  - Matrices pour lancer des jobs sur plusieurs éléments 
  - `steps`: Description d'une étape
    - `run`: le shell à exécuter
    - Démarrable dans un container docker
    - Possibilité de déléguer une étape à une Github Action

> Github Action: Action unitaire avec des entrées / sorties conçu pour intérragir facilement avec git et l'API Github

Exemple: checkout, super-linter, ..., et plein de setup

Objet `github` avec plein d'info sur le contexte d'exécution du pipeline.

Gestion des secrets, masqués dans les logs :+1:

Inconvénients:
- Qu'est-ce qui arrive lorsque le créateur d'une Action la supprime ?
- Que se passe-t-il si un créateur modifie l'Action pour miner du Bitcoin par exemple ?

