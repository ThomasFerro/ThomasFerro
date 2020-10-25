# Internationalize your Svelte app with reactive stores

> You either pick a i18n library or live long enough to write your own

I must confess something to you, and I hope that you will forgive me for it: I am French 🇫🇷🥖🧀🍷🐸🇫🇷.

I tend to write my applications in English in order to make them accessible to the greatest number of developers around the world. However, I sometime forget about the people closest to me.

I recently asked my family to beta test an [adaptation of a board game](https://dev.to/thomasferro/introduction-to-svelte-adapting-a-board-game-emc) that I am building and the first feedback that I got was my girlfriend asking me "*pourquoi c'est en anglais ?*" ("*why is your game in English?*").

Fair point. I immediately added an issue on this subject and start thinking about how to internationalize the application. Oh, and for those who wonder what **i18n** means, it is a commonly used abbreviation for *internationalization*, where the eighteen characters between the *i* and the *n* are represented by the *18*.

I already worked on that subject in my early days of Vue.js developer, using [Kazupon's `Vue I18n` library](https://kazupon.github.io/vue-i18n/). I loved the ease of use of the API, simply calling a `$t` method with the translation key in order to make everything work.

I wanted to find a Svelte alternative with the same appeal, but then think that it could make a great use case to learn to use a tool that I never practice before: the stores.

Again, I do not encourage you to build a new solution from scratch for every technical subject, especially when there is already [well maintained alternatives](https://github.com/kaisermann/svelte-i18n). However, for now, let us learn how to use Svelte's stores to build a reactive internationalization mechanism 😁

## Building the i18n mechanism

Practicing *Test Driven Development* has taught me something that I love to use when I have the opportunity, always starts by building the API that fits you the most. Only starts working on implementation details once the intention is clear.

Let us do just that and imagine the API of our dreams, starting with how to ask the system for a specific translation.

I like the idea of calling a simple method, directly from the template or from the `<script>`, something like this:

```svelte
<script>
  import { i18n } from './i18n';

  // A localized message injected in the script
  $: authenticationErrorMessage = i18n('authenticationError')
</script>

<!-- A localized message directly called from the template -->
<h1>{i18n('welcomeMessage')}</h1>

<p>{authenticationErrorMessage}</p>
```

Obviously we will need a method to change the current locale, a method that will hopefully be able to change automatically every translation in the page without a refresh (spoiler alert: it will).

```svelte
<script>
  import { i18n, changeLocale } from 'path/to/i18n';
</script>

<button on:click={() => changeLocale('fr')}>{i18n('switchToFrench')}</button>
<button on:click={() => changeLocale('en')}>{i18n('switchToEnglish')}</button>
```

We could use *JSON* objects to manage the translations, one file per locale for instance:

```json
{
  "welcomeMessage": "Welcome!",
  "authenticationError": "Unable to authenticate...",
  "switchToFrench": "Switch to french",
  "switchToEnglish": "Switch to english"
}
```

Having already worked in large scoped projects, I know that the number of labels can grow pretty fast. It would be nice if we could allow for the use of nested objects.

```svelte
<h1>{i18n('home.welcomeMessage')}</h1>

<!-- With the JSON formatted like this: 
{
  "home": {
    "welcomeMessage": "Welcome!"
  }
}
 -->
```

Knowing our expected behaviour, it seems that we need a reactive mechanism accessible from any component in our application. We can manage this by using a global store, but how to implement it in Svelte? Heck, what is a global store?

### Read the fantastic manual!

Leaving the Svelte world for a paragraph or two, a store can be seen as a way to manage reactive data outside of a component. It is especially useful when a lot of components share logic for a given matter.

Take the connected user management for instance. You may have one component managing the authentication process, another one responsible for the display of the connected user information, another one who takes care of editing the profile, etc. They all play with the same data and they need to be informed when this piece of data changes to update themselves accordingly.

This is where you could be tempted to create a `user` store. I am too, so let us create it!

Svelte provide us with a module for creating stores. We can create:

- [*readable stores*](https://svelte.dev/docs#readable): See them as read-only stores. I have no use case for them by now, but they must be useful since they are available 🤷‍♀️
- [*writable stores*](https://svelte.dev/docs#writable): "Classical" stores, offering us ways to subscribe and unsubscribe to the data's changes and methods to actually modify the data.
- [*derived stores*](https://svelte.dev/docs#derived): A store based on other stores. We will see a specific use case for our i18n mechanism.

Here is a minimalist `user` store:

```js
import { writable } from 'svelte/store';

export const user = writable({});
```

I warned you, it is minimalist. Here is how you can consume and change this store's data:

```svelte
<script>
  import { user } from 'path/to/user/store'
  
  let username 
  user.subscribe(newUserInformation => {
    username = newUserInformation.name
  });

  // Can be called when typing the name in an input for instance
  user.set({ name: 'Thomas Ferro' });
</script>

<h1>Welcome {username}!</h1>
```

Subscribing to a store can seem like a lot of busy work with this method. Svelte also provide a way to subscribe with a shortcut, prefixing your store name with `$`:

```svelte
<script>
  import { user } from 'path/to/user/store'
</script>

<h1>Welcome {$user && $user.name}!</h1>
```

The complete API can be found, as always, in the [documentation](https://svelte.dev/docs#svelte_store).

Here is one extra feature that I enjoy a lot: any object with a correctly implemented `.subscribe` and `.unsubscribe` and optionally `.set` methods can be considered as a store by Svelte. Kudo for being able to create a framework-agnostic module.

However, for the sake of simplicity and brevity, we will use the provided methods to create our stores. 

### Finally building something

We know what we want to build, we know how we are going to build it... Time to code!

The first thing we want is a store with the labels for the current locale. We can manage this by creating a *writable store* with the labels and a method changing this store's data according to the new locale:

```js
import { derived, writable } from 'svelte/store';
import enLabels from './en.json';
import frLabels from './fr.json';

const labelsStore = writable(enLabels);

export const EN_LOCALE = "en";
export const FR_LOCALE = "fr";
export let currentLocale = EN_LOCALE;

export const changeLocale = (newLocale) => {
    if (newLocale === EN_LOCALE) {
        labelsStore.set(enLabels)
        currentLocale = newLocale
    } else if (newLocale === FR_LOCALE) {
        labelsStore.set(frLabels)
        currentLocale = newLocale
    }
}
```

One could use these exposed method and constants to make a local switcher:

```svelte
<script>
  import { changeLocale, EN_LOCALE, FR_LOCALE } from './i18n';
</script>

<button on:click={() => changeLocale(FR_LOCALE)}>🇫🇷</button>
<button on:click={() => changeLocale(EN_LOCALE)}>🇬🇧</button>
```

As explained in the description of the targeted API, I do not want the developers to directly access the `labelsStore`. Instead, I want them to use an exposed method and provide a translation key.

How can we expose this store in a way that fits our expected API? Using a **derived store**! This derived store will be called `i18n` and will not return directly an object with the labels, but a function that takes the translation key as an argument and return the label:

```js
import { derived, writable } from 'svelte/store';

// [...] labelsStore implementation

export const i18n = derived(labelsStore, (labelsForCurrentLocale) => {
    return key => labelsForCurrentLocale[key]
})
```

This way, when the `labels` *store* is updated, the `i18n` derived store is notified and update itself too, making the components who depends on it refresh their templates.

We now need to manage the nested objects. We can extract this logic and use it directly in the method returned by the `i18n` store:

```js
import { derived, writable } from 'svelte/store';
import enLabels from './en.json';
import frLabels from './fr.json';

const labelsStore = writable(enLabels);

const OBJECT_PROPERTY_SEPARATOR = "."

const crawlLabelsToFindRequestedTranslation = (currentLabels, translationKey) => {
    const pathToFollowInLabels = translationKey.split(OBJECT_PROPERTY_SEPARATOR)
    let currentPositionInLabels = currentLabels
    for (let i = 0; i < pathToFollowInLabels.length; i++) {
        currentPositionInLabels = currentPositionInLabels[pathToFollowInLabels[i]]
        if (!currentPositionInLabels) {
            return translationKey
        }
    }
    return currentPositionInLabels
}

export const i18n = derived(labelsStore, (labelsForCurrentLocale) => {
    return (translationKey) => {
        if (!translationKey.includes(OBJECT_PROPERTY_SEPARATOR)) {
            return labelsForCurrentLocale[translationKey] || translationKey
        }
        return crawlLabelsToFindRequestedTranslation(labelsForCurrentLocale, translationKey)
    }
})
```

There we go, our i18n is fully implemented, let us use it in a component 😃 

```svelte
<script>
  import { i18n } from './i18n';

  // A localized message injected in the script
  $: authenticationErrorMessage = $i18n('authenticationError')
</script>

<!-- A localized message directly called from the template -->
<h1>{$i18n('welcomeMessage')}</h1>

<p>{authenticationErrorMessage}</p>
```

Notice the slight difference in the usage, we need to prefix the store's call with a `$` to directly access the value and for this value to be reactive. See the [documentation](https://svelte.dev/docs#4_Prefix_stores_with_$_to_access_their_values) for more details on that matter.

## Possible next steps

I do not think that I will continue to work specifically on the i18n mechanism since it already covers everything I needed in my application.

However, they are some possible improvements and new features.

I think that it could be great to manage the pluralization and the translation with parameters. For instance, when a translation takes a parameter that won't go in the same place for different languages.

A dynamic local management could add value too, so the core of the mechanism won't change when adding new managed language.

And of course, one could think that this mechanism could be a standalone library 😬

## A last word

I learned a lot while building this i18n mechanism and writing this article. I think that it is the best kind of learning, picking a specific subject only when you actually need it. I do not have the time not the will to go through the entire Svelte documentation and make a project that mixes all of the framework's features.

I hope that you discovered something too! 

Localization is a common need for a lot of applications. I think that it would be a blast to have the communities of every front-end frameworks work together on a framework-agnostic reactive internationalization mechanism, don't you? 😃
