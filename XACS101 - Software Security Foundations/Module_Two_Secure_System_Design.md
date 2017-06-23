# Secure Systems Design

---

# 2.1 Understanding Threats

One of the most important tasks of security systems designers is identifying most significant threats & implementing preventative or detection or containment counter-measures.

## 2.1.1 Defacement

**Online vandalism** 

- replacing legit pages with illegitimate ones
- often targets political sites
- more or less of a concern depending on the kind of site
	- for political sites, can be significant since the message is the purpose

Ex. Defacement of White House website by anti-NATO activists and Chinese hackers

- in the 80s, were a lot of kids just trying to prove their hacker worth by defacing sites; have become more economically motivated 

## 2.1.2 Infiltration

**Gaining unauthorized access to resources of the computer system**

- infiltrators want to get full control of the site
- infiltration sites said to be "owned" 
- you want to be able to detect these kinds of attacks

Ex. Defacement attack would be bad for a financial website because it could cost the trust of the users. However, if fully infiltrated, that's far worse.

## 2.1.3 Phishing

**Attacker sets up spoofed site that looks real and lures users into giving credentials**

- usually sent through an email asking users to "verify" their account
- in early 2000s would take the form of emails with links, would attack popular institutions

## 2.1.4 Pharming

**Attacker redirects legit url to a spoofed site by attacking translation process of the domain**

- domain names don't mean much to the browser; it's the IP address that matters
- Pharmers attack translation process of the domain going to the IP address
- DNS cache poisoning to redirect a legit url to the spoofed site
- since the DNS translation is "poisoned", and the result gets cached, it's poisoned for future replies

## 2.1.5 Insider Threats

**People who work for organizations leak data**

- studies show a large number of successful attackers were by insiders
- database admins have been given root access to a company's database and can basically do what they want

**Separation of Privilege**
Instead of giving complete access, configure so they can only be given credentials that allow them to do only what they need to do rather than read/write everything.

**Least Privilege Principle**
Give privileges that are no more powerful than need be. Ex. When you buy a car, you get a master key and valet key. Valet key can only start the car, but not access trunk/glove box/etc.

Mitigating insider threats also include background checks.

## 2.1.6 Click Fraud

**False advertising clicks by bots or 3rd world click farms used to deplete ad budgets and bump other ads to more visible space**

- Two main attacks:
	- Competitor click fraud attacks
	- Site publishers clicking on their own ads to get revenue

## 2.1.7 Denial of Service (DoS)

**Attacker inundates server with packets causing it to drop legit packets**

- makes services unavailable
- downtime = lost revenue, therefore big threat for e-comm vendors and financial institutions
- financial institutions - starting in 2003, 2004 - would get ransom letters: check your web logs, I've DoS'd your site, pay me some amount and I won't do it during business hours
- defending against DoS involves over provisioning so you can handle extra traffic 

## 2.1.8 Data Theft and Data Loss

**Data goes missing**

- Bank of America: backup data tapes lost in transit
- ChoicePoint: kept data about users; attackers were able to pose queries to database and get sensitive info about users
- Veterans Administration: mid 2000s, employee took a computer drive home and was burglarized 
- California passed a law requiring companies to disclose theft/loss
	- This law requires disclosure when unencrypted
	- if encrypted, where's the key? Hopefully not with the data
- Card Systems: credit card payment processor was compromised in 2004; had a database with card numbers and were responsible for handling POS card swipe authentication
	- someone connected a database to the site and do a SQL injection
	- stole a ton of shit
	- led to PCI Data Security measures being implemented

## Threat Modelling ###

**Determine kinds of threats an organization faces**

What are some of the worst things that could happen to get in the way of our operations, trust of customers, etc?

Ex. Civil Liberties website -- defacement.
Ex. Financial Institution -- denial of services; compromise of accounts
Ex. Military -- infiltration; access to classified data

Before figuring out where to invest security resources, the goal of threat modelling should be to figure out where to spend those dollars to mitigate effects of threat.

### Threat Modelling Frameworks

**STRIDE**: developed by Microsoft

What are the different ways that data could be:

* Spoofing identity
* Tampering
* Repudiation
* Information Disclosure
* Denial of Service
* Escalation of Privilege

# 2.2 Designing-In Security

One key problem with system design is looking at security _after_ a system has been built. It's not just a feature you add-on later.

Sometimes you can incrementally improve security, but it's not ideal.

**Design features with security in mind**

Think about security goals you want to achieve; have security requirements and goals _part_ of the requirements document.


## 2.2.1 Windows 98

**Diagnostic Mode**

- F8 while booting; lets you bypass password protections giving attacker complete access to hard disks/data
- device drivers run with OS privileges 
- if you installed some piece of hardware + driver and system wouldn't boot properly, when you entered Diagnostic Mode it won't load the device drivers/OS parts
- goal was for it to be a diagnostic mode, but it gave restricted version of OS but also full access to the entire disk
- so...no real point for username/pass 
- the username/password feature was added as an afterthought, and this is what happened
- safe mode probably would have been designed differently otherwise

## 2.2.2 The Internet

**Commercialization led to new hosts connecting with existing hosts regardless of trust**

- all nodes originally university or military, so trustworthy (DARPA)
- assumptions of trust no longer true once new nodes added from external sources
- started deploying firewalls to only let in "trusted" traffic
- but there weren't as many security measures built into TCP/IP protocol, so this was hard
	- IP protocol that lets OSes on different machines send small data-grams to each other
	- you label an envelope with your address and recipient's address
	- host looks at the addresses
	- when all nodes were trusted, you could put any source and destination address
	- but if you could put any address that gets routed to some other host, you could lie about what your source was
		- IP whitelisting: I will only process a packet if it comes from a certain IP address
		- if you can lie about your IP address, you can pretend to be another system
- So new layers added onto IP (TCP)
	- added sequence numbers which made forging addresses more complicated
- IPSec not introduced until much later -didn't really reach significant deployment until the 90s
- Putting these security counter measures into the original protocols but it might have made them harder to use/set-up/use, which may have gotten in the way of adoption

**The idea is to come up with security that's good enough for where the technology is currently in order to not impact adoption, and add additional measures later.**

## IP Whitelisting & Spoofing

**IP Whitelisting**: only accept packets from host with a particular IP address

**IP Spoofing attack**: attacker mislabels source address on packets, slips past firewall 

Today, you have to do more than just mislabelling source addresses. Check out the book for examples of how to get around simple IP Whitelisting. 


## 2.2.3 Turtle Shell Architectures

firewalls that try to defend systems based on the IP addresses traffic is coming from, or what IP port numbers traffic is coming from are said to be _Turtle Shell Architectures_.

**Some inherently insecure system and you're trying to put some secure shell around it.**

- The "shell" can defend certain attacks, but if something gets through then the internal software is very exploitable

Ex. Death Star: strong outer defence, but vulnerable
Ex. Firewalls guard vulnerable inner systems

_Many of the attacks that take place today are like this_

# 2.3 Convenience and Security

**Good technologies strive to increase security at only a slight inconvenience to the user.**

Sometimes inversely proportional relationship
- More secure can mean less convenient, and vice versa
- If too inconvenient then users try to find workarounds (ex. writing down passwords)

