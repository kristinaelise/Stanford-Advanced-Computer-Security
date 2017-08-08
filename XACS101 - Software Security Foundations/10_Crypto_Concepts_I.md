# Crypto Concepts I

Cryptography is only reliable when it's implemented _reliably_, so it's important to stick to standards and reliable implementations.

## Goal 1: Secure Communication
- encrypt data transmitted on the network (using SSL protocol)
- prevent the attacker from eavesdropping on traffic and prevent tampering

### Secure Sockets Layer / TLS

**Handshake Protocol**: Establish shared secret key using public-key cryptography

**Record Layer**: Transmit data using this key negotiated by the browser and server; data is encrypted using that key

## Goal 2: Protected Files
- prevent eavesdropping and tampering of a file before it reaches its intended recipient
- network traffic and storage on disk can be viewed as two sides of same coin
  - in the slide example communication from Alice today to Alice tomorrow, but communicating through disk
  - whenever talking about applications of crypto, think of storage on disk _and_ network traffic

## Building Block: Symmetric Encryption

**Symmetric Cypher**: encryption algorithm and a decryption algorithm
- called 'symmetric' because of a shared key, `k` that both sides know
- not always the case in other forms of encryption; sometimes they know different keys
- but in symmetric encryption, both know the exact same key
- Alice generates a nonce
- given message and key, Alice uses encryption algorithm to generating corresponding cypher text
- cypher text sent to Bob with `nonce`; Bob uses decryption alg with key to decrypt cypher text

**m**: plaintext
**c**: cyphertext
**n**: nonce; ensures that if you encrypt the same message twice, you get different cypher texts
- prevents eavesdropper from realizing it's the same message

- key itself is relatively short; traditionally 128 bits or 256 bits
- typically much shorter than message being translated
- encryption algs E, D are publicly known
- the only thing that's secret is the secret key, `k`
  - system itself is public; attacker just don't know `k`
- you should be using standardized algs; don't use proprietary algs

## Two Types of Encryption Algs

### Single Use Key
- secret key is only used to encrypt _one_ message
- "one-time key"
- thrown away and never used again
- no need for nonce (set to 0)
- with email messages, new key generated for every email and then never used again

### Multi-Use Keys
- used to encrypt multiple messages
- ex. SSL -- same key used to encrypt many packets sent from sender to receiver
- have to make sure that we're using an encryption that's secure for multi-use keys
- therefore important to have a nonce so even if same message sent twice, data is different
- requires a _unique_ or _random_ nonce

## One Time Pad (Single Use Key Case)
- designed in beginning of 20th century by Vernam
- aka Vernam cypher

### How it Works
- secret key is just a sequence of bits
- plaintext message is encoded as a sequence of bits
- ASCII text broken into bytes, and then each encoded using ascii 8-bit representation 
- Encrypt plaintext by xor two strings
  - XOR: addition modulo 2
    - 0 XOR 1 = 1
    - 1 XOR 1 = 1
    - etc.
- Decrypt: XOR the cypher text with the key
  - undoes the encryption operation
- very fast

### Why is this Secure?
- Claude Shannon first asked this (1949)
- proved a theorem showing that OTP is actually secure against one-time eavesdropping
- if attacker gets to see one cyphertext, it tells him nothing about plaintext
- if you don't have the key, and only see cyphertext, reveals no information

### Problem
- uses very very long keys
- key is as long as the message


## Stream Cypher (Single Use Key)
- OTP key is as long as the message; how do we generate short keys with the same benefit?

### How it Works
- short key, but can be used to encrypt short messages
- Encrypt: pseudorandom generator (PRG)
  - takes short key, expands into a long pseudo random sequence
  - input is a short key; output is sequence as long as what we're encrypting that looks random
  - if PRG is secure, output is indistinguishable from random
  - once we have this random string, we can XOR with the message, and this gives us a cyphertext
- Decrypt: cyphertext XORed with pseudo-random sequence  

**Famous Stream Cyphers**: Salsa20/12, Sosemanuk, RC4
- much faster than other ways of encryption

- NB: only designed for _single use case_
- if you try to use it for multiple messages, things becuase _extremely insecure_

### Example of "Two Time Pad"
- two messages: M1, M2
- encrypt using One Time Pad twice
- gives two cyphertexts
- problem is attacker can compute XOR of C1 and C2
  - since same pad used twice, it cancels out
  - you get to see the XOR of the two plain texts
  - if you give the XOR of 2 English plainttexts, enough redundancy, you can recover M1 and M2

### Project Venona
- used to break One Time Pad encryptions that Russians were sending from US to Moscow
- were using OTP, but using it incorrectly
- were using single pad multiple times
- US was able to intercept cyphertext and using XORS were able to recover plaintext


## Block Cyphers: The Workhorse of Crypto
- pair of algorithms: E, G
- uses a key for encryption, decryption
- operates on relatively short, fixed size blocks
  - ex. operates on n-bit blocks; takes n-bit input and produces n-bit output
  
**AES, 3DES**: common examples
- AES' k-size varies depending on security needs of the application (128, 192, 256)
- longer the key, slower the block cypher but presumably more secure

### How They Actually Work
- nb: operates on very small blocks, and one block at a time
- built by iteration
  - round function (`R`)
- Encrypt:  message comes in as input (`m`) and is encrypted using key `k` 1, 2, 3...etc. until you get the cyphertext back
- Decrypt: using round function in reverse until you get the original message
- all of the k keys are generated by key expansion function that expands original key into smaller, separate keys (n keys for each round)
- AES faster than 3DES (triple DES) 

### Round Function
- 3DES: take plain text block (64-bit block), break into two halves, and apply round function
  - output is output of one round, do that 48 times using each of 48 round keys
- AES you'd do it 10 times
  - AES on an intel processor is super fast since implemented in software
- **Don't design block cyphers yourself!**
  - use Advanced Encryption Standard (AES); approved by National Institute of Standards
  - always use AES
  - last thing you want to do is design your own...they're designed to resist subtle attacks

### Attack Examples
- differential attacks, linear attacks, brut-force, etc.

## Using Block Cyphers

### The Incorrect Way
- Electronic Code Book: incorrect
  - take long plaintext message and break into 16-byte blocks
  - AES: 16-byte-blocks
  - encrypt each block separately
  - **SUPER WRONG**; **INSECURE**
    - if you have a repeated block and encrypt the two blocks, you get the same cyphertext block in 2 places and this makes it easier for people to determine plaintext

### The Correct Way

### Cypher Block Chaining (CBC)
- start with a random initialization vector (IV)
- IV is fixed -- used as a nonce
- break message into blocks
  - instead of encrypting each block separately, XOR the first block with the IV and encrypt the result
  - then take cyphertext output and chain it into the next block, XOR the two together and encrypt the result
  - continue for the rest
  - resulting cyphertext is the resulting blocks along with the IV
- good exercise to decrypt -- decrypt same way -- first block to last -- except XOR moves to bottom
- once you use CBC, problem of leaking info goes away; everything is totally random
  - same plaintext block may appear, but what gets encrypted both times is different

## How do we choose an IV
- single use: no IV needed (IV = 0)
  - means you just encrypt the plaintext without IV; only secure if you only use key once
- multiple times: setting IV to 0 is a very common mistake
  - **super insecure**
  - generating it at random for every message
    - blocks are 128 bits, for every message you generate a 128bit IV and encrypt the message using that and include in cyphertext

### Problem
- CBC is very sequential; you can't encrypt block M2 without encrypting block M1
- is there a way to use AES which is secure and lets us use parallelism? YES!
  - **Counter Mode Encryption!**

### Counter Mode Encryption
- generate the IV
- encrypt not the message, but the encrypt the IV!
- encrypt the IV +1, +2, ... up to number of blocks in message
  - counting, starting from IV, encrypting counter and generating OTP through the counter
  - take the OTP output generated through block cypher like AES and XOR it with plaintext and that gives cipher text
  - include IV with cyphertext
- one time use: IV = 0; multiple times: IV = random for each message
- reduces the block cypher to a pseudo-random generator
  - pseudorandom sequence then used to encrypt the actual message

### Why is it Secure?
- learn more through full crypto course 

### Performance
- stream vs block
- stream cyphers are _way_ faster than corresponding block cyphers
- but stream cyphers are single use; blocks can be used for multi-use
- many websites opt for stream cyphers rather than block

## Warning!
- everything we talked about so far provides no integrity
- it's all just eavesdropping integrity
  - we aren't protecting against tampering (where attacker tries to modify cypher text)
- for most apps, eavesdropping security by itself is _insufficient_ and provides _no security at all_
- so everything we've discussed is really just a building block; shouldn't be used as a standalone encryption method
- we prevent tampering by using a method integrity mechanism

## Message Integrity: MACs
- want to ensure message isn't changed as it's moving from Alice to Bob
- ex. Ad: ad isn't secret; attacker can see it
  - advertiser cares the attacker doesn't change what the ad is

### How it Works
- Alice and Bob share a secret key
- Alice computes an "integrit tag" (S for 'signing') where she uses secret key and message to generate a tag on the message
  - tag is very short
  - append tag to message and send to Bob
- Bob verifies the tag using the algorithm V (V for 'verification')
  - outputs Yes or No depending on whether tag is valid

**Message integrity always involves a shared key that the attacker doesn't know**

- if you don't have a shared key, you can't provide integrity
- standard checksums (ex. CRC) that's going to be insecure in adversarial settings
  - attacker can intercept message M, modify, and recompute CRC without Bob knowing
- only thing preventing attacker from recomputing is that he doesn't know the shared key

## Secure MACs
- security defined in terms of _what can the attacker do_? and _what is the attacker trying to do?_
- with MAC, attacker allowed to obtain tag on any message
  - may send user an email, user computes MAC and sends to disk
  - attacker may expose disk and gets to see tags on messages of his choice
  - = "Chosen Message Attack"
- given message-pair tags, attacker's goal is to generate message tag pair that's different
  - if he can compute the tag for some message he hasn't seen, then he's broken the MAC
- attacker might get to see all stored binaries and corresponding binaries
  - shouldn't be able to invent own binary stored on desk and produce tag for that particular binary
  - shouldn't be able to do it even if tag is invalid
  - want to ensure there's no message at all that attacker can provide a MAC for

## Constructing Secure MACs
- construct from primitives we already have

### ECBC : Block Cypher
- "Encrypted CBC Mode": Encrypted Cypher Block Chaining Message Integrity Mode
- take message and break into blocks
  - 16-byte blocks in case of AES
  - encrypt each block with MAC key `k`
  - do usual CBC path (xor + encrypt) until you've hashed whole message where you get output called `raw CBC MAC`
  - do one more encrpytion called CBC1
  - output is tag
- `k` = key used for chaining step; `k1` used for final encryption
  - final encryption gives MAC on entire message that could be arbitrarily long 
- integrity tag is very short, even when message is very long

## HMAC (Hash-MAC)
- uses cryptographic hash function (ex. **SHA-256**)
- takes arbitrarily long message and hashes it down to 256 bits
- **collision resistance**: hard to find 2 messages that produce the same hash
  - if you can't find 2 messages that produce same hash, called 'collision resistant'

### How It Works
- take message and short key:
  - hash the key XOR some fixed pad
  - concatenate the message to that and you hash the pair
  - after hashing, output will be 256 bits
  - concatenate to that the same key XOR another pad
  - hash those two words together again to 32 bytes (256 bits)
  - and that's the output
-  **ipad**: inner pad; **opad**: outer pad

###ECBC
- commonly used in the banking industry for clearing cheques 

###HMAC
- commonly used on the internet

### The Parallel Problem Again
- both constructions are _sequential_
- is there a MAC construction that would allow us to take advantage of parallelism?

## PMAC - a parallel MAC
- have message broken into blocks
- before applying encryption function, you compute a masking function that depends on a certain key along with the block index
- the masking function is a simple function to computer
- XOR message block with appropriate mask
- encrypt results in parallel
- XOR all of outputs together
- one more encryption
- gives you the final output

### What's wrong with doing just raw CBC?
- why do we add on last encrpytion step?
- without last encryption step, ECBC is insecure
  - if you give the tag on one message, attacker can deduce tag on some other message n'
  - can choose message `m`, such that he can forge message on `m'`

# How Do We Combine Encryption & Integrity?

## Authenticated Encryption

### Combining MAC and ENC (encryption alg)

**Option 1 (SSL)**:
- aka HTTPS
- take plaintext message, compute a MAC on it using a MAC key (`ki`)
- concat MAC to message and then encrypt this pair using encrpytion key `ke`

**Option 2 (IPsec)**:
- works opposite to SSL
- takes message, encrypts it first using encryption key
- computes MAC on cyphertext

**Option 3 (SSH)**:
- totally different algorithm
- takes message, encrypts it, and then computes MAC on plaintext message and concats with cyphertext

### Only one is secure universally
- IPsec is secure because the encryption alg makes sure eavesdropping doesn't reveal any info about message
- MAC algorithm then locks the cyphertext

### Encrypt, then MAC

### Standards (High Level)
- don't invent your own system
- use one of these standards
- do more than just encrypt and authenticate, they provide **authenticated encryption with associated data**
  - both encrypts data, but then provides a header such that the header has integriy but isn't encrypted
  - part of data can be encrypted, part can be plain, but preserves integrity
  - ex. packet payloads may need to be encrypted, but header may need to be plain so that router can know where to route packets

## Implementation Problems
- security isn't just about a correct algorithm
- also about whether implementation itself is correct

### Side Channel Attack
- done back in 1999; still happens today
- attacker measures power that the computer is using at the time it's doing encryption and decryption
- attacker doesn't know key, but can measure amount of power
- amount of power consumed depends on instruction and data being given
  - would carefully measure amount of power at every step
  - reveals secrey key
  - graph shows DES encryption (set-up phase, final phase, 16 rounds shown clearly)
    - if you zoom in you can read off key bits one at a time
    - exposes secret key
- don't encrypt your own algs, and don't implement your own
  - rely on standardized options

## Generating Randomness
- where does randomness come from?
  - many products that are vulnerable because of weak entropy (weak randomness)
- random number generator has "internal states"
  - call generator, from internal states it gives you some random output
- LINUX has device `dev/random` that generates random output for you
  - used as input into cryptographic generator (openSSL, etc.)
- continuously measure system and add entropy

### What are the entropy sources?
- **`RdRand`** from Intel
  - Intel added entropy generation function into processor
  - `RdRand` generates true randomness by listening to physical processes on the chip
  - fast generator: 3Gb/sec
  - enough to generate entropy for everything
- **Other sources**:
  - generators listen to various hardware interrupts and when they happen
    - timing of interrupts fed into entropy pool and that's used to generate randomness
  
### How do you get entropy right after the machine boots?
- entropy pool is relatively empty
  - crypto ops very close to boot time is a bad idea
  - wait till machine operates for a while, and _then_ perform crypto
  - otherwise not enough entropy and could be insecure
