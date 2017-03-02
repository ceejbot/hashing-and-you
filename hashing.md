# [fit] __hash functions__ and you

---

## [fit] or, why breaking __SHA-1__
## [fit] is a thing

---

# [fit] this deep-dive is on
# [fit] __hash functions__
# [fit] not cryptography in general

^ Going to dive into hashing today. Topic came up because...

---

## [fit] Google figured out how to
## [fit] generate two PDF files with
## [fit] __the same SHA-1 hash__

---

## [fit] why does this matter?
## [fit] what should npm do about it?

---

## [fit] the term __hash__
## [fit] like so many computer terms,
## [fit] is __metaphorical__

---

# [fit] hash functions chop up data
# [fit] and __hash it up__
# [fit] ha ha get it?
# [fit] \(except we're not sure of the origin)

---

# [fit] what's a __hash function__?

---

## __hash function__

> Any function that can be used to map data of arbitrary size to data of fixed size.

-- [the wikipedia entry on hash functions][wiki]

^ The wikipedia entry is good & I recommend it!

---

## [fit] data ➜ function ➜ number

^ And yes, it's always a number, even when it's represented as a hexadecimal string.

---

## [fit] data ➜ fn ➜ number
## [fit] message ➜ __hash__ ➜ digest/hash/code

^ Say hello to some jargon.

---

## really simple hash function, from Knuth

If *k* is an integer key, and *n* is the number of buckets, not a power of 2:

`f(k) = k(k+3) mod n`

```
const buckets = 19;
function knuth(k) {
	return (k * (k + 3)) % buckets;
}
```

---

## mostly we hash character data...

So we do this, roughly:

- initialize to 0
- take the input in chunks of the size that you want
- do something to the chunk & XOR it into the result
- when out of chunks, return

The "something" can either be very simple, like the Knuth division hash, or very complex.

---

## [fit] the hash value is (usually)
## [fit] a lot __smaller__ than the data!

^ This is very handy: you have a representation of the data, related to it, that isn't a full copy of it.

---

## hash tables!

Suppose you have a lot of data that you want to stick into memory for fast access by a *key*. You use a hash function to map the keys into *buckets*, one for every output number, and put the matching data for a key into the key's bucket. There might be more than one thing in the bucket, but that's fine because you can look inside the bucket to find your item in a small collection, instead of having to search the whole thing.

This is the data structure underlying associative arrays, caches, and a million other things in software.

---

## Data integrity!

Alice publishes a package on npm. Bob wants to know if the package he's downloading is the data Alice published, because he's worried about data corruption or tampering.

---

Alice creates a *checksum* of the data:

```
⇒ md5 <  package.tgz
0030be42121988078dca0ec982d04f72
```

She gives the output to Bob, and he can compare with the md5 sum of his download to see if he has the data she meant to give him.

---

## features of a good hash function

- output is *deterministic*
- output is *uniformly distributed*, not clustered
- usually: fast
- output value has a fixed size
- sometimes: similar inputs produce nearby outputs
- sometimes: similar inputs produce distant outputs (avalanche effect)

---

## [fit] __avalanche effect__
## [fit] a small change in input
## [fit] large change in output

---

# hashing __cat__ and __car__ with MD5

```
⇒ echo cat | md5
54b8617eca0e54c7d3c8e6732c6b687a
⇒ echo car | md5
5cd3e81fb747479797b62794c6bf6aaf
```

Even the battered md5 has the avalanche effect.

---

## [fit] locality-sensitive hashes
## [fit] or __similarity__ hashes
## [fit] do exactly the opposite

---

## [fit] copyright violation __detection__ on youtube
## [fit] will have similarity hashing behind it

---

## [fit] mostly we want the __avalanche__ effect
## [fit] sometimes called the __butterfly__ effect

---

## [fit] often we want
## [fit] __cryptographic__ hashes

---

# a good __cryptographic__ hash

- deterministic
- fast to compute
- has the avalanche effect
- reconstructing the original message from the hash is infeasible
- finding collisions is infeasible

---

## [fit] __collisions__ mean we can't tell that
## [fit] two inputs are different
## [fit] deliberate collisions would be an attack

---

## Back to the classic example!

Carol wishes to trick Bob into running her npm package instead of the one Alice wrote. Because Alice used the weak MD5 algorithm to sign her data, Carol is able to craft a tarball that has the same MD5 digest but different data inside. She man-in-the-middle attacks Bob and serves him her cleverly-crafted package.tgz instead of Alice's.

Bob is now pwned.

---

## [fit] __collision-resistance__ is crucial
## [fit] for verifying data integrity
## [fit] This is why "breaking" SHA-1 matters.

^ Hello!

---

## [fit] Collision-resistance
## [fit] might not __always__ matter

---

# non-cryptographic hashes

- usually a lot faster than cryptographic hashes
- finding collisions might be feasible
- ditto reconstructing the original

---

# non-cryptographic hashes

- use when speed matters
- use when defense against malicious input doesn't matter

---

# uses for non-cryptographic hashes

- bloom filters 🌻
- lookup tables
- sharding data uniformly

^ Distribution of package names in npm is an example of non-uniform data dist.

---

# non-cryptographic hashes

* [MurmurHash][murmurhash]
* [CityHash][cityhash]
* [HighwayHash][highwayhash]
* [xxHash][xxhash]
* [seahash][seahash]

^ First three from Google, which employs a lot of CS peeps. I like xxhash for its speed.

---

# [fit] you can [design your own][designing]
# [fit] non-crypto hash with __a little math__
# [fit] cryptographic hashes are harder

---

## [fit] cryptographic hashes
## [fit] are the workhorses of __security__

---

## use a cryptographic hash

- to verify message integrity
- to verify passwords without knowing them
- to identify data

---

## [fit] choose a cryptographic hash
## [fit] to __defend__ against
## [fit] malicious input or attack

---

## [fit] hashes suitable for __passwords__
## [fit] have some unusual properties

---

## hashing passwords

* salt + password ➜ hashing function ➜ output
* store the output *only*
* repeat the transformation when checking the password to see if you get the same result

An attacker who can run that transformation frequently & quickly is one who can __brute-force attack__ your users' passwords.

---

## [fit] password hashes are
## [fit] __tunably expensive__

---

## [fit] slow to run, use a lot of memory
## [fit] to __slow down__ attackers

---

# password hash algorithms

- [bcrypt][bcrypt]
- [scrypt][scrypt]
- [argon2][argon2]
- [PBKDF2][PBKDF2]

^ bcrypt is the current industry standard; based on Bernstein's blowfish cipher; pbkdf2 is attackable with GPU-based algos. scrypt & argon are new but probably fine to use.

---

## [fit] some cryptographic hashes you should know!

- [md5][md5]
- the SHA family
- [blake2][blake2]
- [siphash][siphash]

---

## [MD5][md5]

- designed in 1991
- blown apart by 1996
- really fast
- __do not use__ as a cryptographic hash
- can use for data bucketing if you trust the input

^ So many horror stories.

---

## [fit] the __SHA__ family
## [fit] Secure Hashing Algorithm
## [fit] NIST standards

---

## [fit] standardization means SHA algos are
## [fit] widely __available__ & widely used

---

## [SHA-1][sha1]

- 160-bit result (40 hex digits)
- very widely used (git shasums!)
- designed by NSA in 1995, 1st attack in 2005
- better attack in 2015
- collision at > brute force speed by Google in 2017
- __do not use__ as a cryptographic hash

---

## [fit] npm uses SHA-1
## [fit] for __data integrity__ checks for tarballs

---

# Back to that classic example!

Alice is using SHA-1 to sign her package tarballs. Carol is an employee of Google or maybe of a nation-state's spy agency. Carol has a lot of computing power available, and really wants to pwn Bob. She cleverly crafts a tarball that has the same shasum as Alice's, and serves it to Bob.

In ten years, Carol will be able to do this with a Raspberry Pi 7 the size of her thumbnail instead of a fleet of cloud computers.

---

## [SHA-2][sha2]

- comes in 224, 256, 384 or 512 bit variants
- SHA-256 & SHA-512 are the most-used
- designed by NSA in 2001
- no feasible attacks known
- __do use__ freely
- replace SHA-1 with this, generally

^ You will notice that we used to trust the NSA on cryptography.

---

## [SHA-3][sha3]

- comes in 224, 256, 384 or 512 bit variants
- won a competition to choose next SHA standard
- adopted as standard in 2015
- chosen for its differences from SHA-2
- no feasible attacks known
- __do use__ freely

---

## [blake2][blake2]

- a SHA-3 finalist
- 32 bit word variant (produces 256-bit results)
- 64 bit word variant (produces 512-bit results)
- faster than SHA-3 selection
- __do use__ freely

---

# hash __flooding__ attacks

[Hash-flooding DOS attacks][flood] use collisions to mount denial-of-service attacks by attacking a language's underlying hash implementation.

- send crafted data to an app, which stores it in a hash table
- the keys are designed to cause collisions
- all the data goes into one hash bucket
- hash table performance collapses into merely a linked list
- which is, as you know, Bob, a truly awful data structure

---

# [fit] somebody did this to __perl__ in 2003
# [fit] in 2011 to a lot of __other__ languages

^ This produced ...

---

# [siphash][siphash]

* designed to defend against hash flooding attacks
* optimized for speed with small input
* used in hash table implementations in many languages
* use it for your own hash table

---

## which one should __npm__ adopt?

* SHA-2, SHA-3, and BLAKE2 are all fine choices
* SHA-2 is safer because of implementation availability
* we are not size-sensitive about the output
* we care more about picking an algo that will last
* SHA-512 is a solid, safe choice

^ sha-1 is not something we'd pick if starting from scratch today. cost of migrating goes up every year we delay, so we should do it now.

---

## summary

* stop using MD5
* stop using SHA-1
* do use SHA-2 and newer hashes
* use bcrypt to store passwords
* don't invent unless you're an expert
* in which case you should be giving this talk not me

---

## [fit] __Questions?__
## [fit] More-of-a-comment-reallys?

---

## all the links

[Hash function wiki page][wiki] • [bcrypt][bcrypt] [scrypt][scrypt] • [PBKDF2][PBKDF2] • [argon2][argon2] • [MD5][md5] • [SHA-1][sha1] • [SHA-2][sha2] • [SHA-3][sha3] • [BLAKE2][blake2] • [seahash][seahash] • [xxhash][xxhash] • [siphash][siphash] • [designing your own][designing] • [Hash-flooding][flood]

Go! Try them! Learn more!

love,
@ceejbot

[wiki]: https://en.wikipedia.org/wiki/Hash_function "wikipedia page on hashing functions"
[murmurhash]: https://github.com/aappleby/smhasher/blob/master/src/MurmurHash3.cpp "MurmurHash 3"
[cityhash]: https://github.com/google/cityhash
[highwayhash]: https://github.com/google/highwayhash
[bcrypt]: https://en.wikipedia.org/wiki/Bcrypt "bcrypt"
[scrypt]: http://www.tarsnap.com/scrypt.html "scrypt"
[PBKDF2]: https://en.wikipedia.org/wiki/PBKDF2
[argon2]: https://github.com/P-H-C/phc-winner-argon2
[md5]: https://en.wikipedia.org/wiki/MD5 "MD5"
[sha1]: https://en.wikipedia.org/wiki/SHA-1 "SHA-1"
[sha2]: https://en.wikipedia.org/wiki/SHA-2 "SHA-2"
[sha3]: https://en.wikipedia.org/wiki/SHA-3 "SHA-3"
[blake2]: https://blake2.net "BLAKE2"
[seahash]: https://github.com/ticki/tfs/tree/master/seahash "seahash"
[xxhash]: https://cyan4973.github.io/xxHash/ "xxhash"
[siphash]: https://131002.net/siphash/ "siphash"
[designing]: http://ticki.github.io/blog/designing-a-good-non-cryptographic-hash-function/  "Designing a good non-cryptographic hash function"
[flood]: http://emboss.github.io/blog/2012/12/14/breaking-murmur-hash-flooding-dos-reloaded/ "Hash-flooding DOS reloaded"
