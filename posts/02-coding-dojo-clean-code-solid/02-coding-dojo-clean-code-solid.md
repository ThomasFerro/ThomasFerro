# Coding Dojo - Clean Code - SOLID

> An introduction to Clean Code / Architecture in disguise.

A Coding Dojo based on the universe of Metal Gear Solid. Five exercises will be presented to introduce the principles behind the SOLID acronym.

This article is a concatenation of all of the Coding Dojo parts. You can follow the dojo by reading through the post or via the [GitHub repository](https://github.com/ThomasFerro/dojo-clean-code-solid).

## Introduction

**Transmission received...**

```
Snake, we need your help.

Our agent management system has been compromised and we are trying to rebuild another one in Node.js.

The system is used to keep track of all of our agents and their missions. No big deal for a clean coder like you, am I right ?

We are almost done, but something is missing, like if we were building the project without thinking of the maintainability, flexibility and understandability...

Your goal is to find the project's flaws and eliminate them. Our first analysis has shown that there is one of them per principle behind the SOLID acronym, whatever it means.
```

Follow the instructions and try to complete the exercises so you will have a proper introduction to the Clean Code / Clean Architecture as instructed by Uncle Bob !

For each section, you will have to checkout into the git branch with a flaw to fix. There is no perfect solution to those problems, try your best to clean the code but do not spend to much time scratching your head.

A branch with a solution is provided for every exercise, feel free to compare it with your own !

I highly recommend you to read "Clean Architecture: A Craftsman's Guide to Software Structure and Design" by Robert C. Martin after you completed the dojo. It contains a deeper dive into the SOLID principles and many more.

## Dojo

### Single Responsibility Principle

> "A class should have only one reason to change." Robert C. Martin.

Think of the last time you tried to debug that particular method that was responsible of :

1. Checking if there is enough coffee and water left;
2. Prepare the coffee if the first condition is fulfilled;
3. Pour the coffee and the milk if necessary;
4. Add sugar; and
5. Notify the user that his coffee is ready !

All of that in the `CoffeeService` !

Now, imagine a world where all the coffee business is managed by the `CoffeeService`, the sugar business by the `SugarService`, and so on..

Wouldn't it be nice ? Easily testable and debuggable ?

This is the philosophy behind the **Single Responsibility Principle**, every part of the code should be responsible for one thing and it should be very talented at doing it !

**Transmission received...**

```
Snake, we have found your first target.

This is a common case of a junk room class ! Clean it by making sure that the responsibilities are at the right place.
```

#### Exercise

Checkout into the `exercise-1` branch.

Try to clean up the `addMission` in the `MissionsService` by splitting the responsabilities in different classes.

```
    public async addMission(mission: Mission): Promise<boolean> {
        if (!mission
            || !mission.getId() ||
            !isAgentValid(mission.getAgent()) ||
            !mission.getStartDate()
        ) {
            throw new InvalidMission(mission);
        }

        if (getMissionWithinPeriod(
            await this.getAgentMissions(mission.getAgent().getId()),
            mission.getStartDate(),
            mission.getEndDate(),
        )) {
            throw new MissionConflict(mission);
        }

        return await this.missionsRepository.add(mission);
    }
```

You will find a solution to this exercise in the `exercise-1-solution` branch.

#### Solution

In order to simplify the `MissionsService`, I have created a class responsible for the policies linked to a mission.

Here, this `MissionPolicies` class exposes two pure methods: `isMissionValid` and `hasAlreadyAMissionWithinThisPeriod`.

```
export const isMissionValid = (mission: Mission): boolean => {
    return !!(mission && mission.getId() && isAgentValid(mission.getAgent()) && mission.getStartDate());
};

export const hasAlreadyAMissionWithinThisPeriod = (
    agentMissions: Mission[],
    newMission: Mission,
): boolean => {
    return getMissionWithinPeriod(
        agentMissions,
        newMission.getStartDate(),
        newMission.getEndDate(),
    ) !== undefined;
};
```

The `MissionsService` can then focus on his primary responsability, and simply call the policies when needed.

```
    public async addMission(mission: Mission): Promise<boolean> {
        if (!isMissionValid(mission)) {
            throw new InvalidMission(mission);
        }

        if (hasAlreadyAMissionWithinThisPeriod(
            await this.getAgentMissions(mission.getAgent().getId()),
            mission,
        )) {
            throw new MissionConflict(mission);
        }

        return await this.missionsRepository.add(mission);
    }
```

### Open / Closed Principle

> "A class is closed, since it may be compiled, stored in a library, baselined, and used by client classes. But it is also open, since any new class may use it as parent, adding new features. When a descendant class is defined, there is no need to change the original or to disturb its clients." Bertrand Meyer.

This is one of the most important principle, if you want to be able to make your application grow.

To paraphrase Bertrand Meyer, the goal is to build an application when a new feature will take the form of a new class, a new method or a new module, while leaving the rest of the application untouched.

To apply this principle efficiently, you must already have a modular application, respecting the **Single Responsability Principle**.

To put it in context, let's take back our **CoffeeService** example. What if we want to be able to serve tea as well ?

One could rush the making of this feature by adding a parameter with the type of drink and surround the treatment in a big and dirty `if`... But not us !

We can, for instance, make a new `TeaService` with the specific treatment, and make both this service and the coffee one implements the `DrinkService`, providing the necessary API. 

```
Well done with the previous mess !

Now, if we want to be able to add more specificities to the project, we will need to rethink our classes strategy.
```

#### Exercise

Checkout into the `exercise-2` branch.

We have add a new feature, the missions with backup. These are just an extension of the previous missions, but with a list of agents that can be used as backup.

Unfortunately, we rushed the feature and now with have a messy `MissionsService`.

Try to clean it by putting all of the `BackedMission` specifics in an extension of the `MissionsService`.

```
export class MissionsService {
    private missionsRepository: InMemoryMissionsRepository;
    private backedMissionsRepository: BackedMissionsRepository;

    constructor(missionsRepository: InMemoryMissionsRepository, backedMissionsRepository: BackedMissionsRepository) {
        this.missionsRepository = missionsRepository;
        this.backedMissionsRepository = backedMissionsRepository;
    }

    // [...]

    public async getAllBackedMissions(): Promise<Mission[]> {
        return await this.backedMissionsRepository.findAll();
    }

    public async getAgentBackedMissions(agentId: string): Promise<Mission[]> {
        return await this.backedMissionsRepository.findByAgent(agentId);
    }

    public async getBackedMissionInformation(missionId: string): Promise<BackedMission | undefined> {
        return await this.backedMissionsRepository.findById(missionId);
    }

    public async removeBackupFromMission(mission: BackedMission, backupId: string): Promise<boolean> {
        if (!isAgentInBackup(backupId, mission)) {
            return false;
        }

        return this.backedMissionsRepository.removeBackup(mission.getId(), backupId);
    }
}
```

You will find a solution to this exercise in the `exercise-2-solution` branch.

#### Solution

To solve this issue and make the extension of the application more straightforward, I have added a `BackedMissionsService` that extends the `MissionsService`.

This new class is responsible for the providing an API used to manage specifically the missions with backup. This way, backward compatibility is kept clean.

```
export class BackedMissionsService extends MissionsService {
    private backedMissionsRepository: BackedMissionsRepository;

    constructor(backedMissionsRepository: BackedMissionsRepository) {
        super(backedMissionsRepository);

        this.backedMissionsRepository = backedMissionsRepository;
    }

    public async getBackedMissionInformation(missionId: string): Promise<BackedMission | undefined> {
        return await this.backedMissionsRepository.findById(missionId);
    }

    public async removeBackupFromMission(mission: BackedMission, backupId: string): Promise<boolean> {
        if (!isAgentInBackup(backupId, mission)) {
            return false;
        }

        return this.backedMissionsRepository.removeBackup(mission.getId(), backupId);
    }
}
```

However, I found this way of keeping the classes open redundant. To make this new feature, I had to add :

- A new set of helpers;
- A new repository;
- A new implementation of this repository.

### Liskov Substitution Principle

> "Let φ(x) be a property provable about objects x of type T. Then φ(y) should be true for objects y of type S where S is a subtype of T." Barbara Liskov.

> "Functions that use pointers of references to base classes must be able to use objects of derived classes without knowing it." Robert C. Martin.

This is, in my opinion, the most complicated of the five principles.

Try to think of it this way : 

1. You should never strengthened preconditions in a subtype. Meaning that if you extends the `CoffeeService` class with a `pourCoffee` method, this method in your subclass should accept the same arguments as the original.
2. You should never weakened postconditions in a subtype. Meaning that if the method in the parent class have a condition that throws an error, the same error should be thrown in every subclasses.

This way, there is no need to do specific validation when calling the `pourCoffee` method, whether it is from the `CoffeeService` itself one or his subclasses.

```
Still with us ? We need you more than ever with this issue.

The project act oddly depending on the implementation, no matter how precise the API is..

Spot the weak substitution and clear it, so we can move on.
```

#### Exercise

Checkout into the `exercise-3` branch.

The team has decided to add a new condition for creating a mission with backup.

This condition is that there must be at least one of those backups.

The first implementation seems to fulfill this requirement, adding a `BackedMission` with no backup throws an `InvalidMission` exception.

```
export class BackedMissionsService extends MissionsService {
    private backedMissionsRepository: BackedMissionsRepository;

    constructor(backedMissionsRepository: BackedMissionsRepository) {
        super(backedMissionsRepository);

        this.backedMissionsRepository = backedMissionsRepository;
    }

    public async addMission(mission: BackedMission): Promise<boolean> {
        if (!hasBackup(mission)) {
            throw new InvalidMission(mission);
        }

        return super.addMission(mission);
    }
    // [...]
}
```

You may have noticed the violation here : The same entity doest not behave the same while being added in the `MissionsService` or in the `BackedMissionsService`. Thus, we cannot substitue those classes, even if `BackedMissionsService` extends `MissionsService`.

Try to find a way to fix this violation but do not wait too much before reading the solution, and please share with me your solution or your feeling about mine.

You will find a solution to this exercise in the `exercise-3-solution` branch.

#### Solution

I could not find a suitable solution for this specific problem without letting the `MissionsService` know about the `BackedMission`.

Knowing this, and the fact that the new domain rule (no backed mission without backup) is specific to the `BackedMissionsService`, I have decided to create a whole new method, `addBackedMission`, that checks this rule.

```
    public async addBackedMission(mission: BackedMission): Promise<boolean> {
        if (!hasBackup(mission)) {
            throw new InvalidMission(mission);
        }

        return super.addMission(mission);
    }
```

### Interface Segregation Principle

> "The interface-segregation principle (ISP) states that no client should be forced to depend on methods it does not use." Robert C. Martin.

> "ISP splits interfaces that are very large into smaller and more specific ones so that clients will only have to know about the methods that are of interest to them. Such shrunken interfaces are also called role interfaces." Wikipedia.

Just like the *Liskov Substitution Principle*, the **Interface Segregation Principle** helps for a more maintainable application.

To stay with the coffee example, imagine that you have a `DrinkService` that expose three method : 

- `fillWater`
- `pourDrink`
- `doMaintenance`

The **Interface Segregation Principle** says that you should segregate these operation in *role interfaces*. So you will have three interfaces that all uses the `DrinkService` for a different purpose :

- `WaterFillerService` with a `fillWater` method exposed
- `DrinkPourerService` with a `pourDrink` method exposed
- `MaintenanceService` with a `doMaintenance` method exposed

This way, a `WaterFillerService`'s client will not depend on any of the maintenance business.

```
The previous mission was the most difficult, keep on cleaning !

A lot of integrators expressed their frustration about our API. They keep telling us that there is way too much useless things to implement, even unnecessary methods...
```

#### Exercise 

Checkout into the `exercise-4` branch.

Our engineers are working on a new feature, the possibility to manage the information of the missions' locations.

The data will be fetched from a `Repository`, but they need to be able to delete elements from this repository.

To do so, they have started to add a new `delete` method.

```
export interface Repository<T, K> {
    add(element: T): Promise<boolean>;
    findAll(): Promise<T[]>;
    findById(id: K): Promise<T | undefined>;
    delete(id: k): Promise<boolean>;
}
```

As you can imagine, the compiler indicates a lot of error based on this new addition, exemple :

```
Property 'delete' is missing in type 'InMemoryMissionsRepository' but required in type 'MissionsRepository'
```

It might be due to the fact that the interface holds to many methods !

Split this interface to best suit the new needs of the team.

You will find a solution to this exercise in the `exercise-4-solution` branch.

#### Solution

I have splitted the `Repository` into three interfaces. This way, every implementation can choose which action they want to provide, depending on their specifications :

```
export interface Add<T> {
    add(element: T): Promise<boolean>;
}

export interface FindAll<T> {
    findAll(): Promise<T[]>;
}

export interface FindById<T, K> {
    findById(id: K): Promise<T | undefined>;
}
```

And now, the team can simple add a new `delete` interface and make their repository implements what they need !

### Dependency Inversion Principle

> "High level modules should not depend upon low level modules. Both should depend upon abstractions." Robert C. Martin.

> "Abstractions should not depend upon details. Details should depend upon abstraction." Robert C. Martin.

This may be the least expansive principle to apply.

If you are not already convinced that you should always depends on abstraction (abstract classes or interfaces), try to think of the last time you have committed to this specific database, let us say *MongoDB*.

You build your entire application with explicit references to this provider, using the technology as it's full potential.

Then, right before the shipping, your boss tells you that a contract has been signed with Microsoft to use *Microsoft SQL Server*, and that you **have to** use this database !

Now you have to search for every *MongoDB* reference in the codebase, replacing it with the new provider... And, of course, you have used specific features that is not available in the new provider !

This is a common case where the **Dependency Inversion Principle** can save you hours, even days !

In the previous example, instead of referencing *MongoDB*, an abstract **Repository** would have been used, later implemented with the chosen provider. This way, changing a provider can be simply done by adding a new implementation, and configure the application to use this one instead.

```
One last thing to greatly improve our project and you will be free to go.

The team wants to be able to try numerous technical choices before deciding which one to use in production.

To do so, we need to rethink our application to remove any explicit references !
```

#### Exercise 

Checkout into the `exercise-5` branch.

The `AgentsService` make explicit reference to an implementation of the `AgentsRepository`. Abstract the used repository.

```
export class AgentsService {
    private agentsRepository: InMemoryAgentsRepository;

    constructor(agentsRepository: InMemoryAgentsRepository) {
        this.agentsRepository = agentsRepository;
    }
    // [...]
}
```

You will find a solution to this exercise in the `exercise-5-solution` branch.

#### Solution

This exercise was not quite elaborate, it was just a means to talk to you about the principle.

You should always depends on abstraction, even if you are doing a """disposable service""" (spoiler: it will never be disposed).

In this case, the `AgentsService` should depend on an abstract `AgentsRepository`.

The only thing that the service should know is that the repository can perform various actions (here: fetching and adding agents) :

```
export class AgentsService {
    private agentsRepository: AgentsRepository;

    constructor(agentsRepository: AgentsRepository) {
        this.agentsRepository = agentsRepository;
    }
    // [...]
}
```

## Conclusion

```
Great job Snake, you have managed to tremendously clear the project !
```

I hope that you have enjoyed the trip and that you will apply all of those principles in your work !

Don't hesitate to notice me if you have any observation about the dojo, I will do my best to answer quickly :) 
