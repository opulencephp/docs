# Authentication

## Note
The authentication library was originally meant to be released in a subsequent version of Opulence.  However, we've decided it's important to provide something out of the box, so we are currently working on our authentication library.

<a href="https://github.com/opulencephp/Opulence/tree/develop/src/Opulence/Authentication" target="_blank">Follow the updates</a> on the `develop` branch.

## Table of Contents
1. [JSON Web Tokens](#jwt)
  1. [Introduction](#jwt-introduction)
  2. [Building JWTs](#building-jwts)
  3. [Signing JWTs](#signing-jwts)
  4. [Verifying JWTs](#verifying-jwts)
  5. [Creating JWTs from Strings](#creating-jwts-from-strings)

<h2 id="jwt">JSON Web Tokens</h2>

<h4 id="jwt-introduction">Introduction</h4>
JSON web tokens are great ways for passing claims (such as a user's identity) between a client and the server.  They consist of three parts:
1. Header - The algorithm used to sign the token, the content type (JWT), and the token type (JWT)
2. Payload - The data actually being sent in the token (also called "claims")
  * <a href="https://en.wikipedia.org/wiki/JSON_Web_Token" target="_blank">Read more about standard payload claims</a>
3. Signature - The hashed result of the header and the payload (prevents tampering with payload data)

Typically, you'll see JWTs as strings in the following format: "{base64-encoded header}.{base64-encoded payload}.{base64-encoded signature}".

<h4 id="building-jwts">Building JWTs</h4>
```php
use DateTimeImmutable;
use Opulence\Authentication\Tokens\JsonWebTokens\JwtHeader;
use Opulence\Authentication\Tokens\JsonWebTokens\JwtPayload;
use Opulence\Authentication\Tokens\JsonWebTokens\UnsignedJwt;
use Opulence\Authentication\Tokens\Signatures\Algorithms;
use Opulence\Authentication\Tokens\Signatures\Factories\SignerFactory;

// The algorithm can be any of the constants in Algorithms
$algorithm = Algorithms::SHA256;

// The signer will be used when we actually encode our JWT
// Keys can either be strings or resources
// Private keys are only necessary for Algorithms::RSA_* algorithms
$signer = (new SignerFactory)->createSigner($algorithm, "myPublicKey", "myPrivateKey");

// Create our JWT's components
$header = new JwtHeader($algorithm);
$payload = new JwtPayload();
$payload->setIssuer("http://mysite.com");
$payload->setValidTo(new DateTimeImmutable("+30 days"));
// Set a custom field in our payload
$payload->add("username", "dave");

// Create our unsigned JWT
$unsignedJwt = new UnsignedJwt($header, $payload);
```

<h4 id="signing-jwts">Signing JWTs</h4>
To encode your JWT, you'll need to sign it using an `ISigner`.

```php
use Opulence\Authentication\Tokens\JsonWebTokens\SignedJwt;

$signature = $signer->sign($unsignedJwt->getUnsignedValue());
$signedJwt = new SignedJwt(
    $unsignedJwt->getHeader(),
    $unsignedJwt->getPayload(),
    $signature
);
$signedJwt->encode(); // Returns the encoded JWT
```

<h4 id="verifying-jwts">Verifying JWTs</h4>
Verifying a JWT is simple.  Create a `VerificationContext` and specify the fields we want to verify against.  The verifier will then compare those fields to the JWT's claims.  It will also verify that the signature is correct.

```php
use Opulence\Authentication\Tokens\JsonWebTokens\Verification\JwtVerifier;
use Opulence\Authentication\Tokens\JsonWebTokens\Verification\VerificationContext;

$context = new VerificationContext($signer);
$context->setIssuer("http://mysite.com");
$verifier = new JwtVerifier();
$verifier->verify($signedJwt, $context); // Throws an exception on failure
```

> **Note:** The not-before and expiration times are only verified if they're specified.  If your context does not set particular values, eg no issuer is set in the context, then that claim is skipped during verification.

<h4 id="creating-jwts-from-strings">Creating JWTs from Strings</h4>
You can easily create a `SignedJwt` from a string in the format "{base64-encoded header}.{base64-encoded payload}.{base64-encoded signature}".

```php
$signedJwt = SignedJwt::createFromString($tokenString);
```

> **Note:** Tokens created in this way are not verified.  You must pass them through `JwtVerifier::verify()` to verify them. 