# microservices-workshop

A Microservices Workshop Spring Cloud and Kubernetes.

## Before you start

You need to install following software to finish this workshop:

- [VirtualBox](https://www.virtualbox.org/)
- [Vagrant](https://www.vagrantup.com/)
- IDE like [Visual Studio Code](https://code.visualstudio.com/), [IntelliJ IDEA](https://www.jetbrains.com/idea/) or [Eclipse](https://www.eclipse.org/downloads/)
- [MySQL Community](https://dev.mysql.com/downloads/mysql/)
- [MySQL Workbench](https://dev.mysql.com/downloads/workbench/)
- [Postman](https://www.getpostman.com/)
- [JDK-8](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

## Step 0: Clone this repo

Be sure that you have enough storage space (1~2 GB) and memory (2 GB). 

And clone this repo via `https://github.com/wizardbyron/microservices-workshop`.

Then checkout the step 1 via `git checkout step-1`.

## Step 1: Create an user management API to access database.

Our goal now to create a basic Spring Boot API for user management. We can use this API do CRUD operations on a mysql database.

Tasks:

1. Create a Spring boot API.
2. Create a local mysql instance.
3. Create user and database in mysql.

[Spring Guides](https://spring.io/guides) has many beginner projects in good code taste and enginnering practices. We can start from them.

Now we are going to start the journey from [Accessing data with MySQL](https://spring.io/guides/gs/accessing-data-mysql/).

You need to clone the repo via `git clone https://github.com/spring-guides/gs-accessing-data-mysql.git`.

You can following the guide start from `init` folder. But we just need `complete` folder for a quick win.

Use your IDE or editor open the folder.

Create a local mysql instance with creadentials and database name in the `application.properties` file.

You can use **MySQL Wrokbench** to connect your local MySQL instance as root to create database and user:

```SQL
CREATE database db_example;
CREATE USER 'springuser'@'%' IDENTIFIED BY 'ThePassword';
GRANT ALL ON msdb.* TO 'springuser'@'%';
GRANT USAGE ON msdb.* TO 'springuser'@'%';
FLUSH PRIVILEGES;
```

Now you can start your spring application from IDE or `./gradlew bootRun`.

`./gradlew` is a script means `gradle wrapper`, it will download gradle wrapper in the project folder to help you automated tasks. You can list all tasks via `./gradlew tasks`.

After a few seconds(or minutes), you can test the API from `PostMan` or `curl` by making a request to `http://localhost:8080/demo/all`.

If you get a `[]` which means an empty array, you have setup the API. Let's add a user.

Now you you can made a GET request to `http://localhost:8080/demo/add?name=test_user&email=test_user@google.com` to add a new user. It will response `Saved` to you.

You can verify the request by accessing database via `select * from user` in MySQL Workbench or make a new query request to `http://localhost:8080/demo/all` to get the user message you add.

It will response a JSON like:

```JSON
[
    {
        "id": 1,
        "name": "test_user",
        "email": "test_user@google.com"
    }
]
```

Now, You are finish the first mile stone, Congrats!

Let's take the next step by `git checkout step-2`

## Step 2: Hey, there is a Bug！

Let's back to the code, open `gs-accessing-data-mysql/complete/src/main/java/hello/MainController.java` in your IDE or editor. You can find out the code which handle `add` operation in line 20.

Here is a problem, what will happend if we add a new user with same user and email?

You can make same add request twice then you can get two users with same name and email like this:

```JSON
[
    {
        "id": 1,
        "name": "test_user",
        "email": "test_user@google.com"
    },
    {
        "id": 2,
        "name": "test_user",
        "email": "test_user@google.com"
    }
]
```

The only difference is the ID. The user can have different name but same email, or an email address with different account name, or there a unique user-email pair in the database. No matter what kind of constraint you want, You have to add it.

The constraint in the data model which generated from the user class. Spring boot will create a table for us when we write `spring.jpa.hibernate.ddl-auto=create` in file `application.properties`.

We can add the constraint via annotaion, You can add following line between `@Entity` and `public class User {` to assure user-email pair unique in database.

```Java
@Table(name = "USER", uniqueConstraints = @UniqueConstraint(columnNames = {"Name", "Email"}))
```

Let's rerun the application and make same request twice. It should return an unfriendly message like this:

```JSON
{
    "timestamp": "2018-12-19T14:15:16.304+0000",
    "status": 500,
    "error": "Internal Server Error",
    "message": "could not execute statement; SQL [n/a]; constraint [UKrjpkssy3ogf0vjvjbn7le85qt]; nested exception is org.hibernate.exception.ConstraintViolationException: could not execute statement",
    "path": "/demo/add"
}
```

Anyway we fixed a bug! Let's see how to keep this bug away.

Let's take the next step by `git checkout step-3`
