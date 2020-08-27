# Composition API v Renderless Components - Let's use Vue 3's features to clean our components!

> Iteratively and incrementally improve components quality

Making the perfect component in the first draft is impossible. Impossible because you do not know exactly what will be needed before actually building the component. Impossible also because you will always learn new, more effective ways of doing things.

Too many times have I overengineered, trying to make components that no one would ever need to modify or fix, components that was meant to meet all present and future use cases.

Let me tell you the best place that I found for those components: in a post-mortem.

*Why is he telling me all that*, you may ask yourself. I wanted to introduce this article with this little digression to discuss the importance of iterations.

First, build the minimum viable product, the basic implementation that works and does what is expected. Only then starts the refinement phase to ship a clean and maintainable code.

You do not know if you are building the right thing until you try it. Ship it fast and get feedback.

It is a concept you should be familiar with [when doing TDD](https://medium.com/@t.ferro184/tdd-in-action-debouncer-in-go-b0f7d7d75931) or even if you recognize yourself in the Agile values.

This article follows the same pattern, we will start with a component that works, even though it is far from being maintainable. Then we will incrementally improve it, without the new Composition API in the first place so we will be able to compare with previously existing tools.

I will not discuss the mostly important matter of tests in this article. The reason being that I am not confident enough on the subject of Front-End testing to give you my opinion. I might dig into the subject in the future, but for now I leave you with a few resources:

- [Vue Test Utils - the official unit testing utility library for Vue.js](https://vue-test-utils.vuejs.org/);
- [Testing Library - a collection of utilities that encourage "good" testing practices](https://testing-library.com/). I haven't tried it yet, but the promise is good enough for me to share it with you.

## The legacy component

Before starting any refactoring, we need to understand what we are working with.

We will be creating a TODO-list with only a few features:

- Listing the tasks to do;
- Creating a new task;
- Tag a task as finishing.

The first thing we want to do is to make the application work, so let us do it!

```html
<template>
    <h1>My TODO list! ({{ doneCount }} / {{ totalCount }})</h1>

    <!-- Error management -->
    <p v-if="loadingError">
        {{ loadingError }}
        <button @click="loadTodos">Reload</button>
    </p>

    <ul v-else>
        <li v-for="todo in todoList" :key="todo.id">
            {{ todo.content }}
            <button @click="achieveATodo(todo.id)">Validate</button>
        </li>
    </ul>
    <form @submit.prevent="() => addTodo(newTodoContent)">
        <label>
            What do you have to do?
            <input v-model="newTodoContent">
        </label>
        <input type="submit" value="Create">
        <!-- Error management -->
        <p v-if="todoCreationError">{{ todoCreationError }}</p>
    </form>
</template>

<script>
export default {
    name: 'todo-list',
    data() {
        return {
            loadingError: '',
            todoList: [ ],
            newTodoContent: '',
            todoCreationError: '',
        };
    },
    computed: {
        doneCount() {
            return this.todoList.filter(todo => todo.done).length;
        },
        totalCount() {
            return this.todoList.length;
        },
    },
    methods: {
        loadTodos() {
            this.loadingError = '';
            fetch(import.meta.env.VITE_TODOS_URL)
                .then((response) => {
                    if (!response.ok) {
                        throw new Error('An error has occurred while loading todos');
                    }
                    return response.json();
                })
                .then((todos) => {
                    this.todoList = todos;
                })
                .catch((error) => {
                    this.loadingError = error;
                });
        },
        achieveATodo(id) {
            // [...] Call the API to achieve the task
        },
        addTodo(content) {
            this.todoCreationError = '';
            fetch(import.meta.env.VITE_TODOS_URL, {
                method: 'post',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ content })
            })
                .then((response) => {
                    if (!response.ok) {
                        throw new Error('An error has occurred while creating todo');
                    }
                    return response.json();
                })
                .then((newTodo) => {
                    this.todoList = [
                        ...this.todoList,
                        newTodo,
                    ]
                })
                .catch((error) => {
                    this.todoCreationError = error;
                });
        }
    },
    created() {
        this.loadTodos();
    },
}
</script>
```

Great, we have a working component. The client is happy since he can try the application even though it is a *work in progress*. Now that we know that his expectation are met, it is time to meet ours.

Listing the pros and cons of the component can be useful in this case. It will allow us to have a complete picture of the state of the component and we will be able to prioritize the tasks to be done.

On the bright side, the component works. He has every needed features and we managed to build it relatively fast. On the other hand, we have a lot to improve before shipping it:

- It has too many responsibilities (data fetching, business rules applying to the data, actions to add and modify data, the display of all of those information);
- Nothing here is reusable;
- It seems hard to maintain, at least it is hard to read through (a hundred lines, without the task achievement logic).

Let us iterate a few times until we are happy about the result!

## Before Vue 3, a first step toward clean components

This is kind of a bonus section where we will refactor the component using present tools. You can skip it if you are only interested in the Composition API or if you are not familiar with the advanced concept of [*scoped slots*](https://vuejs.org/v2/guide/components-slots.html#Scoped-Slots). On the other hand, I do think that it is an interesting pattern to see if not to learn and use.

I had the chance to work with a friend and former colleague, [Edouard](https://twitter.com/ecattez) [Cattez](https://github.com/ecattez), on a project for a big French retailer. The Front-End of this project was made, you guessed it, using Vue.js.

We had an issue with the code base that we could not name. For several months we worked hard on it, but could not figure what it was that make so difficult to add or modify features.

This was about the same time where I started to really dive into the concepts of Clean Code, Clean Architecture and Software Craftsmanship.

One day, when talking to that friend, we finally were able to find the underlying issue, our code base lacked separation of concerns.

Every components in our application started to get quite big since they managed their template, data managing and styles. This way of writing components can work fine, as long as it does not get out of hands.

Our components, however, managed a lot of business logic and associated templates. It causes a great amount of cognitive load to read through since the components held the intent *and* the implementation of the business logic.

We needed a way to separate the concerns, to have the business logic in one place and the templates in another. We could drop the *Single File Components* or even write mixins, but those solutions sounded wrong in our context.

The issue was not that the template and the data managing were in the same file. It had more to do with the fact that we mixed the intent and the implementation. Like an application without *interfaces*, only implementations.

This is were we found out about the great article and pattern from [*Adam Wathan*](https://adamwathan.me), [**Renderless Components in Vue.js**](https://adamwathan.me/renderless-components-in-vuejs/).

I will not dig too deep inside the matter since his article already explain it all. Just know that it works by creating a **renderless component**, responsible for the data managing. This renderless component then provides information for the "*view component*" to use thanks to **scoped slots**.

How could we apply this pattern in our TODO-list? Let us first try to extract the logic inside a renderless component, named **`TodoListManager`**:


```html
<!-- No template tag, we will use a render function -->
<script>
export default {
    name: 'todo-list-manager',
    data() {
        return {
            loadingError: '',
            todoList: [ ],
            todoCreationError: '',
        };
    },
    computed: {
        doneCount() {
            return this.todoList.filter(todo => todo.done).length;
        },
        totalCount() {
            return this.todoList.length;
        },
    },
    methods: {
        loadTodos() {
            this.loadingError = '';
            fetch(import.meta.env.VITE_TODOS_URL)
                .then((response) => {
                    if (!response.ok) {
                        throw new Error('An error has occurred while loading todos');
                    }
                    return response.json();
                })
                .then((todos) => {
                    this.todoList = todos;
                })
                .catch((error) => {
                    this.loadingError = error;
                });
        },
        achieveATodo(id) {
            // [...] Call the API to achieve the task
        },
        addTodo(content) {
            this.todoCreationError = '';
            fetch(import.meta.env.VITE_TODOS_URL, {
                method: 'post',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ content })
            })
                .then((response) => {
                    if (!response.ok) {
                        throw new Error('An error has occurred while creating todo');
                    }
                    return response.json();
                })
                .then((newTodo) => {
                    this.todoList = [
                        ...this.todoList,
                        newTodo,
                    ]
                })
                .catch((error) => {
                    this.todoCreationError = error;
                });
        }
    },
    created() {
        this.loadTodos();
    },
    render() {
        // Only display the content inside of the default slot, with every needed information
        return this.$slots && this.$slots.default && this.$slots.default({
            loadTodos: this.loadTodos,
            loadingError: this.loadingError,
            todoList: this.todoList,
            doneCount: this.doneCount,
            totalCount: this.totalCount,
            achieveATodo: this.achieveATodo,
            addTodo: this.addTodo,
            todoCreationError: this.todoCreationError,
        });
    },
}
</script>
```

While the view component could be like this one:

```html
<template>
    <!-- Use our renderless component -->
    <!-- You can see that not only data are provided but also methods, computed, etc -->
    <todo-list-manager v-slot="{
        loadTodos,
        loadingError,
        todoList,
        doneCount,
        totalCount,
        achieveATodo,
        addTodo,
        todoCreationError,
    }">
        <!-- Here, we can use every reactive information provided by the renderless component -->
        <h1>My TODO list! ({{ doneCount }} / {{ totalCount }})</h1>

        <!-- Error management -->
        <p v-if="loadingError">
            {{ loadingError }}
            <button @click="loadTodos">Reload</button>
        </p>

        <ul v-else>
            <li v-for="todo in todoList" :key="todo.id">
                {{ todo.content }}
                <button @click="achieveATodo(todo.id)">Validate</button>
            </li>
        </ul>
        <form @submit.prevent="() => addTodo(newTodoContent)">
            <label>
                What do you have to do?
                <!-- newTodoContent may come from the view component or the renderless one -->
                <input v-model="newTodoContent">
            </label>
            <input type="submit" value="Create">
            <!-- Error management -->
            <p v-if="todoCreationError">{{ todoCreationError }}</p>
        </form>
    </todo-list-manager>
</template>

<script>
// [...]
</script>
```

We could go even further by extracting the API call inside a JS module, creating a generic loading and error display management component, etc. Those enhancements are out of the scope of the article, but still great to do. What we can do now, however, is to keep iterating on the renderless component.

Our `TodoListManager` seems greatly filled to me. What if we only need to list the tasks? What if we only need to create a new one?

We could ignore the data exposed by the renderless component that we do not need. However, I find it more clear to explicitly use the renderless component responsible for the creation of a task and / or the one responsible for the listing. Here is how we can achieve that.

First, the creation logic is extracted in a new renderless component, **`TodoCreationManager`**:

```html
<script>
export default {
    name: 'todo-creation-manager',
    data() {
        return {
            todoCreationError: '',
        };
    },
    emits: [ 'todo-created' ],
    methods: {
        addTodo(content) {
            this.todoCreationError = '';
            fetch(import.meta.env.VITE_TODOS_URL, {
                method: 'post',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ content })
            })
                .then((response) => {
                    if (!response.ok) {
                        throw new Error('An error has occurred while creating todo');
                    }
                    return response.json();
                })
                .then((newTodo) => {
                    // We don't have any reference to the list here
                    // We can, however, send an event with the created task
                    this.$emit('todo-created', newTodo)
                })
                .catch((error) => {
                    this.todoCreationError = error;
                });
        }
    },
    render() {
        return this.$slots && this.$slots.default && this.$slots.default({
            addTodo: this.addTodo,
            todoCreationError: this.todoCreationError,
        });
    },
}
</script>
```

Our **`TodoListManager`** component is now only responsible for the fetching of the tasks list.

Then, in our view component, we need to nest the two renderless component and use the logic from them both in the template:

```html
<template>
    <!-- Use our renderless component -->
    <!-- You can see that not only data are provided but also methods, computed, etc -->
    <todo-list-manager v-slot="{
        loadTodos,
        loadingError,
        todoList,
        doneCount,
        totalCount,
        achieveATodo,
        todoCreated,
    }">
        <!-- A second renderless component, managing the creation of a task -->
        <!-- 
            When this component notify us that a new task is created,
            we can add it directly to the list by calling a method
            on the todo-list-manager renderless component
         -->
        <todo-creation-manager
            v-slot="{
                addTodo,
                todoCreationError,
            }"
            @todo-created="todoCreated"
        >
            <!-- Here, we can use every reactive information provided by the renderless component -->
            <h1>My TODO list! ({{ doneCount }} / {{ totalCount }})</h1>

            <!-- Error management -->
            <p v-if="loadingError">
                {{ loadingError }}
                <button @click="loadTodos">Reload</button>
            </p>

            <ul v-else>
                <li v-for="todo in todoList" :key="todo.id">
                    {{ todo.content }}
                    <button @click="achieveATodo(todo.id)">Validate</button>
                </li>
            </ul>
            <form @submit.prevent="() => addTodo(newTodoContent)">
                <label>
                    What do you have to do?
                    <!-- newTodoContent may come from the view component or the renderless one -->
                    <input v-model="newTodoContent">
                </label>
                <input type="submit" value="Create">
                <!-- Error management -->
                <p v-if="todoCreationError">{{ todoCreationError }}</p>
            </form>
        </todo-creation-manager>
    </todo-list-manager>
</template>
```

-----

It is a pattern that I adopted for every component with complex business logic. It helps keeping your view component clean and concise. However, since it is based on a renderless *component*, it adds one to the components tree each time you use it. **It is also worth noting that it is an advance pattern that adds to the entry cost of your code base**.

How is this elegant solution compared to the new Composition API? Let us find out. 

## Refactoring in Vue 3 with the Composition API

In this section, I will assume that you are already familiar with the intent and the basic syntax of the Composition API.

I made [an article introducing the API in case you never heard about it](https://dev.to/thomasferro/vue-3-s-composition-api-and-the-segregation-of-concerns-1k5c). Please read it first if you are afraid of being confused by the syntax.

We have two implemented features:

- Fetch the todo list;
- Add a new one.

You can try to follow the same pattern while implementing the task achievement if you want.

Let us start with the list fetching. First, we will create a new ES module with a method that contains every pieces of information about the todo list. It is basically the same as the data inside the carryall component, but with a different syntax:

```js
import { ref, computed } from 'vue';

export const useTodoList = () => {
    // First, we create the reactive data and computed
    const todoList = ref([ ]);
    const doneCount = computed(() => {
        return todoList.value.filter(todo => todo.done).length;
    });
    const totalCount = computed(() => {
        return todoList.value.length;
    });

    const loadingError = ref('');

    // Then we create the method that will manipulate those data
    const loadTodos = () => {
        loadingError.value = '';
        fetch(import.meta.env.VITE_TODOS_URL)
            .then((response) => {
                if (!response.ok) {
                    throw new Error('An error has occurred while loading todos');
                }
                return response.json();
            })
            .then((todos) => {
                todoList.value = todos;
            })
            .catch((error) => {
                loadingError.value = error;
            });
    }

    const achieveATodo = (id) => {
        // [...] Call the API to achieve the task
        // Move it in a new method useTodoAchiever
    };

    // This method will be useful soon
    const todoCreated = (newTodo) => {
        todoList.value = [
            ...todoList.value,
            newTodo
        ]
    }
    
    // Finaly, we return the information that could be useful for our clients
    return {
        todoList,
        doneCount,
        totalCount,
        loadingError,
        loadTodos,
        achieveATodo,
        todoCreated,
    }
}
```

These information will be consumed by our view component's `setup` method. Here is the **`TodoList`**:

```html
<template>
    <!-- The template remains untouched -->
</template>

<script>
import { useTodoList } from './index.js';

export default {
    name: 'todo-list',
    setup() {
        // You cannot destructure the returned value here or you will loose Vue's reactivity
        const todoListData = useTodoList();

        todoListData.loadTodos();

        return {
            todoList: todoListData.todoList,
            doneCount: todoListData.doneCount,
            totalCount: todoListData.totalCount,
            loadingError: todoListData.loadingError,
            loadTodos: todoListData.loadTodos,
            achieveATodo: todoListData.achieveATodo,
        }
    },
}
</script>
```

We can now do the same with the task creation process:

```js
export const useTodoCreation = ({
    // Method called when a todo is created
    onTodoCreated = () => {},
}) => {
    // Create the reactive data
    const todoCreationError = ref('');

    // The method used to create a new task
    const addTodo = (content) => {
        todoCreationError.value = '';
        fetch(import.meta.env.VITE_TODOS_URL, {
            method: 'post',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ content })
        })
            .then((response) => {
                if (!response.ok) {
                    throw new Error('An error has occurred while creating todo');
                }
                return response.json();
            })
            .then(onTodoCreated)
            .catch((error) => {
                todoCreationError.value = error;
            });
    }

    // Return the needed information
    return {
        todoCreationError,
        addTodo,
    }
}
```

Plug what is needed inside the view component:

```html
<script>
import { ref } from 'vue';
import { useTodoList, useTodoCreation } from './index.js';

export default {
    name: 'todo-list',
    setup() {
        // You cannot destructure the returned value here or you will loose Vue's reactivity
        const todoListData = useTodoList();
        const todoCreationData = useTodoCreation({
            // Plug the method that will update the list when a task is created
            onTodoCreated: todoListData.todoCreated,
        });
        const newTodoContent = ref('');

        todoListData.loadTodos();

        return {
            todoList: todoListData.todoList,
            doneCount: todoListData.doneCount,
            totalCount: todoListData.totalCount,
            loadingError: todoListData.loadingError,
            loadTodos: todoListData.loadTodos,
            achieveATodo: todoListData.achieveATodo,
            todoCreationError: todoCreationData.todoCreationError,
            addTodo: todoCreationData.addTodo,
            newTodoContent,
        }
    },
}
</script>
```

The last thing we can do is to create reusable component for the display of a task and for the creation form.

```html
<!-- TodoCreation.vue -->
<template>
    <form @submit.prevent="() => addTodo(newTodoContent)">
        <label>
            What do you have to do?
            <input v-model="newTodoContent">
        </label>
        <input type="submit" value="Create">
        <!-- Error management -->
        <p v-if="creationError">{{ creationError }}</p>
    </form>
</template>

<script>
export default {
    name: 'todo-creation',
    // Declare what events will our component emit
    emits: [
        'create-todo',
    ],
    props: {
        creationError: String,
    },
    data() {
        return {
            newTodoContent: '',
        }
    },
    methods: {
        addTodo(content) {
            this.$emit('create-todo', { content });
        }
    },
}
</script>
```

```html
<!-- TodoDisplay.vue -->
<template>
    {{ content }}
    <button @click="achieveTodo()">Validate</button>
</template>

<script>
export default {
    name: 'todo-display',
    emits: [
        'achieve-todo',
    ],
    props: {
        content: String,
    },
    methods: {
        achieveTodo() {
            this.$emit('achieve-todo');
        }
    },
}
</script>
```

```html
<!-- TodoList.vue -->
<template>
    <!-- Here, we can use every reactive information provided by the renderless component -->
    <h1>My TODO list! ({{ doneCount }} / {{ totalCount }})</h1>

    <!-- Error management -->
    <p v-if="loadingError">
        {{ loadingError }}
        <button @click="loadTodos">Reload</button>
    </p>

    <ul v-else>
        <li v-for="todo in todoList" :key="todo.id">
            <todo-display
                :content="todo.content"
                @achieve-todo="() => achieveATodo(todo.id)"
            ></todo-display>
        </li>
    </ul>
    <todo-creation
        :creation-error="todoCreationError"
        @create-todo="addTodo"
    ></todo-creation>
</template>

<script>
    // [...]
</script>
```

-----

This is even cleaner than the solution using *renderless component* to me since it does not add components to the tree. The Composition API allows for a strict segregation of concerns. Our components can use business logic without knowing implementation details.

It will, however and just like the *renderless components*, adds to the entry cost of our projects. That is why I will use it pragmatically and try to make it the most readable possible for newcomers. For instance, in this application, I only used it in the **`TodoList`** component.

## On the next episode...

I hope that this series helps you better understand the benefits of Vue 3's Composition API! Please send feedback of your journey learning to use this new toy :)

What do you think about those two methods? Which one is the clearer, the one that you will start to use?

Next on the series, I will show you how to get rid of Vuex thanks to the Composition API.
