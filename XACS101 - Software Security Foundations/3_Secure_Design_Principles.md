# Secure Design Principles

---

# 3.1 Principle of Least Privilege

- give just the  privilege the user needs to get shit done, and nothing else (ex. valet key)

## 3.1.1 Simple Webserver Example
If you run a web server under root, you can access _everything_. Bad news.
- let's say a user wants to get access to a password file stored on the server
- if there are no aditional checks built into the server, and you're running under root then the user can access whatever the heck they want

`GET ../../../../etc/shadow HTTP/1.0`

*DANGER ZONE*

Once the attacker has access, it can launch a *Dictionary Attack* to try to find corresponding passwords.

- so you need to validate the path name, as well as whether the input isn't malformed

## 3.1.2 Canonicalizing Pathnames
- build checks into the server
- we don't want to give access to clients any parts of directory tree that are outside/above where you started
- `checkPath()` makes sure that the target path is below; eliminates any `..` in the pathname
- say this file isn't found if the user isn't in the right directory (basically lying; but you don't want to give the user any information about existing files)

# 3.2 Defence-in-Depth

aka. redundancy/diversity
- don't rely only on one layer of security
- ex. don't just rely on not running the SWS as root, also have other checks in place

## 3.2.1 Prevent, Detect, Contain and Recover
- want to have multiple measures for each of these
- detection is important for network security
- as security professionals we should try to _prevent_ attacks whenever possible
  - but in reality, you can't prevent _everything_
  - so it's important to have *detection, containment and recovery* measures in place for when that happens

Example of recovery measure: SCATANA (9/11 response, landing every plane in the country)

## 3.2.2 Don't Forget Containment and Recovery

# 3.4 Securing the Weakest Link

*Information System is only as strong as its weakest link.*

- War dialing: dial a bunch of numbers and hope some may get picked up by modems and try using (ex. Movie "War Games"

### Weak Links in Today's Systems
- weak passwords
- password systems with vulnerabilities
- people tend to be a weak link (social engineering, for example)
- buffer overflows: a way to take control of a running program even though you're not authorized to
- software design and security design can be a weak link, but implementation vulnerabilities have probably been the worst

## 3.4.1 Weak Passwords
- 1970, Creators of Unix did a study and found 1/3 of users choose a password that can be found in the dictionary
- many websites have password systems that don't really enforce restrictions or are easy to crack
- you have to assume that if there are many many user accounts, some of them are going to become compromised -- so you have to have measures in place (ex. employ principle of least privilege)
- detecting which accounts are compromised and recovering from them is important

## 3.4.2 People
- *spear fishing emails*: emails that claim to be legit but aren't (phishing)
- malicious programmers can put backdoors into programs ("features" like Richard Pryor's character in Superman 3)
- any piece of code that gets put into a prod system should be reviewed by at least one other employee (or two) depending on the system
- if you have a company full of unhappy people, that can lead to insider attacks
- when info's distributed, distribute on an "need to know" basis -- but this can also impact company (particularly hi-tech companies) culture

## 3.4.3 Implementation Vulnerabilities
- if there's user input or untrusted input that can be added to the system, it can be used to hijack a system (ex. SQL injection)
- mixing of control and data can result in a whole slew of problems 

# 3.5 Fail-Safe Stance

- expect and plan for system failure
- ie. Elevator: if power fails, you don't want the elevator ot fail. So they're designed with the expectation that they're going to lose power. Various counter-measures built in.
- how do we have a failsafe stance in software? what if some of the inputs don't arrive? what if they're malformed? what if they have unexpected data?

*Program defensively; program in a paranoid way*

- ex. firewalls shouldn't let anyone through if they fail; for example if an attacker wanted to get somewhere prevented by a firewall, they could cause the firewall to fail. 

## 3.5.1 SWS Fail-Safe Example
- once client decided which file to serve, entire file read from memory and sends to client
- what if amount of memory in server runs out?
  - it crashes.
  - that's good -- that's a fail-safe stance
  - better to let server crash and prevent access to any other clients 
- if an attacker requests a file that's large enough, it can force it to crash and result in the same denial of service issue 

*What if there are no files big enough?*
- in Linux, there's a virtual file `/dev/random` often used to get cryptographic key data / random data
- if you open up the file and keep reading, it'll just keep giving you more random bits
- so if an attacker says get `/dev/random`, it'll keep giving the web server more to read infinitely 

## 3.5.2 Checking the File Length
- initial fix doesn't work for `/dev/random` since length not accurately reported (doesn't exist on disk, really, so file length is returned as 0)

## 3.5.3 Don't Store the File in Memory
- to avoid crashing, can just not store the file in memory
- instead of reading into buffer and then outputting once entire file is read in, can just read/write one character at a time and continue so nothing is stored in memory
- but if you access `/dev/random`, the SWS won't run out of memory but _will_ get tied up, since it'll be an infinite read in/write out

## 3.5.4 ... and Impose a Download Limit
- handles the issue with `/dev/random` by limiting downloads
- if you choose a limit that's too low as your max, then legit files will get truncated because they obviously won't have each character read-in

# 3.6 Secure by Default
- there are many features that might be available in software, and there can be many vulnerabilities -- to mitigate risk, don't turn on all the futures. 
- maybe only enable 20% of features used by majority of the population

*Microsoft, early 2000s*
- implemented this approach when they were receiving a large number of attacks
- in 2001, the OS would start services by default (like servers), and worms started taking advantage of these "on-by-default" functionalities
- all of these "on-by-default" features led to worms being spread through systems


- this is a *"more secure by default"* configuration since every features is likely to have some kind of vulnerability
- you can harden any system regardless of the kind of system
- if you think of your program as a surface, you may have multiple dents in the surface of your armour and an attacker can take advantage of them
  - the more features you expose, the larger the attack surface


# 3.7 Simplicity 
- software is hard to get right; hard to get to perform from a correctness standpoint _and_ to be completely secure
- *complexity is your enemy in security*
- go for simplicity when you're worried about security -- easier to understand; easier to audit

ex. want to check whether a user/pass combo is valid. 
- if you have 3 different ways a user can enter the program, and 3 different areas a check is being done, they're likely not all going to be consistent
- advantageous to employ a choke point: *one point of centralized code through which all of the checks must pass (authentication, control, etc.)*
  - the more you can do to keep the choke point code small and audited, the more secure your system is likely to be
- *reduce your attack surface*

# 3.8 Usability
- there are often tradeoffs between usability and security
- users tend to not read documentation, and tend to have problems using software
- if you're shipping a product and the user has to be able to do certain things to use the product securely, you shouldn't rely on a set of documentation that you're assuming the user will read
  - *make it secure by default*

*Why Johnny Can't Encrypt : Alma Whitten*
- looks at a system that allowed users to do pgp encryption (pretty good privacy)
- paper found that due to the design of some of the initial pgp applications, most users did insecure things even though the whole point of the application was to let you be _more_ secured (ex. some sent out private keys rather than public)

# 3.9 Security Features Do Not Imply Security
- ex. website may have SSL but, just because a website uses SSL doesn't mean it's secure
  - in the 90s, what it meant to run a secure webserver was running SSL
  - you could be running SSL and the user can have a secure connection to the _website_ but if the website allows the user to make a weak password, the attackers can exploit that

*Security features can _help_ security, but do not _imply_ security. Many things have to be in place for security to really be achieved.*

*"Security is a process, not a product."*



