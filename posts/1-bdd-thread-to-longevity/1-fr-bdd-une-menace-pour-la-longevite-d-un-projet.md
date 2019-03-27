# Le BDD est-il une menace pour la longévité de votre projet ?

> Cet article n'est le reflet que de mon propre avis sur la question. Il a pour but d'ouvrir le débat et de potentiellement service de base de discussion.

## Les promesses du BDD

Le *Behavior-driven development* (ou **BDD**) est un processus de développement d'application faisant de plus en plus d'adeptes depuis quelques années.

La promesse est simple: piloter le développement par des comportements attendus, rédigés lors d'ateliers réunissant des *profils techniques* et des *sachants du métier*.

Sans entrer dans les détails, ces ateliers permettront de rédiger des spécifications dans un langage naturel (souvent la langue des intervenants), sous la forme de tests d'accéptences: les Gherkins.

> Exemple basique de Gherkin tiré de la documentation de [Cucumber.io](https://docs.cucumber.io) :
>
> Scénario : Manger 5 des 12 concombres
>
> - Étant donné qu'il y a 12 concombres
> - Lorsque je mange 5 concombres
> - Alors je dois avoir 7 concombres

Cette pratique a donc pour but de nous aider à parler le même langage entre les différentes parties prenantes du projet.

Côté développement, la promesse est d'avoir une solution qui réponde exactement aux cas d'usages définis.

## Le cycle de vie dans un projet Agile

À une époque où tout le monde souhaite **faire** (*sic*) de l'Agile, rares sont les projets avec des spécifications définies et gravées dans le marbre.

L'Agilité me semble parfaitement légitime, je ne le remets pas en cause ici. Cependant, les vraies difficultés commencent lorsqu'on souhaite entièrement travailler en *BDD* et avec une méthodologie Agile.

En effet, et comme présenté plus haut, nous allons écrire des tests sous la forme de Gherkins. Un test représentera une promesse que notre projet devra respecter.

Au fil des Gherkins, nous ajouterons de plus en plus de promesses, et il faudra naturellement refactoriser au fil du développement afin d'avoir une base de code saine, maintenable et évolutive.

Cette phase de refactorisation va, à la fin, créer une architecture **pilotée par les Gherkins, et donc par les comportements attendus**, mais est-ce vraiment souhaitable ?.

## Les dangers de l'Agilité : Des spécifications toujours changeantes

Toute personne ayant travaillait sur un projet Agile le sait bien, cette méthodologie implique deux grands axes dans la définition des spécifications. Ces dernières sont généralement :

- Plus petites (voir atomique) afin d'être incrémentales; et
- Surtout **changeantes**.

Nous arrivons donc au coeur du problème : changez un comportement (via le ou les Gherkins associés) et c'est toute votre architecture qui s'invalide !

Chaque modification dans les promesses de votre application risque de faire s'effondrer le château de cartes (Pun intended).

## Aparté sur la notion de métier

Chaque application résout des problématiques pour certains corps de métier.

Une application de gestion de livres pourra, par exemple, résoudre des cas d'usages pour des *libraires*, des *éditeurs* et des *auteurs*. Elle aura donc trois domaines métier distincts.

Il ne s'agit pas ici de queston *technique*, et vous devriez toujours faire attention à bien séparer ces deux entités.

Je vous redirige vers les concepts de Clean Architecture de Robert C. Martin pour plus de détails sur la façon de séparer métier et technique.

## Le DDD à la rescousse ?

Imaginez maintenant une autre façon de penser l'architecture de votre projet.

Ne pilotez plus vos développements par le *comportement*, mais par le **métier**.

Piloter par le métier, c'est ce que propose un autre courant de pensée, le **Domain Driven Design** (ou *DDD*).

Le **DDD** est une approche de conception visant à remettre le métier au coeur du développement applicatif.

Sans entrer dans les détails (je vous invite à lire **Domain-Driven Design: Tackling Complexity in the Heart of Software** d'*Eric Evans* pour cela), cette pratique partage beaucoup avec le *BDD* en ce qui concerne le dialogue avec les personnes du métier.

Mais nous irons ici un peu plus loin en mettant l'accent non sur les comportements attendus, mais sur les différents domaines métier.

Cet accent sera mis via la création d'un contexte par domaine, dans lequel primeront **les entitées** et **la logique** liée à ce dernier.

Cette approche vous permettrait donc d'avoir une base solide, basée sur le métier, sur laquelle vous pourrez construire des applications **durables**, **testables** et **Agiles** !

En effet, une spécifications changeante n'impactera que la façon dont vous appelez les différents contextes de votre métier. Il vous faudra peut-être retravailler certains domaines, mais le plus gros du travail consistera à agencer les briques qui composent votre application différemment.

## La place des Gherkins en approche DDD

Est-ce donc la fin des tests d'acceptances ? Doit-on jeter tous les Gherkins rédigés par nos *tres amigos* ?

Bien sûr que non, je vous propose plutôt de les utiliser à une autre fin, qui me semble plus naturelle : **comme garde-fou**.

Ainsi, nous aurons un projet qui reflête parfaitement le **métier** en jeu, et nous validerons que ce projet répond aux attentes des responsables et utilisateurs via les Gherkins.

Ces derniers ne pourraient utiliser le projet que comme un consommateur lambda, via l'API exposée. Cette pratique nous évitera de polluer le projet à cause des tests.

![Mariage DDD / Gherkins](https://github.com/ThomasFerro/readmes/blob/master/posts/1-bdd-thread-to-longevity/Mariage%20DDD%20_%20Gherkins.png)

> Gardez bien en tête que nous ne sommes pas là pour créer de la valeur *technique*, mais pour utiliser la technique afin de créer de la valeur **métier**.

## Conclusion

Je ne voulais surtout pas, dans cet article, rentrer dans les détails technique d'implémentation. Ce n'est qu'une introduction au sujet dans lequel sont exposées mes craintes et ce que je pense être une meilleure solution.

N'hésitez pas à partager votre avis sur la question, ainsi que vos retours d'expériences !
