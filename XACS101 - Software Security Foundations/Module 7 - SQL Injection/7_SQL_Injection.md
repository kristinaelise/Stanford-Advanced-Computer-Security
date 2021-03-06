# SQL Injection

---

**Command injection vulnerability** - untrusted input inserted into a query or command
- SQL injection is a form of command injection
- attackers may try to run commands on your database
- prepared statements and bind variables provide a stronger defense

### SQL Injection Impact IRL
- CardSystems, a credit card payment processor
- had a role as a payment gateway 
- entire company ruined by a SQL injection attack in June 2005
- over 265k credit card numbers that were stolen
- had a DB of about 40mil+ CC numbers which were unencrypted
- someone connected a website to the CardSystems database
  - used SQL injection vulnerability in the website to steal data and install software and DB scripts to the server
  - would periodically mail attackers thousands of CC numbers
- company assets got acquired by another company because no one wanted to take on the liability

# 8.1 Attack Scenario

- let's say that pizza site has a function that allows users to see order history
  - example: submit month, and returned all orders made in that month
  - assume simple form with a url param corresponding to the month submitted
- using HTTPS, but that doesn't protect you against SQL injection attacks

### SQL query
- get all orders for the specified month
- `request.getParameter("month");` returns the passed month from the request

### What's the problem?
- no input validation
- what happens if attacker doesn't just input `10` for a month; what if they input `0 OR 1 = 1`
  
```
https://www.deliver-pizza.com/show_orders?month=0%20OR%201%3D1
```
- since the spaces and equal sign are encoded when passed through the URL in the request
- no month 0, so you can't return anything for the month '0'

```
...
WHERE userid = 4123
AND order_month = 0 OR 1=1
```
- `1=1` will always have a result, so all of a sudden your other condition is always `true`, which means that `userid = x AND order_month = 0` is suddenly ignored (X AND Y OR Z)
- will return _everyone's_ user data (or at least everything in that particular `orders` table)
- this could be an issue for companies whose revenue could be predicted from seeing all order data, for example

### A more damaging attack

```
0 AND 1 = 0 UNION SELECT cardholder, number, exp_month, exp_year FROM creditcards
```
- for every row in orders table that's evaluated against predicate, return will be false (since 1 != 0)
- we won't get any customer orders in the result
- _but_ the attacker specified a `UNION` as part of the query, the database will retrieve all of the cardholder names, numbers, etc. and join it as part of the results in the query
- all columns are the same data type, so the query will run fine and return a table of all CC information

### Even _worse_

```
month = 0; DROP TABLE creditcards;
```
- attacker will have deleted all credit card information
- future orders will fail, and the site will be DoSed

### Problematic SQL Statements
- sql injection as a vector of attack is pretty significant
- you can also shut down the DB or control the OS using administrative SQL query commands

# 8.2 Solutions to SQL Injection Attacks

## 8.2.1 Why Blacklisting Does Not Work
- you could look for dangerous characters and blacklist them (ex. quotes)
- you could always miss a dangerous character
- killing quotes, for example, doesn't prevent numeric parameter attacks
- may conflict with functional requirements (ex. the name O'Brien)
- attackers can use JavaScript or encoding, etc. to specify the characters that are blacklisted without explicitly writing them

## 8.2.2 Whitelisting-Based Input Validation

**Whitelisting**: specify what valid inputs should look like and if input doesn't match the pattern of valid input, then don't do the transaction
- one approach is to use **regular expressions**
- not a good idea to ignore input and try to proceed; better to just not respond or go into some kind of failsafe mode
  - "Please restate your input" or something
  - you don't want to tip off attackers so much; don't give them internal errors 

## 8.2.3 Escaping
- escape the character to tell the DB that it's _data_, not control information
- numeric parameters could still be vulnerable, though
- only works for string inputs

## 8.2.4 Second-Order SQL Injection

**Second-Order SQL Injection Attack**: data stored in database used to conduct SQL injection later
- ex. a user's username and password are escaped before being added to database, but not when updating an existing profile

```
admin' --
```
- `--` means ignore the rest of the statement
- quote terminates the string; if `admin` is a real username, their password wil be reset

**tldr;** all parameters are dangerous even if they've been previously escaped

## 8.2.5 Prepared Statements & Bind Variables

**Bind variables**: placeholders guaranteed to be data

**Prepared Statements**: allow creation of static queries with bind variables

- essentially put in placeholders wherever there exists something that you should always consider data
- ex. `userid=?` where the `?` is then replaced with an integer for the current user ID

### Why aren't stored procedures a solution?
- suppose a stored procedure exists called `change_password`
- purpose is to serve as a shorthand for a bunch of other SQL statements; the structure of the statement can still be affected in the same way
- so you still need to use bind variables

- be careful about where the placeholders should be

## 8.2.6 Mitigating the Impact of SQL Injection Attacks
- most attackers poke and prod and make guesses
- a lot use blind sql injection as well -- basically a method of reverse engineering the db schema
  - pose a lot of queries: ex. are there any tables that start with the character A?
  - they'll have programs that will auto-place web requests and interrogate the database using some kind of binary search
- don't display detailed error messages
- make sure to configure the app, server and components to be in production mode rather than dev mode
  - ex. "invalid month" rather than giving away schema information

### Limiting Privileges
- employ principle of least privilege
- don't give users more privileges to the database than they need
  - may need to do inserts/updates, but probably don't need to drop tables
  - configure the database account so that schema changes can't be made like that
  - mitigates damage of sequel attacks

### Encrypt Sensitive Data
- second line of defence
- without a key, the attacker doesn't really have much
- **don't** store the key in the DB since the attacker can just use a SQL injection attack to get it
- **some db allow auto-encrpytion but they still return plaintext queries!!!**

### Hardening DB Server and Host O/S
- ideally keys should be stored in tamper-resistent hardware
- some SQL databases allow users to open sockets -- you shouldn't allow _all_ features by default

 


