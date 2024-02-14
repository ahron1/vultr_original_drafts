# How to Encrypt Data Columns in PostgreSQL

## Introduction

Databases that contain sensitive information often need multiple layers of security to protect that information. In some cases, the organization needs to safeguard (parts of) the stored data. In other cases, securing users' personal information allows the organization to comply with regulations (e.g. HIPAA, GDPR, etc.). Encryption is one of the methods used to secure (access to) sensitive information.

Encrypting the information in the database is known as *encryption at rest*. This is different from *encryption in transit*. Encryption at rest can be done at three different layers:

1. Encrypting the entire disk of the server or parts of the filesystem used for storing data files.
1. Encrypting the [database cluster](https://www.postgresql.org/docs/current/creating-cluster.html) or individual [tablespaces](https://www.postgresql.org/docs/current/manage-ag-tablespaces.html) - this is also known as [Transparent Data Encryption (TDE)](https://wiki.postgresql.org/wiki/Transparent_Data_Encryption).
1. Encrypting the columns containing the sensitive information. This is known as *column-level encryption* and is the topic of this article.

### Scope

The scope of this article is limited to encrypting the data in specific columns. The use of both symmetric key encryption and public (asymmetric) key encryption is covered. This article does not cover TDE or disk-level encryption.

This guide also does not cover encrypting connections to PostgreSQL. Nor does it cover encrypting database credentials, i.e., passwords used to log in to the database. Both these are important components of a comprehensive security setup. [Securely storing user passwords is also a separate topic](https://www.postgresql.org/docs/current/pgcrypto.html#PGCRYPTO-CRYPT-ALGORITHMS) and not covered in this guide. Note that passwords are hashed, not encrypted.

### Prerequisites

To benefit from this guide, it is necessary to have prior exposure to PostgreSQL. To test the examples, it is assumed you already have PostgreSQL running either on a [standalone server](https://www.vultr.com/docs/how-to-install-configure-backup-and-restore-postgresql-on-ubuntu-20-04-lts/) or [as an instance of Vultr Managed Databases for PostgreSQL](https://www.vultr.com/docs/postgresql-managed-database-guide/).

It is helpful, but not mandatory, to be familiar with the basic concepts of cryptography, like encryption and decryption, [symmetric](https://en.wikipedia.org/wiki/Symmetric-key_algorithm) and [asymmetric](https://en.wikipedia.org/wiki/Public-key_cryptography) keys, and the like.

### Compatibility

Encrypting data columns using the `pgcrypto` extension works on all recent versions of PostgreSQL. The commands in the section on generating public and private keys are based on the [GNU PG (GNU Privacy Guard) tool](https://en.wikipedia.org/wiki/GNU_Privacy_Guard), `gpg`. A default installation of most common Linux distros includes `gpg`. There exist [similar tools for Mac, Windows, and other Operating Systems](https://www.gnupg.org/download/index.html), but their usage is not covered in this guide. Key generation commands in this article are tested on Ubuntu 22.04. In general, they should be compatible with all recent Linux releases.

## The `pgcrypto` Extension

Encryption in PostgreSQL is based on the [OpenPGP standard](https://www.openpgp.org/about/standard/). The `pgcrypto` extension has the necessary functions for encrypting and decrypting data according to the OpenPGP standard. Since `pgcrypto` is a builtin extension, you do not need to download or install any additional software for it. Enable the extension:

    CREATE EXTENSION pgcrypto ;

## Test Data

Consider a hypothetical example where you encrypt the clinical records of users. Create a test database with four columns - an automatically generated ID field, name, and two text fields for symmetrically and asymmetrically encrypted *clinical notes* data.

    CREATE TABLE patients (id SERIAL, name VARCHAR(50), notes_symmetric TEXT UNIQUE, notes_asymmetric TEXT UNIQUE) ;

The examples below encrypt and insert mock excerpts of clinical notes about hypothetical patients:

* *A 66-year-old female presents symptoms of pain in their lower left molars for the last 2 weeks. No prior history of dental problems. Occasional smoker.*
* *A 66-year-old male presents symptoms of pain in their lower left abdomen for the last 2 weeks. No prior history of kidney problems. Occasional drinker.*

## Using Symmetric Keys

Symmetric key encryption is a method of encrypting data that uses the same key to both encrypt and decrypt data. This is also known as single-key cryptography or private-key cryptography. Anyone who has access to the key can decrypt the data - hence, access to the key should be restricted.

    --pseudocode
    cipher = encryption_function (plaintext, key)

    plaintext = decryption_function (cipher, key)

**Note about terminology:** Symmetric key cryptography is also known as private key cryptography. Asymmetric key cryptography (discussed later) is also known as public key cryptography. Every public key has a corresponding private key. To avoid confusion, in the context of this guide, the term "private key" is used only in the context of asymmetric keys, and as the counterpart of the corresponding public key. Symmetric key cryptography is referred to only as "symmetric key cryptography", and not by the synonymous "private key cryptography". The terms "asymmetric key cryptography" and "public key cryptography" are used interchangeably.

### Symmetric Key Encryption

To encrypt data using Symmetric Key Encryption, use the encryption function `pgp_sym_encrypt()`:

    --pseudocode
    pgp_sym_encrypt('sensitive data to encrypt', 'symmetric_key')

`symmetric_key` is a string value chosen by the administrator. The encryption function is called while inserting data into the table:

    INSERT INTO patients (name, notes_symmetric) 
    VALUES (
        'Jane Doe 1',
        pgp_sym_encrypt(
            'A 66-year-old female presents symptoms of pain in their lower left molars for the last 2 weeks. No prior history of dental problems. Occasional smoker.', 
            'this_is_a_dummy_secret_key'
        )
    ) ;

Insert another row:

    INSERT INTO patients (name, notes_symmetric) 
    VALUES (
        'John Doe 1',
        pgp_sym_encrypt(
            'A 66-year-old male presents symptoms of pain in their lower left abdomen for the last 2 weeks. No prior history of kidney problems. Occasional drinker.', 
            'this_is_a_dummy_secret_key'
        )
    ) ;

Take a look at the table with the encrypted data:

    SELECT * FROM patients ;

The data in the `notes_symmetric` column appears as [ciphertext](https://en.wikipedia.org/wiki/Ciphertext).

### Symmetric Key Decryption

To decrypt, use the `pgp_sym_decrypt()` function on the encrypted data:

    --pseudocode
    pgp_sym_decrypt('encrypted sensitive data', 'symmetric_key')

The key used to encrypt the data is also used for decrypting. Decrypt the columns containing the encrypted data:

    SELECT 
        name, 
        pgp_sym_decrypt(
            notes_symmetric::BYTEA, 
            'this_is_a_dummy_secret_key'
        ) 
    FROM patients ;

To run a query conditional on the actual (unencrypted) value of the encrypted column, base the `WHERE` condition on the decrypted column value. For example, to find patients whose notes mention the word *smoke*:

    SELECT 
        name, 
        pgp_sym_decrypt(
            notes_symmetric::BYTEA, 
            'this_is_a_dummy_secret_key'
        ) 
    FROM patients
    WHERE 
        pgp_sym_decrypt(
            notes_symmetric::BYTEA, 
            'this_is_a_dummy_secret_key'
        ) ILIKE '%smoke%' ;

### The `BYTEA` Datatype

In the above examples, notice that the encrypted data is typecast into `BYTEA`. This data type is used for binary strings. Encryption produces a cipher text of binary data. Similarly, decryption works on strings of binary data. In this guide, columns containing encrypted data are of datatype `TEXT`. Therefore, before decrypting, the encrypted data (in text format) is typecast into binary.

Alternatively, you can also design the table such that the encrypted data columns are of type `BYTEA`. If the column type is `BYTEA`, the decryption functions can work on it directly without typecasting. Note, however, that if the column type is `BYTEA`, it might display differently depending on the interface. For example, a terminal program and a GUI program might display the binary value differently, depending on the configured encoding of the software. To avoid this potential confusion, this guide uses `TEXT` columns for encrypted data.

## Using Asymmetric Keys

Asymmetric key cryptography is also known as public key cryptography. It involves a pair of keys - a public key, and a private key. The data is encrypted using the public key. It is decrypted using only the corresponding private key. For ease of use, the public key is shared with a wider audience. Access to the private key is restricted only to those who need to be able to decrypt and read the data.

    --pseudocode
    cipher = encryption_function (plaintext, public_key)

    plaintext = decryption_function (cipher, private_key)

### Key Generation

To use asymmetric key encryption, you need a key pair (public and private keys). Note that PGP keys can only be generated on the Operating System, not within PostgreSQL. Generate a key pair using `gpg`:

    $ gpg --gen-key 

The above command asks for a name and an email address. After generating a key pair, you get the option of setting a passphrase. To skip this step, press `Enter` without inputting a passphrase.

If you entered a passphrase, remember it (or note it down) for future use. The passphrase is necessary for decryption, not for encryption. It is not possible to decrypt data without the passphrase - there is no "forgot password" feature.

To check all generated keys:

    $ gpg --list-keys

This lists out all generated key pairs. Each key pair is displayed as shown in the example below:

    pub   rsa3072 2023-01-23 [SC] [expires: 2025-01-22]
          7C43A1A93FB91FB86F8487B609C6A0F98287BED1
    uid           [ultimate] foobar <foo@bar.com>
    sub   rsa3072 2023-01-23 [E] [expires: 2025-01-22]

In the above example, the ID of the key is the string `7C43A1A93FB91FB86F8487B609C6A0F98287BED1`. The following examples refer to the key ID with the placeholder `KEY_ID`. The keys are stored in the `~/.gnupg` directory. After generating a new key pair, export the public and private keys into text files.

Export the public key:

    $ gpg --export KEY_ID > ~/.my_public_key.txt

Take a look at the public key file:

    $ less ~/.my_public_key.txt

Notice that the key is in binary - this makes it difficult to handle, especially on commonly used text-based interfaces. To be copied and pasted, the key needs to be expressed as ASCII. This is achieved by *armoring* the key from binary into ASCII. The `--armor` option of the `gpg` command does this:

    $ gpg --export --armor KEY_ID > ~/.my_public_key.txt

Take a look at the public key file again:

    $ less ~/.my_public_key.txt

The public key is now readable and looks like this:

    -----BEGIN PGP PUBLIC KEY BLOCK-----

    random-looking-block-of-text
    -----END PGP PUBLIC KEY BLOCK-----

Similarly, export the armored private key:

    $ gpg --export-secret-keys --armor KEY_ID > ~/.my_private_key.txt

If you set a passphrase, you are asked for it while exporting the private key.

Take a look at the (armored) private key:

    $ less ~/.my_private_key.txt

The private key looks like this:

    -----BEGIN PGP PRIVATE KEY BLOCK-----

    random-looking-block-of-text
    -----END PGP PRIVATE KEY BLOCK-----

### Asymmetric Key Encryption

To encrypt data using Asymmetric Key Encryption, use the encryption function `pgp_pub_encrypt()`:

    --pseudocode
    pgp_pub_encrypt('sensitive data to encrypt', 'public_key')

The encryption function is called while inserting data into the table. The encryption function needs the key in binary. So, the key is *dearmored* back from the ASCII text format into binary. This is done using the `dearmor()` function.

    --pseudocode
    pgp_pub_encrypt('sensitive data to encrypt', dearmor('public_key')) ;

To insert dummy rows with patient data into the table:

    INSERT INTO patients (name, notes_asymmetric) 
    VALUES (
        'Jane Doe 2',
        pgp_pub_encrypt(
            'A 66-year-old female presents symptoms of pain in their lower left molars for the last 2 weeks. No prior history of dental problems. Occasional smoker.', 
            dearmor('public_key')
        )
    ) ;

In the code above, copy the armored value of `public_key` from the exported text file, as described earlier.

Similarly, insert another row of dummy data:

    INSERT INTO patients (name, notes_asymmetric) 
    VALUES (
        'John Doe 2',
        pgp_pub_encrypt(
            'A 66-year-old male presents symptoms of pain in their lower left abdomen for the last 2 weeks. No prior history of kidney problems. Occasional drinker.', 
            dearmor('public_key')
        )
    ) ;

In the examples in the earlier section, patient data encrypted using symmetric keys was inserted into the `notes_symmetric` column. In the examples in this section, patient data is encrypted using asymmetric keys and inserted into the `notes_asymmetric` column.

#### Practical Tip

In practice, keys are part of the application code that reads/writes data from/to the database. In these examples, you manually copy and paste the keys. Pasting multi-line strings (like the public and private keys) is tricky on the command line. To try the examples, it is recommended to use a GUI-based database front-end (such as [Postico](https://eggerapps.at/postico2/) on a Mac, or [Beekeeper Studio](https://www.beekeeperstudio.io/) on Ubuntu, or any other software you are comfortable with) to access the database and enter SQL commands.

Open the exported text files containing the keys in a GUI-based text editor and copy the entire content with `CTRL+A` followed by `CTRL+C`. Do not inadvertently edit the keys. Be careful while copying keys manually - omitting a single character or introducing an extra character (such as space, tab, or newline) changes the key. Paste the entire key within single quotes in the SQL query. Avoid formatting or aligning the SQL query after pasting the key.

### Asymmetric Key Decryption

To decrypt data (which was encrypted using the public key), you need the private key. Call the `pgp_pub_decrypt()` function on the encrypted data:

    --pseudocode: using a passphrase
    pgp_pub_decrypt('encrypted sensitive data', 'private_key', 'passphrase')

    --pseudocode: without a passphrase
    pgp_pub_decrypt('encrypted sensitive data', 'private_key')

As before, the ASCII text value of the private key is also dearmored into binary before it is used. Decrypt the columns containing the encrypted data:

    --using a passphrase
    SELECT 
        name, 
        pgp_pub_decrypt(
            notes_asymmetric::BYTEA, 
            dearmor('private_key'),
            'passphrase'
        ) 
    FROM patients ;

In the above example, copy the value of `private_key` from the text file described earlier. Write the passphrase you had set earlier within single quotes. If you did not set a passphrase, omit that argument in the function call:

    --without a passphrase
    SELECT 
        name, 
        pgp_pub_decrypt(
            notes_asymmetric::BYTEA, 
            dearmor('private_key')
        ) 
    FROM patients ;

To run a query conditional on the actual (unencrypted) value of the encrypted column, base the condition on the decrypted column value. For example, to find patients whose notes mention the word *smoke*:

    SELECT 
        name, 
        pgp_pub_decrypt(
            notes_asymmetric::BYTEA, 
            dearmor('private_key'),
            'passphrase'
            ) 
    FROM patients
    WHERE pgp_pub_decrypt(
        notes_asymmetric::BYTEA, 
        dearmor('private_key'),
        'passphrase'
    ) ILIKE '%smoke%' ;

## Caveats

Encrypting data in individual columns is a fine-grained approach and works well for many use cases. However, its use comes with some caveats:

1. Encrypted data cannot be meaningfully indexed.
1. Running queries based on the decrypted values of data consumes considerably more computing resources - each encrypted value is first decrypted and then checked against the `WHERE` condition.
1. Before the data is encrypted, it is available as plaintext. This makes it vulnerable to unscrupulous employees with access to the database and/or server. Thus, the system described above relies on trusting developers and administrators of the application. If this trust cannot be established, consider using encryption methods on the client side. This way, unencrypted data never shows up on the server.

### Approaches to Avoid

Implementing column encryption is a one-time effort. Securing access to the keys is an ongoing operational challenge. A compromised database should not lead to compromised data - that defeats the purpose of encryption.

It is convenient to store the key either on the database itself or as part of a stored procedure. Do not do this. If an attacker gains access to the database (or a dump of it), they also get access to the keys. The key must be stored separately from the encrypted data.

Similarly, it is also possible to store the exported key file in the data directory of PostgreSQL. This allows the application to directly access the key file from a SQL query, for example:

    --pseudocode
    pgp_pub_decrypt('encrypted sensitive_data', dearmor(pg_read_file('/path/to/key_file'))) 

While convenient, this approach is not secure. An attacker who compromised the database server also gets access to the keys. Hence, as before, store the key separately from the data.

## Conclusion

Any security system is only as strong as its weakest link. You need to figure out how to store, protect, access, share, and if needed, recover, the key.

Choosing whether to use symmetric or asymmetric keys depends on the security goals, the infrastructure setup, and the organizational setup. In general, public keys are more secure, but they come with the burden of managing the private key. Evaluate the security implications of each approach before making a decision. In practice, keys are often part of the application code that accesses the database. Hence, access to the application code that reads sensitive data also needs to be restricted. While using public keys, WRITE access is granted to a larger audience, and READ access is restricted. With symmetric keys, anyone who has WRITE access also has READ access.

The [`pgcrypto` documentation](https://www.postgresql.org/docs/current/pgcrypto.html) also discusses a few additional functions for encrypting columns and working with binary data. In particular, understand the use of the functions `pgp_sym_encrypt_bytea()` and `pgp_sym_decrypt_bytea()`, as well as `pgp_pub_encrypt_bytea()` and `pgp_pub_decrypt_bytea()`.

In addition to column encryption, [the PostgreSQL documentation discusses a few other encryption options](https://www.postgresql.org/docs/current/encryption-options.html). It is important to not rely on a single security measure, and instead adopt multiple layers of security. Recent versions of PostgreSQL also support [*row-level security*](https://www.postgresql.org/docs/current/ddl-rowsecurity.html). Combined with column-level security, it leads to an even more granular approach to data security.

## Notes to Editor

    The last line in the Scope section has a placeholder link to the pgcrypto module's page. It should link to the earlier article on storing passwords, after that article is published.
