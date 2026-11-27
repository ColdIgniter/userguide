# Encryption Service

!!! warning
    DO NOT use this or any other *encryption* library for password storage! Passwords must be *hashed* instead.

The Encryption Service provides two-way symmetric (secret key) data encryption. The service will instantiate and/or initialize an encryption **handler** to suit your parameters as explained below.

Encryption Service handlers extend `ColdIgniter\Encryption\EncrypterInterface`, which is a plain component whose methods throw, because CFML on ColdFusion MX 6.1 has no interfaces.

The ColdIgniter handler uses the Java cryptography classes (`javax.crypto`) of the server's JVM, not OpenSSL. The handler is still called the OpenSSL handler, and the driver name is `OpenSSL`.

!!! warning "Not ported yet"
    The Sodium driver is not ported, because the server has no libsodium. Asking for it throws an `EncryptionException` that says no handler is available. Padding with `blockSize` belongs to the Sodium handler, so it has no effect. See [Padding](#padding).

This is not a full cryptographic solution. If you need more capabilities, for example, public-key encryption, use a library built for that.

## Using the Encryption Library

Like all services in ColdIgniter, it can be loaded with the global function `service()`:

```cfm
<cfset encrypter = service("encrypter")>
```

Assuming you have set your starting key (see [Configuring the Library](#configuring-the-library)), encrypting and decrypting data is simple - pass the appropriate string to `encrypt_()` and/or `decrypt_()` methods:

```cfm
<cfset plainText = "This is a plain-text message!">
<cfset ciphertext = encrypter.encrypt_(plainText)>

<!--- Outputs: This is a plain-text message! --->
<cfoutput>#encrypter.decrypt_(ciphertext)#</cfoutput>
```

!!! note "Difference from CodeIgniter"
    The method names end with an underscore, because `encrypt` and `decrypt` are built-in functions in CFML: `encrypt_()` and `decrypt_()`. The result of `encrypt_()` is a byte array, not a string, unless `rawData` is `false` (see below). The result of `decrypt_()` is text (UTF-8).

And that's it! The Encryption library will do everything necessary for the whole process to be cryptographically secure out-of-the-box. You don't need to worry about it.

### Configuring the Library

The example above uses the configuration settings found in **app/Config/Encryption.cfc**.

| Option | Possible values (default in parentheses) |
|---|---|
| `key` | Encryption key starter |
| `previousKeys` | Old keys, for [key rotation](#encryption-key-rotation) |
| `driver` | Preferred handler (`OpenSSL`). `Sodium` is not ported |
| `digest` | Message digest algorithm (`SHA512`) |
| `blockSize` | [**SodiumHandler** only] Padding length in bytes (`16`). Not ported |
| `cipher` | [**OpenSSLHandler** only] Cipher to use (`AES-128-CTR`) |
| `encryptKeyInfo` | [**OpenSSLHandler** only] Encryption key info (`""`) |
| `authKeyInfo` | [**OpenSSLHandler** only] Authentication key info (`""`) |
| `rawData` | [**OpenSSLHandler** only] Whether the cipher-text should be raw (`true`) |

In **app/Config/Encryption.cfc** a boolean is the string `"true"` or `"false"`. The keys can also be set in the **.env** file, for example `encryption.cipher`.

!!! note "Difference from CodeIgniter"
    The default `cipher` is `AES-128-CTR`, not `AES-256-CTR`. The JVM of ColdFusion MX 6.1 (Java 1.4.2) has no unlimited-strength policy, so it cannot use a key longer than 128 bits. Asking for `AES-192` or `AES-256` throws an `EncryptionException` that says the policy is needed, unless the JVM allows it. To use AES-256, install the Java unlimited-strength policy files on every server, then set `cipher` to `AES-256-CTR` and a key of 32 bytes. Behind a load balancer, every server needs the same key, cipher and policy, or data written on one server cannot be read on another.

    The ciphers are `AES-128`, `AES-192` and `AES-256`, each in the modes `CTR`, `CBC` and `ECB`. Any other cipher throws an `EncryptionException`. The layout of the data is the one of CodeIgniter, so the same key, cipher and digest give data that CodeIgniter can read.

You can replace the config file's settings by passing a configuration object of your own to the `service()` call. The `config` variable must be an instance of the `Config\Encryption` component.

```cfm
<cfset config = config("Config\Encryption")>
<cfset config.key = "aBigsecret_ofAtleast16Characters">
<cfset config.driver = "OpenSSL">

<cfset encrypter = service("encrypter", config)>
```

The key is only read from the config file and the **.env** file for the request. It is never kept in the server or application scope, a cache or an exception message.

#### Configuration to Maintain Compatibility with CI3

You can decrypt data encrypted with CI3's Encryption. If you need to decrypt such data, use the following settings to maintain compatibility.

```cfm
<cfset config = new_("Config\Encryption")>
<cfset config.driver = "OpenSSL">

<!--- Your CI3's 'encryption_key' --->
<cfset config.key = hex2bin("64c70b0b8d45b80b9eba60b8b3c8a34d0193223d20fea46f8644b848bf7ce67f")>
<!--- Your CI3's 'cipher' and 'mode' --->
<cfset config.cipher = "AES-128-CBC">

<cfset config.rawData = false>
<cfset config.encryptKeyInfo = "encryption">
<cfset config.authKeyInfo = "authentication">

<cfset encrypter = service("encrypter", config)>
```

#### Supported HMAC Authentication Algorithms

For HMAC message authentication, the Encryption library supports usage of the SHA-2 family of algorithms:

| Algorithm | Raw length (bytes) | Hex-encoded length (bytes) |
|---|---|---|
| SHA512 | 64 | 128 |
| SHA384 | 48 | 96 |
| SHA256 | 32 | 64 |

!!! note "Difference from CodeIgniter"
    SHA224 is not supported, because the Java of the server does not have it.

The reason for not including other popular algorithms, such as MD5 or SHA1 is that they are no longer considered secure enough and as such, we don't want to encourage their usage.

## Default Behavior

By default, the Encryption Library uses the OpenSSL handler. That handler encrypts using the AES-128-CTR algorithm, your configured *key*, and SHA512 HMAC authentication.

## Setting Your Encryption Key

Your encryption key **must** be as long as the encryption algorithm in use allows. For AES-128, that's 128 bits or 16 bytes (characters) long. For AES-256, it is 32 bytes.

The key should be as random as possible, and it **must not** be a regular text string, nor the output of a hashing function, etc. To create a proper key, you can use the Encryption library's `createKey()` method.

```cfm
<cfset encryption = new_("ColdIgniter\Encryption\Encryption")>

<!--- key will be assigned a 16-byte (128-bit) random key, as a byte array --->
<cfset key = encryption.createKey(16)>
```

`createKey()` has a default length of 32 bytes. A key longer than 16 bytes only works with a cipher of the matching strength, see above.

The key can be stored in **app/Config/Encryption.cfc**, or you can design a storage mechanism of your own and pass the key dynamically when encrypting/decrypting.

To save your key to your **app/Config/Encryption.cfc**, open the file and set:

```cfm
<cfset this.key = "YOUR KEY">
```

!!! note "Difference from CodeIgniter"
    `createKey()` is a normal method of an `Encryption` instance here, not a static one. A key is text, or a byte array. The key you get from `createKey()` is a byte array. Binary data in CFML on ColdFusion MX 6.1 is a byte array, because a string cannot hold a NUL character.

### Encoding Keys or Results

You'll notice that the `createKey()` method outputs binary data, which is hard to deal with (i.e., a copy-paste may damage it), so you may use `bin2hex()`, or `ToBase64()` to work with the key in a more friendly manner. For example:

```cfm
<!--- Get a hex-encoded representation of the key: --->
<cfset encoded = bin2hex(encryption.createKey(16))>

<!--- Put the same value with hex2bin(), so that it is still passed as binary to the library: --->
<cfset key = hex2bin("your-hex-encoded-key")>
```

You might find the same technique useful for the results of encryption:

```cfm
<!--- Encrypt some text and make the results text --->
<cfset encoded = ToBase64(encrypter.encrypt_(plaintext))>
```

### Using Prefixes in Storing Keys

You may take advantage of two special prefixes in storing your encryption keys: `hex2bin:` and `base64:`. When these prefixes immediately precede the value of your key, `Encryption` will intelligently parse the key and still pass a binary string (a byte array) to the library.

```cfm
<!--- In Encryption.cfc, you may use --->
<cfset this.key = "hex2bin:<your-hex-encoded-key>">
<!--- or --->
<cfset this.key = "base64:<your-base64-encoded-key>">
```

Similarly, you can use these prefixes in your **.env** file, too!

```
# For hex2bin
encryption.key = hex2bin:<your-hex-encoded-key>

# or
encryption.key = base64:<your-base64-encoded-key>
```

A `base64:` key that is not valid base64 throws an exception.

## Encryption Key Rotation

When you need to rotate your encryption key (for security best practices or compliance requirements), you can use the `previousKeys` configuration option to maintain the ability to decrypt data encrypted with old keys while using a new key for all new encryption operations.

### How It Works

- **Encryption** always uses the current `key` value
- **Decryption** tries the current `key` first
- If decryption fails, it automatically falls back to trying each key in `previousKeys`
- This allows seamless key rotation without data loss

### Configuration

Add your old keys to the `previousKeys` property in **app/Config/Encryption.cfc**:

```cfm
<cfset this.key = "hex2bin:your_new_encryption_key_in_hex">

<cfset this.previousKeys = ArrayNew(1)>
<cfset ArrayAppend(this.previousKeys, "hex2bin:your_old_encryption_key_in_hex")>
<cfset ArrayAppend(this.previousKeys, "hex2bin:another_old_key_if_needed")>
```

When keys are set, the encrypter you get is a `KeyRotationDecorator` that wraps the handler. It has the same `encrypt_()` and `decrypt_()` methods.

### Using .env File

You can also configure previous keys in your **.env** file (recommended) using a comma-separated list:

```
encryption.key = hex2bin:your_new_key
encryption.previousKeys = hex2bin:old_key_1,hex2bin:old_key_2
```

The framework automatically parses the comma-separated string into an array and processes each key.

### Key Rotation Workflow

1. **Before rotation**: Data is encrypted and decrypted with `key`
2. **Start rotation**: Move current `key` value to `previousKeys` array, set new value for `key`
3. **During rotation**: New data encrypted with new `key`, old data still decryptable via `previousKeys`
4. **Re-encrypt data** (optional): Decrypt and re-encrypt existing data with the new key
5. **Complete rotation**: Once all data is re-encrypted, remove old keys from `previousKeys`

!!! warning
    The `previousKeys` feature is for **decryption fallback only**. All new encryption operations always use the current `key`. If you pass an explicit key via the `params` argument to `encrypt_()` or `decrypt_()`, the previousKeys fallback will not be used.

## Padding

Sometimes, the length of a message may provide a lot of information about its nature. If a message is one of "yes", "no" and "maybe", encrypting the message doesn't help: knowing the length is enough to know what the message is.

Padding is a technique to mitigate this, by making the length a multiple of a given block size.

!!! warning "Not ported yet"
    Padding is implemented in CodeIgniter by the `SodiumHandler` with the functions `sodium_pad` and `sodium_unpad`. The Sodium handler is not ported, so the `blockSize` setting has no effect. The `CBC` and `ECB` modes of the OpenSSL handler use the standard block padding of the cipher, but that does not hide the length.

!!! warning
    Passwords should not be padded. Usage of padding in order to hide the length of a password is not recommended. A client willing to send a password to a server should hash it instead (even with a single iteration of the hash function).

## Encryption Handler Notes

### OpenSSL Notes

CodeIgniter's OpenSSL handler uses OpenSSL. ColdIgniter's handler uses the AES ciphers of the Java `javax.crypto.Cipher` class. The default is the AES-128-CTR cipher.

The *key* your configuration provides is used to derive two other keys, one for encryption and one for authentication. This is achieved by way of a technique known as an [HMAC-based Key Derivation Function](https://en.wikipedia.org/wiki/HKDF) (HKDF).

The HMAC is compared byte by byte over its whole length, so the time does not tell how much is right. A message that fails the authentication throws an `EncryptionException`, and so does an unsupported cipher when decrypting.

### Sodium Notes

!!! warning "Not ported yet"
    The Sodium handler is not ported.

## Message Length

An encrypted string is usually longer than the original, plain-text string (depending on the cipher).

This is influenced by the cipher algorithm itself, the initialization vector (IV) prepended to the cipher-text, and the HMAC authentication message that is also prepended. If `rawData` is `false`, the encrypted message is also Base64-encoded so that it is safe for storage and transmission regardless of the character-set in use.

Keep this information in mind when selecting your data storage mechanism. Cookies, for example, can only hold 4K of information.

## Using the Encryption Service Directly

Instead of (or in addition to) using `service()` as described in [Using the Encryption Library](#using-the-encryption-library), you can create an "Encrypter" directly, or change the settings of an existing instance.

```cfm
<!--- create an Encryption instance --->
<cfset encryption = new_("ColdIgniter\Encryption\Encryption")>

<!--- reconfigure an instance with different settings --->
<cfset encrypter = encryption.initialize_(config)>
```

Remember, that `config` must be an instance of the `Config\Encryption` component.

## Class Reference

### Encryption

The class is `ColdIgniter\Encryption\Encryption`.

#### createKey()

`createKey(length)`

- `length` (numeric, optional, default `32`): output length in bytes.
- Returns a pseudo-random cryptographic key with the specified length, as a byte array.

Creates a cryptographic key by fetching random data from the secure random source of the server. The length must be greater than zero.

#### initialize_()

`initialize_(config)`

- `config` (`Config\Encryption`, optional): configuration parameters.
- Returns the encrypter (an `EncrypterInterface` instance).
- Throws `ColdIgniter.Encryption.Exceptions.EncryptionException`.

Initializes (configures) the library to use different settings. It throws if no driver is set, if the driver is unknown or not ported, and if the key is empty or `0`.

!!! note "Difference from CodeIgniter"
    The name ends with an underscore, like other methods whose plain name could clash with a built-in. The argument is a `Config\Encryption` component. An array of settings, like `['cipher' => 'AES-256-CTR']` in CodeIgniter, is not accepted: set the property on the config object and pass that.

Please refer to the [Configuring the Library](#configuring-the-library) section for detailed info.

### EncrypterInterface

#### encrypt_()

`encrypt_(data, params)`

- `data` (string or byte array): data to encrypt.
- `params` (string, byte array or struct, optional): configuration parameters (key).
- Returns the encrypted data (a byte array, or base64 text if `rawData` is `false`).
- Throws `ColdIgniter.Encryption.Exceptions.EncryptionException`.

Encrypts the input data and returns its ciphertext.

If you pass parameters as the second argument, the `key` element will be used as the starting key for this operation if `params` is a struct; or the starting key may be passed as a string or a byte array.

Examples:

```cfm
<cfset ciphertext = encrypter.encrypt_("My secret message")>
<cfset ciphertext = encrypter.encrypt_("My secret message", "New secret key")>

<cfset params = StructNew()>
<cfset params["key"] = "New secret key">
<cfset ciphertext = encrypter.encrypt_("My secret message", params)>
```

#### decrypt_()

`decrypt_(data, params)`

- `data` (string or byte array): data to decrypt.
- `params` (string, byte array or struct, optional): configuration parameters (key).
- Returns the decrypted data, as text.
- Throws `ColdIgniter.Encryption.Exceptions.EncryptionException`.

Decrypts the input data and returns it in plain-text.

If you pass parameters as the second argument, the `key` element will be used as the starting key for this operation if `params` is a struct; or the starting key may be passed as a string or a byte array.

Examples:

```cfm
<cfoutput>#encrypter.decrypt_(ciphertext)#</cfoutput>
<cfoutput>#encrypter.decrypt_(ciphertext, "New secret key")#</cfoutput>

<cfset params = StructNew()>
<cfset params["key"] = "New secret key">
<cfoutput>#encrypter.decrypt_(ciphertext, params)#</cfoutput>
```

!!! warning "Not ported yet"
    The `blockSize` key of `params` belongs to the Sodium handler and is not used.
