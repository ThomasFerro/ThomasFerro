# Emancipate yourself from Vuex with Vue 3's Composition API

> Is there still room for Vuex with Vue 3?

Before wrapping up this series about *Vue 3* and the *Composition API*, I wanted to show you one last use-case that I found interesting. If you did not already, please take a look at [my introduction of the Composition API](https://dev.to/thomasferro/vue-3-s-composition-api-and-the-segregation-of-concerns-1k5c) so that you won't get lost with the syntax.

This article is especially for the people who have already learned *Vuex* and tend to use it in every place where they manage data.

If you do not fit into this category or do not even know what *Vuex* is, here is a concise introduction.

## One store to rule them all

According to the documentation, *Vuex* is "a state management pattern + library for Vue.js applications". Think of it as the place to store and manipulate reactive data outside of a component before we had the Reactivity and the Composition APIs.

I cannot recommend you enough to watch **Vue Mastery**'s introduction on the subject, available on [the library's home page](https://vuex.vuejs.org/).

To sum up greatly, you can use this library to externalize reactive data shared by components far from each other in the components tree.

Instead of communicating by sending props down the tree and emitting events up, you can use a *Vuex store*. This way, your components are always up to date when one of them modify the store's state.

These two schemas are from [*Vue Mastery*'s introduction](https://www.vuemastery.com/courses/mastering-vuex/intro-to-vuex/). First you can see a complex component tree with a lot of events and props to manage in order to make the two leaf components communicate:

![Complex component tree communication - Vue Mastery](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1578371882429_1.png?alt=media&token=2c411c9f-d5cf-4404-8009-00b73e24a622)

Here we use a Vuex store to simplify this communication:

![Components communication with Vuex - Vue Mastery](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1578371889694_2.png?alt=media&token=f9cc01f0-4644-4867-92ad-17ccd6cc7a6e)

A *Vuex store* is composed of these different parts:

- [`state`](https://vuex.vuejs.org/guide/state.html): The place where your reactive data lives. **You cannot directly modify the state**. To do so, you have to *commit* **mutations**;
- [`getters`](https://vuex.vuejs.org/guide/getters.html): Just like computed properties, this is a way to expose the state in a different shape. They are usually used to avoid rewriting logic inside every component consuming the store;
- [`mutations`](https://vuex.vuejs.org/guide/mutations.html): The only way to modify the state is by **committing mutations**. They should be synchronous and the smallest possible;
- [`actions`](https://vuex.vuejs.org/guide/actions.html): For asynchronous treatments or for logic that implies modifying many elements in the state, we can **dispatch actions**;
- [`modules`](https://vuex.vuejs.org/guide/modules.html): In order to split the state, we can create stand-alone stores, the modules.

The state is represented as a stack of mutations that you can replay or just analyze in-depth:

![Mutations stack shown in Vue Devtools](https://github.com/ThomasFerro/readmes/blob/master/posts/17-replace-vuex-with-composition-api/mutations.png?raw=true)

This is just a theoretical introduction and it is not enough to get started. Please read through [the documentation for more information](https://vuex.vuejs.org/).

### My issue with Vuex

*Vuex* is just like any technology, it comes with a cost. First, the price of learning the library. It usually takes me half a day to only introduce the subject in the trainings that I gave. You can add a couple of days of practicing before actually tame the beast.

Secondly, when using *Vuex*, you tend to lost the concept of **data responsibility**. No component is responsible for the data when everyone can modify the store's state. This way of thinking usually leads to application that are hard to maintain and debug. It is hard to keep track of whom did the mutations and why, even when using great tools such as [Vue Devtools](https://github.com/vuejs/vue-devtools).

When I say "usually", I generalize on purpose. They are applications where *Vuex* is used pragmatically and where the code base is still easily maintainable and scalable. However, I tended to overuse *Vuex* when I first learned it and I think that I am not the only one.

My take is to almost never use *Vuex*, especially when a simple "Parent-Child" communication pattern is enough. You will save yourself from long hours of debugging and headaches.

Where to use it, then? There are use-cases where it comes handy. Or should I say "where it used to came handy", now that we have the Composition API. Let me explain with an example, a sample application with user information displayed and editable.

## User data management using Vuex

I will not be covering Vuex installation in this article, please follow [the documentation](https://vuex.vuejs.org/installation.html) if you need to.

Let us first take a look at what we will be building. We are not creating an entire Web application, it is way out of this article's scope. We will, however, build common pieces that you will certainly encounter if you have not already. Those two pieces are the following:

1. A `Header` component, displaying the user nickname and his profile picture;
2. A `UserSettings` component where the data will be editable.

Here, using *Vuex* is overkill. Just imagine that the rest of the application is ignored, that we have a complex component tree and *Vue Router* installed.

The actual API call will be externalize in the `api.js` file. Just know that it returns a `Promise`, like `fetch` or `axios` would have.

Let us start our implementation with the store's user module:

```js
import { loadUserInfo, saveNewUserInfo } from './api';

const AVAILABLE_STATUS = {
    LOADING: 'LOADING',
    UPDATING: 'UPDATING',
    ERROR: 'ERROR',
};

export const user = {
    namespaced: true,
    state() {
        return {
            nickname: '',
            pictureUrl: '',
            status: '',
        };
    },
    getters: {
        isLoading: state => state.status === AVAILABLE_STATUS.LOADING,
        isUpdating: state => state.status === AVAILABLE_STATUS.UPDATING,
        errorOccurred: state => state.status === AVAILABLE_STATUS.ERROR,
    },
    mutations: {
        changeStatus(state, newStatus) {
            state.status = newStatus;
        },
        changeNickname(state, newNickname) {
            state.nickname = newNickname;
        },
        changePicture(state, newPicture) {
            state.pictureUrl = newPicture;
        },
    },
    actions: {
        // Called by the "App" component to ensure that the initial data are loaded
        load({ commit }) {
            commit('changeStatus', AVAILABLE_STATUS.LOADING);
            loadUserInfo()
                .then(({ nickname, pictureUrl }) => {
                    commit('changeNickname', nickname)
                    commit('changePicture', pictureUrl)
                    commit('changeStatus', '');
                })
                .catch(() => {
                    commit('changeStatus', AVAILABLE_STATUS.ERROR);
                })
        },
        update({ commit }, newUser) {
            commit('changeStatus', AVAILABLE_STATUS.UPDATING);
            saveNewUserInfo(newUser)
                .then(({ nickname, pictureUrl }) => {
                    commit('changeNickname', nickname)
                    commit('changePicture', pictureUrl)
                    commit('changeStatus', '');
                })
                .catch(() => {
                    commit('changeStatus', AVAILABLE_STATUS.ERROR);
                })
        },
    },
};
```

Here we have two important things. First, the state to consume with the nickname and the picture's url. We also have the possibility to modify the profile thanks to the `update` *action*.

A loading status is also managed in the store, which allows components to display the appropriate message to the user.

The header component can now consume the store's data:

```html
<template>
    <header>
        <template v-if="isLoading">
            User information are loading
        </template>
        <template v-else-if="isUpdating">
            User information are updating
        </template>
        <template v-else-if="errorOccurred">
            Unable to manage user information
        </template>
        <template v-else>
            Welcome {{ nickname }}
            <img :src="pictureUrl" alt="User picture" class="user-picture">
        </template>
    </header>
</template>

<script>
import { mapState, mapGetters } from 'vuex';

export default {
    name: 'app-header',
    computed: {
        ...mapState({
            nickname: state => state.user.nickname,
            pictureUrl: state => state.user.pictureUrl,
        }),
        ...mapGetters({
            isLoading: 'user/isLoading',
            isUpdating: 'user/isUpdating',
            errorOccurred: 'user/errorOccurred',
        }),
    },
}
</script>

<style >
.user-picture {
    height: 40px;
    width: 40px;
    border-radius: 50%;
}
</style>
```

Last, the `UserSettings` component will do just the same and will use the action when the user validate his modifications:

```html
<template>
    <form @submit.prevent="updateUser">
        <label>
            Nickname
            <input type="text" v-model="newNickname">
        </label>
        <label>
            Picture url
            <input type="text" v-model="newPicture">
        </label>
        <input type="submit" value="Validate changes" :disabled="formDisabled">
        <p v-if="errorOccurred">An error has occurred while managing user information...</p>
    </form>
</template>

<script>
import { mapState, mapGetters } from 'vuex';

export default {
    name: 'user-settings',
    data() {
        return {
            newNickname: '',
            newPicture: '',
        };
    },
    computed: {
        ...mapState({
            nickname: state => state.user.nickname,
            pictureUrl: state => state.user.pictureUrl,
        }),
        ...mapGetters({
            isLoading: 'user/isLoading',
            isUpdating: 'user/isUpdating',
            errorOccurred: 'user/errorOccurred',
        }),
        formDisabled() {
            return this.isLoading || this.isUpdating
        },
    },
    watch: {
        nickname: {
            handler() {
                this.newNickname = this.nickname;
            },
            immediate: true,
        },
        pictureUrl: {
            handler() {
                this.newPicture = this.pictureUrl;
            },
            immediate: true,
        },
    },
    methods: {
        updateUser() {
            if (!this.formDisabled) {
                this.$store.dispatch('user/update', {
                    nickname: this.newNickname,
                    pictureUrl: this.newPicture,
                })
            }
        },
    },
};
</script>
```

One can say that this solution works and he would be right. However, I see several drawbacks:

- We need to make a component responsible for the initial data load;
- A complete and complex library is needed for what seems to be a simple task.

Will the results be any better with the Composition API? Let us see!

## The same result with the Composition API?

Refactoring this application to use the Composition API should not take too long.

First, we will create the ES module that will replace our store. How can we make a module that share the data between every consumers? We can use the **singleton** design pattern:

```js
import { ref, computed } from "vue";

import { loadUserInfo, saveNewUserInfo } from './api';

const AVAILABLE_STATUS = {
    LOADING: 'LOADING',
    UPDATING: 'UPDATING',
    ERROR: 'ERROR',
};

// These data will only be created once and thus be shared by the consumers
const nickname = ref('');
const pictureUrl = ref('');
const status = ref('');

// Computed properties based on the status
const isLoading = computed(() => status.value === AVAILABLE_STATUS.LOADING);
const isUpdating = computed(() => status.value === AVAILABLE_STATUS.UPDATING);
const errorOccurred = computed(() => status.value === AVAILABLE_STATUS.ERROR);

// No need for mutations anymore, we can simply create JS methods
const apiCallReturnedWithNewUserInformation = ({ nickname: loadedNickname, pictureUrl: loadedPictureUrl }) => {
    nickname.value = loadedNickname;
    pictureUrl.value = loadedPictureUrl;
    status.value = '';
}
const load = () => {
    status.value = AVAILABLE_STATUS.LOADING;
    loadUserInfo()
        .then(apiCallReturnedWithNewUserInformation)
        .catch(() => {
            status.value = AVAILABLE_STATUS.ERROR;
        });
};

const update = (newUser) => {
    status.value = AVAILABLE_STATUS.UPDATING;
    saveNewUserInfo(newUser)
        .then(apiCallReturnedWithNewUserInformation)
        .catch(() => {
            status.value = AVAILABLE_STATUS.ERROR;
        })
};

// Fetch the user info when the module will be used for the first time
load();

// Export a method that returns every needed piece of information
export const useUserManager = () => ({
    load,
    update,
    nickname,
    pictureUrl,
    status,
    isLoading,
    isUpdating,
    errorOccurred,
});
```

Next, we need to change the way we consume data in our components:

```html
<template>
    <header>
        <template v-if="isLoading">
            User information are loading
        </template>
        <template v-else-if="isUpdating">
            User information are updating
        </template>
        <template v-else-if="errorOccurred">
            Unable to manage user information
        </template>
        <template v-else>
            Welcome {{ nickname }}
            <img :src="pictureUrl" alt="User picture" class="user-picture">
        </template>
    </header>
</template>

<script>
import { useUserManager } from './user/userManager';

export default {
    name: 'app-header',
    setup() {
        const userManager = useUserManager();
        return {
            pictureUrl: userManager.pictureUrl,
            nickname: userManager.nickname,
            isLoading: userManager.isLoading,
            isUpdating: userManager.isUpdating,
            errorOccurred: userManager.errorOccurred,
        }
    },
}
</script>

<style >
.user-picture {
    height: 40px;
    width: 40px;
    border-radius: 50%;
}
</style>
```

```html
<template>
    <form @submit.prevent="updateUser">
        <label>
            Nickname
            <input type="text" v-model="newNickname">
        </label>
        <label>
            Picture url
            <input type="text" v-model="newPicture">
        </label>
        <input type="submit" value="Validate changes" :disabled="formDisabled">
        <p v-if="errorOccurred">An error has occurred while managing user information...</p>
    </form>
</template>

<script>
import { ref, computed, watchEffect } from 'vue';
import { useUserManager } from './userManager';
export default {
    name: 'user-settings',
    setup() {
        const newNickname = ref('');
        const newPicture = ref('');

        const userManager = useUserManager();
        const formDisabled = computed(() => {
            return userManager.isLoading.value || userManager.isUpdating.value;
        });

        watchEffect(() => newNickname.value = userManager.nickname.value);
        watchEffect(() => newPicture.value = userManager.pictureUrl.value);

        const updateUser = () => {
            if (!formDisabled.value) {
                userManager.update({
                    nickname: newNickname.value,
                    pictureUrl: newPicture.value,
                });
            }
        }

        return {
            newNickname,
            newPicture,
            pictureUrl: userManager.pictureUrl,
            nickname: userManager.nickname,
            isLoading: userManager.isLoading,
            isUpdating: userManager.isUpdating,
            errorOccurred: userManager.errorOccurred,
            formDisabled,
            updateUser,
        }
    },
};
</script>
```

Finally, you can delete the store's files and remove the dependency from your application!

We now have the same result for the end-user, but our application does not depend on a library other than Vue.

I have to say, however, that this is not a silver bullet. Your application will still be hard to debug and maintain if you put everything into one big module. The Composition API is a tool, a great one, but still nothing more. It can do much more harm than good if used non-pragmatically.

-----

What we built can be seen as - and is, actually - a simple [**State Management Pattern**](https://vuejs.org/v2/guide/state-management.html#Simple-State-Management-from-Scratch). We did leverage the clarity and modularity provided by the Composition API to create what I think is the most developer and user-friendly *State Management Pattern*.

## Something ends... 

What do you think about this State Management Pattern? Will you consider using it in your application?

It was the first time that I write so much about a single subject. I hope that you have learned as much as I did along the way :)

Please send me feedback about the series and your experiences, I will be glad to read from you all!
