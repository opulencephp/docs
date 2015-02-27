# Cryptography

## Table of Contents
1. [Introduction](#introduction)
2. [Hashers](#hashers)
  1. [BCrypt](#bcrypt)

<h2 id="introduction">Introduction</h2>
Keeping user data secure is of the utmost importance.  Unfortunately, PHP's built-in cryptographic support is somewhat fragmented and not easy to use.  Lucky for you, RDev has a `Cryptography` library to simplify all this.

<h2 id="hashers">Hashers</h2>
Hashers take input and perform a one-way mapping to a hashed value.  It is impossible to *decrypt* a hashed value because of the way this mapping is generated.  This hashes suitable for storing sensitive data, like user passwords.

<h4 id="bcrpyt">Bcrypt</h4>
`Bcrypt` is a popular hashing function that has built-in protection methods against timing attacks.  It accepts a "cost" parameter, which tells `Bcrypt` how long to take (higher numbers take longer) when attempting to verify an unhashed value.  This prevents GPUs and future GPUs from being able to efficiently perform rainbow table attacks against compromised data.  Let's take a look at how to use it:

```php
use RDev\Cryptography;

$stringUtility = new Cryptography\StringUtility();
$bcryptHasher = new BCryptHasher($stringUtility);

// Let's create a hash with a pepper of "bar"
// $hash is automatically salted and suitable for database storage
$hashedValue = $bcryptHasher->generate("foo", ["cost" => 10], "bar");
```

To verify that an unhashed value hashes to a particular value, use `verify()`:

```php
$unhashedValue = "foo";
echo $bcryptHasher->verify($hashedValue, $unhashedValue, "bar"); // 1
```