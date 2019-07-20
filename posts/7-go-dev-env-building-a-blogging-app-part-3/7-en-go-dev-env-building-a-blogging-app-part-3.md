# Test, build and Dockerize your Go app with no effort ! - Building a blogging application part 3

> My take on the application of the Docker's promises

## Do we transvestite Docker ?

As you may notice through the reading of my articles, I am no Docker expert.

On the other hand, the premises and the potential of containerization really speak to me. That is why I am willing to step out of my comfort zone to learn the principles and apply them as much as I can !

The thing that really bugs me about Docker is its usage that seems to be generally done.

I may be mistaking, but the really strength of Docker for me is in the ability to have the certainty to put in production the exact same environment that you developed on your machine.

However, all I see is people (including myself) developing software locally, pushing the code in any repository and only then building a Docker image based on the latest sources.

Docker seems to be relegated to an afterthought, as if we were building the image because of the containerization hype ?

Am I wrong when I picture the following development workflow ?

1. Do the needed changes in your IDE / text editor;
2. Build a Docker image;
3. Run the container, check if the work is done;
4. Promote the image in any staging area for the domain experts to test it;
5. Promote the image in the production environment. 

Only then will we be sure that the image used in production is working just like the one on our machine.

With that in mind, let us talk a little bit about a very important matter when developing before diving into the main subject.

## On feedback loop

This may be an opinion-based subject, but I do think that this is the most frustrating part when I first started using Docker.

![Scrum feedback loop](https://github.com/ThomasFerro/readmes/blob/master/posts/7-go-dev-env-building-a-blogging-app-part-3/scrum_feedback_loop.jpg)

![Kanban feedback loop](https://github.com/ThomasFerro/readmes/blob/master/posts/7-go-dev-env-building-a-blogging-app-part-3/kanban_feedback_loop.jpg)

You may be familiar with the concept of feedback loop if you are working in an Agile team. The point is to have the shortest loop possible in order to now right away if you are heading in the right direction. You want to have regular feedback from your clients, remember that they are the reasons you are making the software in the first place !

Now, in software development, you can see the feedback loop just as described above. The only change is in the actors.

The first actor is the compiler. Take a software that does not compile, do you really need to wait for the end user to tell you that it does not work ? No, you have the information right away in your IDE / terminal. **The compilation is the shortest feedback loop when developing.**

Things start to get more interesting with the second actor : the tests. I am not making any distinction between the different kinds of tests, they complete each other so you have to run them all locally. **Trust me, you want your tests to run as fast and as frequently as possible.**

The third actor is you, the developer, the (wo)man in the chair. After testing and building your app, you should be checking by hand your work. **This is most likely to be the longest feedback loop of the three since you have to wait for the software to compile and run.**

To summarize, here are the three actors of the development feedback loop :

![Development feedback loops](https://github.com/ThomasFerro/readmes/blob/master/posts/7-go-dev-env-building-a-blogging-app-part-3/development_feedback_loops.jpg)

1. The compiler (if you got one) : If the software does not compile, *it does not work*;
2. The tests (if you got some **AND YOU SHOULD**) : If the test suites does not pass, *it does not work*;
3. The human (if indeed you are human) : If it does not work, *it does not work*;

When talking about this software development feedback loop, how long is too long ?

This is when things get even more opinionated. For me it is when I have the time to think of something else, when I lose focus. It is up to you to decide your own limit.

Going back to the subject, how is this all related to me building a Go development environment ?

When I first started to play with Go, I had this "all by hand" workflow :

1. Change the code;
2. Run the tests;
3. Build;
4. Run;
5. When everything was ok, build a Docker image and run a container;

Notice the similarities with the Docker usage described in the first part.

It may seem like a standard development workflow, but boy if it is a long one when entering every command by hand. Especially when you are a Go and a Docker neophyte like me !

Here is how I tried to reduce the loop, and greatly simplify my life while training my Go and Docker skills.

## Introduction go-dev-env !

First thing first, we need to make sure that the initial needs are being addressed.

The main issue is to reduce the time of the development feedback loop by providing a solution that runs frequently and requires minimum effort from the developer.

This minimum effort could be just a button to press, but it will be even better if the process starts itself whenever a file has been modified.

The second issue is to build an artifact that the developer could use instantly and deploy in any environment.

Using the same terms as the ones that described the needs, here is a draft of a solution.

I need a development environment that runs a *workflow* (this workflow will eventually build a Docker image) every time I receive a notification from a *trigger* (for instance, a "file changed trigger").

We can clearly see the two main parts of the project from this sentence :

- The workflow; and
- The trigger.

## The workflow

A workflow is the description of everything that needs to happen for the artifact to be built.

```go
package workflows

import "github.com/go-dev-env/triggers"

// Workflow A workflow to be executed in the dev env
type Workflow interface {
	Execute()
	Init()
	Trigger() triggers.Trigger
}
```

A workflow is composed of an `Execute` method that start the workflow and an `Init` method to initiate the process. We also have a reference to the workflow's trigger.

For now, I only need to manage workflow that are based on a builder (here, a Dockerfile). To do so, I only needed to create a `BuildWorkflow`, that calls the builder every time a notification is sent from the trigger.

The builder is also quite simple :

```go
package builders

// ArtifactPath The path to the built artifact
type ArtifactPath string

// Builder An artifact builder
type Builder interface {
	Build(contextPath string) (ArtifactPath, error)
}
```

It builds an artifact and can also throw an error in the process. The last part is to create a `DockerBuilder` for our specific case.

This builder uses the standard library to create and execute a `docker build` command.

## The trigger

Here is how I describe a `Trigger` :

```go
package triggers

/// Trigger A standard trigger
type Trigger interface {
	Init() chan bool
}
```

We simply have to call the `Init` method to initiate the trigger, and what we get in return is a new channel through which we will be notified.

We need to know when to start the workflow. As I said earlier, we can achieve that with a simple button to press, ordering the workflow to run again.

However, I found it more interesting to have a trigger based on the modification in the source code.

This is one of the few places where I have allowed myself to rely on an external library :  [`fsnotify`, a file system notifications library.](https://github.com/fsnotify/fsnotify).

With a little bit of setup, I was able to wrap this tool under the `FileWatcherTrigger`. I can now simply create a new trigger using the exposed method :

```go
// NewFileWatcherTrigger Create a new file watcher trigger
func NewFileWatcherTrigger(path string) *FileWatcherTrigger {
	log.Println("Creating a new file watcher trigger")

	return &FileWatcherTrigger{
		path: path,
	}
}
```

Few noticeable details :

- This trigger uses a homemade debouncer to avoid duplicate notifications ([here is an article describing the making of this debouncer](https://medium.com/@t.ferro184/tdd-in-action-debouncer-in-go-b0f7d7d75931));
- It only notifies if a file or a folder is created, modified or deleted;
- It will watch everything inside the provided path, including files inside a newly created folder.

Putting it all together, as done in the `main.go` file, we can now create our workflow :

```go
package main

import (
	"os"

	"github.com/go-dev-env/triggers"
	"github.com/go-dev-env/workflows"
	"github.com/go-dev-env/builders/docker"
)

func getPath() string {
	path := os.Getenv("SRC_PATH")
	if path == "" {
		path = "/src"
	}
	return path
}

func main() {
	path := getPath()
	
	trigger := triggers.NewFileWatcherTrigger(path)
	builder := docker.NewBuilder()

	workflow := workflows.NewBuildWorkflow(path, trigger, builder)
	workflow.Init()

	for {}
}
```

How does it all look from the developer perspective ?

## Story time

Let us say I am a developer working on a blogging application in Go (which turns out to be true).

The first thing I will need to do in order to use the development environment is to describe my workflow. Lucky for us, it is a pretty straightforward one :

1. Run the test suites;
2. Build the application;
3. Run it.

This workflow can be translated as this multistep Dockerfile :

```dockerfile
# Run the tests with a Go specific version
FROM golang:1.12.6 as tests

WORKDIR /src

ADD . .

RUN go test ./...


# Build the artifact for the same Go version
FROM golang:1.12.6 as builder

WORKDIR /src

ADD . .

RUN CGO_ENABLED=0 GOOS=linux go build -o /dist/go-app

# Run the software from an empty Docker image
FROM scratch

WORKDIR /root

COPY --from=builder /dist .

CMD [ "./go-app" ]
```

Setup time is over, I can now run the `go-dev-env` and start to work on the actual blogging application ! Every time I change something in my code base, the workflow will run and I will be noticed about every issue in the test suites or the compilation.

The last thing I need to do is to run the Docker container based on the image generated by the development environment 😃.

## Next steps

Obviously, this environment only provide a solution for my current needs.

In order to go full circle, it would be nice if the development environment was himself inside a Docker container. I have tried to tackle this feature, but I was not able to build a Docker image inside a Docker container.

The other and more important feature to be added is the possibility to make it work on bigger project, with more than one code bases. I will certainly work on this when building an application with many microservices.

Do not hesitate to give me your opinion on the project, and even submit or work on an issue, it is [hosted on GitHub and is open source](https://github.com/ThomasFerro/go-dev-env) !

## In the next episode...

This article was part of my "Building a blogging application" series, where I describe the steps that seem important to me while working on this project.

It was really fun to learn all those new things while making a development environment that tackles my actual issues. I do encourage you to build things really useful when learning a new language or a new technology. It feels way more rewarding than building a todo app for instance, and you are way less likely to give up if you need that piece of software.

Stay tuned for more articles, the next ones are going to be focused on the implementation of the first features !
