# Crypto Concepts II
- where do the shared keys come from?
- how do we generate a session key?

# Public Key Encryption
- basically a tool for generating or managing symmetric key
- unlike symmetric key crypto, there are actually **two keys**
  - public key, secret key
- could be many senders and all of them would use same public key to send info to Bob

## How It Works
- Bob generates public key and secret key
- advertises public key to anyone who wants to send confidential info
- Alice encrypts using public key
  - sends resulting cyphertext to Bob
  - no one knows corresponding secret key other than Bob
- Bob applies secret key to cyphertext to decrypt
- Encryption and decryption algs are known
- Only thing not known is Bob's secret key

## Building Block: Trapdoor Permutations
- public key encryption consists of **3 algorithms**
  - **key generation algorithm**
    - basically generates public key and secret key
  - **one-way function**
    - given public key, anyone can evaulate the function
      - given input x and public key can evaluate function at point x
    - inverting the function is quite difficult _just_ given public key
      - this is why it's called 'one-way'
  - **secret key**
    - if you have the secret key, and output of the function, inverting becomes easy 
    - `F(SK, y) = x`
- function is one way unless you have the secret key

**Trapdoor permutations used to generate public key encryption and digital signatures**

### Example: RSA
- Rivest, Shamir, Adelman (1978)
- first example of trapdoor permutation
- key generation:
  - generate two random primes, `p` and `q` (these are huge numbers; 500 digit numbers)
  - multiple p and q to give N
- fundamental property is that multiplcation is efficient, but if you give modulous, factoring is very difficult
  - creating public key is easy, but finding secret q is difficult
- fixed `e` - the "encryption exponent" - 65537
- private decryption key: some sort of inverse function modulous of N; some other integer dependent on e and N
- how do we evaluate RSA?
  - take `x -> (x^e mod N)`
- if all you have is public-key, then inverting function is as difficult as factoring N
- if you give the number `d`, inverting becomes simple
  - `y -> (y^d mod N)

## Public Key Encryption with a TDF
- generate public key and secret key for trapdoor permutation

### How to Encrypt
- want to encrypt `m`
- generate random x between 1 and N (in case of RSA)
- hash x so it's an AES key, for example
- apply function to x - gives C0; encrypt M with secret key, k
- C0 is result of applying trapdoor permutation to x
- C1 is result of encrypting M using hash of x as a symmetric encryption key

### How to Decrypt
- if you have secret key, look at C0
- invert C0 using secret key that gives x
- rederive from x the symmetric encryption key
- apply symmetric encryption key to C1 to give plaintext

- every time we encrypt, uses new random key (since new random x, and then hashed)

# Digital Signatures

**Goal**: bind document to author
**Problem**: attacker can copy signature from one document to another
**Idea**: make the signature depend on the document

_The signature itself is a function of the document._

Example: build from trapdoor functions

```
signer(SK, m) := F^-1(SK, H(m))
```
- signer uses RSA secret key; verifier uses RSA public key to verify
- hash message so it's a small number (ex. using SHA-256) and you get a small hash value (H(m))
- invert the function F using the secret key

```
verify(PK, m, sig) := accept if F(PK, sig) = H(m)
```
- verifier has public key, message and signature on the message
- will reapply function to the signature
- if signed correctly, should get the hash of the message

## How Does the Verifier/Encrypter Obtain Bob's Public Key?
- how do you know it's really _their_ public key (PK)?

**Certificate**
- binds a particular user to a particular public key (ex. binds Bob to his PK)

### HTTPS Example

**Set-up**
- Browser wants to verify that it really is receiving server's PK
- Server convinces browser by presenting certificate issued by Certificate Authority (CA)
  - CA has its own secret key that only it knows
- Public Key lives on the server and browser; whole world can know CA's public key
- Verisign, example of CA (acquired by Symantec)
- All browser's have CA's public key stored in cache

**How the Server Obtains Certificate**
- server generates secret key and public key
  - stores secret key internally
- server sends public key to CA along with a "proof" that says it's really who it says it is
- certificate verifies the proof and issues certificate
- certificate is really just a signature saying that Bob's public key really _is_ the value PK

**Certificate Obtained**
- browser can now validate certificate
  - and now can encrypt with PK
- certificates live for a long time (1 - 2 years)
  - interaction between server and CA happens only rarely

# Schematic SSL session setup

## Abstract TLS (simplicifed)
- transport layer security (TLS); SSL was predecessory but both often referred to as SSL
- web server has RSA secret key and a certificate on that secret key
- browser connects to server and sends `ClientHello` message to establish a connection (handshake to generate shaked secret key)
  - hello message contains some randomness (nonce)
- server receives message and responds with server certificate plus public key and nonce from the server
- client generates 48 bytes of "pre master secret" (PreK)
  - just truly random 48 bytes of data
- client sends back message that's basically an encryption of the 48 bytes under the server's public key (ClientKeyExchange)
- server has corresponding secret key and can decrypt
  - gets the PreK
- can use pseudorandom function to crunch 48 bytes and nonces to generate session keys to encrypt traffic
- finishing messages to make sure both sides agree on key
- shared secret key now exists and rest of communication is used with secret key

### How Is It Secure?
- eavesdropper would see RSA certificate
  - server's public key and signature on public key
- sees encryption using server's public key
- because encryption scheme is secure, given encryption of 48 bytes, attacker knows nothing

## Properties

### Why do we need the nonces?
- if attacker recorded handshake, could replay the entire interaction to the server
  - server wouldn't be able to tell it was a replay of an old session

### No forward secrecy
- imagine client and server established secure session
- imagine a week later attacker was able to break into server and steal secret key
  - attacker could record conversation and go back to what was recorded, decrypt cyphertext with secret key and decrypt entire session
- says even though _today_, server is secure, in the future it could be come insecure
- old sessions might become exposed if server broken into
- TLS does have support for forward secrecy though
- more sophisticated methods slow things down; some don't use for efficiency
  - 3:1 ratio of slowness

### One-Sided Identification
- browser knows it's talking to a particular server because of the certificate
- however server knows nothing about who the client is
- TLS has support for mutual identification
  - but requires client public key/secret key and client certificate
  - rarely used

# A Brief Overview of Modern Cryptography

## Secure Multi-Party Computations
- keep input secret while disclosing output

"Anything that can be done with a trusted authority can also be done without"
- rather than sending votes to trusted authority, parties talk amongst themselves
  - at end of conversation, final result produced

## Privately Outsouring Computation
- Alice sends encrypted query to Google
- system that allows computations on encrypted query
- you get back an encrypted result (Google learned nothing)
- user decrypts and gets results without Google knowing anything
- **Fully Homomorphic Encryption**
- pretty theoretical right now though
- fairly ineffient; more of a conceptual idea right now

## Zero Knowledge Proof of Knowledge
- I can convince you that I know something in such a way that you learn nothing at all about what I know
- ex.
  - RSA modulus is product of two primes
  - you have RSA modulus n
  - want to prove I know the factorization (p, q) but without telling you anything about the factors
- often used in building access (lock doesn't learn what key is but knows key is able to enter building)

## Privacy: Group Signatures
- **Group Signature**: 
  - key issuer has master key
  - each user given a different secret key
  - allow user to sign messages
  - signature reveals nothing about who actually signed the signature
  - observer has no idea which member issued signature
- allows you to sign on behalf of group without telling who actually issued signature
- simple solution: give all users same key
  - would mean there's a global key distributed
  - generally a bad idea
    - means no way to revoke
  - ex. CSS encryption for DVDs used global key
- should be a trace just in case you need to see who actually signed
- **Example**
  - Vehicle Safety Comm. : car-to-car system
  - implant short range transmitters in cars
  - used for vehicle safety
    - imagine one car needs to put on brakes in emergency
    - to avoid pile-up, use transmitters to send signal when action performed
      - would probably still affect one car, but others can be alerted to fact that first car is braking
  - security problem:
    - once deployed, people could install a transmitter saying they're an ambulance
    - need to authenticate transmitter signals
      - would use digital signature
  - if you sign a signal, now you have a privacy problem as well
    - cars would be transmitting position and velocity
  - **group signatures solve this issue**

## Broadcast Encryption
- transmit data to a large number of recipients 
- ex. internet radio
  - in each radio player embed secret key
  - transmit in such a way that active subscribers are the ones who can decrypt     
- want cypher texts to be short
- only authorized subscribers can decrypt
- have good constructions for this
- ensure collusion resistance: even if all unauthorized users' secret keys pooled together they can't figure out message

 
