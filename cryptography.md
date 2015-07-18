# Cryptography

## Table of Contents
1. [Introduction](#introduction)
2. [Hashing](#hashing)
  1. [bcrypt](#bcrypt)
3. [Encryption](#encryption)
  1. [Encrypting Data](#encrypting-data)
  2. [Decrypting Data](#decrypting-data)
4. [String Utility](#string-utility)
  1. [Generating Random Strings](#generating-random-strings)
  2. [String Comparison](#string-comparison)

<h2 id="introduction">Introduction</h2>
Keeping user data secure is of the utmost importance.  Unfortunately, PHP's built-in cryptographic support is somewhat fragmented and not easy to use.  Lucky for you, Opulence has a `Cryptography` library to simplify all this.

<h2 id="hashing">Hashing</h2>
Hashers take input and perform a one-way mapping to a hashed value.  It is impossible to decrypt a hashed value because of the way this mapping is generated.  This hashes suitable for storing sensitive data, like user passwords.

<h4 id="bcrypt">bcrypt</h4>
`bcrypt` is a popular hashing function that has built-in protection methods against timing attacks.  It accepts a "cost" parameter, which tells `bcrypt` how long to take when attempting to verify an unhashed value.  Every time the cost goes up by one, the hasher takes 10 times longer to hash.  This prevents GPUs from being able to efficiently perform rainbow table attacks against compromised data.  You can adjust this value to make it future-proof.  Let's take a look at how to use it:

```php
use Opulence\Cryptography\Hashing\BcryptHasher;
use Opulence\Cryptography\Utilities\Strings;

$bcryptHasher = new BcryptHasher(new Strings());

// Let's create a hash with a pepper of "bar"
// $hash is automatically salted and suitable for database storage
$hashedValue = $bcryptHasher->hash("foo", ["cost" => 10], "bar");
```

To verify that an unhashed value hashes to a particular value, use `verify()`:

```php
$unhashedValue = "foo";
echo $bcryptHasher->verify($hashedValue, $unhashedValue, "bar"); // 1
```

<h2 id="encryption">Encryption</h2>
Sometimes, your application needs to encrypt data, send it to another component, and then decrypt it.  This is different from hashing in that encrypted values can be decrypted.  To make this process as secure and simple as possible, Opulence has an easy-to-use wrapper around `OpenSSL` in its `Encrypter` class:

```php
use Opulence\Cryptography\Encryption\Encrypter;
use Opulence\Cryptography\Encryption\EncryptionException;
use Opulence\Cryptography\Utilities\Strings;

// This should be a unique, random string
$myApplicationKey = "mySecretApplicationKey";
$encrypter = new Encrypter($myApplicationKey, new Strings());
```

You can change the underlying `OpenSSL` cipher using `setCipher()`.

> **Note:** The default cipher is AES-128-CBC.  Although you can use any OpenSSL cipher you'd like, it is strongly recommended you use an AES CBC cipher such as AES-128-CBC or AES-256-CBC. 

<h4 id="encrypting-data">Encrypting Data</h4>
```php
try
{
    $encryptedData = $encrypter->encrypt("foobar");
}
catch(EncryptionException $ex)
{
    // Handle the exception
}
```

A **Message Authentication Code** (**MAC**) is created using the encrypted value to detect any tampering with the encrypted data.  If there was any issue encrypting the data, an `Opulence\Cryptography\Encryption\EncryptionException` will be thrown.

<h4 id="decrypting-data">Decrypting Data</h4>
```php
try
{
    $encryptedData = $encrypter->encrypt("foobar");
    $decryptedData = $encrypter->decrypt($encryptedData);
}
catch(EncryptionException $ex)
{
    // Handle the exception
}

// Verify the decrypted data matches our original value
echo $decryptedData === "foobar"; // 1 
```

If there was any issue decrypting the data, an `Opulence\Cryptography\Encryption\EncryptionException` will be thrown.

<h2 id="string-utility">String Utility</h2>
Opulence has a utility class `Opulence\Cryptography\Utilities\Strings` to make it easy to do cryptographically-secure string functions.

<h4 id="generating-random-strings">Generating Random Strings</h4>
It turns out that computers are not very good at generating random numbers.  Calling `rand()` over and over will begin to yield a pattern, which is obviously the opposite of random.  PHP recommends using `openssl_random_pseudo_bytes()`, which Opulence provides a simple wrapper around:

```php
use Opulence\Cryptography\Utilities\Strings;

$stringUtility = new Strings();
echo $stringUtility->generateRandomString(16); // A random 16-character string
```

<h4 id="string-comparison">String Comparison</h4>
A hacker can sometimes backwards-engineer a secret value by timing how long comparisons take against his input.  For example, the following will return false after only comparing 1 character:

```php
"aaaaa" == "bbbbb";
```

This will return false after comparing 5 characters and will take a little longer than the previous comparison:

```php
"bbbba" == "bbbbb";
```

Because of this comparison timing vulnerability, it's important to compare the entire length of the user input against the known value, even if you know they're not equal from the first character.  Opulence has a wrapper function around **Symfony's** excellent safe string comparison function:

```php
echo $stringUtility->isEqual("aaaaa", "bbbbb"); // 0
```

This comparison will take the same amount of time as:

```php
echo $stringUtility->isEqual("bbbba", "bbbbb"); // 0
```