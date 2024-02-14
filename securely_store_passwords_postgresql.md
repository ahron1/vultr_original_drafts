# How to Securely Store Passwords Using PostgreSQL

## Introduction

Applications that deal with user accounts need to authenticate (establish the identity of) the user before allowing (authorizing) the user to do different tasks within the application. Passwords are a time-tested and common method used to authenticate users. In typical password-based authentication, user credentials consist of a login ID (username, email, and so on) and a password. These credentials are stored in the database. For every login attempt, the credentials entered by the user are compared with those stored in the database.

When storing user credentials on the database, never (ever) store passwords as plaintext (unencrypted readable text). The opposite of plaintext is ciphertext.

1. All computer systems are hackable. In the unfortunate event that your server is compromised and an attacker gains even read-only access to the database (or to a database dump), they can get the login credentials of users, including privileged users.
1. Users often ignore security best practices, and use the same password for multiple services. Exposing the password of a user on your service can lead to the user's account getting compromised on other services.
1. If passwords are readable by the service provider, the user can blame the service provider in case something unwanted happens. Also, unscrupulous developers and sysadmins can misuse user passwords.
1. Many national and supranational regulations (e.g., GDPR) make it illegal to store passwords as plaintext.

### Scope

The scope of this article is limited to securely storing user passwords. This article does not cover general data encryption in PostgreSQL, nor does it cover encrypting database credentials (i.e., passwords used to log in to the database itself) or connections.

### Prerequisites

To benefit from this guide, it is necessary to have prior exposure to PostgreSQL. To test the examples, it is assumed you already have PostgreSQL running either on a [standalone server](https://www.vultr.com/docs/how-to-install-configure-backup-and-restore-postgresql-on-ubuntu-20-04-lts/) or [as an instance of Vultr Managed Databases for PostgreSQL](https://www.vultr.com/docs/postgresql-managed-database-guide/).

It is helpful, but not mandatory, to be familiar with the basic concepts of cryptography, like encryption and hashing.

Note that all code samples in this guide are SQL statements. Note also that text strings in SQL statements are enclosed in single quotes.

## First Principles - Password Hashing

If user passwords should not be stored as plaintext, they need to be stored in encrypted form. However, encryption algorithms have corresponding decryption algorithms that are used to recover the encrypted data. The goal in encryption is to obfuscate data temporarily and decrypt it back to the original when needed. But this is not desirable in the case of passwords. Storing an encrypted password that can easily be decrypted is of no added benefit. Hence, passwords are not encrypted but hashed.

A `hash` is a random-looking string that is generated from an input string. The algorithm that generates the hash is called a *hash function*. It is easy to compute the hash of an input string, but nearly impossible to compute the value of a string given its hash. Hashing is said to be a *one-way computation*.

Therefore, the most practical attack vector against hashed passwords is brute-forcing. Brute-forcing attempts are typically based on dictionaries. The general idea is to start with a list (dictionary) of the most commonly used passwords and sequentially compute the hash of each possible password until there is a match.

Hashing is a computationally intensive task; this makes brute-forcing difficult. In addition, password hashing functions make use of techniques like [*key stretching*](https://en.wikipedia.org/wiki/Key_stretching) to further slow down brute-forcing attempts. An example of a key stretching technique is to repeatedly hash a string (first compute the hash, then the hash of the hash, and so on).

## Hashing Passwords in PostgreSQL

### The `pgcrypto` Extension

In PostgreSQL, the `pgcrypto` extension has the necessary functions for hashing passwords. Since `pgcrypto` is a builtin extension, you do not need to download or install any additional software for it. Enable the extension:

    CREATE EXTENSION pgcrypto ;

### Salt

A hash function always generates the same hash for a given input string. This leads to two main problems:

1. If two users have the same password, their hashes are the same. It is preferable that all hashes are unique.
1. Brute-forcing is computationally expensive. So, attackers typically use [*rainbow tables*](https://en.wikipedia.org/wiki/Rainbow_table) instead of brute-forcing. These tables contain pre-computed hash values of commonly used passwords.

Both these problems are solved by computing the hash using two input values - the input string (the password) and the *salt*. The salt is a pseudo-random-generated string that helps ensure the uniqueness of hash values. Even if two users have the same password, their salt values will differ. Hence, all hashes will be unique.

#### The `gen_salt()` Function

The salt is generated using the `gen_salt()` function. Usually, this function takes one argument - `type`. The syntax is:

    -- pseudocode
    gen_salt(type) ;

The `type` argument denotes the type of cryptographic algorithm to use for hashing. It can take one of four possible values:

* `des` - for the Data Encryption Standard (DES) algorithm
* `xdes` - for the Extended DES algorithm
* `md5` - for the MD5 message-digest algorithm
* `bf` - for the Blowfish algorithm

For example, to generate a salt using the Extended DES algorithm:

    SELECT gen_salt('xdes') ;

Similarly, to generate a salt using the MD5 algorithm:

    SELECT gen_salt('md5') ;

Note that the salt also encodes information on the type of cryptographic algorithm to use for hashing, as well as (if applicable) the hashing parameters. Try to generate the same type of salt repeatedly; observe that salts of the same type follow a consistent pattern.

### Store New Password

When a user creates a new password (or changes their existing password), the database needs to store (the hash of) the password. The hash is generated using the `crypt()` function.

#### The `crypt()` Function

The `crypt()` function is based on the [Unix Crypt library](https://man7.org/linux/man-pages/man3/crypt.3.html). It generates a hash based on the following arguments:

1. the password string
1. a salt value

The syntax of the `crypt()` function is:

    -- pseudocode
    crypt(password_string, salt_string) ;

The `salt_string` argument is generated using the `gen_salt()` function. For example, to use the `md5` algorithm to compute the hash:

    -- pseudocode
    SELECT crypt(password_string, gen_salt('md5')) ;

#### Hashing without Salt

To simulate the use of the `crypt()` function without a randomized salt, use a constant string, `salt`, for the salt. Generate the hash for the password, `supersecurepassword`:

    SELECT crypt('supersecurepassword', 'salt') ;

The above command generates the ciphertext `saUkChKIZTKFs`. A non-randomized salt (or no salt) leads to predictable hashes, and makes the system vulnerable to rainbow table attacks. Hence, it is necessary to use randomized salts.

#### Password Hashing with Randomized Salt

Generate a hash for the password string `supersecurepassword` using, for example, the MD5 algorithm:

    SELECT crypt('supersecurepassword', gen_salt('md5')) ;

Copy the hash value generated by the above command. You will use it in the next section.

### Check if the Entered Password is Correct

When an existing user logs in, they enter their username and password. The server needs to check if the entered password matches the stored password. This is done by calling the same `crypt()` function with the following arguments:

1. The entered password
1. The stored hash of the actual password (instead of a generated salt)

If the entered password is the same as the actual password, the hash of the entered password matches the stored hash (of the actual password).

The general syntax is:

    --pseudocode
    crypt(entered_password, stored_hash_of_actual_password) ;

Suppose the user enters the correct password, `supersecurepassword`. To test this, call the `crypt()` function as follows:

    SELECT crypt('supersecurepassword', 'generated_hash_value_from_actual_password') ;

The second argument above is the hash generated by the previous command - paste the hash value you had copied earlier. This command outputs the same hash as was generated by the actual password.

Now, suppose the user enters the wrong password, `wrong_password`. Call the `crypt()` function as follows:

    SELECT crypt('wrong_password', 'generated_hash_value_from_actual_password') ;

The output hash value is different from the hash of the actual password.

In other words, calling the `crypt()` function with a string (`string1`) and the hash (`hash1`) of that string (`string1`) generates the same hash (`hash1`).

## Practical Usage

In actual usage, hashes are stored in and queried from tables. The examples in this section show how to do that:

Create a `user_account` table with three columns - an automatically generated ID, the user name, and the hash of the user's password:

    CREATE table user_account (user_id SERIAL, user_name VARCHAR(10), password_hash VARCHAR(100)) ;

Insert a row of test data into the table:

    INSERT INTO user_account (user_name, password_hash) 
    VALUES ('user1', crypt('user1_password', gen_salt('md5'))) ;

The above command inserts the username `user1`, and the MD5 hash of the password, `user1_password`.

To change the (hash of the) user's password, use the SQL `UPDATE` command:

    UPDATE user_account 
    SET password_hash = crypt('user1_new_password', gen_salt('md5')) 
    WHERE user_name = 'user1' ;

To match the entered password with the correct password, call the `crypt()` function with the entered password and the hash of the correct password as the salt.

If the user attempts to log in with the correct password, `user1_new_password`:

    SELECT (password_hash = crypt('user1_new_password', password_hash)) 
        AS password_match 
    FROM user_account 
    WHERE user_name = 'user1' ;

This should output `t`, for `true`, as the value of `password_match`.

If the login attempt uses a wrong password, `user1_wrong_password`:

    SELECT (password_hash = crypt('user1_wrong_password', password_hash)) 
        AS password_match 
    FROM user_account 
    WHERE user_name = 'user1' ;

This should output `f`, for `false`, as the value of `password_match`.

## Advanced Usage

When the hash is computed using either the Extended DES or the Blowfish algorithms, it is possible to customize (tune) the number of iterations that the algorithm undergoes to generate the hash. In this case, the `gen_salt()` function can accept an additional argument, `iter_count`, for the number of iterations. The syntax is:

    -- pseudocode
    gen_salt(type, iter_count)

In the above statement, `type` is either `xdes` or `bf`.

1. The iteration count for the Extended DES (`xdes`) algorithm can be an odd number between 1 and 16777215. The default value is 725.
1. The iteration count for the Blowfish (`bf`) algorithm can be an integer between 4 and 31. The default value is 6.

If the hash has been generated after *N* iterations, any brute-forcing attempt also needs to hash password guesses *N* times. The larger the value of *N*, the harder it is to brute-force the password hash. However, making the hash computation too slow is impractical for regular use. As a demonstration, hash the password `supersecurepassword` using the Blowfish algorithm with the default number of iterations, 6:

    SELECT crypt('supersecurepassword', gen_salt('bf', 6)) ;

Notice approximately how long it takes (using the clock on your computer or phone). Now run the same hash function with a higher iteration count:

    SELECT crypt('supersecurepassword', gen_salt('bf', 30)) ;

It takes much longer. If it takes too long, cancel the operation using `CTRL` + `C` and retry with a smaller number of iterations.

If you enter an invalid number of iterations, it gives an error like this:

    ERROR:  gen_salt: Incorrect number of rounds

### Tuning the Hash Function

Choosing the number of iterations is a balance between usability and security. The higher the number of iterations, the longer it takes to compute the hash. This makes it less user-friendly, but also increases security by thwarting brute-force attempts. A common recommendation is to choose the number of iterations such that standard server hardware can compute between 4 and 100 hashes per second.

## Conclusion

In reality, passwords are a suboptimal solution. Storing passwords on the server (even in hashed form) places the entire responsibility on the application developer and admins. Authentication is a complex topic and it is advisable to use dedicated service providers who specialize in it, like [OpenID](https://openid.net/connect/). Allowing users to log in with a standard set of credentials (such as their Google account) is advantageous both for users as well as application developers.

Nonetheless, password-based authentication is convenient in many cases and is widely used. Hence, due care must be taken to securely store user passwords, and safeguard both the user's security and the application's integrity.

### Caveats

1. This article only discusses how to securely store passwords in the database. This is also called *securing data at rest*. Equally important is *securing data in transit*. When transmitting passwords over the internet (for example, from a web front-end) always make sure to 1) use HTTPS and 2) send the login credentials as form data, not as URL parameters.
1. Before the password has been hashed, it is available as plaintext. This makes it vulnerable to unscrupulous employees with access to the database and/or webserver. Thus, the system described above relies on trusting developers and administrators of the application. If this trust cannot be established, consider using encryption methods at the client side. This way, plaintext passwords never show up at the server.
1. Furthermore, hashing passwords is effective only against an attacker that has gained read-only access to the database (or a database dump). An attacker that has write-access to the database will find it easier to simply overwrite the password hashes.

Ultimately, no security is truly foolproof. Greater security measures imply either more complex system design or less user-friendly systems. You need to strike a balance depending on the security needs of your particular application.
