# Vue 3's Composition API and the segregation of concerns

> A novelty worth digging

I always think that it is best to put things in context. That is why I wrote [a first article](https://dev.to/thomasferro/the-state-of-vue-will-the-3-0-still-be-approachable-4mgc) about my take on the state of Vue before the 3.0, which is currently available as a release candidate.

However, the main subject of this series is one of Vue 3's new feature: the Composition API. It is the one that I anticipate the most and it is finally time we discuss it!

## Shiny new toys

This article should be the shortest of the series, since the subject has already been discussed [many](https://auth0.com/blog/getting-started-with-vue-3-composition-api/) [times](https://composition-api.vuejs.org/) by people way more interesting and smart than I am.

The Composition API was created to answer two common issues that occur when your Vue application starts to get big.

### Code organization

Have you ever had to maintain really big components, with complex logic implying a lot of `data`, `computed`, `methods`, etc? When trying to read those types of components, the main issue is to keep track of what everything is doing and how do they interact with each other. With the current Options API, you have to navigate back and forth inside the Vue instance, which causes a great cognitive load.

Vue 3 tries to resolve this issue by adding a new method to the Vue instance, the [`setup`](https://composition-api.vuejs.org/api.html#setup). This method can be seen as **the component's entry point**, being called before the `beforeCreated` hook and receiving the `props` as an argument. The **returned value is an object containing every available information for the template to use**.

This is the place where you will write all of your component logic, no matter if we are talking about `data`, `computed`, `watcher`, etc.

There is still a missing piece to this puzzle, how do we write `data`, `computed`, `watcher`, `methods` and more in that new `setup` method?

Vue 3 **provides a new tool to create those reactive data and more: the Reactivity API**.

It may not be relevant for now, but here is a small example of how to create a reactive data using the Reactivity API:

```js
import { ref } from 'vue';

const count = ref(0);

// Accessing ref's value in JS
console.log('Count:', count.value)

// Modifying the value
count.value += 1
```

As you can see, you have to explicitly access the `ref`'s value when manipulating it in JS. It bugs me a little, but you won't have to do so in the template and can directly access the value, as we will see later.

Please take a look at the [Reactivity API's reference](https://composition-api.vuejs.org/api.html#reactivity-apis) for more information about what is contained in it.

Ok, but how do those two APIs fit together? Let us see that with an example. First, we will write it using the Options API, then we will do it *à la Vue 3*.

Let us say that we have a component managing the loading and display of blog posts, a minimalist version of it could be like this:

```js
export default {
    name: 'blog-posts',
    data() {
        return {
            posts: [],
            loadingStatus: '',
            error: '',
        };
    },
    computed: {
        blogPostsLoading() {
            return this.loadingStatus === 'loading';
        },
        blogPostsLoadingError() {
            return this.loadingStatus === 'error';
        },
    },
    methods: {
        loadBlogPosts() {
            this.loadingStatus = 'loading';
            fetch(process.env.VUE_APP_POSTS_URL)
                .then((response) => {
                    if (!response.ok) {
                        throw new Error(response.status);
                    }
                    return reponse.json();
                })
                .then((posts) => {
                    this.posts = posts;
                    this.loadingStatus = 'loaded';
                })
                .catch((error) => {
                    this.error = error;
                    this.loadingStatus = 'error';
                });
        },
    },
    created() {
        this.loadBlogPosts();
    },
}
```

Using the new provided tools, we can put all of the logic in the `setup`:

```js
import { ref, computed } from 'vue';

export default {
    name: 'blog-posts',
    setup() {
        const loadingStatus = ref('');
        const error = ref('');
        const posts = ref([]);

        const blogPostsLoading = computed(() => {
            return loadingStatus.value === 'loading';
        });
        const blogPostsLoadingError = computed(() => {
            return loadingStatus.value === 'error';
        });

        const loadBlogPosts = () => {
            loadingStatus.value = 'loading';
            fetch(process.env.VUE_APP_POSTS_URL)
                .then((response) => {
                    if (!response.ok) {
                        throw new Error(response.status);
                    }
                    return reponse.json();
                })
                .then((fetchedPosts) => {
                    posts.value = fetchedPosts;
                    loadingStatus.value = 'loaded';
                })
                .catch((apiError) => {
                    error.value = apiError;
                    loadingStatus.value = 'error';
                });
        };

        // Return every information to be use by the template
        return {
            loadingStatus,
            // You can rename those information if needed
            loadingError: error,
            loadBlogPosts,
            blogPostsLoading,
            blogPostsLoadingError,
        };
    },
};
```

It can seem not so useful in a component with few logic, but it already helps developers keep track of the different pieces without scrolling between the Vue instance's options. We will see later in this article and the next ones how to leverage the most out of it.

We can also extract the logic by creating an ES module (`posts.js`) managing the data and exposing the useful information:

```js
import { ref, computed } from 'vue';

export const useBlogPosts = () => {
    const loadingStatus = ref('');
    const error = ref('');
    const posts = ref([]);

    const blogPostsLoading = computed(() => {
        return loadingStatus.value === 'loading';
    });
    const blogPostsLoadingError = computed(() => {
        return loadingStatus.value === 'error';
    });

    const loadBlogPosts = () => {
        loadingStatus.value = 'loading';
        fetch(process.env.VUE_APP_POSTS_URL)
            .then((response) => {
                if (!response.ok) {
                    throw new Error(response.status);
                }
                return reponse.json();
            })
            .then((fetchedPosts) => {
                posts.value = fetchedPosts;
                loadingStatus.value = 'loaded';
            })
            .catch((apiError) => {
                error.value = apiError;
                loadingStatus.value = 'error';
            });
    };
    
    // Return every information to be use by the consumer (here, the template) 
    return {
        loadingStatus,
        // You can rename those information if needed
        loadingError: error,
        loadBlogPosts,
        blogPostsLoading,
        blogPostsLoadingError,
    };
}
```

Our component will then only manage the template based on data provided by the module. Complete separation of concerns:

```js
import { useBlogPosts } from './posts.js';

export default {
    name: 'blog-posts',
    setup() {
        const blogPostsInformation = useBlogPosts();
        return {
            loadingStatus: blogPostsInformation.loadingStatus,
            loadingError: blogPostsInformation.loadingError,
            loadBlogPosts: blogPostsInformation.loadBlogPosts,
            blogPostsLoading: blogPostsInformation.blogPostsLoading,
            blogPostsLoadingError: blogPostsInformation.blogPostsLoadingError,
        };
    },
};
```

Again, it helps clarify your code and decouple the intent from the implementation, which is always nice.

You may already have thought about it, but this way of creating modules can help us reuse logic!

### Logic reuse

We already have some tools that help us create logic to be used by many components. [Mixins](https://vuejs.org/v2/guide/mixins.html), for instance, let write you any Vue instance's options to be injected in one or many components.

These approaches share one downside, they lack clarity.

You never clearly know which mixin imported which option unless you read through all of them. It can easily become a nightmare for developers trying to understand how the components work, having to navigate through the Vue instance and the globally and locally injected mixins. Moreover, mixins' options can collide with each others, leading to a knot bag, not to say a mess.

With the Composition API, any component can pick what he needs from different modules. Clearly stated in the `setup` method, the developers can see what was taken from where and even rename variable if it helps understand the intent better.

I think that clarity is a, if not *the*, most important concern when writing applications to be actively maintain for years. The Composition API gives us the tool to do so in an elegant and practical way.

### Wait, there is more?

The two main goals look pretty achieved to me, but the Composition API should not be reduced to those two concerns.

It will also **benefit the testability of our applications**, let me explain how.

Before Vue 3 we had to instantiate the component, inject mocked dependencies if needed, and only then start hacking around assertions. This way of testing **can lead to tests suites strongly coupled to the actual implementation**. The kind of tests that age poorly and can do way more harm than good. 

Now we can create ES modules encapsulating the domain logic and exporting the data and methods to be used. Those modules will be written in almost-pure Javascript since they will still use Vue's API, but not in the context of a component.

Our tests can simply consume the exported information, just like our components will!

## The art of teaching

You may have noticed by the fact that I am writing a whole series about it, I am really excited about this new API. It scratches itches that I've had for a long time, trying to apply clean code principles into my front-end applications. I think it will help us tremendously increase the quality of our components and applications if well used.

However, the Composition API is an advanced concept. I do not think that it should replace the actual way of writing components. Moreover, we will still encounter legacy code written before Vue 3, so our previous knowledge is still useful.

I already discuss this issue in the previous article, but it is really important to keep this in mind: not everyone is lucky enough to discover the 3.0 after two years of practicing Vue almost daily.

Some people will start using Vue with the 3.0, and a whole new API like this one adds a lot to the already big entry cost. A newcomer now have to understand it in addition to the "classical" Options API.

**How do you think that the new API should be introduced to newcomers?** I personally think that it should be just like Vuex or Vue Router, introduced later as an advanced tool. It should be added on top of a solid foundations of knowledge and used practically.

## Once again, share your thoughts!

What do you think about the new Composition API?

Are you ready to use it as soon as the 3.0 is released?

Please let everyone know and let us discuss all that :)

Now that the subject is theoretically introduced, what is next? I will get my hands dirty and try the make the most out of the Composition API, starting in the next article with a refactoring session!
