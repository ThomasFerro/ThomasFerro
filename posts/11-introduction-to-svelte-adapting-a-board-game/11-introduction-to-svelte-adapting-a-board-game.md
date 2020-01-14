# Introduction to Svelte - Adapting a board game

> Or how being trollish almost made me skip a gem.

## Yet another state of JS

![Extracted from the 2019 survey of "State of JS - Front-End frameworks": https://2019.stateofjs.com/front-end-frameworks/](https://github.com/ThomasFerro/readmes/blob/master/posts/11-introduction-to-svelte-adapting-a-board-game/state-of-js.png)

> Extracted from the 2019 survey of ["State of JS - Front-End frameworks"](https://2019.stateofjs.com/front-end-frameworks/)

So, you want to build a Web application? You have been searching for the best tool to do the job, reading articles and tech radars? The landscape has not change that much since 2019, with three behemoths dominating the market. 

React, Angular and Vue, three frameworks (and associated cults) with quite a similar promise : helping us build reactive application, while taking full advantage of the components approach.

I will not be able to talk much about Angular and React, since I have chosen to focus on Vue for the last three years. I bet that you will be able to do the analogy of what I will say with the framework of your heart.

It is common knowledge that Vue.js is the one of the three with the cheapest entry cost. I agreed with this statement for a long time, preaching for my parish as we developers do. Yet, as time goes by, I found myself being more critical about the Vue approach.

I have been teaching Vue for about a year now, and every time I do, I lost some of the students with all of the concepts. The issue is not that there are too many of them, but everyone comes with caveat, exception and sometimes different syntax. It is very confusing for people trying to start using the framework.

Those "issues" can be described as the entry cost of Vue.js. Yet, the more I think about those pain points, the more I feel that we developers pay the price of the technical complexity that should be abstracted by the framework. When learning Vue you will eventually get over it, but why should we?

Please note that I am talking about Vue 2.x. The Composition API might be the solution, but I have yet to try it.

I do not know if the same can be said about the other two, please let me know your point of view on this matter!

## The native way

Are native Web Components a thing? I would love to believe that. No framework shipped to the user, only a standard way of creating, exposing and using components!

But let's be honest, we are far from it. Far from building our complex applications with no framework. The current state of the art is to ship lots and lots of dependencies for our application to run. The more Javascript the better!

The standard way is stillborn. Or is it?

## What about Svelte?

"Svelte, you say? Another contender? What is it going to be, two months or three until we all forget about it?" These were my first thoughts when I heard about Svelte.

"Cybernetically enhanced web apps? Was there no other marketing nonsense available?" These were my thoughts when I landed on the tool's website.

"Wait, that's actually a great set of promises!" These were my thoughts when I finished reading the main page. 

"Wow, it must be the best tutorial I have ever seen!" These were my thoughts when [I finished reading the tutorials](https://twitter.com/Thomas_Ferro_md/status/1210242504085377025).

Remember my laments about the current state of JS? About native Web Components? Turns out I might have been missing the tool that meets my expectations.

Svelte is not like the golden three frameworks. Instead of doing the heavy lifting in the browser like React, Vue and Angular, Svelte does it in **compile time**. Instead of shipping the framework to the user, we only ship native Web Components.

The only concern that I have is with compatibility. What about browser that does not support Web Components APIs? (I have yet to double-check on this)

## Before we start, what is Armadora

Todo lists applications are ok. They are certainly more valuable than Hello Worlds for learning, but I much prefer a bigger project. I know that games are usually created using 2d or 3d drawing libraries, but I thought that a HTML game would be just fine for my objective : to adapt a board game as a Web application.

Armadöra is a game designed by **Christwart Conrad**, helped by artist **Tony Rochon**.

Here is the description of the game, as stated in the game's guide:

> True to their reputation, Dwarfs amassed a large amount of gold. Armadöra thus became a highly coveted land for Elves, Orcs, Goblins and Mages who are all now assembling their respective forces. Let the siege of Armadöra begin!
>
> Position your troops, raise the barricades, but make no mistake about it, a single error may undermine your strategy!

I would like to thank *Blackrock Games*, the game's editor, for allowing me to use their property for educational purpose.

## How to start?

Before starting any project of your own, I cannot recommend you enough to read and play through the tutorials. They are well written, straight to the point and it does not require too much time. It took me a couple of hours, but I think that it was thanks to my Vue.js background.

I will describe every interesting parts of the making of the game in order to introduce you to Svelte's beauties.

The first part might be the least interesting, the setup. Between you and me, I don't like setups. It's coarse and rough and irritating and it gets everywhere. Not like with Svelte's setup tool. 

The Vue ecosystem makes a great job at abstracting the setup and the configuration with Vue CLI. I was not expecting to find a similar tool with Svelte, but I was wrong (well, for projet initialization at least).

One way to bootstrap a Svelte project is by using the `degit` package, who provides application templates for you to use. I did not use the tool for anything else yet, but the job I asked it to do was well done. Isn't what a tool is supposed to do?

![Default project](https://github.com/ThomasFerro/readmes/blob/master/posts/11-introduction-to-svelte-adapting-a-board-game/default-project.png)

Simply install the dependencies (`npm install`) then run the `npm run dev` command and you are up and running!

## Svelte 101

Now that we have a working application, let me introduce `.svelte` files. Similar to `.vue` files in guess what ecosystem, those files allow you to describe a component that can be used anywhere you like.

Well, *describe* might not be the best choice of word. A description of a component is what you do when using the Vue syntax for instance. 

!["Hello world" example from Vue documentation](https://github.com/ThomasFerro/readmes/blob/master/posts/11-introduction-to-svelte-adapting-a-board-game/vue-hello-world.png)

With Svelte, there is no heavy descriptive syntax to use. You want a data named `applicationName`? How would you do it with plain old Javascript? You are right, with a *variable*.

```html
<script>
  const applicationName = 'Armadora' 
</script>
```

That is all for me, thanks for reading, bye bye! 

Jokes aside, it feels really good not to learn a heavy syntax for such simple tasks. There will be some to learn, but not for simple purposes like that.

Now, you want to display the variable value in the browser? Go on and interpolate it in the template! 

```html
<h1>{applicationName}</h1>
```

Any code in the curly braces is Javascript ran in the context of your component. You can write anything you want, even trying to do math in JS: `{0.1 + 0.2}`.

Going back to our example, in the App.svelte component (for now), if we want to display the number of palisades left, it will look something like this :

```html
<script>
  let palisadesLeft = 35
</script>

<main>
  Palisades left: {palisadesLeft}
</main>
```

![Results of the example above](https://github.com/ThomasFerro/readmes/blob/master/posts/11-introduction-to-svelte-adapting-a-board-game/palisades-left.png)

## Actions and dynamic attributes

You can argue that what we have done so far is nothing more than a static page made with way more tools than necessary. You will be right. You impatient reader.

To make the magic happen, let us add some interactions.

The game's implementation is going to be way more complex, but for know we are going to say that for a player to put a palisade, he just have to click a button. Such fun, I know.

You can, in vanilla Javascript, listen for a DOM event in an element. For instance, listening to a click on a button will look like this:

```js
document.querySelector('button').addEventListener('click', methodCalledOnClick)
```

This syntax is simplified with Svelte:

```html
<button on:click={methodCalledOnClick}>Click me if you dare!</button>
```

Of course, every [DOM event](https://developer.mozilla.org/en-US/docs/Web/Events) can be listened for.

Before applying this concept, remember what I said about the variables? That simply declaring them makes them available in the HTML template? Well, the same goes for the functions!

Decrement our palisade count will therefore be done this way:

```html
<script>
  let palisadesLeft = 35

  const putPalisade = () => palisadesLeft--
</script>

<main>
  Palisades left: {palisadesLeft}
  <button on:click={putPalisade}>Put a palisade</button>
</main>
```

![Results of the example above](https://github.com/ThomasFerro/readmes/blob/master/posts/11-introduction-to-svelte-adapting-a-board-game/put-palisade.png)

"Wait a minute", I hear you say, "We can go below zero and have minus 20 palisades left ! We trusted you Thomas !"

Baby steps ! Now that we have a button, we can make the `putPalisade` method a bit smarter, and we can also introduce a new concept: dynamic attributes.

Remember when we interpolate JS variable into the template as text to be displayed? Well, we can do just that with HTML attributs to !

You want to dynamically disable a button, based on a JS variable? Let's do it !

```html
<button disabled={shouldTheButtonBeDisabled}>My awesomely pointless button</button>
```

In our palisades example, it will look something like this:

```html
<script>
  let palisadesLeft = 35
  let hasPalisadesLeft = true

  const putPalisade = () => {
    if (palisadesLeft > 0) {
        palisadesLeft--
    }
    hasPalisadesLeft = palisadesLeft > 0
  }
</script>

<main>
  Palisades left: {palisadesLeft}
  <button on:click={putPalisade} disabled={!hasPalisadesLeft}>Put a palisade</button>
</main>
```

Any HTML attribute can be made dynamic with this technique.

If your hair stood up while seeing the `hasPalisadesLeft` variable, hold your horses, we will discuss this just now !


## To reactivity and beyond !

This `hasPalisadesLeft` business smells. Having to manage multiple data for something that clearly could be computed (excuse my Vue) seems like a lot of work.

Is there any way to have a variable that is automatically recomputed when any of the dependency changes? Of course it is, I would have swept it under the carpet otherwise.

I have to say, the syntax of this feature bugs me a little, but here is how we do it using *labels*. In this example, the fullname will be recomputed whenever the firstname or lastname changes:

```html
<script>
    let firstname = 'Thomas'
    let lastname = 'Ferro'

    $: fullname = `${firstname} ${lastname}`
</script>

<span>{fullname}</span>
```

We can go even further and creating a whole function that will be called again if any dependency changes:

```html
<script>
    let firstname = 'Thomas'
    let lastname = 'Ferro'

    $: {
        console.log(firstname, lastname);
    }
</script>
```

What does any of these reactivity magic have to do with our `hasPalisadesLeft`? Well, as I said earlier, this data is simply another representation of the `palisadesLeft` count. So we can make a reactive data instead of setting it manually:

```html
<script>
  let palisadesLeft = 35

  $: hasPalisadesLeft = palisadesLeft > 0

  const putPalisade = () => {
    if (palisadesLeft > 0) {
        palisadesLeft--
    }
  }
</script>

<main>
  Palisades left: {palisadesLeft}
  <button on:click={putPalisade} disabled={!hasPalisadesLeft}>Put a palisade</button>
</main>
```

## Conditional rendering and looping

One common need in templating is the ability to render parts of HTML only if a predicate condition is met. One will achieve that in Svelte with the `#if`, `:else if` and `:else` keywords.

```html
<script>
    const displayContent = true
</script>

{#if displayContent}
<h1>My content !</h1>
{:else}
The content cannot be displayed.
{/if}
```

As the tutorial says:

> A `#` character always indicates a block opening tag. A `/` character always indicates a block closing tag. A `:` character, as in `{:else}`, indicates a block continuation tag. Don't worry — you've already learned almost all the syntax Svelte adds to HTML.

Where to put that in practice in our application? Pretty much everywhere. Let us say that we have to display the winner at the end of the game, a big old `if` can do the trick !

```html
<script>
    const winner = 'Orcs'
	// [...]
</script>

{#if !winner}
[The game board and controls...]
{:else}
<section class="winner-display">
	{winner} wins !
</section>
{/if}
```

Another thing you can do is to repeat some part of the template based on data. You will do that with the `each` keyword.

```html
<script>
  const blogPosts = [
		'My first article',
		'Why my first article was dumb',
		'Is my second article to trust?',
		'Lesson learned - or why will I never blog again',
	]
</script>

<main>
	<h1>My blog !</h1>
    {#each blogPosts as post}
	<article>
		<h2>{post}</h2>
	</article>
	{/each}
</main>
```

You can put whatever you need inside the loop, and you also have access to the index as the second argument.

Note: There is a notion of `keyed each blocks` that I will let you [discover afterwards](https://svelte.dev/tutorial/keyed-each-blocks).

Displaying every player can be a good example of use-case:

```html
<script>
    const players = [
        {
            character: 'Orc',
            nickname: 'README.md'
        },
        {
            character: 'Elf',
            nickname: 'Javadoc'
        },
        {
            character: 'Mage',
            nickname: 'noobmaster69'
        },
    ]
</script>

{#each players as player}
<section class="player">
	{player.nickname} playing {player.character}
</section>
{/each}
```

Here, `player` can be destructured to directly access the attributes:

```html
<script>
    const players = [
        {
            character: 'Orc',
            nickname: 'README.md'
        },
        {
            character: 'Elf',
            nickname: 'Javadoc'
        },
        {
            character: 'Mage',
            nickname: 'noobmaster69'
        },
    ]
</script>

{#each players as { nickname, character }}
<section class="player">
	{nickname} playing {character}
</section>
{/each}
```


## More bindings

Remember the time when creating a form and fetching the data from it in Javascript was obnoxious? All the events to listen for, all the data to maintain up-to-date... Those days are long gone now with the help of **inputs bindings** !

You can see those binding as a way to connect a JS variable to a HTML input. When updating the value, the field will be updated and vice versa.

This feature can be used with many types of inputs, from the simple text to a `select` with multiple values. In any of those cases, you will use the `bind:value` syntax as follow:

```html
<script>
	let name = 'world';
</script>

<input type="text" bind:value={name}>

<h1>Hello {name}!</h1>
```

I will not go into much details here since all I can do is paraphrase the [documentation or the tutorial](https://svelte.dev/tutorial/text-inputs).

What I will do, however, is to continue implementing the game ! Input bindings can be useful with "New player form", a basic form with a name and the chosen character:

```html
<script>
	let nickname = ""
	let character = undefined
	
	let availableCharacters = [
		'Orc',
		'Elf',
		'Mage',
		'Gobelin'
	]
</script>

<ul>
	<li>Nickname: {nickname}</li>
	<li>Character: {character}</li>
</ul>

<form>
	<label>
		Nickname
		<input type="text" bind:value={nickname}>
	</label>
	<label>
		Character
		<select bind:value={character}>
			{#each availableCharacters as availableCharacter}
			<option value={availableCharacter}>{availableCharacter}</option>
			{/each}
		</select>
	</label>
</form>
```

## The elephant in the room - What about components?

For maintainability, testability and whateverotherwordsthatendswithbility, we developers tend to split our applications in small modules. Many years of Clean Code made us think that for a module to be efficient, we need it to be *small*, *self-contained* and *reusable* if possible. These principle can also be applied in front-end development via the use of **components**.

Luckily enough, we already have played with components since the beginning of our Svelte course ! In fact, every `.svelte` file represent a component.

We will go through the process of *creating a new component*, to *importing and using it* and then we will see how we can manage *communication between components*.

To create a new component, go ahead and create a new `MyFirstComponent.svelte` file next to the `App.svelte` (for now, we will clean up our rooms later). In that new file, you can apply every principle that we have seen so far ! We can for instance have internal data, bind those data in the template, etc.

```html
<!-- MyFirstComponent.svelte -->
<script>
  const title = "My first component !!!!"
</script>

<article>
	{title}
</article>
```

Great, now we have a component ! Next step is to import and use it. Let us say that we want this new component to be displayed in `App.svelte`. The first thing to do is to import the component, just like a JS module:

```html
<!-- App.svelte -->
<script>
	import MyFirstComponent from './MyFirstComponent.svelte'
</script>
```

We can now use the component in the template, **just like any other HTML tag** !

```html
<!-- App.svelte -->
<script>
	import MyFirstComponent from './MyFirstComponent.svelte'
</script>

<MyFirstComponent></MyFirstComponent>
```

For a more concrete example, let us extract the display of a player in a `Player.svelte` component:

```html
<!-- Player.svelte -->
<section class="player">
	A player...
</section>
```

Oh.. Seems like we need to receive data about a player in order to display them in this component.. How could we manage to do that?

This is the first *parent-child communication* tool that we will see: **Props** are used to communicate from the parent to a child. A props is nothing more than a variable, it can be a string, a number or any complex object. Here is how to declare a props:

```html
<!-- Player.svelte -->
<script>
    export let character
    export let nickname
</script>
<section class="player">
    <!-- Props can be used just like any internal data -->
    {nickname} playing {character}
</section>
```

As implied in the name, you need to provide data from the parent to effectively have a "*parent-child communication*".

You can do so by adding attributes when calling the custom component. Yes, just like any native HTML attribute:

```html
<!-- App.svelte -->
<script>
	import Player from './Player.svelte'
</script>

<Player character="Orc" nickname="README.md"></Player>
```

Here, I set the value of the attributes manually, but nothing prevent us from binding values ! The reactivity black-magic will do the trick and any modification in the data of the parent will be reported in the children.

```html
<!-- App.svelte -->
<script>
    import Player from './Player.svelte'

    const players = [
        {
            character: 'Orc',
            nickname: 'README.md'
        },
        {
            character: 'Elf',
            nickname: 'Javadoc'
        },
        {
            character: 'Mage',
            nickname: 'noobmaster69'
        },
    ]
</script>

{#each players as player}
<Player character={player.character} nickname={player.nickname}></Player>
{/each}
```

We have ourselves a working one-way data flow ! Try to code a form to add a player and see what happen !

... Wait... "one-way data flow"? What about the other way? There is another way?

We only have cover the parent to children communication, but there are use-cases when a component needs to talk to his parent. How to do that without breaking the golden rules of components? How could a component be *self-contained* if he needs to know whom his parent is? You can achieve that by dispatching events.

The mantra for *parent-child communication* - and it applies to Vue.js too - is the following: **Props down / Events up**.

The first thing you will need to do is to create an *event dispatcher*, using Svelte's exposed methods:

```html
<!-- Child.svelte -->
<script>
    import { createEventDispatcher } from 'svelte';
    
    const dispatcher = createEventDispatcher();
</script>
```

Then, when you want to dispatch an event, call the dispatcher with the name of the event as the first argument.

```html
<!-- Child.svelte -->
<script>
    import { createEventDispatcher } from 'svelte';
    
    const dispatcher = createEventDispatcher();

    const sendEvent = () => {
        dispatcher('my-custom-event')
    }
</script>

<button on:click={sendEvent}>Send event !</button>
```

You can also send data through the event with the second argument. Those data are often referred to as *the payload* and can be of any type you need.

Note: The payload can be retrieve in the `detail` attribute in the actual event.

```html
<!-- Child.svelte -->
<script>
    import { createEventDispatcher } from 'svelte';
    
    const dispatcher = createEventDispatcher();

    const sendEvent = () => {
        dispatcher('my-custom-event', {
            name: 'Thomas',
        })
    }
</script>

<button on:click={sendEvent}>Send event !</button>
```

Let us say that we are creating a form, which can take the default name and age as *props*. The form is **not responsible** for the data, it is not his duty to modify them. When the form is complete, the child component will send an event, notifying his parent that the job is done:

```html
<!-- ClientForm.svelte -->
<script>
  import { createEventDispatcher } from 'svelte';
    
  const dispatcher = createEventDispatcher();
	
	export let name = undefined
	export let age = undefined
	
	let newName = undefined
	let newAge = undefined
	
	$: newName = name
	$: newAge = age
	
	const validate = () => {
		dispatcher('validate', {
			name: newName,
			age: newAge,
		})
	}
</script>

<input type="text" bind:value={newName}>
<input type="number" bind:value={newAge}>
<button on:click={validate}>
	Validate
</button>
```

```html
<!-- App.svelte -->
<script>
	import ClientForm from './ClientForm.svelte';
	let name = 'Thomas'
	let age = 24
	
	const updateData = (payload) => {
		name = payload.name
		age = payload.age
	}
</script>

<ul>
	<li>Name: {name}</li>
	<li>Age: {age}</li>
</ul>

<!-- Here we need to fetch the 'detail' attribute from the event -->
<ClientForm name={name} age={age} on:validate={(event) => updateData(event.detail)}></ClientForm>
```

## Another example: building a grid

Before we wrap up, let me show you another example with the game's grid.

This *grid* is composed of *cells* in which you can click to put a warrior.

The cell can either contain a *warrior*, *gold* or it can be empty. We will abstract that by saying that it receive a *content* and we will be simply displaying the content. When clicking on a cell, the component simply propagate the event via the shortcut `on:click`.

```html
<!-- Cell.svelte -->
<script>
	export let content;
</script>

<button on:click>
	{content}
</button>
```

The grid component receive the state of each cell from his parent, via the `state` attribute which is a two dimensional array. It manages the click on a cell by emitting an event for his parent to listen.

```html
<!-- Grid.svelte -->
<script>
	import { createEventDispatcher } from 'svelte';
	import Cell from './Cell.svelte';
	
	export let state;
	
	const dispatch = createEventDispatcher();
	
	const cellClicked = (rowIndex, colIndex) => {
		dispatch('cell-selected', {
			row: rowIndex,
			col: colIndex,
		})
	}
</script>

<article class="grid">
	{#each state as row, rowIndex}
	<section class="row">
		{#each row as cell, colIndex}
			<Cell
			    content={cell}
			    on:click={() => cellClicked(rowIndex, colIndex)}
			></Cell>
		{/each}
	</section>
	{/each}
</article>
```

The root component `App.svelte` is responsible for the actual data, changing the state of the grid when it receives an event from the grid. It is not the responsibility of the grid nor the cell to change the state.

```html
<!-- App.svelte -->
<script>
	import Grid from './Grid.svelte';
	
	const state = [
		[ 'land', 'land', 'land', 'land', 'land', 'land', 'land', 'land' ],
		[ 'land', 'land', 'land', 'land', 'land', 'land', 'land', 'land' ],
		[ 'land', 'land', 'land', 'land', 'land', 'land', 'land', 'land' ],
		[ 'land', 'land', 'land', 'land', 'land', 'land', 'land', 'land' ],
		[ 'land', 'land', 'land', 'land', 'land', 'land', 'land', 'land' ],
	]
	
	const cellSelected = (payload) => {
        // Change the state based on the payload, the current player and the chosen action
		state[payload.row][payload.col] = 'warrior'
	}
</script>

<Grid state={state} on:cell-selected={(e) => cellSelected(e.detail)}></Grid>
```

This is a simplified example of how those concepts can all fit together in a real application. The game's rules are not implemented to avoid noise, the point here is to show you parent-child communication in action.

Now that you have the building blocks of an application, all you have to do is build your own ! Try to build this game or any other, or even a more traditional Web application :)

## Disclaimers

The components and examples from this article are for studying purpose only. Building the actual game would require some kind of server with the game data to which every player will connect and interact with. However, I hope that it gives you a good idea of what you can do with Svelte !

Please refer to the [game's repository](https://github.com/ThomasFerro/armadora) if you want to take a look at the current state of the game.

I purposely did not discussed what I consider to be advanced topics, such as *class binding*, *transitions*, *lifecycle* or *slots*. Please read through the documentation and build some projects on your own to fully discover the tool !

## Wrapping up !

You may have noticed, I am really getting into Svelte. The promises are in line with my own opinions about Javascript frameworks and the Web at large, and it feels just good writing applications with this tool.

I might not recommend it for every use-case yet, especially not knowing how it behave on old browsers. However, it is definitely worth learning if you feel tired of the Reacts, the Angulars and the Vues or just if you want to get into front-end development !
