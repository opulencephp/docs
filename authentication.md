# Authentication

## Table of Contents
1. [Introduction](#introduction)
2. [Subjects](#subjects)
  1. [Principals](#principals)
  2. [Example](#subject-example)
3. [Credentials](#credentials)
  1. [Credential Factories](#credential-factories)
  2. [Authenticators](#authenticators)
      1. [Username/Password](#username-password-authenticator)
      2. [JWT Authenticator](#jwt-authenticator)
      3. [JWT Refresh Token Authenticator](#jwt-refresh-token-authenticator)
4. [Authentication Contexts](#authentication-contexts)
  1. [Middleware](#authentication-context-middleware)
5. [JSON Web Tokens](#jwt)
  1. [Introduction](#jwt-introduction)
  2. [Building JWTs](#building-jwts)
  3. [Signing JWTs](#signing-jwts)
  4. [Verifying JWTs](#verifying-jwts)
  5. [Creating JWTs from Strings](#creating-jwts-from-strings)
  6. [JWT Ids](#jwt-ids)

<h2 id="introduction">Introduction</h2>

Opulence's authentication library provides many models out of the box that are common to most authentication schemes.  Unlike other frameworks, Opulence's authentication library is not coupled at all to the rest of the framework, including the authorization library.  So, you may use it in conjunction with 3rd party authentication and authorization libraries.  It is meant to provide you with the tools you need to build your authentication scheme.  It is very possible that, with community support, these tools can be used to create new or bridge existing OpenID Connect implementations.

<h2 id="subjects">Subjects</h2>

A subject is the entity that is requesting access to a resource.  Although a subject could be a user, it could also be something like a system process.  So, it's important to distinguish the concept from users.  Subjects have [principals](#principals) and the [credentials](#credentials) to prove their authenticity.

<h4 id="principals">Principals</h4>

A principal is a piece of information that identifies a subject.  For example, your social security number is one of your principals.  Another might be your name.  In the world of programming, a principal might be a user Id or username.  Opulence requires that exactly one "primary" principal (usually an Id) is set for a subject.

<h4 id="subject-example">Example</h4>

Let's take a look at subjects in Opulence:

```php
use Opulence\Authentication\Principal;
use Opulence\Authentication\PrincipalTypes;
use Opulence\Authentication\Subject;

// This is a list of all roles assigned to this principal
// You are free to define your role names as you'd like
$roles = ['administrator'];
$userIdPrincipal = new Principal(PrincipalTypes::PRIMARY, 123, $roles);
// For now, let's assume $credentials was set previously
$subject = new Subject([$userIdPrincipal], $credentials);
```

To grab the credentials a subject has, call `$subject->getCredentials()`.

To grab the primary principal, call `$subject->getPrimaryPrincipal()`.

To grab all principals, call `$subject->getPrincipals()`.

To grab all roles, call `$subject->getRoles()`.

To check if a subject has a role, call `$subject->hasRole($roleName)`.

<h2 id="credentials">Credentials</h2>

A credential is simply a value or collection of values used to prove a subject's authenticity.  For example, this could be a username/password combination or an OAuth2 access token.

Credentials have a type and value(s):

```php
use Opulence\Authentication\Credentials\Credential;
use Opulence\Authentication\Credentials\CredentialTypes;

// This is a mapping of credential value names to their values
$values = ['username' => 'dave', 'password' => 'mypassword'];
$credential = new Credential(CredentialTypes::USERNAME_PASSWORD, $values);
```

To get the type, call `$credential->getType()`.

To get the credential's values, you can either call `$credential->getValues()` to get all of them, or pass in the name of the value you wish to get, eg `$credential->getValue("username")`.

<h4 id="credential-factories">Credential Factories</h4>

Opulence makes it simple to generate JWT-based credentials for a subject.  A couple credential factories come with Opulence, and they both implement`Opulence\Authentication\Credentials\ICredentialFactory`, which defines a single method `ICredentialFactory::createCredentialForSubject($subject)`:

* `Opulence\Authentication\Credentials\AccessTokenCredentialFactory`
* `Opulence\Authentication\Credentials\RefreshTokenCredentialFactory`

<h4 id="authenticators">Authenticators</h4>

Authenticators do what their name implies - they authenticate credentials.  Not only that, but they also create a `Subject` object on success, or an error message on failure.  They all implement `Opulence\Authentication\Credentials\Authenticators\IAuthenticator`.

<h5 id="username-password-authenticator">Username/Password</h5>

To authenticate a username/password combination, pass in a user repository and a role repository:

```php
use Opulence\Authentication\Credentials\Authenticators\UsernamePasswordAuthenticator;
use Opulence\Authentication\Roles\Orm\IRoleRepository;
use Opulence\Authentication\Users\Orm\IUserRepository;

// You must write your own user repository since it is specific to your persistence layer
// Your user repository needs to implement Opulence\Authentication\Users\Orm\IUserRepository
$userRepository = new MyUserRepository();
// You must write your own role repository, too
// Your role repository needs to implement Opulence\Authentication\Roles\Orm\IRoleRepository
$roleRepository = new MyRoleRepository();
$authenticator = new UsernamePasswordAuthenticator($userRepository, $roleRepository);

// $subject will be set on success
$subject = null;
// $error will be set on failure
$error = null;

// Assume $credential was previously set
if (!$authenticator->authenticate($credential, $subject, $error)) {
    die("There was an error authenticating you: $error");
}

// $subject is now set
```

If your passwords also include a pepper, simply pass it as a 3rd parameter to the `UsernamePasswordAuthenticator` constructor.

<h5 id="jwt-authenticator">JWT Authenticator</h5>

Opulence supports authenticating [JSON web tokens](#jwt), which can be useful for authenticating JWT OAuth2 access tokens.  The `JwtAuthenticator` requires a [verifier and a verification context](#verifying-jwt).

```php
use Opulence\Authentication\Credentials\Authenticators\JwtAuthenticator;

// Assume $jwtVerifier and $verificationContext were previously set
$authenticator = new JwtAuthenticator($jwtVerifier, $verificationContext);

if (!$authenticator->authenticate($credential, $subject, $error)) {
    die("There was an error authenticating you: $error");
}

// $subject is now set
```

<h5 id="jwt-refresh-token-authenticator">JWT Refresh Token Authenticator</h5>

Refresh tokens are used to generate new access tokens when using OAuth2.  They are identical to `JwtAuthenticator`, except they also require a refresh token repository parameter.  That repository is left to you to create, and it must implement `Opulence\Authentication\Tokens\JsonWebTokens\Orm\IJwtRepository`.

<h2 id="authentication-contexts">Authentication Contexts</h2>

The `Opulence\Authentication\AuthenticationContext` is a simple wrapper that contains the current subject as well as its status, eg authenticated or unauthenticated.

```php
use Opulence\Authentication\AuthenticationContext;
use Opulence\Authentication\AuthenticationStatusTypes;

// Assume $subject was previously set
$status = AuthenticationStatusTypes::AUTHENTICATED;
$authenticationContext = new AuthenticationContext($subject, $status);
```

You can then retrieve the subject and status:

```php
$subject = $authenticationContext->getSubject();
$status = $authenticationContext->getStatus();
```

You can also update these values:

```php
$authenticationContext->setSubject($newSubject);
$authenticationContext->setStatus(AuthenticationStatusTypes::UNAUTHENTICATED);
```

<h4 id="authentication-context-middleware">Middleware</h4>

Opulence provides the `Opulence\Authentication\Framework\Http\Middleware\Authenticate` middleware to get the current subject from the HTTP request and store it along with its status in an `AuthenticationContext`.

<h2 id="jwt">JSON Web Tokens</h2>

<h4 id="jwt-introduction">Introduction</h4>

JSON web tokens (JWTs) are great ways for passing claims (such as a user's identity) between a client and the server.  They consist of three parts:
1. Header - The algorithm used to sign the token, the content type ("JWT"), and the token type ("JWT")
2. Payload - The data actually being sent in the token (also called "claims")
  * <a href="https://en.wikipedia.org/wiki/JSON_Web_Token" target="_blank">Read more about standard payload claims</a>
3. Signature - The hashed result of the header and the payload (prevents tampering with payload data)

Typically, you'll see JWTs as strings in the following format: "{base64-encoded header}.{base64-encoded payload}.{base64-encoded signature}".

<h4 id="building-jwts">Building JWTs</h4>

You can programmatically build an unsigned JWT.  You can then use a signer to [sign your JWT](#signing-jwts) and encode it as a string.

```php
use DateTimeImmutable;
use Opulence\Authentication\Tokens\JsonWebTokens\JwtHeader;
use Opulence\Authentication\Tokens\JsonWebTokens\JwtPayload;
use Opulence\Authentication\Tokens\JsonWebTokens\UnsignedJwt;
use Opulence\Authentication\Tokens\Signatures\Algorithms;
use Opulence\Authentication\Tokens\Signatures\SignerFactory;

// The algorithm can be any of the constants in Algorithms
$algorithm = Algorithms::SHA256;

// The signer will be used when we actually encode our JWT
// Keys can either be strings or resources
// Private keys are necessary 3rd parameters for Algorithms::RSA_* algorithms
$signer = (new SignerFactory)->createSigner($algorithm, 'myPublicKey');

// Create our JWT's components
$header = new JwtHeader($algorithm);
$payload = new JwtPayload();
$payload->setIssuer('http://mysite.com');
$payload->setValidTo(new DateTimeImmutable('+30 days'));
// We can set custom fields in our payload
$payload->add('username', 'dave');

// Create our unsigned JWT
$unsignedJwt = new UnsignedJwt($header, $payload);
```

<h4 id="signing-jwts">Signing JWTs</h4>

To encode your JWT, you'll first need to sign it using an `ISigner`.

```php
use Opulence\Authentication\Tokens\JsonWebTokens\SignedJwt;

$signature = $signer->sign($unsignedJwt->getUnsignedValue());
$signedJwt = SignedJwt::createFromUnsignedJwt($unsignedJwt, $signature);
$signedJwt->encode(); // Returns the encoded JWT
```

<h4 id="verifying-jwts">Verifying JWTs</h4>

Verifying a JWT is simple.  Create a `VerificationContext` and specify the fields we want to verify against.  The verifier will then compare those fields to the JWT's claims.  It will also verify that the signature is correct.

```php
use Opulence\Authentication\Tokens\JsonWebTokens\Verification\JwtVerifier;
use Opulence\Authentication\Tokens\JsonWebTokens\Verification\VerificationContext;

$context = new VerificationContext($signer);
$context->setIssuer('http://mysite.com');
$verifier = new JwtVerifier();

if (!$verifier->verify($signedJwt, $context, $errors = [])) {
    print_r($errors);
}
```

> **Note:** The not-before and expiration times are only verified if they're specified.  If your context does not set particular values, eg no issuer is set in the context, then that claim is skipped during verification.

The errors will correspond to the constants in `Opulence\Authentication\Tokens\JsonWebTokens\Verification\JwtErrorTypes`.

<h4 id="creating-jwts-from-strings">Creating JWTs from Strings</h4>

You can easily create a `SignedJwt` from a string in the format "{base64-encoded header}.{base64-encoded payload}.{base64-encoded signature}".

```php
$signedJwt = SignedJwt::createFromString($tokenString);
```

> **Note:** Tokens created in this way are not verified.  You must pass them through `JwtVerifier::verify()` to verify them.

<h4 id="jwt-ids">JWT Ids</h4>

Any time you create a new JWT payload, it's automatically assigned a unique JWT Id (also known as a JTI).  This Id is a combination of the JWT's claims and a random string.  You can grab the Id like so:

```php
$jwt->getPayload()->getId();
```

If you'd like to manually set the Id, you may do so:

```php
$jwt->getPayload()->setId('foo');
```
