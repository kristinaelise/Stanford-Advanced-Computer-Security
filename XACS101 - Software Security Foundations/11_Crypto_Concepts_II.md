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

