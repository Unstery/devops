# UR1/ESIR DevOps Course
This repository contains the material and content of the DevOps course at the engineering school ESIR of the University of Rennes 1. 

## Membres du groupe
Louis MORLOT--PINTA | Tergel TSAGAAN | Bastien SAUVAT | Bastien FAISANT | Jérôme SCHENBERG | Arthur MORVAN

## Introduction
Afin d'améliorer la qualité du code et à identifier les problèmes le plus rapidement, les tests tel le ```Fuzzing```, ```A/B Testing``` et le ```Canary testing``` permettent aux développeurs d'accélérer le processus de développement et de déploiement tout en s'assurant que le produit final est de haute qualité. En effet dans ce tutoriel, 3 outils de tests vont être présentés : 
- Le ```Fuzzing``` est une technique d'analyse de sécurité qui consiste à envoyer des entrées aléatoires à un logiciel pour voir comment il réagit. Cela peut aider à identifier les vulnérabilités de sécurité dans le code, car le Fuzzing peut révéler des bugs et des failles de sécurité que les tests manuels n'auraient pas détectés. Dans le cadre de la méthode DevOps, le Fuzzing est souvent utilisé pour automatiser les tests de sécurité pendant le cycle de développement et de déploiement.
- L'```A/B testing``` est une technique d'optimisation qui consiste à comparer 2 versions différentes d'un produit ou d'une fonctionnalité auprès des utilisateurs pour déterminer laquelle fonctionne le mieux. Cela peut aider les développeurs à améliorer l'expérience utilisateur, à optimiser les taux de conversion et à prendre des décisions plus éclairées sur la manière d'améliorer le produit ou la fonctionnalité. Dans le cadre de la méthode DevOps, l'A/B testing est souvent utilisé pour mesurer l'impact des changements de code sur l'expérience utilisateur.
- Le ```Canary testing``` est une technique qui consiste à déployer une nouvelle version de code auprès d'un petit groupe d'utilisateurs pour voir comment elle fonctionne avant de la déployer à grande échelle. Cela peut aider à identifier les bugs et les problèmes de performance avant qu'ils ne deviennent un problème majeur. Dans le cadre de la méthode DevOps, le Canary testing est souvent utilisé pour s'assurer que les nouveaux déploiements de code ne perturbent pas l'expérience utilisateur.

## Outils pour Fuzzing, A/B and Canary Testing

### Fuzzing

Le fuzzing test permet de détecter les erreurs de programmation ou les vulnérabilités en envoyant des entrées aléatoires à une application. Pour mettre en place ceci, nous prenons l'exemple d'un projet ayant un backend en NodeJS et frontend en Angular. Ce projet comporte la particularité d'avoir un système d'authentification à l'arrivée sur le page du frontend. Cette authentification permet d'autoriser l'accès au site web seulement aux identifiants enregistrés dans la base de données. Les autres n'ont pas la possibilité de s'enregistrer. La partie logique de l'authentification se trouve dans le backend du projet.

Prérequis :
- Avoir une application comportant un backend NodeJS et un frontend Angular

#### 1- Installation des dépendances nécessaires
Dans ce tutoriel, nous utiliserons la bibliothèque ```fuzzer``` pour NodeJS. Pour installer ce module, entrer la commande suivante à la racine du projet NodeJS :
```shell script
npm install fuzzer
```

#### 2- Création d'un fichier de test pour NodeJS
Une fois l'installation effectuée, nous commençons par écrire les tests de fuzzing directement sur le contrôleur d'authentification.<br>
Pour ce faire créer un fichier ```auth.controller.fuzz.spec.ts```.
Tout d'abord, dans ce fichier, nous importons le controller de ```auth``` : ```AuthController```. Ensuite, nous créons une fonction de test qui utilisera ```fuzzer```. <br>
Voici, dans un premier temps, la base du fichier de test :
```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { AuthController } from './auth.controller';

describe('AuthController', () => {
  let controller: AuthController;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      controllers: [AuthController],
    }).compile();

    controller = module.get<AuthController>(AuthController);
  });
});
```

#### 3- Importation de la bibliothèque ```fuzzer```
Dans la fonction de test, nous appelons le module 'fuzzer' à l'aide de la commande suivante :
```typescript
const fuzzer = new Fuzzer();
```
Cette commande doit important automatiquement la bibliothèque en haut du fichier : ```import { Fuzzer } from 'fuzzer';```

#### 4- Utilisation de ```fuzzer``` pour générer des données aléatoires
Pour générer des données aléatoires, nous utilisons la méthode ```mutate.object()``` de la bibliothèque. Cette méthode renvoie une copie modifiée de l'objet en paramètre via incrémentation/décrémentation, suppression de propriété, ...)

Dans notre cas, nous avons un objet ```AuthInput``` contenant les propriétés ```username``` et ```password``` (tous les 2 de type String).
Voici un exemple d'utilisation de la méthode ```mutate.object()``` pour générer 100 authentifications aléatoires :
```typescript
it('should call login function with random input', () => {
    fuzzer.seed(Date.now());

    const authInput: AuthInput = {
      username: 'joe',
      password: "123456"
    }

    for (let i = 0; i < 100; i++) {
      const authInputModified = fuzzer.mutate.object(authInput);
      expect(() => controller.login(authInputModified)).not.toThrow();
    }
  });
```

Afin d'appliquer le fuzz testing, nous devons maintenant passer ```authInputModified``` comme entrée de la fonction ```login()``` de notre controlleur. Puis, nous vérifions que l'appel de la fonction ne lance pas d'erreur à l'aide de l'assertion ```expect(...).not.toThrow()```.

De plus, nous remarquons l'utilisation d'une méthode interne à la bibliothèque fuzzer : ```fuzzer.seed(Date.now())```. Cela permet de définir une graine pour la génération de nombres aléatoires. Dans cet exemple, nous l'avons initialisé selon l'heure courante donc le générateur de nombres aléatoires va dépendre de cette valeur pour générer notre objet. L'objectif est donc de garantir des mutations différentes à chaque exécution puisque la graine sera toujours différente.<br>
L'utilisation d'une même graine à chaque exécution peut être utile pour déboguer les erreurs car il est possible de reproduire les mêmes mutations à chaque itération.

#### 5- Exécution du test fuzzing 
Enfin, nous pouvons tester notre controller ```auth``` afin de vérifier si le test passe ou non :
```
npm run test auth.controller.fuzz.spec.ts
```

Dans ce tutoriel, nous avons présenté l'utilisation du fuzzing test à l'aide de la bibliothèque ```fuzzer``` sur un exemple jouet afin de détecter des erreurs ou des comportements inattendus à partir de données d'entrée potentiellement malveillantes.

#### Difficultés rencontrées
Pour la réalisation de ce tutoriel, plusieurs difficultés ont été rencontrées. En effet, parmi les nombreuses bibliothèques de fuzzing test, pour la plupart, nous avons eu des difficultés à les mettre en place car il n'était pas adapté à notre projet sur lequel on voulait le tester.
Ensuite, une fois la bibliothèque choisie, il a fallu comprendre comment l'utiliser car la documentation proposée prêtait à confusion.

## A/B TESTS
Les tests A/B sont une technique courante utilisée pour évaluer l'impact d'un changement apporté à un produit ou à un service en comparant deux versions différentes auprès d'un public cible.

### Points clés

#### Amélioration de l'expérience utilisateur
Tester différentes variantes d'une fonctionnalité pour déterminer celle qui fonctionne le mieux.

Cela peut aider à améliorer l'expérience utilisateur et augmenter l'engagement des clients.

#### Réduction des risques
Permettent aux équipes DevOps de tester des changements de manière contrôlée avant de les déployer à grande échelle.

Cela peut aider à réduire les risques liés aux bugs, aux erreurs de configuration et aux problèmes de compatibilité.

#### Optimisation de la performance
Tester différents paramètres de configuration pour optimiser les performances du système.

Cela peut aider à améliorer les temps de réponse, à réduire la charge sur les serveurs et à améliorer la fiabilité globale du système.

### Tutoriel
[Cliquez ici pour voir le tutoriel A/B TEST](./documentation/ABTEST/ABTEST.md)

## CANARY RELEASES
Consiste à déployer une nouvelle version d'une application sur une petite partie de l'infrastructure, avant de la déployer sur l'ensemble de l'infrastructure.

Cette technique permet de réduire les risques liés au déploiement de nouvelles versions et d'identifier rapidement les problèmes avant qu'ils n'affectent tous les utilisateurs.

### Points clés

#### Réduction des risques
En déployant d'abord une nouvelle version sur une petite partie de l'infrastructure, il est possible de détecter les problèmes potentiels avant qu'ils ne se propagent à l'ensemble de l'infrastructure.

Cela permet ainsi de réduire les risques de perturbation de l'ensemble du système.

#### Amélioration de la qualité
Identification rapide des problèmes de qualité de la nouvelle version avant qu'ils n'affectent tous les utilisateurs.

Cela permet de corriger rapidement les problèmes et d'améliorer la qualité globale de l'application ou du service.

#### Meilleure visibilité
En surveillant de près les performances de la version, il est possible d'identifier les problèmes potentiels et de mieux comprendre comment la nouvelle version affecte le système dans son ensemble.

### Tutoriel
[Cliquez ici pour voir le tutoriel CANARY RELEASE](./documentation/canary/canary.md)

## Conclusion

Pour conclure, l'utilisation de ces méthodes de test dans le contexte DevOps permet aux développeurs de minimiser les risques de bugs et de vulnérabilités mais aussi d'optimiser les performances de l'application et de garantir une expérience utilisateur de qualité. Ces techniques améliorent la sécurité et la fiabilité de l'application puisqu'elle se réalise avant la phase de déploiement à grande échelle.

## Auteurs
Ce projet a été réalisé par Louis MORLOT--PINTA, Tergel TSAGAAN, Bastien SAUVAT, Bastien FAISANT, Jérôme SCHENBERG et Arthur MORVAN.
