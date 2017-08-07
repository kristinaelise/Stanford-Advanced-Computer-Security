# Cross-Domain Security in Web Applications

**Domain**:hostname or internet domain where the app/service/website is hosted ex. google.com
- all code that supports the web application is hosted at that domain

**Cross-Domain**: security threats due to interactions between the domain and another application running on another host

# 10.1 Interaction Between Web Pages from Different Domains
- Back in '95/'96 and Java applets were first released by Sun Microsystems and Netscape, browser companies included a Java virtual machine inside the browser
- whole point of this was to allow devs to write small programs that would run in the page
- wouldn't want 2 applets to run at the same time or steal info from each other
- led to creation of same-origin policy
 
**Same-Origin Policy**: When code comes from one domain, it shouldn't interact from another domain
- aka. cross-domain security policy

- a lot of these cross-domain interactions occur because of cookie authentication and http authentication
  - http initially a stateless protocol and still is, web apps need a way to maintain state across interactions between clients
  - when you combine all different protocols together, certain interactions allow for security vulnerabilities

## 10.1.1 HTML, Javascript and the Same-Origin Policy
- document object model (DOM) organizes the components on the page
- behaviour directives through Javascript
- support style layout through CSS
- in developing a number of technologies, same-origin policy exists
  - if code comes from one particular origin, it should only be able to interact with that origin and not other destinations
  - if there are two different destinations -- good origin, and attacker -- don't want attacker to read data from good origin

**Origin**: a protocol + hostname + port (doesn't include the path)

**Same-Origin Policy**: script comes from a particular host and is served by a particular port on that host only allowed to access properties (cookies, DOM objects) of documents of the same origin

### Same Origin
`http://www.examplesite.org/here/` & `http://www.examplesite.org/there`
- same protocol: http; host: examplesite; default port 80

### Different Origin
`http://www.examplesite.org/here/` & `https://www.examplesite.org/there` 
- different protocol: http vs https
- different port since https implies 433

`http://www.examplesite.org/here` & `http://www.examplesite.org:8080/thar`
- different ports: 80 vs 8080

`http://www.examplesite.org/here` & `http://www.hackerhome.org/yonder`
- different host name

## 10.1.3 HTTP Request Authentication
- when a client comes to a server, the server can't by default distinguish the client request from a different client request
- additional information needs to be provided to demonstrate it's the same client

**HTTP Authentication**: username/passwd automatically supplied in HTTP header
- HTTP protocol has a feature that allows a user to specify a username/password
- ex. browser dialog box that comes up rather than a field on the site itself
- if the same username and password shows up multiple times, it's the same user

**Cookie Authentication**: credentials requestedin form, after POST app issues session token
- same cookie provided back to server with each request

**Hidden-form authentication**: hidden form fields to transfer session token

# 10.2 Attack Patterns

1. Cross-Site Scripting (XSS)
2. Cross-Site Request Forgery (XSRF)
3. Cross-Site Script Inclusion (XSSI)

## 10.2.1 Cross-Site Scripting
- what could be done if an attacker gets their own code running on your application?

Ex. app has a query param in the URL that's printed on the page
- input data not filtered
- attacker injects some malicious script

### Pizza Order Example
- attacker gets user to click on some link containing malicious script (could be through email, IM, whatever)
- attacker modifies the `submit_order?price=5.50`
- inserts script in place of the price input
- embeds script by echoing on the page as well as within the form
- goal of the attacker is to get the script run by the browser
- browsers are forgiving with syntax, so a lot of the times there will be "syntatic glue" - that's not really needed a lot of times
- stealing cookies lets attacker appear to be the same as the user
- **stealing cookie allows attacker to be the man in the middle**; allows attacker to impersonate user

### XSS Exploits

**Stealing Cookies**
- script could grab user's cookie and send to their own site
- gives attacker full access to user's session

**Scripting the Vulnerable App**

**Modifying Web Pages**

### XSS Implications
- attacker gets to inject random script into your app and makes the browser think it's legit
- **XSS is a back door that allows the attacker to control your web application**

### Reflected XSS
- attacker injects a script that's immediately reflected in the response
- what was demo'd in the query param example with the Pizza app

### Stored XSS
- script delivered after some time
- stored somewhere in meantime
- attack is repeatable; more easily spread

### MySpace Stored XSS Worm (Oct 2005)
- figured out hwo to get around XSS preventative measures
- whenever someone viewed his page, they automatically became his friend
- attack written so that when anyone viewed one of his friends pages, they were also added as a friend
- propagated from one user profile via friend connections
- 1 to over 1M "infected" profiles in < 6 hours
- MySpace went offline to fix it
- Samy Kamkar: first FBI conviction for XSS attack; was the larges XSS attack to date

### Preventing XSS
- while SQL injection could be handled by sanitizing input, it's not the right solution for XSS
- XSS is an output sanitization problem; not an input validation problem
- problem wasn't that there was JS inside the input -- the problem occurs when the script is _output_
- browser is interpreting data as control
- escape output before it goes out to a webpage
  - ex. `htmlentities()` in PHP

`<` => `&lt;`
`>` => `&rt;`
etc...

- HTML escapting is appropriate anytime you're outputting text to a page or outputting a value to a form
- depending upon where the data is output on a webpage (URL or could appear in a piece of JS that runs) you may need different types of escaping 
  - see slide 9/18 of Module 9 PDF 

## 10.2.1 Cross-Site Request Forgery (XSRF)
- malicious site initiatiates HTTP requests to app on user's behalf without their knowledge
- cached credentials setnt to our server regardless of who made the request
- when a user is logged into multiple sites, the user has authenticated all of those sites

ex. User logged into her bank; lured to some other website while being logged into bank website
- in a normal interaction, Alice is at her bank and enters username/password
- bank gets username/password, checks against database and determines that it's authentic
- sends Alice the sessionID in a cookie that browser continues to send for any interaction with the bank
- Alice goes to view bank balance and session ID used
- Bank app may not have XSS vulnerability, so attacker can't inject any code

### How the Attack Works
- Alice gets login page
- fills out username/password
- bank authenticates and sends sessionID
- attacker gets Alice to visit webpage called `evil.html`
- doesn't ask user for password
- page has a tag that says there's an image, go request it from the bank
- webpages can request from anywhere on the internet
- putting url in the image tag takes advantage of the fact that the browser will make an http request to bank
- attacker tells Alice's browser to make HTTP request to `paybill` script
- browser is told to make http request to `paybill` at `bank.com`

### Why Should Bank.com Do This?
- Alice's browser is already logged in to the bank
- Any request made by the browser will include the cookie
- Bank gets the request _with_ the cookie and authentic session ID
  - looks legit to the bank.com server
  - so server will complete the request

### What Happens After?
- money won't go directly to the address
- address will be some mule whose job is to pick up some cheque and deposit into some other account

### XSRF Impacts
- malicious site can't read info, but can make _write_ requests
- no code injected
- who should worry about XSRF?
  - apps with server-side state (user info, profiles with username/password)
  - apps with financial transactions
  - apps that store user data

# 10.3 Preventing XSRF
- HTTP request was indistinguishable from the legit request
- distinguish request from legit source vs. one by an attacker
  - inspect Referer headers
    - logs would show `evil.org`
    - sometimes proxies strip referers because of privacy 
  - validation by user-provided secret
    - asked to re-enter password to validate some action 
  - validation by action token
    - want to validate that bank.com's own server made the request

## 10.3.3 Validation via Action Token
- in the form the user fills out to pay the bill, introduce a token generated by origin domain that only _they_ know
- pay bill script checks for existence of this token that can only be generated by origin domain

### XSRF Attack Foiled
- alice logs in
- cookie, sessionID, etc.
- within form, token has a value that was generated using secret key
  - hidden parameter, for example, in the form
- bank.com server checks not only for presence of token, but that it was generated by a secret key only known to the server
- attacker doesn't know what token to fill in, and even if they guessed, it'll always be invalid because the secret key is unknown

### Generating Action Tokens
- could come up with a counter, construct a message authentication code with the counter where it users a secret key
- when transaction is attempted, counter will be one of the params of the webpage and origin domain can check whether token (signature) matches the counter
  - if it's _just_ a simple counter, attacker can use the application as an "oracle" to extract hidden form, etc.

## 10.2.2 Cross-Site Script Inclusion (XSSI)
- allows attacker to steal data
- many JS libraries are available to render menus, etc. and it's common that those are used
  - can be leveraged by attackers to steal data

### XSSI Example: Ajax Script
- bank.com has some script that uses a .js library on their own site and an AJAX feature
  - ex. make HTTP request without reloading page
  - processes data sent back

### Normal AJAX Interaction
- Alice logs in and authenticates -> cookie
- makes a request to view balance, cookie provided
- data sent back; page says rendering data - get account number and balance
- renders data with passed paramateres

### XSSI Attack
- attacker can take advantage of fact that the script on the bank's site can be called from another site
- attacker provides own implementation of `renderData` function
- Alice logs in, authenticated, cookie, can view balance
- Alice gets lured to view some malicious site
- malicious site sends back page that contains .js with its own `renderData` function of its own
  - when that function gets called, any data there calls another function which sends params to some attacker
  - includes script from bank.com
  - script gets data, and callback is `renderData` function - but now it's the attacker `renderData` function being called
- when webpage gets to Alice's browser, Alice is already logged in so it gets data from bank site
  - calls `renderData` function and sends account number and balance to the attacker
- attacker able to include script from bank site in their own site to get data they want and is acquired because Alice is already authenticated  

# 10.4 Preventing XSSI
- can prevent using an action token
- similar problem with preventing XSRF where you need to distinguish 3rd party references from legit ones 
