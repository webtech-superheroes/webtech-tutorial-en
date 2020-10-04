# RESTful API

A RESTful API is a method to expose data and perform CRUD operations on top of the HTTP protocol.

In this exercise we will be creating a web server using NodeJS and the ExpressJS framework. Data will be stored in MySQL and we will be using Sequelize to model the tables and relations.

## 1. How to initialize a NodeJS app?

In order to initialize a node app open a directory where you want to create the app and  execute the following command in the terminal

```bash
npm init
```

You will be prompted to fill in the following details:

* Name
* Version
* Description
* Entry point
* Test command
* Git repository
* Keywords
* Author
* License

Before completing this step you need to confirm the data is correct:

If everything went well a _**pagckage.json**_ file is generated

* [ ] TODO: check if the package.json exists
* [ ] TODO: create a new file called _**server.js**_

## 2. How to build a HTTP server using ExpressJS?

ExpressJS is a lightweight, but powerful framework for building web applications or web services.

To include this dependency in your project run the following npm command

```bash
npm install express --save
```

Write the code in the _**server.js**_ file

```javascript
const express = require('express')
```

Next create a constant called`app`

```javascript
const app = express()
```

Use the static files middleware to serve your webpage assets `express.static`

```javascript
app.use('/', express.static('frontend'))
```

The first parameter is the path where you want the files to be served. The second parameter is the middleware configured with the directory where the files are located.

Configure the server to listen to 8080 port for HTTP.

```javascript
app.listen(8080)
```

* [ ] TODO: create a directory called `frontend`
* [ ] TODO: add a new file called `index.html`
* [ ] TODO: open the app in the browser \([http://localhost:8080](http://ip:8080)\) - make sure to replace localhost with your IP if you work in the cloud

## 3. How to install MySQL and create the database?

If you haven't done this already, follow the tutorial published [here](../development-environment/mysql.md).

Login into the console and create a database called **profile**:

```sql
create database profile;
```

* [ ] TODO: check if the database was created by executing `show databases;`
* [ ] TODO: exit the console by typing `exit`and pressing enter

## 4. How to connect your database from NodeJS using Sequelize?

Sequelize is an object oriented library that provides a high level mapping between the database and your code. In practice we call this pattern object-relational mapping \(ORM\). It allows us to represent tables from the database by creating models and relationships. On each model basic CRUD \(Create, Read, Update, Delete\) operations are exposed. 

Sequelize is a wrapper on multiple database drivers. In our example we will use **mysql2.** Both the library and the drivers will be installed using npm.

```bash
npm install --save sequelize
```

```bash
npm install --save mysql2
```

Next edit the server.js file and create an instance of the Sequelize class

```javascript
const Sequelize = require('sequelize')

const sequelize = new Sequelize('profile', 'root', '', {
    dialect: "mysql",
    host: "localhost"
})
```

Notice the difference between `sequelize` as an object and `Sequelize` as a class. 

The first parameter of the constructor is the database name, the second parameter is the username and the third is the passwor. Further options are passed as an object in the fourth parameter.

In order to perform the database connection we call the  **authenticate\(\)** method.

As many Sequelize methods, the authentification will be asyncronus, thus it will return a promise.  `Promise`. When the promise is fulfilled the function passed as a parameterh on the **then\(\)** method is called. If the promise gets rejected the function on the **catch\(\)** method will be called.

```javascript
sequelize.authenticate().then(() => {
    console.log("Connected to database")
}).catch((err) => {
    console.log(err)
    console.log("Unable to connect to database")
})
```

* [ ] TODO: run `node server.js`
* [ ] TODO: check the console for the message: **Conected to database**

## 5. How to define models for my messages table?

The fist parameter is the name of the table. As a convention is preferred to define the tables using english language and use the plural version. The second parameter is an object that describes the structure of the table as key value pairs.

```javascript
const Messages = sequelize.define('messages', {
    subject: Sequelize.STRING,
    name: Sequelize.STRING,
    message: Sequelize.TEXT
})
```

Extended documentation is here - [http://docs.sequelizejs.com/manual/tutorial/models-definition.html](http://docs.sequelizejs.com/manual/tutorial/models-definition.html)

Supported data type - [http://docs.sequelizejs.com/manual/tutorial/models-definition.html\#data-types](http://docs.sequelizejs.com/manual/tutorial/models-definition.html#data-types)

## 6. How to create the tables in the database?

Using _**sync\(\)**_ all the models will be created in your database

Note that by adding `{force: true}` existent tables will be recreated from scratch

To create the tabeles we are exposing a GET /createdb endpoint

```javascript
app.get('/createdb', (request, response) => {
    sequelize.sync({force:true}).then(() => {
        response.status(200).send('tables created')
    }).catch((err) => {
        console.log(err)
        response.status(200).send('could not create tables')
    })
})
```

* [ ] TODO: open the brouser and navigate to http://localhost:8080/createdb
* [ ] TODO: login to the mysql console type `use profile` and `show tables;` to check if the tables were created

## HTTP Methods

Now that we have the model let's implement the Create, Read, Update and Delete operations.

![metode http](../.gitbook/assets/00002-arhitectura-metode-http.jpg)

## 7. How to create resources using POST?

We consider the following endpoint

```text
POST /messages
```

The client will perform a HTTP request containing data in `json` or `urlencoded`. To parse the data we need to add two middleware functions. The endpoint is defined using app.post

```javascript
app.use(express.json())
app.use(express.urlencoded())

//definire endpoint POST /messages
app.post('/messages', (request, response) => {
    Messages.create(request.body).then((result) => {
        response.status(201).json(result)
    }).catch((err) => {
        response.status(500).send("resource not created")
    })
})
```

 **create** ---&gt; ``INSERT INTO messages (`subject`, `name`, `message`) VALUES ('test','test','test')``.

![postman post method](../.gitbook/assets/00701-postman-post.png)

1. Choose POST
2. Type the URL of the resource
3. Choose raw a body type
4. Choose application/json content type
5. Press Send

## 8. How to expose data from a table using GET?

Pentru a lista datele dintr-un tabel vom expune două enpoint-uri. Primul care returnează toată lista de mesaje și al doilea care returnează un mesaj după un ID specific.

```text
GET /messages
```

```text
GET /messages/1
```

Details on data queries: [http://docs.sequelizejs.com/manual/tutorial/querying.html](http://docs.sequelizejs.com/manual/tutorial/querying.html)

```javascript
app.get('/messages', (request, response) => {
    Messages.findAll().then((results) => {
        response.status(200).json(results)
    })
})

app.get('/messages/:id', (request, response) => {
    Messages.findByPk(request.params.id).then((result) => {
        if(result) {
            response.status(200).json(result)
        } else {
            response.status(404).send('resource not found')
        }
    }).catch((err) => {
        console.log(err)
        response.status(500).send('database error')
    })
})
```

* [ ] TODO: testează endpoint-urile create folosind Postman

## 9. How to update a resource using PUT?

Considering the following endpoint

```text
PUT /messages/1
```

First, the resource is retrieved by ID. If the resource was not found the server's response is **404 NOT FOUND**

If the resource was fount we call the `update()` method with the payload sent by the client in the body.

```javascript
app.put('/messages/:id', (request, response) => {
    Messages.findByPk(request.params.id).then((message) => {
        if(message) {
            message.update(request.body).then((result) => {
                response.status(201).json(result)
            }).catch((err) => {
                console.log(err)
                response.status(500).send('database error')
            })
        } else {
            response.status(404).send('resource not found')
        }
    }).catch((err) => {
        console.log(err)
        response.status(500).send('database error')
    })
})
```

* [ ] test the endpoint in POSTMAN

![postman put](../.gitbook/assets/00901-postman-put.png)

## 10. How to remove resources using delete DELETE?

Considering the following endpoint

```text
DELETE /messages/1
```

First, the resources is retrieved by id. If the resource was found we are calling the destroy method to send the delete instruction. This will return a Promise that needs to be treated accordingly. If the delete was successful the server's response is 204 NO CONTENT

```javascript
app.delete('/messages/:id', (request, response) => {
    Messages.findByPk(request.params.id).then((message) => {
        if(message) {
            message.destroy().then((result) => {
                response.status(204).send()
            }).catch((err) => {
                console.log(err)
                response.status(500).send('database error')
            })
        } else {
            response.status(404).send('resource not found')
        }
    }).catch((err) => {
        console.log(err)
        response.status(500).send('database error')
    })
})
```

* [ ] TODO: test the endpoint in POSTMAN

## 11. Putting it all together \(server.js\)

```javascript
const express = require('express')
const Sequelize = require('sequelize')

const sequelize = new Sequelize('profile', 'root', '', {
    dialect: "mysql",
    host: "localhost"
})

sequelize.authenticate().then(() => {
    console.log("Connected to database")
}).catch((err) => {
    console.log(err)
    console.log("Unable to connect to database")
})

const Messages = sequelize.define('messages', {
    subject: Sequelize.STRING,
    name: Sequelize.STRING,
    message: Sequelize.TEXT
})

const app = express()
app.use('/', express.static('frontend'))

app.use(express.json())
app.use(express.urlencoded())

app.get('/createdb', (request, response) => {
    sequelize.sync({force:true}).then(() => {
        response.status(200).send('tables created')
    }).catch((err) => {
        console.log(err)
        response.status(200).send('could not create tables')
    })
})

//definire endpoint POST /messages
app.post('/messages', (request, response) => {
    Messages.create(request.body).then((result) => {
        response.status(201).json(result)
    }).catch((err) => {
        response.status(500).send("resource not created")
    })
})


app.get('/messages', (request, response) => {
    Messages.findAll().then((results) => {
        response.status(200).json(results)
    })
})

app.get('/messages/:id', (request, response) => {
    Messages.findByPk(request.params.id).then((result) => {
        if(result) {
            response.status(200).json(result)
        } else {
            response.status(404).send('resource not found')
        }
    }).catch((err) => {
        console.log(err)
        response.status(500).send('database error')
    })
})

app.put('/messages/:id', (request, response) => {
    Messages.findByPk(request.params.id).then((message) => {
        if(message) {
            message.update(request.body).then((result) => {
                response.status(201).json(result)
            }).catch((err) => {
                console.log(err)
                response.status(500).send('database error')
            })
        } else {
            response.status(404).send('resource not found')
        }
    }).catch((err) => {
        console.log(err)
        response.status(500).send('database error')
    })
})


app.delete('/messages/:id', (request, response) => {
    Messages.findByPk(request.params.id).then((message) => {
        if(message) {
            message.destroy().then((result) => {
                response.status(204).send()
            }).catch((err) => {
                console.log(err)
                response.status(500).send('database error')
            })
        } else {
            response.status(404).send('resource not found')
        }
    }).catch((err) => {
        console.log(err)
        response.status(500).send('database error')
    })
})


app.listen(8080)
```

## 12. Using async/await keywords to simplify the code

```javascript
const express = require('express')
const Sequelize = require('sequelize')

const sequelize = new Sequelize('profile', 'root', '', {
    dialect: "mysql",
    host: "localhost"
})

sequelize.authenticate().then(() => {
    console.log("Connected to database")
}).catch((err) => {
    console.log(err)
    console.log("Unable to connect to database")
})

const Messages = sequelize.define('messages', {
    subject: Sequelize.STRING,
    name: Sequelize.STRING,
    message: Sequelize.TEXT
})

const app = express()
app.use('/', express.static('frontend'))

app.use(express.json())
app.use(express.urlencoded())

app.get('/createdb', async (request, response) => {
    try {
        await sequelize.sync({force:true})
        response.status(200).send('tables created')
    } catch(err) {
        console.log(err)
        response.status(200).send('could not create tables')
    }
})

//definire endpoint POST /messages
app.post('/messages', async (request, response) => {
    try {
        let result = await Messages.create(request.body)
        response.status(201).json(result)
    } catch(err) {
        console.log(err)
        response.status(500).send("resource not created")
    }
})


app.get('/messages', async (request, response) => {
    try {
        let results = await Messages.findAll()
        response.status(201).json(results)
    } catch(err) {
        console.log(err)
        response.status(500).send("server error")
    }
})

app.get('/messages/:id', async (request, response) => {
    try {
        let result = await Messages.findByPk(request.params.id)
        if(result) {
            response.status(200).json(result)
        } else {
            response.status(404).send('resource not found')
        }
    } catch(err) {
        console.log(err)
        response.status(500).send('database error')
    }
})

app.put('/messages/:id', async (request, response) => {
    try {
        let message = await Messages.findByPk(request.params.id)
        if(message) {
            let result = await message.update(request.body)
            response.status(201).json(result)
        } else {
            response.status(404).send('resource not found')
        }
    } catch(err) {
        console.log(err)
        response.status(500).send('database error')
    }
})


app.delete('/messages/:id', async (request, response) => {
    try {
        let message = await Messages.findByPk(request.params.id)
        if(message) {
            await message.destroy()
            response.status(204).send()
        } else {
            response.status(404).send('resource not found')
        }
    } catch(err) {
        console.log(err)
        response.status(500).send('database error')
    }
})


app.listen(8080)
```

## Next steps...

Get more in depth knowledge by reading this articles

* [https://www.restapitutorial.com/](https://www.restapitutorial.com/)
* [https://medium.com/pixelpoint/oh-man-look-at-your-api-22f330ab80d5](https://medium.com/pixelpoint/oh-man-look-at-your-api-22f330ab80d5)

