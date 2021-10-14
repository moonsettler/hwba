# HWBA
## Hardware Wallet Based Authentication

SDS: Seed Derived Secret (never leaves HW)
Method: "HWBA/<version>/<scheme>/<method>
Account: "name@domain" (json content)
H: SHA256

dA (private key) := H(Account|SDS)
QA := dA x G (secp256k1)

dB private key of service
QB public key of service

## Register 2FA (RFC 6238)
Device reads QR {"m":"HWBA/1/TOTP/R","a":"anon@example.com","x":"678a...deb6","y":"49f6...1d5f"}
Parameters (default): "p":{"a":"SHA1","d":6,"p":30}
	Algorithm: SHA1 (default), SHA256, SHA512
	Digits: 6 (default), 8
	Period:	30 (default) seconds
 
Method := "HWBA/1/TOTP/R"
Account := "anon@example.com"
dA := H("anon@example.com"|"909a...8c4f")
QB := {x, y}
SecretA := (dA x QB == dA x dB X G).x

Device displays QR {"m":"HWBA/1/TOTP/RA","x":"fdb0...1f61","y":"bc3f...6bf1"}
QA := {x, y}
SecretB := (dB x QA == dB x dA x G).x

Site requests TOTP confirmation Code to complete setup

## Authenticate / Request 2FA Code (RFC 6238)
Device reads QR {"m":"HWBA/1/TOTP/A","a":"anon@example.com","x":"678a...deb6","y":"49f6...1d5f"}

Method := "HWBA/1/TOTP/A"
Account := "anon@example.com"
dA := H("anon@example.com"|"909a...8c4f")
QB := {x, y}
SecretA := (dA x QB == dA x dB X G).x

Code := TOTP(SecretA, SHA1, 6, 30, t)

Device displays 6 digit Code (updates every 30 seconds)
