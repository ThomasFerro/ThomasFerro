# Golog : My take on palindromes and Go logging

> Logging enhanced, plus a quick integration with the ELK stack

## Should I write logs?

Yes.

Logging is a key part of debugging. How can we understand what went wrong in our application if we are in the dark?

However, logging for the sake of logging is never enough. For the messages to be useful, they need to be contextualized. What is the level of the log? What data are attached to it?

This way, we are able to quickly understand the root cause, which actions led to the observed behavior.

Still need to be convinced ? Want a more in-depth presentation of the why and how ? Please take a look at [Edouard Cattez's awesome article on the subject](https://medium.com/@edouard.cattez/the-art-of-talkative-application-489f03bd89f4).

## The standard way

The Go programming language offers a standard way of logging via the [`log` package](https://golang.org/pkg/log).

I really like how this package *abstract the output*. It makes it so easy to start logging right away for any little project, without having to deal with the low level details. Nobody wants to deal with low level details anyway, right ? *Right ?*

On the other hand, I feel like too much is going on under the hood, especially with the call to `os.Exit(1)` or the `panic` with the `Fatal` and `Panic` methods. I also am not found of the fact that there is *no logging level nor metadata management* out of the box. It makes me very sad and grumpy.

As I do not like to be sad and grumpy, and as I plain to start a [quite big project of my own](https://medium.com/p/building-a-blogging-application-part-1-the-goals-b4a99847584), I need a solution to the previously listed issues.

## Building the new logger

Disclaimer : The point here is not to reinvent the wheel - even tho I will - but to play around with a real example. I also wanted to create a logger that suits **my own needs** for further projects.

![XKCD comics on the proliferation of standards](https://github.com/ThomasFerro/readmes/blob/master/posts/9-golog-my-take-on-palindromes-and-go-logging/standards.png)

> Credits to XKCD for the comics, [link to the licence page](https://xkcd.com/license.html)

Rest assure, I wont describe the whole process like I did in [TDD in action : Debouncer in Go](https://medium.com/p/tdd-in-action-debouncer-in-go-b0f7d7d75931).

I did however built this logger following the TDD principles, to be sure that it worked the way I expected. "It does not work if it is not tested", as a wise human being once said, I guess. But how did I expected it to work again ?

Many loggers such as [Sirupsen's `logrus`](https://github.com/sirupsen/logrus) already tackle the issues of not having metadata nor levels of tracing.

The API of `golog` is as simple as the `logrus` one. You have one method per level of log (`Debug`, `Info`, `Warn`, `Error` and `Fatal`) and an *entry* system for the metadata.

To add metadata to a log, you can use (and chain) the `WithFields` method :

```golang
golog.WithFields(entries.Fields{
	"answer": 42,
	"title": "The Hitchhiker's Guide to the Galaxy",
}).WithFields(entries.Fields{
	"author": "Douglas Adams",
}).Debug("Unashamed reference dropping")
``` 

You can also log a message without any metadata :

```golang
golog.Debug("My message without context")
```

Unlike the standard API, `golog` **does not panic nor exit the program when using the `Panic` or `Fatal` method**. This is because I personally think that it breaks the Single Responsibility Principle and it make the behavior non-obvious. We may disagree on this point but for me it is an unwanted side-effect.

Here is a sneak peak of the project's current structure.

In order to *format* the logs, a `Formatter` must be provided with the following method `Format(fields entries.Fields, level string, message string) string`.

The *fields* are simply a map with the metadata's names and values (`type Fields map[string]interface{}`).

In the same package (`entries`) you will find the actual `Entry`, with a few methods : 

```golang
type Entry interface {
	Fields() Fields
	WithFields(fields Fields) Entry
	Debug(message string)
	Info(message string)
	Warn(message string)
	Error(message string)
	Fatal(message string)
	WriteLog(level string, message string)
}
```

The project also has a `loggers` package with the interface of a logger and a method to create a new one. As you may have understand already, a logger must have a `Formatter` and an `Output` : 

```golang
func NewLogger(output io.Writer, formatter formatters.Formatter) Logger {
	return GoLogger{
		output,
		formatter,
	}
}
```

With these concerns split, I was able to add the last feature : **Being able to configure multiple loggers**. This way, I can have the log written as key-value pair in the terminal and as JSON in a file.

The method used to actually write a logging message iterates through all of the configured loggers. For each of them, it then write the message formatted using the provided formatter. 

```golang
func (entry LogEntry) WriteLog(level string, message string) {
	for _, logger := range GetLoggers() {
		logger.Output().Write(
			[]byte(
				logger.Formatter().Format(entry.Fields(), level, message),
			),
		)
	}
}
```

Perfect, we know write the formatted message! The time is upon us when we will need no debugging tool, all we need to do is to read the logs! ...But wait, how could we configure *golog* ?

The next step was to provide a configuration system with the following rules :

1. Have a default configuration with a log file and a console output;
2. Being able to override the configuration with in-code configuration.

This is where the `configurations` package steps in. It provides the following default configuration :

```golang
var defaultLoggers = []gologgers.Logger{
	gologgers.NewLogger(os.Stdout, formatters.NewKvpFormatter()),
	gologgers.NewLogger(outputs.NewSimpleLogFile(
		"log",
	), formatters.NewKvpFormatter()),
}
```

It also exposes a `SetLoggers` method, used to override the default configuration.

When fetching the loggers to use, the default ones are used when no other are provided.

## Going further : integration with centralized logs manager

Before we wrap it all up, let us integrate the new logging system with a centralized logs manager.

The main focus for me was to decorrelate the application from the logs manager. [*Filebeat* modules](https://www.elastic.co/products/beats/filebeat) allows me to do so by simply reading log files and sending the data wherever I want. For this example, I will be sending those data to an [*ELK* stack](https://www.elastic.co/what-is/elk-stack).

In order to easily start an *ELK* stack, I have started a *Elastic Cloud* free trial.

Then, I installed and configure *Filebeat* (v7.4.0) based on the [documentation](https://www.elastic.co/products/beats/filebeat).

I decided to configure my logger to write the message as *JSON* object in a file, so the *Filebeat*'s input configuration will look like this :

```yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /my/path/to/the/log/files/*.log
  json.keys_under_root: true
```

One use case that I do not cover yet is the rotating logs. When log files start to get too big, one common practice is to keep the oldest messages in "archive" files. *Filebeat* is able to manage this use case, you can find the related documentation [here](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-input-log.html#rotating-logs).

The *Filebeat* now watch the log files and send the data to *Elasticsearch*, and here is the result in *Kibana* :

![Results in Kibana](https://github.com/ThomasFerro/readmes/blob/master/posts/9-golog-my-take-on-palindromes-and-go-logging/log-in-kibana.png)

You may have noticed that I cheated a little. I needed to log the message as *JSON* object because I could not find any documentation about decoding key-value pair in the *Filebeat*. Feel free to let me know if you have any information about this subject.

Anyway, no reference to the *ELK* nor *Filebeat* are in the Go code, mission accomplished!

## Special thanks

I would like to thanks [Fabrice Claeys](https://twitter.com/fabriceclaeys) and [Matthieu Fernandes](https://twitter.com/passionne42), two of my coworkers who helped me with their knowledge on logging et the Go language!

Another special thanks to my friend and coworker [Edouard Cattez](https://twitter.com/ecattez) for his precious inputs on logging, the *ELK* stack and *Filebeat*!
