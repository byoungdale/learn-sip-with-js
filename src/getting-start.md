# TODOs:
- using drachtio-freeswitch image to play a recording back to the client (much easier to understand)


# Chapter 1: Setup Your SIP Development Environment

Welcome to the first chapter of 'Learn SIP with Javascript'! In this chapter, we'll guide you through setting up your SIP development environment using sip.js and drachtio. By the end of this chapter, you'll have a basic HTML page with JavaScript that can interact with a SIP server running in a Docker container, and a simple drachtio-srf app to handle signaling from the sip.js client.

## Prerequisites

Before we begin, ensure you have the following installed on your computer:

- [Node.js](https://nodejs.org/): A JavaScript runtime environment that's required to run JavaScript code outside of a web browser.
- [Docker](https://www.docker.com/): A platform for developing, shipping, and running applications inside containers. We'll use Docker to run our SIP server (drachtio).

## Setting Up sip.js

1. **Create a New Project Directory:**
   Open a terminal and create a new directory for your project:

   ```bash
   mkdir sipjs-demo
   cd sipjs-demo
   ```

2. **Initialize a New Node.js Project:**
   Initialize a new Node.js project by running:

   ```bash
   npm init -y
   ```

   This command creates a `package.json` file in your project directory.

3. **Install sip.js:**
   Install the `sip.js` library using npm:

   ```bash
   npm install sip.js
   ```

4. **Create an HTML File:**
   Create a new file named `index.html` in your project directory with the following content:

   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <title>SIP.js Demo</title>
   </head>
   <body>
       <h1>SIP.js Demo</h1>
       <script src="node_modules/sip.js/dist/sip.js"></script>
       <script src="app.js"></script>
   </body>
   </html>
   ```

   This HTML file includes the `sip.js` library and a reference to a JavaScript file `app.js` that we'll create next.

5. **Create a JavaScript File:**
   Create a new file named `app.js` in your project directory. For now, you can leave it empty. We'll add SIP logic to this file later.

## Setting Up Drachtio

Drachtio is a SIP server that we'll use to handle SIP messages. We'll run Drachtio inside a Docker container for ease of setup and isolation.

1. **Create a Drachtio Configuration File:**
   Create a new file named `drachtio.conf.xml` in your project directory with the following content:

   ```xml
   <drachtio>
       <admin port="9022" secret="adminsecret"/>
       <sip>
           <contact>*:5060</contact>
       </sip>
   </drachtio>
   ```

   This configuration file sets up an admin interface on port 9022 and listens for SIP messages on port 5060.

2. **Run Drachtio in Docker:**
   Run the following command to start a Drachtio container with the configuration file you just created:

   ```bash
   docker run --name drachtio -d --rm -p 5060:5060/udp -p 9022:9022 -v $(pwd)/drachtio.conf.xml:/etc/drachtio.conf.xml drachtio/drachtio-server
   ```

   This command pulls the Drachtio image from Docker Hub, starts a container named `drachtio`, and maps the necessary ports. It also mounts your configuration file into the container.

## Setting Up Drachtio-srf

Drachtio-srf is a Node.js framework that allows you to build SIP server applications using the drachtio server. We'll create a basic drachtio-srf app to handle signaling from the sip.js client.

1. **Install Drachtio-srf:**
   In the same project directory, install the `drachtio-srf` library using npm:

   ```bash
   npm install drachtio-srf
   ```

2. **Create a Drachtio-srf App:**
   Create a new file named `server.js` in your project directory with the following content:

   ```javascript
   const Srf = require('drachtio-srf');
   const srf = new Srf();

   srf.connect({
       host: 'localhost',
       port: 9022,
       secret: 'adminsecret'
   });

   srf.on('connect', (err, hostport) => {
       console.log(`Connected to drachtio server at ${hostport}`);
   });

   srf.invite((req, res) => {
       console.log('Received INVITE');
       res.send(200, {
           body: 'Hello

, this is a SIP response from drachtio-srf'
       });
   });
   ```

   This code creates a drachtio-srf app that connects to the drachtio server and handles incoming INVITE requests by responding with a 200 OK message.

3. **Run the Drachtio-srf App:**
   Start the drachtio-srf app by running:

   ```bash
   node server.js
   ```

   You should see a message indicating that the app has connected to the drachtio server.

## Testing Your Setup

At this point, you have sip.js set up in a basic HTML page, a Drachtio SIP server running in Docker, and a simple drachtio-srf app to handle signaling from the sip.js client. In the next chapters, we'll dive into writing JavaScript code to interact with the SIP server using sip.js and handling SIP messages in the drachtio-srf app.

To test your setup, you can open `index.html` in a web browser and check the console for any errors. You should also see the message "Connected to drachtio server" in the terminal where you're running the drachtio-srf app.

Congratulations! You've successfully set up your SIP development environment using sip.js, drachtio, and drachtio-srf. In the following chapters, we'll explore how to use these tools to build a fully functional SIP application with JavaScript




## TODO: integrate this into one chapter

# Chapter 2: Basic SIP.js Usage - Sending an INVITE

In this chapter, we'll dive into using sip.js to create a User Agent Client (UAC) that sends an INVITE request to a SIP server without needing to register. This is a common scenario for initiating a SIP session, such as starting a VoIP call.

## Setting Up the HTML and JavaScript

We'll continue using the `index.html` and `app.js` files from the previous chapter. Make sure your `index.html` file includes the sip.js library and references the `app.js` file:

```html
<!DOCTYPE html>
<html>
<head>
    <title>SIP.js Demo</title>
</head>
<body>
    <h1>SIP.js Demo</h1>
    <script src="node_modules/sip.js/dist/sip.js"></script>
    <script src="app.js"></script>
</body>
</html>
```

Now, let's add the necessary JavaScript to `app.js` to create a UAC and send an INVITE request.

## Creating a User Agent Client

First, we need to create a User Agent (UA) instance. The UA represents the SIP endpoint (in this case, the client). Add the following code to `app.js`:

```javascript
// Configuration for the User Agent
const uaConfig = {
    uri: 'sip:client@example.com', // Your SIP address (can be a dummy address)
    transportOptions: {
        wsServers: 'ws://localhost:5060' // WebSocket server URL
    },
    sessionDescriptionHandlerFactoryOptions: {
        constraints: {
            audio: true, // Request audio
            video: false // Do not request video
        }
    }
};

// Create a User Agent instance
const ua = new SIP.UA(uaConfig);

// Start the User Agent
ua.start();
```

In this configuration, we specify a dummy SIP URI (`sip:client@example.com`) and the WebSocket server URL (`ws://localhost:5060`). We also indicate that we only want audio in our session.

## Sending an INVITE Request

Now that we have a UA instance, we can use it to send an INVITE request. Add the following code to `app.js`:

```javascript
// Send an INVITE request
const session = ua.invite('sip:server@example.com'); // SIP address of the target (the SIP server)

// Listen for session events
session.on('progress', () => {
    console.log('Call is in progress...');
});

session.on('accepted', () => {
    console.log('Call has been accepted!');
});

session.on('rejected', () => {
    console.log('Call has been rejected.');
});

session.on('terminated', () => {
    console.log('Call has ended.');
});
```

In this code, we use the `invite` method of the UA instance to send an INVITE request to the SIP server (`sip:server@example.com`). We also set up event listeners to log the progress of the call.

## Testing the Setup

To test the setup, make sure your drachtio server and drachtio-srf app from Chapter 1 are running. Then, open `index.html` in a web browser. You should see console messages indicating the progress of the call. If everything is set up correctly, the drachtio-srf app should receive the INVITE request and respond to it.

## Conclusion

Congratulations! You've successfully used sip.js to create a User Agent Client that sends an INVITE request to a SIP server without registering. This is a fundamental step in building SIP-based communication applications. In the next chapters, we'll explore more advanced features and scenarios.
