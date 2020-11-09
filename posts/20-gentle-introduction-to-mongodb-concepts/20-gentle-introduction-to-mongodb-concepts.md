# Gentle introduction to MongoDB concepts (M001 - Part 1)

> Learn the intention before the implementation

I could have started this series by writing CRUD (*Create / Read / Update / Delete*) operations and explaining MongoDB base concepts on top of working example. However, I think that the best way to introduce a concept to some people is by allowing them to try along.

With that in mind, we will see what MongoDB actually is and how to get started with your own database for free!

## Wherever there is data...

First of all, what is a database?

It is a structured way to store and access data. You usually need one when you need to persist any of your applications' data.

Take a note taking application for instance, where you can write and manage notes. Imagine the difference of provided value between those two versions:

- One that loose all of your notes when refreshing the page;
- Another one who allows you to retrieve your notes every time you start it.

This is a trivial example, but data is actually everywhere. I never had a single professional experience where no data was involved, have you?

There are two main families of databases usually used: the *SQL* and the *NoSQL* databases.

*SQL* actually refers to the **Structured Querying Language**, the language used to manipulate the data inside of a relational database management system (*RDBMS*). It also gave its name to the databases family of the same name.

Here are the main differences between the two kinds of databases.

> **SQL databases** (OracleDb, PostreSQL, SQLite, ...): Where data is stored in strongly constrained and related tables. 

> **NoSQL databases**: Variety of database types that do not use the approach of related data tables.

In the *NoSQL* family, you can find:

> **Key–value stores** (Redis for instance): Where data is stored as key-value pairs with the keys only appearing once in the database. Think of it as a `map` in your favorite programming language.

> **Document store** (MongoDB): Where data is stored as documents in collections. We will of course discuss that kind of NoSQL database in length in this series.

> **Graph** (Noe4j): Used for data with strong relationships. Think LinkedIn's relations web for instance.

These are the main types of NoSQL databases that you will encounter, but [not the only ones](https://en.wikipedia.org/wiki/NoSQL#Types_and_examples).

So, MongoDB is a NoSQL document database, but what does it really means? Let us find out!

## MongoDB basics

MongoDB is classified as a *document database* because of how it stores data. Every entry in your database will take the form of **documents**, representing your data as **field-value pairs**. Picture yourself a *JSON object* and you basically see a *MongoDB document*.

Want to stock user information? Here is a valid document:

```json
{
  "firstName": "Thomas",
  "lastName": "Ferro"
}
```

> **Field**: A unique identifier for the document's attribute.

Again, this is strongly similar to any JSON object where you cannot have the same attribute twice or more.

Are we supposed to throw all of our documents in a single spot and hope for the best when trying to retrieve valuable information? It is not the case since MongoDB offers a first level of data arrangement, the *collections*.

> **Collection**: Where the documents are organized and stored.

The second level of arrangement are the *databases*. One database can have multiple collections and you will be able to search for your data by providing the couple *database* / *collection*.

For instance, the previous example can be stored in a `users` collection inside the database named after your project.

## Different ways to host your database

It is time to get yourself an up and running MongoDB instance! We will see different way to have your own database, starting with solutions to run it locally.

I won't get into much details on this part, but I think that it can be useful to know how to get a local MongoDB instance for development or test purposes. You will have to main solution to do so.

This is not the way that I would recommend, but it is the most basic one. You can start a MongoDB Community Server by downloading it [here](https://www.mongodb.com/try/download/community).

If you are already familiar with Docker or trying to get better at it, I would recommend that you run a [`mongo` container](https://hub.docker.com/_/mongo/).

Both those solutions require you to manage the administration part, even though they can work out of the box for basic usage. This can suit you for starting and getting to know MongoDB. However, when starting a real project, I would recommend using a managed database.

Managed databases are MongoDB (or other solutions) instances provided by a third party. As a developer, I found this solution to be the perfect compromise. It allows me to focus on my field of expertise while resting assured that my data will not be corrupted or deleted because of my lack of operational skills.

In this case, we often talk about **database as a service** such as *MongoDB Atlas* or the solution provided on *Amazon Web Services*,  *Microsoft Azure* or other. Those providers usually take responsibility for deploying, running and maintaining your MongoDB infrastructure with different pricing strategies.

In this series we will be using *MongoDB Atlas* since they are the ones providing the course and they offer a free plan for us to try their services.

## MongoDB Atlas in theory

On top of the MongoDB concepts, Atlas add a few principles.

> **Clusters**: Groups of servers that store data.

You can picture a cluster as Atlas's magic sauce that brings you all of the services related to your database.

In order to provide high availability and resiliency, Atlas use the concept of *replica sets*.

> **Replica set**: Connected MongoDB instances that store the same data.

Replica sets are useful to provide a resilient infrastructure by replicate the data in all instances. If one instance goes down, the others can still provide access to your data.

> **Instance**: Single machine running MongoDB in the cloud.

Atlas also provide tools to monitor your databases, but we will not cover them here.

With that theoretical section out of the way, it is time to play with Atlas!

## Start using MongoDB Atlas

The first thing to do is to create a cluster through Atlas' web application and store some data in it.

You can sign in with the authentication method of your choice [here](https://account.mongodb.com/account/login?nds=true), then *create an organization* which will contain your databases.

Inside this new organization, create a new project that will serve as a sandbox. You can build a free cluster in the *cloud provider* of your choice. It should not matter for this example, the Atlas services are supposed work the same for any cloud provider.

You now can serve yourself a cup of coffee or tea while the cluster is being provisioned for you ☕️

Congratulations, you have your first MongoDB cluster in the Cloud!

We now want to access the cluster. Two things to configure to do so:

- Grand access to our IP address (more information on security features [here](https://docs.atlas.mongodb.com/setup-cluster-security/#add-ip-addresses-to-the-whitelist))
  - You can allow access from anywhere for now, but **you really should define who can access your database before going into production.**
- Create a *database user* that will have the rights to perform actions on the data.

To start manipulating data, I suggest you load the sample dataset into your sandbox by clicking the ellipsis on your cluster then "Load sample dataset".

### Visualize data

Click the "collections" button in the cluster to see the data stored in it. You should be able to navigate through the databases, collections and documents imported from the sample dataset.

If you want to access and manipulate your data through a shell, you first need to click the "connect" button in your cluster then "Connect with the `mongo` shell". Here you will find every needed information to download the tool if needed.

Enter the provided connection command line in your terminal, replacing `dbname` and `dbuser` with your database's information: `mongo "mongodb+srv://sandbox.xsnoi.mongodb.net/<dbname>" --username <dbuser>`.

You should be prompted to enter your user's password. Once done, you are connected to your cluster!

Try to use the `db` command to see the name of your database 😁

---

You now have your own sandbox, it is time to learn how to actually manipulate data.

Thanks for reading through, next time we will dive deep into documents creation and manipulation!
