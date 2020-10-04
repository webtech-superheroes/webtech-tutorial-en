# RESTful API

A RESTful API is a method to expose data and perform CRUD operations on top of the HTTP protocol.

In this exercice we will be creating a web server using NodeJS and the ExpressJS framework. Data will be stored in MySQL and we will be using Sequelize to model the tables and relations.

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

* [ ] TODO: crează un director denumit `frontend`
* [ ] TODO: adaugă în directorul creat un fișier `index.html`
* [ ] TODO: deschide în browser aplicația accesând adresa URL \([http://ip:8080](http://ip:8080)\)

## 3. How to install MySQL and create the database?

If you havent done this allready, follow the tutorial published [here](../development-environment/mysql.md).

Login into the console and create a database called **profile**:

```sql
create database profile;
```

* [ ] TODO: check if the database was created by executing `show databases;`
* [ ] TODO: exit the console by typing `exit` and pressing enter

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

Pentru a permite crearea de înregistrări expun o metodă de tip POST.

Fiecare endpoint din API-ul REST este definită de metoda HTTP și numele resursei la care se referă.

```text
POST /messages
```

Clientul va trimite datele prin cererea HTTP în format `json` sau `urlencoded`. Pentru a interpreta aceste date voi adăuga două bodyParser. Apoi definesc endpoint-ul apelând functia `app.post`.

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

Conținutul trimis în body va fi accesibil pe proprietatea `request.body` ce va fi pasat ca parametru pentru modelul Sequelize în metoda create\(\).

Metoda **create** Sequelize va genera automat instrucțiunea ``INSERT INTO messages (`subject`, `name`, `message`) VALUES ('test','test','test')``.

Dacă comanda va fi executată cu succes rezultatul va fi returnat prin functia callback definită pe metoda `then(callback)`

Dacă aplicația va întâmpina o eroare la scriere în baza de date va apela functia callback definită pe metoda `catch(callback)`

Pentru a testa enpoint-ul creat folosim Postman.

![postman post method](../.gitbook/assets/00701-postman-post.png)

1. Selectează metoda POST
2. Adaugă adresa URL a resursei
3. În tabul body alege optiunea raw și adaugă un obiect json care descrie resursa creată
4. Selectează tipul de continut application/json
5. Trimite cererea HTTP apăsând butonul Send

## 8. Cum expun datele dintr-un tabel folosind metoda GET?

Pentru a lista datele dintr-un tabel vom expune două enpoint-uri. Primul care returnează toată lista de mesaje și al doilea care returnează un mesaj după un ID specific.

```text
GET /messages
```

```text
GET /messages/1
```

Pentru interogarea tabelului modelul sequelize expune metode precum `findAll`, `findByPk`, `findOne`. Functia va returna un obiect de tip `Promise` ce va fi executat imediat ce datele sunt primite de la serverul de baze de date. Mai multe detalii despre interogari aici: [http://docs.sequelizejs.com/manual/tutorial/querying.html](http://docs.sequelizejs.com/manual/tutorial/querying.html)

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

## 9. Cum actualizez o înregistrare folosind metoda PUT?

Actualizarea unei resurse se realizează prin intermediul metodei PUT

```text
PUT /messages/1
```

În primul pas se interoghează baza de date. Dacă resursa nu există serverul va returna statusul 404 și mesajul „not found„.

Dacă resursa a fost găsită o actualizez apelând metoda `update()` cu obiectul trimis în body-ul cererii HTTP.

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

Pașii pentru a testa metoda PUT sunt aceiași ca pentru metoda POST.

![postman put](../.gitbook/assets/00901-postman-put.png)

## 10. Cum șterg o înregistrare folosind metoda DELETE?

Ultima metodă permite ștergerea unei resurse

```text
DELETE /messages/1
```

Dacă resursa este găsită după ID, apelez metoda `destroy`,iar sequelize va transmite către baza de date instructiunea sql ``DELETE FROM `messages` WHERE id = 1`` și va returna un obiect de tip `Promise`. În final serverul web va raspunde cu statusul **204 NO CONTENT** semnalând că cererea a fost îndeplinită cu succes.

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

* [ ] TODO: testează enpoint-ul în Postman folosind metoda DELETE

## Next steps...

Dacă ai reușit să parcurgi tutorialul până aici, în primul rând felicitări pentru efort!

Iată câteva resurse care te vor ajuta să aprofundezi dezvoltarea de servicii web REST:

* [https://www.restapitutorial.com/](https://www.restapitutorial.com/)
* [https://medium.com/pixelpoint/oh-man-look-at-your-api-22f330ab80d5](https://medium.com/pixelpoint/oh-man-look-at-your-api-22f330ab80d5)
* [https://www.toptal.com/laravel/restful-laravel-api-tutorial](https://www.toptal.com/laravel/restful-laravel-api-tutorial)
* [https://www.codementor.io/sagaragarwal94/building-a-basic-restful-api-in-python-58k02xsiq](https://www.codementor.io/sagaragarwal94/building-a-basic-restful-api-in-python-58k02xsiq)

