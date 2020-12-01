# HTML and JavaScript interface

In the first part we've built a web server capable to serve static files and expose each CRUD \(create, read, update and delete\) operation. In this second part we will develop a minimal interface to demonstrate the communication between the frontend and the backend components.

Note that frontend development is an iterative process. You usually start from some sketches and aim to get then validated with the users. Then you move into more details as you create a high fidelity design. 

In the end you turn the design into HTML and CSS. Design principles are not covered here since we will be focusing on the technologies and we will be using frameworks like Bootstrap or Material UI.

## HTML Document

This is the html document to start from. It contains the table and the form we will be using to implement the CRUD operations.

```markup
<!DOCTYPE html>
<html>
    <head>
        <title>Message list</title>
    </head>
    <body>
        <h1>Messages</h1>
    </body>
    <div id="content">
        <table style="width:100%;">
        <tr>
            <th>ID</th>
            <th>Subject</th>
            <th>Name</th> 
            <th>Message</th>
            <th>Actions</th>
        </tr>
        <tr>
            <td>1</td>
            <td>Subject data</td>
            <td>Name data</td>
            <td>Message data</td>
            <td>
                <button>Edit</button>
                <button>Delete</button>
            </td>
        </tr>
    </table>
    
    <form>
      <input type="hidden" name="id" id="id" /><br />
      Name:<br />
      <input type="text" name="name" id="name" /><br />
      Subject:<br />
      <input type="text" name="subject" id="subject"><br/>
      Message:<br />
      <textarea name="message" id="message"></textarea> <br/>
      <input type="submit" value="Save message">
      <input type="reset" value="Cancel">
    </form>
</div>
</html>
```

## Displaying data on load

Each time a page is loaded the `onload` event is triggered. We will use this event to fire the showMessages\(\) function

```markup
<script type="text/javascript">
    window.onload = () => {
        console.log('Page loaded')
    }
</script>
```

## Reading messages from the backend \(GET\)

```javascript
async function showMessages() {
    try {
        let results = await fetch('/messages').then(response => response.json())
    
    
        let html = ` <table style="width:500px;">
                <tr>
                    <th>ID</th>
                    <th>Subject</th>
                    <th>Name</th> 
                    <th>Message</th>
                    <th>Actions</th>
                </tr>`
    
        results.forEach(function(element) {
            html += `<tr>
                        <td>${element.id}</td>
                        <td>${element.name}</td>
                        <td>${element.subject}</td>
                        <td>${element.message}</td>
                        <td>
                            <button onClick="editMessage(${element.id})">Edit</button>
                            <button onClick="deleteMessage(${element.id})">Delete</button>
                        </td>
                    </tr>`
        })
    
        html += `</table>`
        document.getElementById('content').innerHTML = html
    } catch (error) {
        console.log(error)
    }
}
```

Add the function on the event`onload`

```markup
<script type="text/javascript">
    window.onload = () => {
        console.log('Page loaded')
        showMessages()
    }
</script>
```

## Handling form submit

We noticed that the default behavior of a form is to submit data via the GET method. We need to override this by passing a function on the `onSubmit` handle. This function will get the event that was triggered as a parameter.

```markup
<form onSubmit="saveMessage(event)">
    ...
</form>
```

```javascript
async function saveMessage(event) {
    event.preventDefault()

    let id = event.target.id.value
    
    let data = {
        name: event.target.name.value,
        subject: event.target.subject.value,
        message: event.target.message.value
    }
    
    let url = ''
    let method = ''
    
    if(id) {
        //make a request to PUT /messages/:id
        url = '/messages/' + id
        method = 'PUT'
    } else {
        url = '/messages'
        method = 'POST'
    }
    
    try {
        let result = await fetch(url, {
            method: method, 
            headers: {
            'Content-Type': 'application/json'
            },
            body: JSON.stringify(data)
        }).then(response => response.json())
        
        showMessages()
    } catch(err) {
        alert('unable to save message')
    }
    
}
```

## Delete method

DELETE`/messages/:id`

```javascript
async deleteMessage(id) {
    try {
        let url = '/messages/' + id
        let result = await fetch(url, {method: 'DELETE'})
        showMessages()
     } catch(err) {
         alert('unable to delete message')
     }
     
}
```

