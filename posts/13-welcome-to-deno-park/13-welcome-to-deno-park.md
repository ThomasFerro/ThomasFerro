# Welcome.. To Deno Park!

> Tired of hacking Node applications? Introducing Deno

## What? Why? How?

My rule of thumb for effective technology watch is the following: don't loose your time on shiny new things.

Wait for the hype to pass and see if the opinions are still mostly positive after that, once the emotions and feelings are out of the way. With [the release of the 1.0](https://deno.land/v1), I think that it is time to really dig into Deno!

First thing first, what problem does Deno solve? Mostly the same as Node.js. From building CLI's and HTTP API to creating developers' tools.

I have been working with Node.js for more than two years now, and it was far from a cliché love story between us.

There are aspects of this runtime that I like. It is especially a great tool for front-end developers wanting to do some back-end work.

The heavy setup needed to test your code or use a superset of Javascript, I'm not so found of.

You can guess that my curiosity was piqued when I first eared of Deno, and here are the reasons why I was interested in learning it:

### Dependencies hell

Deno dependencies management is quite different from Node.js. It works with a cache on your computer where the third party modules will be stored.

Using a dependency does not require you to install it with a command, but just reference it in an import. Just like importing a module from another local file:

```ts
import { serve } from "https://deno.land/std/http/server.ts";
```

Like the [documentation explains it](https://deno.land/manual/linking_to_external_code#it-seems-unwieldy-to-import-urls-everywhere) and how we will see in the next sections, importing from urls can seem messy and inconsistency prone. Some patterns are already used to manage your dependencies' locations in a unique place.

Assuring the consistency of the dependencies between your development environment and the production is also a huge deal. To do so, you can [generate a lock file](https://deno.land/manual/linking_to_external_code/integrity_checking) that will be used to validate the cache integrity.  Another good practice is to reference a specific version number:

```ts
import { copy } from "https://deno.land/std@0.50.0/fs/copy.ts";
```

Instead of:

```ts
import { copy } from "https://deno.land/std/fs/copy.ts";
```

### Secure by design

Have you ever wonder what could happen if one of the dependencies of your Node.js application's dependencies did something malicious? Sending personal data to who knows where? Reading through your files? I never heard such stories, but it worth considering, **better safe than sorry**!

Deno is secure by default, which means that your application is not allowed to do things like reading through your file system without explicit authorization when running the code. Here is what happens if you try to run an application without the requested permissions:

```bash
deno run index.ts --http
Serving the HTTP API on port 8000
error: Uncaught PermissionDenied: network access to "0.0.0.0:8000", run again with the --allow-net flag
    at unwrapResponse ($deno$/ops/dispatch_json.ts:43:11)
    at Object.sendSync ($deno$/ops/dispatch_json.ts:72:10)
    at Object.listen ($deno$/ops/net.ts:51:10)
    at Object.listen ($deno$/net.ts:155:22)
```

### Types and tests

I have never been a Typescript aficionados, but I admit that it helps you build a safer code base, thanks to the statically typed transcompiler. If you are not allergic either: rejoice! [Deno supports Typescript out of the box.](https://deno.land/manual/getting_started/typescript)

I am, however, a test aficionados and I admit that it helps you build a safer code base, thanks to the safeguard that it provides. If you are not allergic either: rejoice! [Deno supports testing out of the box.](https://deno.land/manual/testing) You will need to write your test using `Deno.test` and if needed using [the assertions library](https://deno.land/manual/testing/assertions).

```ts
// Spoiler alert!!
Deno.test("Initial park with two dinosaurs", () => {
  const initialPark = initiatePark();

  assertEquals(initialPark?.dinosaurs?.length, 2);
});
```

## Building a Jurassic park manager

> [Welcome... to Deno Park!](https://www.youtube.com/watch?v=-w-58hQ9dLk)

I am sure that there are a lot of articles and tutorials about the making of "Hello World"s and "TODO list"s out there. This is fine if you want to get started, but it is not enough for me to make my own opinion about a tool. You can also find basic examples in [the documentation](https://deno.land/manual/examples).

What I want to build to try Deno is something that feels more complete. A project that may not be useful on his own, but who is fun to build and who can show me the strengths and limitations behind the hype.

This project is **Deno Park**, defenitly not a rip off. You will be able to manage your dinosaurs: breeding, feeding and euthanize them if necesarry. Those actions will be available via a CLI and a HTTP API.

Building this project will highlight several common themes of "real world applications", such as writing well tested domain code and building APIs on top of it. The only important part missing is a database connection.

You can find [the final product on Github](https://github.com/ThomasFerro/deno-park) if you are interested.

### Setting up

According to a survey that I just made up, 92% of all side projects follow this pattern:

1. Write down ideas about the project;
2. Spend two days setting it up;
3. Get bored / find something similar on Github / realize that there is little to no added value;
4. Archive and never touch it again.

I don't say that Deno will make you finish project. It won't. But it comes with enough tooling and compatibility options to reduce the setup. *Hooray*, right?

So, what do we need to start a Deno project? A `package.json` file with the dependencies and a description? Dozens of tools, plugins and configuration files? Not exactly. Not at all.

First, we will download and install Deno. I will let you do that following the [*getting started* guide](https://deno.land/manual/getting_started/installation).

Then create a new folder... And we are ready! (We previously saw that a lock file can be used for dependency management, but let us keep this simple for now)

One thing that I really enjoyed while trying Deno is the tooling that comes out of the box. Remember when you needed to spend half a day on tools configuration? Now you only need to spend some time on the documentation!

You want to run all of your tests? `deno test .`

Run the project locally? `deno run index.ts` (if no permission is needed)

Format your code base? `deno fmt`

Bundle your application and his dependencies into a single `js` file? `deno bundle index.ts deno-park.js`

And you can count on the community to create tools for more advanced need like [hot reloading](https://deno.land/x/denon).

*Ok! Great!* I hear you say, *Little to no setup! But what about actual code??* Actual code? Silly you, I will show you something far more valuable than code: tests!

### Red, green, refactor: a mantra for a healthy domain code

This is not an article about *Test Driven Development* - or *TDD* - so I won't be long on the subject. [Just know that it is a set of principles and practices that helps you build better software](https://www.oreilly.com/library/view/test-driven-development/0321146530/).

The main principle is to write the application starting with a failing test, then a naive implementation, and finally do the necessary refactoring while keeping the tests suites passing.

Following TDD principles with Deno feels as smooth and good [as it does with Go](https://medium.com/@t.ferro184/tdd-in-action-debouncer-in-go-b0f7d7d75931). Thanks to the tooling provided out of the box, you can write the test with no additional library to install and setup.

I started this project by listing the features that I wanted:

- Being able to create a new park with two dinosaurs;
- Being able to breed two dinosaurs, with the child being added to the park;
- The dinosaurs loose "hunger points" over time, until starvation;
- The manager can feed and euthanize dinosaurs.

What is the shortest feature to implement here? The initial park!

```ts
Deno.test("Initial park with two dinosaurs", () => {
  const initialPark = initiatePark();

  assertEquals(initialPark?.dinosaurs?.length, 2);
});
```

To answer this request, the minimal solution is to create the `initiatePark` method that returns a park with two dinosaurs. No need to implement anything else for now, the dinosaurs list can be an array of anything.

Then the second test comes in, with the need to breed dinosaurs:

```ts
Deno.test("Breed two dinosaurs", () => {
  let park = initiatePark();

  park = park.breed(0, 1, "Billy");

  assertEquals(park?.dinosaurs?.length, 3);
  assertEquals(park?.dinosaurs[2]?.name, "Billy");
});
```

We add a new `breed` method on the park, taking the dinosaurs to breed and the name of the child.

I choose to return the modified park instead of mutating the initial one. This is an implementation detail, but I like immutability.

Now comes the first edge case, what if the user tries to breed dinosaurs who do not exist? Let us create a test for that:

```ts
Deno.test("Cannot breed with a dinosaur not in the park", () => {
  const park = initiatePark();

  assertThrows(
    () => {
      park.breed(0, 12, "Billy");
    },
    CannotBreedDinosaursNotInPark,
  );

  assertThrows(
    () => {
      park.breed(12, 1, "Billy");
    },
    CannotBreedDinosaursNotInPark,
  );
});
```

And so on until we covered every feature!

### Building a CLI and a HTTP API on top of the domain

We have seen that Deno can help us create solid domain code with its tools, but what about infrastructural code?

First, we can build a CLI on top of the domain code, managing the interactions with the user over the terminal.

To do so, Deno provides what I found to be an aesthetic and practical way to read through the standard input asynchronously:

```ts
import { readLines } from "https://deno.land/std@0.60.0/io/bufio.ts";
for await (const nextLine of readLines(Deno.stdin)) {
  // ...
}
```

You can display information to the user just like with Node.js, using the `console` object:

```ts
console.clear();
console.log("Welcome... to Deno Park!");
```

It also provides more tools in his standard libraries, but I let you read through them on your own!

Using many of those tools, you can build your own CLI! [The one that I built](https://github.com/ThomasFerro/deno-park/blob/master/cli/index.ts) may be a little bit complex to grasp at first so let's break down the most important parts.

The CLI presents to the user the information needed to manage the Park, such as the commands that can be used and the current state of the dinosaurs. This is done in the `updateDisplay` methods, called after every update:

```ts
const updateDisplay = (park: Park) => {
  console.clear();
  console.log("Welcome... to Deno Park!");
  if (park.gameOver) {
    console.log("You have no dinosaur left, game over!");
    return;
  }
  displayDinosaurs(park);
  displayCommands(commands);
};
```

> Welcome... to Deno Park!
> [0] 🙂  - 5
> [1] 🙂  - 5
> Type B <first dino> <second dino> <name> to breed (eg: "B 0 1 Sam" to breed the first two dinosaurs and name the new one "Sam")
> Type E <dino> to euthanize a dinosaur (eg: "E 1" to euthanize the dinosaur at index 1)
> Type F <dino> to feed a dinosaur (eg: "F 2" to feed the dinosaur at index 2)

We also need to set an interval, passing the time on a regular basis and updating the display when it is done:

```ts
  setInterval(() => {
    park = park.passTime();
    updateDisplay(park);
  }, 6000);
```

The user can now enter his command, as shown in the examples. His input will be managed in a loop, checking if the command exists and executing it if so:

```ts
  for await (const command of readLines(Deno.stdin)) {
    let error = null;
    const commandHandler = getCommandHandler(commands, command);
    if (commandHandler) {
      try {
        park = commandHandler(park, command);
      } catch (e) {
        error = e.message;
      }
    }
    updateDisplay(park);
    if (error) {
      console.log("Error:", error);
    }
  }
```

Regarding HTTP API, I first tried to create one only with the standard libraries. You have to manage very low-level concerns and heavy lifting, but you can make it work.

A framework managing those complex and repetitive concerns can be used. In fact, you are probably using one when doing API with Node.js as well. I personally often use [*Express*](https://expressjs.com/) for these use cases.

The Deno ecosystem may be young, but we already have plenty of framework to use for building HTTP API. I tried [*oak*](https://deno.land/x/oak) since it has an API very similar to *Express* and a clear documentation. I am not going to explain how the framework works, you can refer to the documentation for that. However, here are the endpoints that I implemented:

```ts
export const initiateHttp = async (initialPark: Park) => {
  let park = initialPark;
  setInterval(() => {
    park = park.passTime();
  }, 6000);
  const router = new Router();

  router
    .get("/", (context) => {
      context.response.body = {
        ...park,
        gameOver: park.gameOver,
      };
    })
    .post("/feed", (context) => {
      try {
        park = park.feed(Number(helpers.getQuery(context)?.dinosaur));
      } catch (e) {
        context.response.status = 500;
        context.response.body = e.message;
      }
    })
    .post("/euthanize", (context) => {
      try {
        park = park.euthanize(Number(helpers.getQuery(context)?.dinosaur));
      } catch (e) {
        context.response.status = 500;
        context.response.body = e.message;
      }
    })
    .post("/breed", (context) => {
      const dinosaurs = helpers.getQuery(context)?.dinosaurs.split(",").map(
        Number,
      );
      const childName = helpers.getQuery(context)?.name;
      try {
        park = park.breed(dinosaurs[0], dinosaurs[1], childName);
      } catch (e) {
        context.response.status = 500;
        context.response.body = e.message;
      }
    });

  const app = new Application();
  app.use(router.routes());
  app.use(router.allowedMethods());

  const port = 8000;
  console.log("Serving the HTTP API on port", port);
  await app.listen({ port: 8000 });
};
```

As you may have noticed, the domain code is only used as an external source, which provide clear boundaries between concerns.

### To production, and beyond!

Writing software is cool. Delivering it to the users is even cooler, if not mandatory!

Setting up a basic *continuous integration* workflow using *Github Actions* can help us get automatic feedback on every commit. It will ensure that our project is in a stable state and that we can put it in production - more - safely.

There seems to be no official Docker image, so I used the following: [hayd/alpine-deno](https://github.com/hayd/deno-docker).

The workflow is two steps long, running the `deno test command` after checking out the code:

```yaml
name: CI

on: push

jobs:
  test:
    runs-on: ubuntu-latest
    container:
      image: hayd/alpine-deno:1.1.3

    steps:
    - uses: actions/checkout@v2

    - name: Run the tests
      run: deno test .
```

You can put your application in production using this Docker image too, I recommend that you take a look at [LogRocket's article](https://blog.logrocket.com/how-to-deploy-deno-applications-to-production/) on the subject.

## Wrapping up

I think you could tell, I really enjoyed trying Deno. All of what use to overwhelm me when doing Node.js projects is gone, and I can focus on what matters most: building application.

I am confident enough in it to make it a candidate for future professional pieces of software when the opportunity will come.

I am now eager to see if it will have the same popular success than Node.js, and to see if it keeps his promises with bigger, more complex projects in production!
