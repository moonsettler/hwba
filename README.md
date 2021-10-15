# HWBA
## Hardware Wallet Based Authentication

Describes proposed methods for using hardware wallets (secure, 'air-gapped' device) to manage critical secrets used for authentication.

## Motivation

The line of thought that led to the development of hardware wallets can be extended to all parts of ones digital life and security. Securely managing keys and passwords and other secrets is a daunting task, various password managers reveal the critical secrets in plain text at one point or an other. Same is true for some popular 2FA solutions where a shared secret is exposed as plain text upon registration.

## Concept

* Only reveal secrets in plain text on the display of the hardware wallet!
* Cryptograhically secure communication of critical secrets via unsecured device!
* Use the seed+passphrase as the root and single point of failure of secret management!
* Use the cryptographic primitives of the bitcoin protocol already present in the hardware wallets as much as possible!
* Do not complicate the recovery protocols for hardware wallets with any additional data they must store!
* Do not expose the HD seed directly to HWBA module SDS is derived with deterministic irreversible one way cryptographic function!
* Use secp256k1 ECDH Elliptic-curve Diffie–Hellman for key exchange!
* Provide 2FA with RFC 4226 HMAC-based One-Time Passwords!
* Provide 2FA with RFC 6238 Time-based One-Time Passwords!
* Provide crpytographically secure non-replayable authentication via secp256k1 ECDSA!
* Provide a method to decrypt and display passwords (for legacy systems)!
* Provide deterministic secure password generator functionality (for legacy systems)!

## Requirements for the hardware wallet

* Supporting primitives for bitcoin secp256k1 ECDSA, SHA256, HMAC-SHA512
* Camera capable of reading QR codes.
* Display capable of presenting QR codes.
* Internal clock (for TOTP support)

## Protocol
### ECDHA secret sharing basics
GUID: binary form of "33771967-c51f-4e84-99e8-7541fd64e14b" \
SDS: Seed Derived Secret HMAC-SHA512(GUID|Seed|Phrase) never leaves HW \
Method: "HWBA/\<version\>/\<scheme\>/\<method\> \
Account: "name@domain" (json content) \
H: SHA256 \
\
dA (private key) := H(Account|SDS) \
QA := dA x G (secp256k1) \
\
dB private key of service \
QB public key of service

### Secret sharing with internet Service
Device reads QR {"m":"HWBA/1/SSR","a":"anon@example.com","x":"678a...deb6","y":"49f6...1d5f"} \
\
Method := "HWBA/1/SSR" \
Account := "anon@example.com" \
dA := H("anon@example.com"|"909a...8c4f") \
QB := {x, y} \
SecretA := (dA x QB == dA x dB X G).x \
\
Device displays QR {"m":"HWBA/1/SSA","x":"fdb0...1f61","y":"bc3f...6bf1"} \
QA := {x, y} \
SecretB := (dB x QA == dB x dA x G).x

### Register 2FA (RFC 6238)
1. Service initiates secret sharing, displays SSR QR code
2. Device completes secret sharing, displays SSA QR code
3. Service and Device have a shared account specific secret
4. Service Requests confirmation PIN to complete setup, displays TOTP/A QR code
5. Device reads TOTP/A, generates and displays PIN
6. User enters PIN Service finalizes registration of 2FA Device

### Authenticate / Request 2FA Code (RFC 6238)
Device reads QR {"m":"HWBA/1/TOTP/A","a":"anon@example.com","x":"678a...deb6","y":"49f6...1d5f"} \
Parameters (default): "p":{"a":"SHA256","d":6,"p":30} \
&nbsp;&nbsp;&nbsp;&nbsp;Algorithm: SHA256 (default), SHA512 (for future use) \
&nbsp;&nbsp;&nbsp;&nbsp;Digits: 6 (default), 8 \
&nbsp;&nbsp;&nbsp;&nbsp;Period:	30 (default) seconds \
\
Method := "HWBA/1/TOTP/A" \
Account := "anon@example.com" \
dA := H("anon@example.com"|"909a...8c4f") \
QB := {x, y} \
SecretA := (dA x QB == dA x dB X G).x \
\
Code := TOTP(SecretA, SHA1, 6, 30, t) \
\
Device displays 6 digit Code (updates every 30 seconds)

### Generate password
Too many options here would hinder the deterministic and simplicistic nature of this function, therefore password strenght and charset will be preset, if it is not suitable the 'Reveal password' function can be used
1. User selects 'Generate password' function on the Device
2. User inputs the Account (like "anon@example.com") identifier using Device keys (alternatively read it from QR code)
3. Device generates a secret H(Iterator|Account|SDS) starting with 0 iterator 128 bit entropy (first 16 bytes) \
example: <code>874e82085aff4e3fac714a5a0dc0a42a</code>
4. Secret is displayed using base58 encoding and alphabet \
example: <code>Hi5Y2qLo9RTP85wUzJSmH3</code>
4. User may use navigation keys to iterate the password
5. Selected iteration is displayed (display may only reveal a small part of the password at once, not just for security reasons, but also to help manual input)
