## Oauth - Webserver

## Author: Sara Bahrini, David Chambers, Tim Li, Trevor Stam

## Links and Resources
* [Webserver](https://github.com/dlchambersjr/lab-36-web-server)
* [Auth Server](https://github.com/dlchambersjr/lab-36-auth-server)

## Challenge
Create a login and authentication server using Facebook OAuth.

## Facebook Developer Documentation
[Facebook Login for the Web](https://developers.facebook.com/docs/facebook-login/web)

## Process
Facebook's instructions to set up a login with OAuth is very straightforward with step-by-step directions and examples in their developer docs.

1. Create a developer account - you can use an existing Facebook account. 
2. Create a Facebook App in the dashboard - `Lab-36-OAuth`

3. Follow the steps as outlined in [Facebook Login for the Web](https://developers.facebook.com/docs/facebook-login/web).

## Obstacles
The first challenge was to separate the web server and Auth server into two separate repos.  This was accomplished with relative ease.

The primary challenge we discovered involved HTTPS.  As of October 6, 2018, Facebook required all URLs used for the redirect to utilize HTTPS instead of HTTP.

Node and express do not inherently provide a HTTPS layer.  So the hunt for options started.  We found a number of suggestions, which led us to implement the steps found at [<TimonWeb>](https://timonweb.com/posts/running-expressjs-server-over-https/):

1. Generate a self-signed certificate: ```openssl req -nodes -new -x509 -keyout server.key -out server.cert```
2. Enable HTTPS in Express:
```
var express = require('express')
var fs = require('fs')
var https = require('https')
var app = express()

app.get('/', function (req, res) {
  res.send('hello world')
})

https.createServer({
  key: fs.readFileSync('server.key'),
  cert: fs.readFileSync('server.cert')
}, app)
.listen(3000, function () {
  console.log('Example app listening on port 3000! Go to https://localhost:3000/')
})
```

At first glance this seemed to solve the problem... that is until we attempted to access the server in Chrome.  Concerned with security, Chrome did not accept our self-signed certificates.  It did give us an option to bypass the warning and render our page as regular HTTP. We chose to try the login this way.

We next added the necessary script to the web server's HTML (detailed below).  The Facebook script performed as expected until it was time to act on the response data.  Our expectation was that the response would be redirected to our Auth server `https://localhost:3000` as identified in the Facebook application settings. However, Facebook would not redirect to the Auth server as requested and only returned the reposne to our web server `HTTP://localhost:8080`.

After a couple of hours of research and instructor involvement (every instructor on-site was able to speak into the issue), we have concluded that Facebook was recognizing, as Chrome did, that our certificates were not issued by an actual SSL issuer.  As a result, it returned an empty response to our Auth server and we were not able to continue the project.

## Modules
### `index.js` -> server on port 8080
* Creates the server and renders `index.html` from `/public`

### `public/index.html` -> login button
* The login button is rendered using the JavaScript SDK included in the script tag.  It triggers FB.login() that is created with the SDK.

#### `FB.login()` -> authentication pop-up
* When the login button is clicked the SDK renders the authentication form.

#### `checkLoginState(arity 1)` -> statusChangeCallback(arity 1)
* Used to pass the response from the Authentication server into the function that handles the status of the login.
* receives the response from Facebook that includes the following object: 
```
{
    status: 'connected',
    authResponse: {
        accessToken: '...',
        expiresIn:'...',
        reauthorize_required_in:'...'
        signedRequest:'...',
        userID:'...'
    }
}
```
#### `statusChangeCallback(arity 1)` -> Text
* logs the authentication server response to the console.
* If 'connected' it calls `testApi()`.
* if 'not connected' it prompts the user to log into the app.

#### `testAPI()` -> Text
* Logs 'successful login for **user**' in the console.
* Renders 'Thanks for logging in **user**' to the DOM.


#### Running the app
* Web server - `node index.js` from the root folder
* Auth Server - `node index.js` from the root folder

#### UML
[Facebook OAuth](https://raw.githubusercontent.com/dlchambersjr/lab-36-web-server/master/facebook-oauth-uml.jpg)
