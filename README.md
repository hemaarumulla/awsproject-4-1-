# sqs-project-repo


**STEP 1: ADD REQUIRED REPO FROM OFFICIAL NODEJS WEBSITE.**

```bash
curl -sL https://rpm.nodesource.com/setup_16.x | sudo bash -
```

**STEP 2: INSTALL NODEJS.**

```bash
sudo yum install -y nodejs
```

**STEP 3: VERIFY INSTALLED VERSION.**

```bash
node -v
npm -v
```

**Initialise Node project**

```bash
npm init -y
```

**Install required dependencies**

```bash
npm install express aws-sdk body-parser ejs
```


**STEP 4: CREATE A app.js FILE AND RUN IT.**

Create a file named `app.js` with the provided the below content.

***app.js***


```bash
// app.js

const express = require('express');
const AWS = require('aws-sdk');
const bodyParser = require('body-parser');
const ejs = require('ejs');
const path = require('path'); // Add path module

const app = express();
const port = 3000;

// Configure AWS SDK to use IAM role credentials automatically
AWS.config.update({ region: 'ap-south-1' }); // Replace with your desired AWS region

// Create SQS service object
const sqs = new AWS.SQS({ apiVersion: '2012-11-05' });

// Body parser middleware
app.use(bodyParser.urlencoded({ extended: true }));

// Set EJS as the view engine and set the views directory
app.set('view engine', 'ejs');
app.set('views', path.join(__dirname, 'views'));

// Serve index.html
app.get('/', (req, res) => {
  res.sendFile(path.join(__dirname, 'index.html'));
});

// Serve send.html
app.get('/send', (req, res) => {
  res.sendFile(path.join(__dirname, 'send.html'));
});

// Send message to SQS
app.post('/send', (req, res) => {
  const { message } = req.body;

  const params = {
    MessageBody: message,
    QueueUrl: 'https://sqs.ap-south-1.amazonaws.com/123123123/yourqueue', // Replace with your SQS queue URL
  };

  sqs.sendMessage(params, (err, data) => {
    if (err) {
      console.error('Error sending message to SQS:', err);
      res.status(500).send('Error sending message to SQS');
    } else {
      console.log('Message sent to SQS:', data.MessageId);
      res.redirect('/');
    }
  });
});

// Serve messages.ejs
app.get('/messages', (req, res) => {
  // Retrieve messages from SQS queue
  const params = {
    QueueUrl: 'https://sqs.ap-south-1.amazonaws.com/123123/yourqueue', // Replace with your SQS queue URL
    AttributeNames: ['All'],
    MaxNumberOfMessages: 10, // Adjust as needed
    WaitTimeSeconds: 0,
  };

  sqs.receiveMessage(params, (err, data) => {
    if (err) {
      console.error('Error receiving messages from SQS:', err);
      res.status(500).send('Error receiving messages from SQS');
    } else {
      const messages = data.Messages || [];
      res.render('messages', { messages });
    }
  });
});

// Listen on port
app.listen(port, () => {
  console.log(`Server is running at http://localhost:${port}`);
});

```

this script creates a basic web server that listens on default port 3000, reads the content of an `index.html` file, and sends it as the response when a request is made to the server. The server logs a message indicating that it is running on `http://localhost:3000`.


***create supported files aswell***

***index.html***

```bash
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>SQS Example</title>
</head>
<body>
  <h1>Home Page</h1>
  <p>Welcome to the home page!</p>
  <a href="/send">Go to Send Page</a>
  <br>
  <a href="/messages">View Messages</a>
</body>
</html>

```


***send.html***

```bash
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>SQS Example</title>
</head>
<body>
  <h1>Send Message</h1>
  <form action="/send" method="post">
    <label for="message">Message:</label>
    <input type="text" id="message" name="message" required>
    <button type="submit">Send</button>
  </form>
  <br>
  <a href="/">Go to Home Page</a>
</body>
</html>

```


***messages.ejs***

```bash
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>SQS Example</title>
</head>
<body>
  <h1>Messages from SQS Queue</h1>
  <ul>
    <% messages.forEach(message => { %>
      <li><%= message.Body %></li>
    <% }); %>
  </ul>
  <br>
  <a href="/">Go to Home Page</a>
</body>
</html>

```

***Once all files created, Run "node app.js". Make sure your ec2 instance have role that contains valid access on SQS service***

Once you access webpage on port 3000, Click on "Go to Send Page" to send a message. and "View Messages" to view messages we have in Queue.
