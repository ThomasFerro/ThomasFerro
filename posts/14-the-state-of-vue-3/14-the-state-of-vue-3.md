# The state of Vue - Will the 3.0 still be approachable?

> Easy to get started with, but how hard to master?

It should not be long before we get our hands on Vue.js next major release! I cannot wait - and actually have not waited - to try the new features to build faster and cleaner applications thanks to the great work of Evan You and the community.

However, the pasts months had me wonder about the global reception of those new features, especially the Composition API. I sure am excited and ready to use it, but is everyone on the same page?

Let me try to explain why I am concerned about the future of the framework if it continues down that path.

## Approachable... at first

I remember more than four years ago, when I joined my first company and heard about Vue for the first time. Having tried only Angular.js in my years of study, I was anxious, not knowing if I could learn a completely new approach relatively fast.

Fast forward a few days later, I felt like I was up and running! The entry cost coming from Angular.js appeared quite cheep,. Everything seemed easier, more obvious.

When I say up and running, I did not mean that I knew every part of the framework and the ecosystem. I knew enough, however, to start contributing on the company's projects. The learning phase never stopped, as you may imagine. Every new feature to implement or bug to fix was an opportunity to teach myself good components and applications building practices.

## Digration about training

For now more than a year, I have been lucky enough to train people into Vue.js. The main target are developers already having a solid experience in Javascript so that we can focus on the framework. My goal is to provide them with the keys to build professional applications using this specific tool.

I was confident enough in my technical skills and knowledge, but not in my teaching skills. How to build a compelling and complete training? One that supports students up to a sufficient level of mastery?

With the help of a soft-skills coach and the support of my technical director, a first draft of the training was ready to be tested on the field.

Not long after, I was teaching Vue.js based on that first draft and I had a feeling that something was off. I gave the maximum amount of keys - knowledge about what the framework can do and how - and it seemed to connect with them. However, when asked to build a feature with no technical details, like in our day to day conversation with product owners and business experts, few where the ones who could provide a working solution.

What is it that makes Vue.js so hard to condensed in three days? *Condensed*... It may be it. Instead of *condensing* every feature into the training and risking to overwhelm people with non-essential information, why not *distill* those information through real world examples?

That is what I tried to apply in the latest version of the training, and it seems to work for most people. Of course, everyone has is own way of learning and there is no silver bullet, but I am convinced that it is a better way of teaching.

This training experience and the discovery of another framework, Svelte, had me wonder about my issues with Vue.js. It is certainly approachable at first, but what it is that confused me when learning Vue and seems to confuse those who want to get started?

## A hundred ways to achieve the same goal

Vue.js provides different ways to define *props* that your component accepts. For instance, here are three valid ways to tell that your component can receive an array of students:

```js
// 1. The simpliest solution, without any check

{
  props: [ 'students' ],
}

// 2. Type checked props

{
  props: {
    students: Array,
  },
};

// 3. Type checked props with fallback

{
  props: {
    students: {
      type: Array,
      // Another itch, the default value needs to be
      // inside a method for arrays and objects
      default() { return []; },
    },
  },
};
```

You have the same behaviour with the definition of *computed properties* and *watchers*.

This expandable API allows you to write less code if you do not need the extra options, which is nice... when you already know all the syntaxes.

When learning Vue, however, it tends to add a significant amount to the already big cognitive load.

As you may imagine, this is not going to get any better with the addition of the whole new Composition API. Newcomers will now be exposed with a whole new way or writing components logic **plus** the classical Options API.

I know that there is no silver bullet, that we have to choose between providing shorthands for developers or force everyone to use the expanded, more verbose version. I still think, however, that the following questions deserve to be asked:

Do we really *need* all of those options? Where do the convenience end and the pragmatism begin?

## Reactivity and references caveats in the hands on the developer

You may have seen that the Vue instance's `data` is often described as a function returning the actual data. Do you know why? According to [the documentation](https://vuejs.org/v2/guide/components.html#data-Must-Be-a-Function), you must do that so "[...]each instance can maintain an independent copy of the returned data object".

It sure sounds to me like a Javascript issue invading the framework's API, but I can get it. The thing that bugs me is that you *can* describe the data directly as an object to if you won't reuse the component.

Two syntaxes because of a references issue, bummer.

Going into the reactivity field, have you ever wonder why linters and compilers tend to get mad at you for not adding a `key` attribute when using the `v-for` directive? It is ["to maintain predictable behavior, such as object constancy in animations."](https://vuejs.org/v2/style-guide/#Keyed-v-for-essential) Again, I cannot help but think that this is not something we should manage as users.

I get that it could add complexity into the framework. Yet, I found it better to have technical complexities managed in the framework and leave the applications as clear as possible from them.

The same goes with the `deep` option in watchers, which allows you to not only watch the changes in the reference to an array or an object but also the changes in the array's elements or the object's properties. This last one, however, can be discussed since it can be useful to only listen to changes in the variable's reference.

Shouldn't those concerns be managed by the framework by default? Are we not using a framework to ignore those concerns in the first place? Should the developer know about implementation details in order to use a framework or library?

## Share your thoughts and opinion!

I know that I am just exposing points without bringing anything to the table. The goal is only to provide a context that we can use to build a constructive debate. I will dive deeper into my thoughts in the last article of this series.

What are your thoughts on the subject?

How did you first learn the basics of Vue.js?

Have you felt overwhelmed at first by the API's options?

Are you concerned about the addition of more APIs?

I will be glad to hear from you about it. On the next articles, we will actually discuss about Vue 3's Composition API and try to make the most out of it! 
