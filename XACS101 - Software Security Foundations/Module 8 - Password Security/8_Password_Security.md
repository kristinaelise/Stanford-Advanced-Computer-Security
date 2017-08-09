# Password Security

# 9.1 A Strawman Proposal
- inital strawman example -- insecured password manager in `java`
- keeps an in-memory cache in a Hashtable data structure
- corresponding code to pull off disk and into memory 
- sync when new users or updated passwords

**How do you add new username/passwords?**
- just adds to the hash table

**How do you check their passwords?**
- user may supply username and password to be checked
- access in-memory hash table that's keyed by username with corresponding values the password
- if username DNE, not authentic
- if is an entry for username, compare value with supplied password

**Disk and Cache**
- hashtable backs up to disk
- init grabs backed up version to load into hash from disk
- flush just backs up the hash table

**What happened if hacker got the file?**
- password file becomes more valuable as number of users and data increases over time

# 9.2 Hashing
**DES** = example of symmetric encryption algorithm
- where do you store the key?
  - what if we encrypted all passwords without a key?

**"one-way encryption" / "hashing"**: for every hashed password you put inside the file, there's no way to decrypt it
- if file stolen, all passwords are stil uncompromised
- security of approach becomes dependent on how hashing takes place

**Pre-Image Resistant**: computationally infeasible to decrypt with only the hashed output

**SHA-256**: "secure hashing algorithm" - 256 bits produced
- invented by Ron Rivest in early 90s (The 'R' in RSA)
- in password file, each line is username : {hash of password} 
  - example in slides is base 64 hash of "automobile" (first example)
- given a hash it's computationally infeasible to figure out a password
  - can you go forwards?

# 9.3 Offline Dictionary Attacks
- assume attacker gets the file

**Offline dictionary attack**: attacker has the file and doesn't need to try throwing usernames and passwords at the actual system itself, but rather offline against the file

**Online dictionary attack**: sending different combination attempts to the system directly

- if attacker takes advantage of the fact that many users use simple words for passwords, it may be possible to do look-ups for potential matches
- attacker can compute hashes of certain common passwords and compare them against the file

# 9.4 Salting

**Salting**: increases the number of combinations the attacker will have to try
- can serve as a viable defense if the number of combos is high enough

**Salt**: random number that -- in this example -- acts as a third field to password storing

- when a new entry is added to the password file, random number is generated, and concatenate password (ex. `automobile`) with salt (ex `1515`) and store that
- even if attacker still gets the whole password file, the hash will be different because of the added salt

**"Salted hash"**

- to check the password, get username entry
  - if user DNE, not authentic
  - if does, check whether hashed password input is equal to salted hash password from the file

### Good News ###
- dictionary attack against arbitrary user now much harder
- hacker would now have to try every dictionary word, hashed with every possible salt, against a particular user

Attacker must hash `n*min(v, 2^k)` strings for an n-word dictionary, k-bit salts, v distinct salts
(since salts will be in password file)
- if many users all using different salts, it's `2^k` harder

### Bad News ###
- useful for arbitrary hacker
- not so good for specific account access since the password file shows the salt that is associated with a user
  - much faster to just get the salted hash against all dictionary words with single salt (assuming user doesn't have a secure passwords)

# 9.5 Online Dictionary Attack
- attacker tries multiple combinations against live system
- monitor attacks for number of failed attempts and block or flag suspicious IPs
- 2FA

# 9.6 Additional Password Security Techniques

## 9.6.1 Strong Passwords
- not a concatenation of 1+ dictionary words
  - ex. Create password from l33ted long passwords
  - "Nothing is really work unless you would rather be doing something else" > n!rWuUwrbds3
- protect your password file
  - limit access to just the admin
  - UNIX used to store in `/etc/passwd` which was accessible by all users
    - figured the passwords were hashed so what the heck
  - now in `/etc/shadow` (which requires privileges)

## 9.6.2 "Honeypot" passwords
- easy-to-guess username/password combos to attract attackers
- lets you monitor attacks

## 9.6.3 Password Filtering
- restrict password selection based on different requirements

## 9.6.4 Aging Passwords
- users must change passwords after certain time period
- OR by only accepting that password a limited number of times
- requiring too many changes leads users to come up with more insecure passwords so they can remember more easily

## 9.6.5 Pronounceable Passwords
- non-dictionary words that are still easy to remember
- you could have a program that concats syllables and vowels together to try dictionary attacks
- not much adoption

## 9.6.6 Limited Login Attempts
- protects against online dictionary attacks
  - account locked or disabled after certain specified number of attempts
- bad for forgetful users
- potential for DoS attack
  - attacker could take a bunch of common usernames, try combos without caring about successful logins
  - would conduct a DoS against a large number of accounts

## 9.6.7 Artificial Delays
- when user tries to login over network
  - if they fail 1x, no problem; if they fail again wait `2^n` seconds after `nth` failure
- minor inconvenience to users but is basically a DoS against attackers
- HTTP proxies can be problematic
  - users using same HTTP proxy could have same IP address 
  - users with same IP address could delay each other 

## 9.6.8 Last Login
- notifies users of last successful or failed login details
- lets users recognize suspicious activity and report it

## 9.6.9 Image Authentication
- primarily to combat phishing attacks
- images act as a second-factor
- user picks image during account creation
  - display at login after username is entered
  - phisher can't spoof the image
  - tell users not to enter password if the user isn't their initial selection
- phishers could say that the image server is down and ask for password
  - many people do that
- this approach relies on _education_ of the users

ex. PassMark - one of first to implement widescale
  - acquired by RSA security

## 9.6.10 One-Time Passwords
- if you use your password multiple times it gives an attacker multiple opportunities
- one-time password could be used as a 2nd factor 
  - ex. cellphone or email or whatever receives the one-time password
- seed chosen (some integer) and known to server and device that's generating the password or sends seed via SMS

 
