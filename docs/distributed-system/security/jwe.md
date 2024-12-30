# JWE

By default, JSON Web Tokens (JWTs) are base64url encoded JSON objects signed using a digital signing algorithm, but it does not provide you with confidentiality. Anyone can read the payload, which can be an issue if the token holds any sort of sensitive data. So, how do you prevent this? How do you encrypt a JWT?

This is where JSON Web Encryption (JWE) comes in. JWE allows you to encrypt a JWT payload so that only the intended recipient can read it while still providing integrity and authentication checks.

## JWE structure

A JWE token is built with five key components, each separated by a period (`.`)

![](https://i.imgur.com/xan7nPp.png)

### JOSE header

The JOSE header is the very first element of the JWE token and it stands for JSON Object Signing and Encryption. It is a base64url-encoded JSON object that includes:

- **alg**: (Required) Specifies the algorithm used to encrypt the Content Encryption Key (CEK). Common algorithms include RSA-OAEP, RSA1_5, A128KW, A256KW, etc.
- **enc**: (Required) Indicates the encryption algorithm used to encrypt the payload (plaintext). Examples include A128GCM, A256GCM, A128CBC-HS256, etc.
- **typ**: (Optional) Indicates the type of the token, typically JWT.
- **cty**: (Optional) Indicates the content type of the encrypted payload if it is something other than the default `application/json`.
- ...

Example

```json
{
  "alg": "RSA-OAEP",
  "enc": "A256GCM"
}
```
This header specifies that the content encryption key is encrypted using the RSA-OAEP algorithm and the payload is encrypted using AES GCM with a 256-bit key.

### JWE Encrypted Key

The CEK is the key used to encrypt the JWE payload. This key is encrypted using the algorithm specified in the `alg` parameter of the JOSE Header.

- If the `alg` is `RSA-OAEP`, the `CEK` is encrypted using the `RSA-OAEP` algorithm with the recipient’s public key.
- If the `alg` is `A128KW` or `A256KW`, a symmetric key wrap is used.

The CEK is different for each token, generated for one-time use. It must never be re-used.

### Initialization Vector (IV)

IV is random value that is used along with the encryption algorithm to ensure that the same plaintext will encrypt differently each time. The IV prevents patterns in the encrypted data, enhancing security.

### Ciphertext

The Ciphertext is the result of encrypting the plaintext (the payload data) with the `CEK` and the `enc` algorithm.

### Authentication Tag

The Authentication Tag (also known as the Tag) is a base64url-encoded value that provides integrity and authenticity to the Ciphertext, Initialization Vector (IV), and Additional Authenticated Data (AAD). It is generated during the encryption process using algorithms like AES GCM.

If any part of the JWE structure is altered after encryption, the decryption process will fail because the Authentication Tag will not match.

## Example of a JWE

Consider a scenario where we want to encrypt a message `Hello, World!` using JWE. Here is a simplified breakdown:

- Protected Header: `{"alg":"RSA-OAEP","enc":"A256GCM"}`
- Encrypted Key: Base64Url(encrypt(symmetric key with recipient’s public key)) using RSA-OAEP algorithm
- Initialization Vector (IV): Base64Url(randomly generated IV)
- Ciphertext: Base64Url(encrypt(“Hello, World!" with the CEK))
- Authentication Tag: Base64Url(GCM Tag)

The final JWE might look something like this:

```
eyJhbGciOiJSU0EtT0FFUCIsImVuYyI6IkEyNTZHQ00ifQ.
g_hE3pPLiSs9C60_WFQ-VP_mQ1BU00Z7Xg.
48V1_ALb6US04U3b.
5eym8mytxoXCBlYkhjBtkmmI.
XFBoMYUZodetZdvTiFvSkQ
```

## JWE decryption flow

![](https://i.imgur.com/dtNFBAI.png)
