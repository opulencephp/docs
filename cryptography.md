# Cryptography

## Table of Contents
1. [Introduction](#introduction)
2. [Hashers](#hashers)
  1. [BCrypt](#bcrypt)
3. [Encrypters](#encrypters)
4. [String Utility](#string-utility)
  1. [Generating Random Strings](#generating-random-strings)
  2. [String Comparison](#string-comparison)

<h2 id="introduction">Introduction</h2>
Keeping user data secure is of the utmost importance.  Unfortunately, PHP's built-in cryptographic support is somewhat fragmented and not easy to use.  Lucky for you, RDev has a `Cryptography` library to simplify all this.

<h2 id="hashers">Hashers</h2>
Hashers take input and perform a one-way mapping to a hashed value.  It is impossible to *decrypt* a hashed value because of the way this mapping is generated.  This hashes suitable for storing sensitive data, like user passwords.

<h4 id="bcrypt">Bcrypt</h4>
`Bcrypt` is a popular hashing function that has built-in protection methods against timing attacks.  It accepts a "cost" parameter, which tells `Bcrypt` how long to take when attempting to verify an unhashed value.  Every time the cost goes up by one, the hasher takes 10 times longer to hash.  This prevents GPUs from being able to efficiently perform rainbow table attacks against compromised data.  You can adjust this value to make it future-proof.  Let's take a look at how to use it:

```php
use RDev\Cryptography\Hashing;
use RDev\Cryptography\Utilities;

$bcryptHasher = new Hashing\BCryptHasher(new Utilities\Strings());

// Let's create a hash with a pepper of "bar"
// $hash is automatically salted and suitable for database storage
$hashedValue = $bcryptHasher->generate("foo", ["cost" => 10], "bar");
```

To verify that an unhashed value hashes to a particular value, use `verify()`:

```php
$unhashedValue = "foo";
echo $bcryptHasher->verify($hashedValue, $unhashedValue, "bar"); // 1
```

<h2 id="encrypters">Encrypters</h2>
Sometimes, your application needs to encrypt data, send it to another component, and then decrypt it.  This is different from hashing in that encrypted values can be decrypted.  To make this process as secure and simple as possible, RDev provides the `Encrypter` class:

```php
use RDev\Cryptography\Encryption;
use RDev\Cryptography\Utilities;

// This should be a unique, random-generated string
$myApplicationKey = "mySecretApplicationKey";
$encrypter = new Encryption\Encrypter($myApplicationKey, new Utilities\Strings());
$unencryptedData = "foobar";
$encryptedData = $encrypter->encrypt($unencryptedData);
echo $unencryptedData === $encrypter->decrypt($encryptedData); // 1 
```

You can change the underlying `MCrypt` cipher and mode using `setCipher()` and `setMode()`, respectively.

<h2 id="string-utility">String Utility</h2>
RDev has a utility class `RDev\Cryptography\Utilities\Strings` to make it easy to do cryptographically-secure string functions.

<h4 id="generating-random-strings">Generating Random Strings</h4>
It turns out that computers are not very good at generating random numbers.  Calling `rand()` over and over will begin to yield a pattern, which is obviously the opposite of random.  PHP recommends using `openssl_random_pseudo_bytes()`, which RDev provides a simple wrapper around:

```php
use RDev\Cryptography\Utilities;

$stringUtility = new Utilities\Strings();
echo $stringUtility->generateRandomString(16); // A random 16-character string
```

<h4 id="string-comparison">String Comparison</h4>
A hacker can sometimes backwards-engineer a secret value by timing how long comparisons take against his input.  For example, the following will return false after only comparing 2 characters:

```php
"aaaaa" == "bbbbb";
```

This will return false after comparing 5 characters and will take a little longer than the previous comparison:

```php
"bbbba" == "bbbbb";
```

Because of this comparison timing vulnerability, it's important to compare the entire length of the user input against the known value, even if you know they're not equal from the first character.  RDev has a wrapper function around **Symfony's** excellent safe string comparison function:

```php
echo $stringUtility->isEqual("aaaaa", "bbbbb"); // 0
```

This comparison will take the same amount of time as:

```php
echo $stringUtility->isEqual("bbbba", "bbbbb"); // 0
```