[[keys_addresses]]
== Keys, Addresses

((("cryptography", "defined")))((("cryptography", see="also keys and addresses")))One of Ethereum's foundational technologies is _cryptography_, which is a branch of mathematics used extensively in computer security. Cryptography means "secret writing" in Greek, but the science of cryptography encompasses more than just secret writing, which is referred to as encryption. Cryptography can also be used to prove knowledge of a secret without revealing that secret (digital signature), or prove the authenticity of data (digital fingerprint). These types of cryptographic proofs are the mathematical tools critical to Ethereum and most blockchain systems, being extensively used in Ethereum applications. ((("encryption")))((("encryption", see="also keys and addresses")))Ironically, encryption is not an important part of Ethereum, as its communications and transaction data are not encrypted and do not need to be encrypted to secure the system. In this chapter we will introduce some of the cryptography used in Ethereum to control ownership of funds, in the form of keys and addresses.

[[keys_addresses_intro]]
=== Introduction

Ethereum has two different types of accounts, which can own and control ether: _Externally Owned Accounts_ (EOA) and _Contracts_. In this section we will examine the use of cryptography to establish ownership of ether by externally owned accounts, i.e. private keys.

((("digital keys", see="keys and addresses")))((("digital signatures", "purpose of")))Ownership of ether in EOAs is established through _digital keys_, _Ethereum addresses_, and _digital signatures_. The digital keys are not actually stored on the blockchain or transmitted on the Ethereum network, but are instead created and stored by users in a file, or simple database, called a _wallet_. The digital keys in a user's wallet are completely independent of the Ethereum protocol and can be generated and managed by the user's wallet software without reference to the blockchain or access to the internet. Digital keys enable many of the interesting properties of Ethereum, including decentralized trust and control, and ownership attestation.

Ethereum transactions require a valid digital signature to be included in the blockchain, which can only be generated with a secret key; therefore, anyone with a copy of that key has control of the ether. The digital signature in an Ethereum transaction proves the true owner of the funds.

((("public and private keys", "key pairs")))((("public and private keys", see="also keys and addresses")))Digital keys come in pairs consisting of a private (secret) key and a public key. Think of the public key as similar to a bank account number and the private key as similar to the secret PIN, that provides control over the account. These digital keys are very rarely seen by the users of Ethereum. For the most part, they are stored inside the wallet file and managed by Ethereum wallet software.

In the payment portion of an Ethereum transaction, the intended recipient is represented by an _Ethereum address_, which is used in the same way as the beneficiary name on a check (i.e., "Pay to the order of"). In most cases, an Ethereum address is generated from and corresponds to a public key. However, not all Ethereum addresses represent public keys; they can also represent contracts, as we will see in <<contracts>>. The Ethereum address is the only representation of the keys that users will routinely see, because this is the part they need to share with the world.

First, we will introduce cryptography and explain the mathematics used in Ethereum. Next, we will look at how keys are generated, stored, and managed.  Finally, we will review the various encoding formats used to represent private and public keys, and addresses.

[[pkc]]
=== Public key cryptography and cryptocurrency

((("keys and addresses", "overview of", "public key cryptography")))((("digital currencies", "cryptocurrency")))Public key cryptography is a core concept of modern day information security. First publicly invented in the 1970s by Martin Hellman, Whitfield Diffie and Ralph Merkle, it was a monumental breakthrough which incited the first big wave of public interest into the field of cryptography. Before the 70s, strong cryptographic knowledge was held under governmental control with little public research untill the open publication of public key cryptography research.

Public key cryptography uses unique keys that are used to secure information. These unique keys are based on mathematical functions that have a unique property: they are easy to calculate in one direction, but very difficult to calculate in the inverse direction. Based on these mathematical functions, cryptography enables the creation of digital secrets and unforgeable digital signatures which are secured by the laws of mathematics.

For example, multiplying two large prime numbers together is trivial. But given the product of two large primes, it is very difficult to find the prime factors (a problem called _prime factorization_). Let's say I present the number 6895601 and tell you it is the product of two primes. Finding those two primes is much harder than it was for me to multiply them to produce 6895601.

Some of these mathematical functions can be inverted easily if you know some secret information. In our example above, if I tell you that one of the prime factors is 1931, you can trivially find the other one with a simple division: 6895601 / 1931 = 3571. Such functions are called _trapdoor functions_ because given one piece of secret information, you can take a shortcut that makes it trivial to reverse the function.

Another category of mathematical functions that is useful in cryptography is based on arithmetic operations on an elliptic curve. In elliptic curve arithmetic, multiplication modulo a prime is simple but division is impossible (a problem known as the _discrete logarithm problem_). Elliptic curve cryptography is used extensively in modern computer systems and is the basis of Ethereum's (and other cryptocurrencies') digital keys and digital signatures.

[TIP]
====
Read more about cryptography and the mathematical functions that are used in modern cryptography:

Cryptography:
https://en.wikipedia.org/wiki/Cryptography

Trapdoor Function:
https://en.wikipedia.org/wiki/Trapdoor_function

Prime Factorization:
https://en.wikipedia.org/wiki/Integer_factorization

Discrete Logarithm:
https://en.wikipedia.org/wiki/Discrete_logarithm

Elliptic Curve Cryptography: https://en.wikipedia.org/wiki/Elliptic_curve_cryptography
====

In Ethereum, we use public key cryptography to create a key pair that controls access to ether and allows us to authenticate to contracts. The key pair consists of a private key, a unique public key, and are considered a "pair" because the public key is derived from the private key. The public key is used to receive funds and the private key is used to create digital signatures to sign transactions to spend the funds. Digital signatures are also used to authenticate owners or users of contracts, as we will see in <<contract_authentication>>.

There is a mathematical relationship between the public and the private key that allows the private key to be used to generate signatures on messages. This signature can be validated against the public key without revealing the private key.

When spending ether, the current owner presents her public key and a signature (different each time, but created from the same private key) in a transaction. Through the presentation of the public key and signature, everyone in the Ethereum system can independently verify and accept the transaction as valid, confirming that the person transferring the ether owned them at the time of the transfer.

[TIP]
====
((("keys and addresses", "overview of", "key pairs")))In most wallet implementations, the private and public keys are stored together as a _key pair_ for convenience. However, the public key can be trivially calculated from the private key, so storing only the private key is also possible.
====

.Why Use Asymmetric Cryptography (Public/Private Keys)?
****
((("cryptography", "asymmetric")))((("digital signatures", "asymmetric cryptography and")))((("asymmetric cryptography")))Why is asymmetric cryptography used in Ethereum? It's not used to "encrypt" (make secret) the transactions. Rather, the useful property of asymmetric cryptography is the ability to generate _digital signatures_. A private key can be applied to the digital fingerprint of a transaction to produce a numerical signature. This signature can only be produced by someone with knowledge of the private key. However, anyone with access to the public key and the transaction fingerprint can use them to _verify_ the signature. This useful property of asymmetric cryptography makes it possible for anyone to verify every signature on every transaction, while ensuring that only the owners of private keys can produce valid signatures.
****

[[private_keys]]
=== Private keys

((("keys and addresses", "overview of", "private key generation")))((("warnings and cautions", "private key protection")))A private key is simply a number, picked at random. Ownership and control over the private key is the root of user control over all funds associated with the corresponding Ethereum address, as well as access to contracts that authorize that address. The private key is used to create signatures that are required to spend ether by proving ownership of funds used in a transaction. The private key must remain secret at all times, because revealing it to third parties is equivalent to giving them control over the ether and contracts secured by that key. The private key must also be backed up and protected from accidental loss. If it's lost, it cannot be recovered and the funds secured by it are lost forever too.

[TIP]
====
The Ethereum private key is just a number. You can pick your private keys randomly using just a coin, pencil, and paper: toss a coin 256 times and you have the binary digits of a random private key you can use in an Ethereum wallet. The public key and address can then be generated from the private key.
====

[[generating_private_key]]
=== Generating a private key from a random number

The first and most important step in generating keys is to find a secure source of entropy, or randomness. Creating an Ethereum private key is essentially the same as "Pick a number between 1 and 2^256^." The exact method you use to pick that number does not matter as long as it is not predictable or repeatable. Ethereum software uses the underlying operating system's random number generator to produce 256 bits of entropy (randomness). Usually, the OS random number generator is initialized by a human source of randomness, which is why you may be asked to wiggle your mouse around for a few seconds, or press random keys on your keyboard.

More precisely, the range of possible private keys is slightly less than 2^256^. The private key can be any number between +1+ and +n - 1+, where n is a constant (n = 1.158 * 10^77^, slightly less than 2^256^) defined as the order of the elliptic curve used in Ethereum (see <<elliptic_curve>>). To create such a key, we randomly pick a 256-bit number and check that it is less than +n - 1+. In programming terms, this is usually achieved by feeding a larger string of random bits, collected from a cryptographically secure source of randomness, into a 256-bit hash algorithm such as Keccak-256 or SHA256 (see <<cryptographic_hash_algorithm>>), which will conveniently produce a 256-bit number. If the result is less than +n - 1+, we have a suitable private key. Otherwise, we simply try again with another random number.

[WARNING]
====
((("random numbers", "random number generation")))((("entropy", "random number generation")))Do not write your own code to create a random number or use a "simple" random number generator offered by your programming language. Use a cryptographically secure pseudo-random number generator (CSPRNG) with a seed from a source of sufficient entropy. Study the documentation of the random number generator library you choose to make sure it is cryptographically secure. Correct implementation of the CSPRNG is critical to the security of the keys.
====

The following is a randomly generated private key (k) shown in hexadecimal format (256 bits shown as 64 hexadecimal digits, each 4 bits):

[[prv_key_example]]
----
f8f8a2f43c8376ccb0871305060d7b27b0554d2cc72bccf41b2705608452f315
----


[TIP]
====
The size of Ethereum's private key space, (2^256^) is an unfathomably large number. It is approximately 10^77^ in decimal. For comparison, the visible universe is estimated to contain 10^80^ atoms.
====


[[pubkey]]
=== Public keys

((("keys and addresses", "overview of", "public key calculation")))((("generator point")))An Ethereum public key is a _point_ on an elliptic curve, meaning it is a set of X and Y coordinates that satisfy the elliptic curve equation.

In simpler terms, an Ethereum public key is two numbers, joined together. These numbers are produced from the private key by a calculation that can _only go one way_. That means that it is trivial to calculate a public key if you have the private key. But you cannot calculate the private key from the public key.

[[WARNING]]
====
MATH is about to happen! Don't panic. If you find it hard to read the previous paragraph, you can skip the next few sections. There are many tools and libraries that will do the math for you.
====

The public key is calculated from the private key using elliptic curve multiplication, which is irreversible: _K_ = _k_ * _G_, where _k_ is the private key, _G_ is a constant point called the _generator point_, and _K_ is the resulting public key. The reverse operation, known as "finding the discrete logarithm"—calculating _k_ if you know _K_—is as difficult as trying all possible values of _k_, i.e., a brute-force search.

In simpler terms: arithmetic on the elliptic curve is different from "regular" integer arithmetic. A point (G) can be multiplied by an integer (k) to produce another point (K). But there is no such thing as _division_, so it is not possible to simply "divide" the public key K by the point G to calculate the private key k. This is the one-way mathematical function described in <<pkc>>.

[TIP]
====
Elliptic curve multiplication is a type of function that cryptographers call a "one way" function: it is easy to do in one direction (multiplication) and impossible to do in the reverse direction (division). The owner of the private key can easily create the public key and then share it with the world knowing that no one can reverse the function and calculate the private key from the public key. This mathematical trick becomes the basis for unforgeable and secure digital signatures that prove ownership of Ethereum funds and control of contracts.
====

Before we demonstrate how to generate a public key from a private key, let's look at elliptic curve cryptography in a bit more detail.


[[elliptic_curve]]
=== Elliptic curve cryptography explained

((("keys and addresses", "overview of", "elliptic curve cryptography")))((("elliptic curve cryptography", id="eliptic04")))((("cryptography", "elliptic curve cryptography", id="Celliptic04")))Elliptic curve cryptography is a type of asymmetric or public key cryptography based on the discrete logarithm problem as expressed by addition and multiplication on the points of an elliptic curve.

<<ecc-curve>> is an example of an elliptic curve, similar to that used by Ethereum.

[TIP]
====
Ethereum uses the exact same elliptic curve, called +secp256k1+, as Bitcoin. That makes it possible to re-use many of the elliptic curve libraries and tools from Bitcoin.
====

[[ecc-curve]]
[role="smallerthirty"]
.A visualization of an elliptic curve
image::images/simple_elliptic_curve.png["ecc-curve"]

Ethereum uses a specific elliptic curve and set of mathematical constants, as defined in a standard called +secp256k1+, established by the National Institute of Standards and Technology (NIST). The +secp256k1+ curve is defined by the following function, which produces an elliptic curve:

[latexmath]
++++
\begin{equation}
{y^2 = (x^3 + 7)}~\text{over}~(\mathbb{F}_p)
\end{equation}
++++

or

[latexmath]
++++
\begin{equation}
{y^2 \mod p = (x^3 + 7) \mod p}
\end{equation}
++++

The _mod p_ (modulo prime number p) indicates that this curve is over a finite field of prime order _p_, also written as latexmath:[\( \mathbb{F}_p \)], where p = 2^256^ – 2^32^ – 2^9^ – 2^8^ – 2^7^ – 2^6^ – 2^4^ – 1, a very large prime number.

Because this curve is defined over a finite field of prime order instead of over the real numbers, it looks like a pattern of dots scattered in two dimensions, which makes it difficult to visualize. However, the math is identical to that of an elliptic curve over real numbers. As an example, <<ecc-over-F17-math>> shows the same elliptic curve over a much smaller finite field of prime order 17, showing a pattern of dots on a grid. The +secp256k1+ Ethereum elliptic curve can be thought of as a much more complex pattern of dots on an unfathomably large grid.

[[ecc-over-F17-math]]
[role="smallersixty"]
.Elliptic curve cryptography: visualizing an elliptic curve over F(p), with p=17
image::images/ec_over_small_prime_field.png["ecc-over-F17-math"]

So, for example, the following is a point Q with coordinates (x,y) that is a point on the +secp256k1+ curve:

[[coordinates_example]]
----
Q = (49790390825249384486033144355916864607616083520101638681403973749255924539515, 59574132161899900045862086493921015780032175291755807399284007721050341297360)
----

<<example_1>> shows how you can check this yourself using Python. The variables x and y are the coordinates of the point Q as above. The variable p is the prime order of the elliptic curve (the prime that is used for all the modulo operations). The last line of Python is the elliptic curve equation (the % operator in Python is the modulo operator). If x and y are indeed points on the elliptic curve, then they satisfy the equation and the result is zero (+0L+ is a long integer with value zero). Try it yourself, by typing +python+ on a command line and copying each line (after the prompt +>>>+) from the listing:

[[example_1]]
.Using Python to confirm that this point is on the elliptic curve
====
[source, pycon]
----
Python 3.4.0 (default, Mar 30 2014, 19:23:13)
[GCC 4.2.1 Compatible Apple LLVM 5.1 (clang-503.0.38)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> p = 115792089237316195423570985008687907853269984665640564039457584007908834671663
>>> x = 49790390825249384486033144355916864607616083520101638681403973749255924539515
>>> y = 59574132161899900045862086493921015780032175291755807399284007721050341297360
>>> (x ** 3 + 7 - y**2) % p
0L
----
====

[[EC_math]]
=== Elliptic curve arithmetic operations

A lot of elliptic curve math looks and works very much like the integer arithmetic we learned at school. Specifically, we can define an addition operator, which instead of adding numbers is adding points on the curve. Once we have the addition operator, we can also define multiplication of a point and a whole number, such that it is equivalent to repeated addition.

Addition is defined such that given two points P~1~ and P~2~ on the elliptic curve, there is a third point P~3~ = P~1~ + P~2~, also on the elliptic curve.

Geometrically, this third point P~3~ is calculated by drawing a line between P~1~ and P~2~. This line will intersect the elliptic curve in exactly one additional place. Call this point P~3~' = (x, y). Then reflect in the x-axis to get P~3~ = (x, –y).

In elliptic curve math, there is a point called the "point at infinity," which roughly corresponds to the role of the number zero in addition. On computers, it's sometimes represented by x = y = 0 (which doesn't satisfy the elliptic curve equation, but it's an easy separate case that can be checked). There are a couple of special cases that explain the need for the "point at infinity."

If P~1~ and P~2~ are the same point, the line "between" P~1~ and P~2~ should extend to be the tangent on the curve at this point P~1~. This tangent will intersect the curve in exactly one new point. You can use techniques from calculus to determine the slope of the tangent line. These techniques curiously work, even though we are restricting our interest to points on the curve with two integer coordinates!

In some cases (i.e., if P~1~ and P~2~ have the same x values but different y values), the tangent line will be exactly vertical, in which case P3 = "point at infinity."

If P~1~ is the "point at infinity," then P~1~ + P~2~ = P~2~. Similarly, if P~2~ is the point at infinity, then P~1~ + P~2~ = P~1~. This shows how the point at infinity plays the role that zero plays in "normal" arithmetic.

It turns out that pass:[+] is associative, which means that (A pass:[+] B) pass:[+] C = A pass:[+] (B pass:[+] C). That means we can write A pass:[+] B pass:[+] C without parentheses and without ambiguity.

Now that we have defined addition, we can define multiplication in the standard way that extends addition. For a point P on the elliptic curve, if k is a whole number, then k pass:[*] P = P + P + P + ... + P (k times). Note that k is sometimes confusingly called an "exponent" in this case

[[public_key_derivation]]
=== Generating a public key

((("keys and addresses", "overview of", "public key generation")))((("generator point")))Starting with a private key in the form of a randomly generated number _k_, we multiply it by a predetermined point on the curve called the _generator point_ _G_ to produce another point somewhere else on the curve, which is the corresponding public key _K_. The generator point is specified as part of the +secp256k1+ standard and is always the same for all implementations of +secp256k1+ and all keys derived from that curve use the same point _G_:

[latexmath]
++++
\begin{equation}
{K = k * G}
\end{equation}
++++

where _k_ is the private key, _G_ is the generator point, and _K_ is the resulting public key, a point on the curve. Because the generator point is always the same for all Ethereum users, a private key _k_ multiplied with _G_ will always result in the same public key _K_. The relationship between _k_ and _K_ is fixed, but can only be calculated in one direction, from _k_ to _K_. That's why an Ethereum address (derived from _K_) can be shared with anyone and does not reveal the user's private key (_k_).

As we described in <<EC_math>>, the multiplication of k * G is equivalent to repeated addition, so G + G + G + ... + G, repeated k times. In summary, to produce a public key _K_, from a private key _k_, we add the generator point _G_ to itself, _k_ times.

[TIP]
====
A private key can be converted into a public key, but a public key cannot be converted back into a private key because the math only works one way.
====

Let's apply this calculation to find the public key for the specific private key we showed you in <<private_keys>>:


[[example_privkey]]
.Example private key to public key calculation
----
K = f8f8a2f43c8376ccb0871305060d7b27b0554d2cc72bccf41b2705608452f315 * G
----

A cryptographic library can help us calculate K, using elliptic curve multiplication. The resulting public key _K_ is defined as a point +K = (x,y)+:

[[example_pubkey]]
.Example public key calculated from the example private key
----
K = (x, y)

where,

x = 6e145ccef1033dea239875dd00dfb4fee6e3348b84985c92f103444683bae07b
y = 83b5c38e5e2b0c8529d7fa3f64d46daa1ece2d9ac14cab9477d042c84c32ccd0
----

In Ethereum you may see public keys represented as a hexadecimal serialization of 66 hexadecimal characters (33 bytes). This is adopted from a standard serialization format proposed by the industry consortium Standards for Efficient Cryptography Group (SECG), documented in http://www.secg.org/sec1-v2.pdf[Standards for Efficient Cryptography (SEC1)]. The standard defines four possible prefixes that can be used to identify points on an elliptic curve:

[[EC_prefix_table]]
|===
| Prefix | Meaning | Length (bytes counting prefix) |
|0x00| Point at Infinity | 1 |
|0x04| Uncompressed Point | 65 |
|0x02| Compressed Point with even Y | 33 |
|0x03| Compressed Point with odd Y | 33 |
|===

Ethereum only uses uncompressed public keys, therefore the only prefix that is relevant is (hex) +04+. The serialization concatenated the X and Y coordinates of the public key:

[[concat_coordinates]]
----
04 + X-coordinate (32 bytes/64 hex) + Y coordinate (32 bytes/64 hex)
----

Therefore, the public key we calculated in <<example_pubkey>> is serialized as:

[[serialized_pubkey]]
----
046e145ccef1033dea239875dd00dfb4fee6e3348b84985c92f103444683bae07b83b5c38e5e2b0c8529d7fa3f64d46daa1ece2d9ac14cab9477d042c84c32ccd0
----

[[EC_lib]]
=== Elliptic curve libraries

There are a couple of implementations of the secp256k1 elliptic curve that are used in cryptocurrency related projects:

((("OpenSSL cryptographic library")))OpenSSL:: The OpenSSL library offers a comprehensive set of cryptographic primitives, including a full implementation of the secp256k1. For example, to derive the public key, the function +EC_POINT_mul()+ can be used. Find it at https://www.openssl.org/

((("libsecp256k1 cryptographic library")))libsecp256k1:: Bitcoin Core's libsecp256k1, is a C-language implementation of the secp256k1 elliptic curve and other cryptographic primitives. The libsecp256 of elliptic curve cryptography was written from scratch to replace OpenSSL in Bitcoin Core software, and is considered superior in both performance and security. Find it at: https://github.com/bitcoin-core/secp256k1

[[hash_functions]]
=== Cryptographic hash functions

((("hash function")))Cryptographic hash functions are used throughout Ethereum. In fact, hash functions are used extensively in almost all cryptographic systems, a fact captured by cryptographer Bruce Schneier who said "Much more than encryption algorithms, one-way hash functions are the workhorses of modern cryptography."

In this section we will discuss hash functions, understand their basic properties and how those properties make them so useful in so many areas of modern cryptography. We address hash functions here, because they are part of the transformation of Ethereum public keys into addresses.

In simple terms, "a hash function is any function that can be used to map data of arbitrary size to data of fixed size." https://en.wikipedia.org/wiki/Hash_function[Source: Wikipedia]. The input to a hash function is called a ((("pre image")))_pre-image_ or _message_. The output is called a _hash_, or _digest_. A special sub-category of hash functions is _cryptographic hash functions_, which have specific properties that are useful to cryptography.

A cryptographic hash function is a _one way_ hash function that maps data of arbitrary size to a fixed-size bit string, where it is computationally infeasible to recreate the input if one knows the output. The only way to determine the input is to conduct a brute-force search of possible inputs, checking for a matching output.

Cryptographic hash functions have five main properties (https://en.wikipedia.org/wiki/Cryptographic_hash_function[Source: Wikipedia/Cryptographic Hash Function]):

Determinism:: Any input message always produces the same hash digest.

Verifiability:: Computing the hash of a message is efficient (linear performance).

Uncorrelated:: A small change to the message (e.g. one bit change) should change the hash output so extensively that it cannot be correlated to the hash of the original message.

Irreversibility (resistance to first pre-image):: Computing the message from a hash is infeasible, equivalent to a brute force search through possible messages.

Collision Protection (resistance to second pre-image):: It should be infeasible to calculate two different messages that produce the same hash output.

Resistance to second pre-image is primarily important to prevent digital signature forgery in Ethereum.

The combination of these properties make cryptographic hash functions useful for a broad range of security applications including:

* Data fingerprinting
* Message integrity (error detection)
* Proof-of-Work
* Authentication (password hashing and key stretching)
* Pseudo-random number generators
* Pre-image commitment
* Unique identifiers

We will find many of these in Ethereum, as we progress through the various layers of the system.

[[keccak256]]
=== Ethereum's cryptographic hash function - Keccak-256

((("SHA-3 Hash Function")))((("Keccak Hash Function")))((("Keccak-256")))Ethereum uses the _Keccak-256_ cryptographic hash function in many places. Keccak-256 was designed as a candidate for the SHA-3 Cryptographic Hash Function Competition held in 2007 by the ((("NIST")))National Institute of Science and Technology (NIST). Keccak was the winning algorithm that became standardized as ((("FIPS")))Federal Information Processing Standard (FIPS) ((("FIPS-202")))202 in 2015.

However, during the period when Ethereum was developed, NIST standardization was being finalized. NIST adjusted some of the parameters of Keccak after the completion of the standards process, allegedly to improve its efficiency. This was occurring at the same time as heroic whistleblower ((("Edward Snowden")))Edward Snowden revealed documents that imply that NIST may have been improperly influenced by the National Security Agency to intentionally weaken the ((("Dual_EC_DRBG")))Dual_EC_DRBG random-number generator standard, effectively placing a backdoor in the standard random number generator. The result of this controversy was a backlash against the proposed changes and a significant delay in the standardization of SHA-3. At the time, the Ethereum Foundation decided to implement the original Keccak algorithm, as proposed by its inventors, rather than the SHA-3 standard as modified by NIST.

[WARNING]
====
While you may see "SHA3" mentioned throughout Ethereum documents and code, many if not all of those instances actually refer to Keccak-256, not the finalized FIPS-202 SHA-3 standard. The implementation differences are slight, having to do with padding parameters, but they are significant in that Keccak-256 produces different hash output than FIPS-202 SHA-3 given the same input.
====

Due to the confusion created by the difference between the hash function used in Ethereum (Keccak-256) and the finalized standard (FIP-202 SHA-3), there is an effort underway to rename all instances of +sha3+ in all code, opcodes and libraries to +keccak256+. See https://github.com/ethereum/EIPs/issues/59[ERC-59] for details.

[[which_hash]]
=== Which hash function am I using?

How can you tell if the software library you are using is FIPS-202 SHA-3 or Keccak-256, if both might be called "SHA3"?

An easy way to tell is to use a _test vector_, an expected output for a given input. The test most commonly used for a hash function is the _empty input_. If you run the hash function with an empty string as input you should see the following results:

[[sha3_test_vectors]]
.Testing whether the SHA3 library you are using is Keccak-256 of FIP-202 SHA-3
----
Keccak256("") =
c5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470

SHA3("") =
a7ffc6f8bf1ed76651c14756a061d662f580ff4de43b49fa82d80a4b80f8434a
----

So, regardless of what the function is called, you can test it to see whether it is the original Keccak-256, or the final NIST standard FIPS-202 SHA-3, by running the simple test above. Remember, Ethereum uses Keccak-256, even though it is often called SHA-3 in the code.

Next, let's examine the first application of Keccak-256 in Ethereum, which is to produce Ethereum addresses from public keys.

[[eth_address]]
=== Ethereum addresses

Ethereum addresses are _unique identifiers_ that are derived from public keys or contracts using a one-way hash function (specifically Keccak-256).

In our previous examples, we started with a private key and used elliptic curve multiplication to derive a public key:

Private Key _k_:
----
k = f8f8a2f43c8376ccb0871305060d7b27b0554d2cc72bccf41b2705608452f315
----

[[concat_pubkey]]
Public Key _K_ (X and Y coordinates concatenated and shown as hex):
----
K = 6e145ccef1033dea239875dd00dfb4fee6e3348b84985c92f103444683bae07b83b5c38e5e2b0c8529d7fa3f64d46daa1ece2d9ac14cab9477d042c84c32ccd0
----

[WARNING]
====
It is worth noting that the public key is not formatted with the prefix (hex) 04 when the address is calculated.
====

We use Keccak-256 to calculate the _hash_ of this public key:

[[calculate_hash]]
----
Keccak256(K) = 2a5bc342ed616b5ba5732269001d3f1ef827552ae1114027bd3ecf1f086ba0f9
----

Then we keep only the last 20 bytes (the least significant bytes in big-endian), which is our Ethereum address:

[[keep_last_20]]
----
001d3f1ef827552ae1114027bd3ecf1f086ba0f9
----

Most often you will see Ethereum addresses with the prefix "0x" that indicates it is a hexadecimal encoding, like this:

[[hex_prefix]]
----
0x001d3f1ef827552ae1114027bd3ecf1f086ba0f9
----

[[eth_address_format]]
=== Ethereum address formats

Ethereum addresses are hexadecimal numbers, identifiers derived from the last 20 bytes of the Keccak-256 hash of the public key.

Unlike Bitcoin addresses which are encoded in the user interface of all clients to include a built-in checksum to protect against mistyped addresses, Ethereum addresses are presented as raw hexadecimal without any checksum.

The rationale behind that decision was that Ethereum addresses would eventually be hidden behind abstractions (such as name services) at higher layers of the system and that checksums should be added at higher layers if necessary.

In retrospect, this design choice lead to a number of problems, including the loss of funds due to mistyped addresses and input validation errors. Ethereum name services were developed slower than initially expected and alternative encodings such as ICAP were adopted very slowly by wallet developers.

[[ICAP]]
==== Inter Exchange Client Address Protocol (ICAP)

The _Inter exchange Client Address Protocol (ICAP)_ is an Ethereum Address encoding that is partly compatible with the International Bank Account Number (IBAN) encoding, offering a versatile, checksummed and interoperable encoding for Ethereum Addresses. ICAP addresses can encode Ethereum Addresses or common names registered with an Ethereum name registry.

Read about ICAP on the Ethereum Wiki:https://github.com/ethereum/wiki/wiki/ICAP:-Inter-exchange-Client-Address-Protocol

IBAN is an international standard for identifying bank account numbers, mostly used for wire transfers. It is broadly adopted in the European Single Euro Payments Area (SEPA) and beyond. IBAN is a centralized and heavily regulated service. ICAP is a decentralized but compatible implementation for Ethereum addresses.

An IBAN consists of up to 34 alphanumeric characters (case-insensitive) string containing a country code, checksum, and bank account identifier (which is country-specific).

ICAP uses the same structure by introducing a non-standard country code "XE" that stands for "Ethereum", followed by a two-character checksum and 3 possible variations of an account identifier:

Direct:: Up to 30 alphanumeric character big-endian base-36 integer representing the least significant bits of an Ethereum address. Because this encoding fits less than 155 bits, it only works for Ethereum addresses that start with one or more zero bytes. The advantage is that it is compatible with IBAN, in terms of the field length and checksum. Example: +XE60HAMICDXSV5QXVJA7TJW47Q9CHWKJD+ (33 characters long)

Basic:: Same as the "Direct" encoding except that it is 31 characters long. This allows it to encode any Ethereum address, but makes it incompatible with IBAN field validation. Example: +XE18CHDJBPLTBCJ03FE9O2NS0BPOJVQCU2P+ (35 characters long)

Indirect:: Encodes an identifier that resolves to an Ethereum address through a name registry provider. Uses 16 alphanumeric characters, composed of an _asset identifier_ (e.g. ETH), a name service (e.g. XREG) and a 9-character name (e.g. KITTYCATS), which is a human-readable name. Example: +XEpass:[##]ETHXREGKITTYCATS+ (20 characters long), where the "##" should be replaced by the two computed checksum characters.

We can use the +helpeth+ command-line tool to create ICAP addresses. Let's try with our example private key (prefixed with 0x and passed as a parameter to helpeth):

[[create_ICAP]]
----
$ helpeth keyDetails -p 0xf8f8a2f43c8376ccb0871305060d7b27b0554d2cc72bccf41b2705608452f315

Address: 0x001d3f1ef827552ae1114027bd3ecf1f086ba0f9
ICAP: XE60 HAMI CDXS V5QX VJA7 TJW4 7Q9C HWKJ D
Public key: 0x6e145ccef1033dea239875dd00dfb4fee6e3348b84985c92f103444683bae07b83b5c38e5e2b0c8529d7fa3f64d46daa1ece2d9ac14cab9477d042c84c32ccd0
----

The +helpeth+ command constructs a hexadecimal Ethereum address as well as an ICAP address for us. The ICAP address for our example key is:

[[ICAP_example]]
----
XE60HAMICDXSV5QXVJA7TJW47Q9CHWKJD
----

Because our example Ethereum address happens to start with a zero byte, it can be encoded using the "Direct" ICAP encoding method that is valid in an IBAN format. You can tell because it is 33 characters long.

If our address did not start with a zero, it would be encoded with the "Basic" encoding, which would be 35 characters long and invalid as an IBAN format.

[TIP]
====
The chances of any Ethereum address starting with a zero byte are 1 in 256. To generate one like that, it will take on average 256 attempts with 256 different random private keys before we find one that works as an IBAN-compatible "Direct" encoded ICAP address.
====

At this time, ICAP is unfortunately only supported by a few wallets.

[[EIP55]]
==== Hex encoding with checksum in capitalization (EIP-55)

Due to the slow deployment of ICAP or name services, a new standard was proposed with Ethereum Improvement Proposal 55 (EIP-55). You can read the details at:

https://github.com/Ethereum/EIPs/blob/master/EIPS/eip-55.md

EIP-55 offers a backward compatible checksum for Ethereum addresses by modifying the capitalization of the hexadecimal address. The idea is that Ethereum addresses are case-insensitive and all wallets are supposed to accept Ethereum addresses expressed in capital or lower-case characters, without any difference in interpretation.

By modifying the capitalization of the alphabetic characters in the address, we can convey a checksum that can be used to protect the integrity of the address against typing or reading mistakes. Wallets that do not support EIP-55 checksums simply ignore the fact that the address contains mixed capitalization. But those that do support it, can validate it and detect errors with a 99.986% accuracy.

The mixed-capitals encoding is subtle and you may not notice it at first. Our example address is:

----
0x001d3f1ef827552ae1114027bd3ecf1f086ba0f9
----

with an EIP-55 mixed-capitalization checksum it becomes:

[[mixed_capitalization]]
----
0x001d3F1ef827552Ae1114027BD3ECF1f086bA0F9
----

Can you tell the difference? Some of the alphabetic (A-F) characters from the hexadecimal encoding alphabet are now capital, while others are lower case. You might not even have noticed the difference unless you looked carefully.

EIP-55 is quite simple to implement. We take the Keccak-256 hash of the lower-case hexadecimal address. This hash acts as a digital fingerprint of the address, giving us a convenient checksum. Any small change in the input (the address) should cause a big change in the resulting hash (the checksum), allowing us to detect errors effectively. The hash of our address is then encoded in the capitalization of the address itself. Let's break it down, step-by-step:

1. Hash the lower-case address, without the +0x+ prefix:

[[hash_lower_case_address]]
----
Keccak256("001d3f1ef827552ae1114027bd3ecf1f086ba0f9")
23a69c1653e4ebbb619b0b2cb8a9bad49892a8b9695d9a19d8f673ca991deae1
----

[start=2]
1. Capitalize each alphabetic address character if the corresponding hex digit of the hash is greater than or equal to +0x8+. This is easier to show if we line up the address and the hash:

[[capitalize_input]]
----
Address: 001d3f1ef827552ae1114027bd3ecf1f086ba0f9
Hash   : 23a69c1653e4ebbb619b0b2cb8a9bad49892a8b9...
----

Our address contains an alphabetic character +d+ in the fourth position. The fourth character of the hash is +6+, which is less than +8+. So, we leave the +d+ lower-case. The next alphabetic character in our address is +f+, in the sixth position. The sixth character of the hexadecimal hash is +c+, which is greater than +8+. Therefore, we capitalize the +F+ in the address, and so on. As you can see, we only use the first 20-bytes (40 hex characters) of the hash as a checksum, since we only have 20-bytes (40 hex characters) in the address to capitalize appropriately.

Check the resulting mixed-capitals address yourself and see if you can tell which characters were capitalized and which characters they correspond to in the address hash:

[[capitalize_output]]
----
Address: 001d3F1ef827552Ae1114027BD3ECF1f086bA0F9
Hash   : 23a69c1653e4ebbb619b0b2cb8a9bad49892a8b9...
----

[[EIP55_error]]
==== Detecting an error in an EIP-55 encoded address

Now, let's look at how EIP-55 addresses will help us find an error. Let's assume we have printed out an Ethereum address, which is EIP-55 encoded:

[[correct_address]]
----
0x001d3F1ef827552Ae1114027BD3ECF1f086bA0F9
----

Now let's make a basic mistake in reading that address. The character before the last one is a capital "F". For this example let's assume we misread that as a capital "E". We type in the (incorrect address) into our wallet:

[[incorrect_address]]
----
0x001d3F1ef827552Ae1114027BD3ECF1f086bA0E9
----

Fortunately, our wallet is EIP-55 compliant! It notices the mixed capitalization and attempts to validate the address. It converts it to lower case, and calculates the checksum hash:

[[hash_demo]]
----
Keccak256("001d3f1ef827552ae1114027bd3ecf1f086ba0e9")
5429b5d9460122fb4b11af9cb88b7bb76d8928862e0a57d46dd18dd8e08a6927
----

As you can see, even though the address has only changed by one character (in fact, only one bit as "e" and "f" are 1-bit apart), the hash of the address has changed radically. That's the property of hash functions that makes them so useful for checksums!

Now, let's line up the two and check the capitalization:

[[incorrect_capitalization]]
----
001d3F1ef827552Ae1114027BD3ECF1f086bA0E9
5429b5d9460122fb4b11af9cb88b7bb76d892886...
----

It's all wrong! Several of the alphabetic characters are incorrectly capitalized. Remember that the capitalization is the encoding of the _correct_ checksum.

The capitalization of the address we input doesn't match the checksum just calculated, meaning something has changed in the address, and an error has been introduced.