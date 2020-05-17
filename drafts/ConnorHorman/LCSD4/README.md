# Connor Horman LCS Draft 4 - Block the Pill Web Platfom

Draft Date: 2020-05-17

## §1 Web Platform

### §1.1 Normative References

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", 
 "RECOMMENDED", "MAY", and "OPTIONAL", when used in all caps in this document, 
 are to be interpreted as described in [[RFC 2119]](https://tools.ietf.org/html/rfc2119).

This Platform uses the Hypertext Transfer Protocol, as described in [[RFC 7231]](https://tools.ietf.org/html/rfc7231) and others, for basic server-client communication.
Data transfered using the Hypertext Transfer Protocol is expected and ensured to be in the JavaScript Object Notation Data Interchange Format, as described by [[RFC 7159]](https://tools.ietf.org/html/rfc7159).

Realtime Communications for this platform are performed using WebRTC, as described by the [[W3 Consortium Technical Report]](https://www.w3.org/TR/webrtc/).

### §1.2 Authentication System

The Web Platform makes available an endpoint available by POST to `/auth` to authenticate a user account. 
This endpoint expects application/json, in the following object structure
- username = The account username or email address
- password = Sha512 hashed password to use for authentication, base64 encoded

If succesful, endpoint returns 200 OK, and application/json data, in the following structure
- blinded\_key = An AES256-CBC-PKCS5 Encrypted DER Encoded RSA Private Key, base64 encoded
- auth\_part = The salt the client MUST append to the unhashed password to produce, by means of the SHA512-256 digest, a proper key for decrypting blinded\_key.

Otherwise, endpoint returns an error status code, and application/json data, in the following structure
- code = The extended error code associated with the error
- msg = The error Message


To register a new user account, a client MUST generate 32 random bytes in a cryptographically strong manner, and base64 encode them as auth\_part. 
Additionally, it MUST generate a new RSA Private Key in a cryptographically strong manner, of strength not lesser than 1024 bits. The key SHOULD have strength not lesser than 2048 bits. The DER Encoded key must then be encrypted using the key derived from the same method as authentication as blinded\_key.
The json object must then be POSTED to the `/auth` endpoint, as follows
- username = The new account's desired username
- password = Sha512 hashed password to use for authentication, base64 encoded
- blinded\_key = The encrypted key derived as above, base64 encoded
- auth\_part = The base64 encoded random bytes, generated above
- register = This field must be set to true.

The same responses are provided. If no error is returned, then the account has been registered. 



 
