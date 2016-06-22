# Cryptography

## Table of Contents
1. [Introduction](#introduction)
2. [Hashing](#hashing)
  1. [bcrypt](#bcrypt)
3. [Encryption](#encryption)
  1. [Encrypting Data](#encrypting-data)
  2. [Decrypting Data](#decrypting-data)

<h2 id="introduction">Introduction</h2>
Keeping user data secure is of the utmost importance.  Unfortunately, PHP's built-in cryptographic support is somewhat fragmented and not easy to use.  Lucky for you, Opulence has a `Cryptography` library to simplify all this.

<h2 id="hashing">Hashing</h2>
Hashers take input and perform a one-way mapping to a hashed value.  It is impossible to decrypt a hashed value because of the way this mapping is generated.  This hashes suitable for storing sensitive data, like user passwords.

<h4 id="bcrypt">bcrypt</h4>
`bcrypt` is a popular hashing function that has built-in protection methods against timing attacks.  It accepts a "cost" parameter, which adjusts the CPU cost to hash a password.  This slows down rainbow table attacks against compromised data.  Increasing the cost parameter by one causes the hashing to take twice as long, which future-proofs it as CPUs get faster.  Let's take a look at how to use it:

```php
use Opulence\Cryptography\Hashing\BcryptHasher;

$bcryptHasher = new BcryptHasher();

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

// This should be a unique, random string
$myApplicationKey = "mySecretApplicationKey";
$encrypter = new Encrypter($myApplicationKey);
```

You can change the underlying `OpenSSL` cipher using `setCipher()`.

> **Note:** The default cipher is AES-128-CBC.  Although you can use any OpenSSL cipher you'd like, it is strongly recommended you use an AES CBC cipher such as AES-128-CBC or AES-256-CBC. 

<h4 id="encrypting-data">Encrypting Data</h4>
```php
try {
    $encryptedData = $encrypter->encrypt("foobar");
} catch (EncryptionException $ex) {
    // Handle the exception
}
```

A **Message Authentication Code** (**MAC**) is created using the encrypted value to detect any tampering with the encrypted data.  If there was any issue encrypting the data, an `Opulence\Cryptography\Encryption\EncryptionException` will be thrown.

<h4 id="decrypting-data">Decrypting Data</h4>
```php
try {
    $encryptedData = $encrypter->encrypt("foobar");
    $decryptedData = $encrypter->decrypt($encryptedData);
} catch (EncryptionException $ex) {
    // Handle the exception
}

// Verify the decrypted data matches our original value
echo $decryptedData === "foobar"; // 1 
```

If there was any issue decrypting the data, an `Opulence\Cryptography\Encryption\EncryptionException` will be thrown.