[![Build Status](https://travis-ci.org/zbohm/lirisi.svg?branch=master)](https://travis-ci.org/zbohm/lirisi)
[![Report Card](http://goreportcard.com/badge/zbohm/lirisi)](http://goreportcard.com/report/zbohm/lirisi)
[![GoDoc](https://godoc.org/github.com/zbohm/lirisi?status.svg)](https://godoc.org/github.com/zbohm/lirisi)


# Linkable Ring Signature


### Abstract

Ring signature is an anonymous signature that both authenticates the message and protects
the identity information of the signer. The signer can generate multiple different signatures for the same message without being discovered by the verifier.
Linkable ring signature (LRS) resolves the problem of duplicity. The verifier can determine if multiple signatures were generated by the same signer, but he cannot determine the actual signer’s identity.


### Implementation

Signature is based on elliptic curve [Edwards-curve Digital Signature Algorithm (EdDSA)](https://en.wikipedia.org/wiki/EdDSA), due to high-speed and high-security.
Features and software is described on [Ed25519](https://ed25519.cr.yp.to/).
As a hash function is used [SHA-3 Keccak](https://en.wikipedia.org/wiki/SHA-3),
the latest member of the [Secure Hash Algorithm](https://en.wikipedia.org/wiki/Secure_Hash_Algorithms) family of standards, released by [NIST](https://en.wikipedia.org/wiki/National_Institute_of_Standards_and_Technology).

This library is written in language [Go](https://golang.org/).
It uses functions from the project of cryptocurrency [go-ethereum](https://github.com/ethereum/go-ethereum).


### Source taken over

The code of main functions [Sign](https://github.com/zbohm/lirisi/blob/master/ring/signature.go#L69) and [VerifySign](https://github.com/zbohm/lirisi/blob/master/ring/signature.go#L216) was taken from the project [ring-go](https://github.com/noot/ring-go). This implementation is based on "Ring Confidential Transactions" proposed for cryptocurrency Monero:
https://eprint.iacr.org/2015/1098.pdf.

### Install

First install [Go](https://golang.org/dl/) if you do not have it in the OS. Then, install the library with the command:

```
$ go get github.com/zbohm/lirisi
```


### Simple console client

This project is primarily a library, but after installation you have a simple client at your disposal. Enter the command `lirisi`:

```
Usage:
  lirisi [params] COMMAND

Commands:
  create-private-key
  extract-public-key
  create-testing-ring
  sign
  verify
  get-key-image

Examples:
  ...
```

This set of commands is sufficient to test the library. We will describe an example of use. First we will create a private key:

```
$ lirisi create-private-key > private-key.hex
```

Private key is only a number. This number is stored in the file in the printable [hexadecimal](https://en.wikipedia.org/wiki/Hexadecimal) format:

```
$ cat private-key.hex
3a99d102db5089355bb6437be27519e6152e2ae9375c58764d47a9156be21684
```

You must keep the private key secret in the real usage. But now we're just testing. After creating the private key, you can extract the public key from it:

```
$ lirisi --private=private-key.hex extract-public-key > public.b64
```

The public key, unlike the private one, must be published.
It is also a number. It is encoded to [base64](https://en.wikipedia.org/wiki/Base64):

```
$ cat public.b64
BFVxqw7II6TnRYVVfDq54qkI8LYYFdHFPBjkEpAHwn373yrmm+PWAkCTpQSWhSnA8C9gHcQ45dOWfiRCCJp/RIY=
```

Now you own private and public key. But to test a ring signature, you need the public keys of other signers. You can generate fake public keys for testing purposes. Then you attach your public key to them. For example you create nine fake keys:

```
$ lirisi --size=9 create-testing-ring > ring.lst
```

Each key is on one line, each encoded in base64:

```
$ cat ring.lst
BCPs3LdNr9TH/PVxt4HjG6Sl6CR5AEGjLr2gzPvmn+k6vjY7wV0aNMUiVo70eLYUDH2gDga/F1FmZs+icOatboI=
BKEVjH4ekVsyBDTTEgLtt2KldcRrkZhQm3cmVvQ4Kb+TAhLloPxd+ONDs6KirBxM3rEyjUXn5j79Hzdj8wLOluo=
BEKmid82WGWyRa4f06s17BkZpaWZXoUAWqT5k4H8Pb4TPSuKm4N1lMvZrdHJjnoByPOUYj0z5l8RQR8gHUqhiDg=
BH5eFm+b49MMkYQ7V3B5PZQCExaDIy5T8eDgSxxYTCS8vmOh3EOYI2CSkN+xz8FytsVzFWKvsTTof0wSNiV0PRw=
BNDu2sbpdZl15vptUs8k8EHLeLEsJw71AG3oIO/nCcGV5h7pSFqO09L5AkDXdm6lQnE1Ye7Zvpd3Jil4lNsw+GM=
BJZxm4C0yzE9jMpcaeYPddIFVz/waiNHvW+cbFNE3TnLULUv1j0iuAGhAoNQwsEAOhCsLl1XRIb66HwZfQNELn0=
BKlVi0RvpCkA+0ezOObfNBVWwCKCPPJOnnevF1U5Rg9rFUhmFjJaqL3L0Uz2t++gFKDHtBgzEYtSv77AdH4JaZk=
BElydk1Q+YeyR+DNmAfwMn8mK9rQLxoBNxTjpFrhf+5vGOof9dduGJ5TxIdq6Isg5lcjJ8G13bKgWPggA16tegs=
BLqRJf4RE38u8bO7Pmo24WYJ2lJaTBEnqmiRShYrcDvskx9dAOTS8PoCBnOjdhhSUTBbe7oxZV31r5Vph0hR4Tc=
```

Now attach your own public key to them:

```
$ cat public.b64 >> ring.lst
```

So you have a set of public keys ready to sign. Your public key is at the bottom of the list, but can be anywhere in it. We can shuffle the list:

```
$ shuf ring.lst > ring-of-public.keys
$ cat ring-of-public.keys
BCPs3LdNr9TH/PVxt4HjG6Sl6CR5AEGjLr2gzPvmn+k6vjY7wV0aNMUiVo70eLYUDH2gDga/F1FmZs+icOatboI=
BH5eFm+b49MMkYQ7V3B5PZQCExaDIy5T8eDgSxxYTCS8vmOh3EOYI2CSkN+xz8FytsVzFWKvsTTof0wSNiV0PRw=
BNDu2sbpdZl15vptUs8k8EHLeLEsJw71AG3oIO/nCcGV5h7pSFqO09L5AkDXdm6lQnE1Ye7Zvpd3Jil4lNsw+GM=
BKEVjH4ekVsyBDTTEgLtt2KldcRrkZhQm3cmVvQ4Kb+TAhLloPxd+ONDs6KirBxM3rEyjUXn5j79Hzdj8wLOluo=
BFVxqw7II6TnRYVVfDq54qkI8LYYFdHFPBjkEpAHwn373yrmm+PWAkCTpQSWhSnA8C9gHcQ45dOWfiRCCJp/RIY=
BJZxm4C0yzE9jMpcaeYPddIFVz/waiNHvW+cbFNE3TnLULUv1j0iuAGhAoNQwsEAOhCsLl1XRIb66HwZfQNELn0=
BLqRJf4RE38u8bO7Pmo24WYJ2lJaTBEnqmiRShYrcDvskx9dAOTS8PoCBnOjdhhSUTBbe7oxZV31r5Vph0hR4Tc=
BElydk1Q+YeyR+DNmAfwMn8mK9rQLxoBNxTjpFrhf+5vGOof9dduGJ5TxIdq6Isg5lcjJ8G13bKgWPggA16tegs=
BEKmid82WGWyRa4f06s17BkZpaWZXoUAWqT5k4H8Pb4TPSuKm4N1lMvZrdHJjnoByPOUYj0z5l8RQR8gHUqhiDg=
BKlVi0RvpCkA+0ezOObfNBVWwCKCPPJOnnevF1U5Rg9rFUhmFjJaqL3L0Uz2t++gFKDHtBgzEYtSv77AdH4JaZk=
```

Now you have a list of ten public keys in a random order. Caution! Once this list is used for signature, its order must not change! Otherwise, the signature cannot be verified.
Finally you make a ring signature:

```
$ lirisi --message='Hello world!' --ring=ring-of-public.keys --private=private-key.hex sign > signature.pem
```

The signature is saved in the format [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail):

```
$ cat signature.pem
-----BEGIN RING SIGNATURE-----
KeyImage: Aeih2pvFV4RR1m9eCAasWHT4IixKfQ+AIf0v+Op8sfHjc1C8jO4fpnu/ngkFnrF8zCKMzdBHwGZyBXqelDUWDw==

QklJQm9BSG9vZHFieFZlRVVkWnZYZ2dHckZoMCtDSXNTbjBQZ0NIOUwvanFmTEh4
NDNOUXZJenVINlo3djU0SkJaNnhmTXdpak0zUVI4Qm1jZ1Y2bnBRMUZnKzZTdGI4
N0U1REpxbDBWcnN5WmRXSnFrdHVQeVVLUVltTzJZdDVQOHpIZHIrZkxJaVFXejlW
MmNRYzVjSzRMKzBLVGRFMzBSNis2Y1pSQ2JHR0tMUVNCL0QrZ1BQcUhESlNjeHpv
dm5PbEE4OHh5eEthVTBxNmhScDVHR2UzazZtQWRXdWI0dnU2azJ0K2s3SlU0S1FO
YUl1V1luTU1hQWJYaG8xMzFXbGNYZ2s4eTdqY2t1Rjl6QTRpYWlpbWRzMjZpUWJN
WlpFY25mQms4ejVQV3l3YVJWRXU5NGltMDNUQjIwNEQzazN6dUprQlBtdmpyeW0x
M1lYVWZoOFdHOElYenZwQ2JKdm5SMHZyUEFId0dxNlJiNFZrV1AyOVVSek1NbUxC
MUVFZjdjKzNlNzdWMWVZQUF3eWFJRmtpenhxRDJwdS9ldXRLNHB0WEc1WkU2SFhE
WU1qVkVBOUdsYnpyQVhKbTM0RDZmU295QkZPSHNUNWcrQnpqUUgwNUZDSHpaWitL
bDRoY1pvOGwzWnBpNlRJWGxaMmwrUHRBdlZPb3Ivb016cm4wV0laUTlDMzlXK2JP
OE92bVcwMWgwZ1hIaExBL2lHdGZoTFdoSDE3bVd0aUU=
-----END RING SIGNATURE-----
```

The signature contains the so-called value "Key Image". This is the unique identifier of your private key. Each signature you create with your private key contains this identifier.
The verifier recognizes the duplicate signature based on this identifier if you create more than one. Nevertheless, it is not possible to find out from the signature which of the signers, to whom the public keys belong, who created the signature. The verifier can only verify the validity of the signature:

```
$ lirisi --message='Hello world!' --ring=ring-of-public.keys --signature=signature.pem verify
SUCCESS
```

If the verifier uses another message or a different key list, the signature is not valid:

```
$ lirisi --message='Hello folks!' --ring=ring-of-public.keys --signature=signature.pem verify
2020/05/25 18:34:22 ERROR
$ lirisi --message='Hello world!' --ring=ring.lst --signature=signature.pem verify
2020/05/25 18:34:41 ERROR
```

Exit status of `SUCCESS` is `0`. Exit status of `ERROR` is `1`. So, you can use it in the [shell scripts](https://en.wikipedia.org/wiki/Shell_script).

If the verifier need to save signature key identifier, extracrt this:

```
$ lirisi --signature=signature.pem get-key-image > private-key-id.b64
$ cat private-key-id.b64
Aeih2pvFV4RR1m9eCAasWHT4IixKfQ+AIf0v+Op8sfHjc1C8jO4fpnu/ngkFnrF8zCKMzdBHwGZyBXqelDUWDw==
```

## Library

Library is written in lang [Go](https://golang.org/). The use of the library is as follows:

```go
package main

import (
	"crypto/ecdsa"
	"log"

	"github.com/ethereum/go-ethereum/crypto"
	"github.com/zbohm/lirisi/ring"
)

// Auxiliary function for creating fake public keys.
func createPublicKeyList(size int) ring.PublicKeysList {
	ring := make(ring.PublicKeysList, size)
	for i := 0; i < size; i++ {
		privkey, err := crypto.GenerateKey()
		if err != nil {
			panic(err)
		}
		ring[i] = privkey.Public().(*ecdsa.PublicKey)
	}
	return ring
}

func main() {
	// Create your private key.
	privKey, err := crypto.GenerateKey()
	if err != nil {
		panic(err)
	}

	// Extract public key from your private.
	pubKey := privKey.Public().(*ecdsa.PublicKey)

	// Create a list of fake public keys for testing signature.
	pubKeysRing := createPublicKeyList(9)
	// Append your own public key.
	pubKeysRing = append(pubKeysRing, pubKey)

	// Define message to sign.
	message := []byte("Hello world!")

	// Create signature.
	sign, err := ring.CreateSign(message, pubKeysRing, privKey)
	if err != nil {
		panic(err)
	}

	// Verify signature.
	result := ring.VerifySign(sign, message, pubKeysRing)
	if result {
		log.Println("SUCCESS")
	} else {
		log.Fatalln("ERROR")
	}
}
```


### Library for other languages

Library [lib/lirisilib.go](https://github.com/zbohm/lirisi/blob/master/lib/lirisilib.go) is designated for calling functions from other languages. This project already contains prepared wrappers for [Python](https://www.python.org/) (>=3.5) and [Node.js](https://nodejs.org/) languages.

First you need to compile the library with a switch `-buildmode=c-shared` to get the shared object binary:

```
$ go build -o wrappers/lirisilib.so -buildmode=c-shared lib/lirisilib.go
```

#### Python (>= 3.5)

Wrapper for python is ready in subfolder `wrappers/python/lirisi/`.
Before usage, you have to make symlink to binary:

```
$ ln -s ../../lirisilib.so wrappers/python/lirisi/lirisilib.so
```

Now you can run example script `example.py`:

```
$ python wrappers/python/example.py
```

The script demonstrates how the library is used.

```python
import random

from lirisi import (CreatePrivateKey, CreateRingOfPublicKeys, CreateSignature,
                    GetPubKeyBytesSize, ExtractPublicKey, PEMtoSign, SignToPEM,
                    ToBase64, ToHex, VerifySignature)


# ---------------------------------------
# Create your private key.

privateKey = CreatePrivateKey()
print("Your private key:")
print(ToHex(privateKey).decode())
# Output:
# Your private key:
# 74aa6a01921f598a384b7c3896e9cf6264c207b1b8c045042b7ea58eef6f65b6

# ---------------------------------------
# Extract public key.

publicKey = ExtractPublicKey(privateKey)
print("\nYour public key:")
print(ToBase64(publicKey).decode())
# Output:
# Your public key:
# BLP66/E9HNhdao1p/7/Aw6V+8BLB0fiKRPL4MR/vJV+nEHdquzXpWThL+Hhpuqel/7R6HpuUEfIHoiNn4clmVB8=

# ---------------------------------------
# Create the ring of fake public keys.

pubList = []
ring = CreateRingOfPublicKeys(9)

# Append your public key.
ring += publicKey

# Randomly shuffle list of public keys.
size = GetPubKeyBytesSize()
ringPubs = []
for pos in range(int(len(ring) / size)):
    i = pos * size
    ringPubs.append(ring[i:i+size])
random.shuffle(ringPubs)

# Concat shuffled keys.
ringPubKeys = []
for chunk in ringPubs:
    ringPubKeys.extend(chunk)

# Prepare public keys for save.
pubList = []
for chunk in ringPubs:
    pubList.append(ToBase64(chunk).decode())

print("\nRing of public keys:")
print('\n'.join(pubList))
# Output:
# Ring of public keys:
# BECA9qR+T+bePvJXVVYUA9GLYWfQ799/5EwlN6DH+VygPAZ3JtHxCPWpO3VdA9DALuids39Mb3/d1JYubU1cxg8=
# BNoLhSi/zHveaI+fHSSGVjUwb6PInRIm7GO1k4hWkkXo9wHtpTwks+yzN7ZZzElTpqBUtfIMM1UtAdhlhQVUJ+4=
# BJGTB+5MlE84LrQPDPr+7zVlGlnsF26QJkYKej4A3VFq8ilx5ZV1Gy4ZEM/F6Tn5LrzJ5Lw2I51eXOWGPu2AHbI=
# BKjS6IXUBJ2wIDKxxhkKXfhBCUSCQB37wHjZK0WXQUHTnZCxCQqtggpRAlPkfXVjFBMTJZkTniTFDI/293A3yBk=
# BB5dsoqdrTdqANzp7MRSZrhbLXF3V5AcIzfmy3/HaKSmGVzdpDw3dUUTWjz5z2ZKI/pWAZV0KJBveMV757Uxa7Q=
# BGBtFDAteGv8atveX+0pn86cQXyCakn2pXlbszUup51wwrTH57DbzjlYfaowH4lk6++TnAaLpJNCDKI4SH67A/g=
# BLP66/E9HNhdao1p/7/Aw6V+8BLB0fiKRPL4MR/vJV+nEHdquzXpWThL+Hhpuqel/7R6HpuUEfIHoiNn4clmVB8=
# BIplYHkY1EqeTCmsoieGiNVc2wBTVPcQSmb/tGVRg4fjy0ZClWPzdaXW2N2wHI4vVj453RpdG2+QzZmjTuaaDNk=
# BIBjSmlHDqmfLuI9AMw6+yUg+ms0ctRc0u35tadiDzIT0jKDotTLXm2JTdpiKDzoJJDY0Zw/z3D6Jgx/UVCMgwE=
# BPOnZeQoVH/nvDsL4I/NXUtAbabiGbh6k3atTpMyxgmXQWTNc4TQKiFGvbh3E0JzQqzjHV3n+UxGbr7oTio3HFs=


# ---------------------------------------
# Prepare message to sign.
# It is a list of bytes - bytearray.
# message = [ord(c) for c in "Hello world!"]
message = "Hello world!".encode()

# ---------------------------------------
# Make signature.
sign = CreateSignature(message, ringPubKeys, privateKey)
pemBytes = SignToPEM(sign)
print("\nSignature in PEM:")
print(bytearray(pemBytes).decode())
# Output:
# Signature in PEM:
# -----BEGIN RING SIGNATURE-----
# KeyImage: n3eMbqe1K+ngj0eKJ2+1so3uwwEIaie7HPCUKLc0/jutSM14cdqpRTZqvrgLw1Mfh8J0ylvqJI3DI52G8SmkpQ==

# QklJQm9KOTNqRzZudFN2cDRJOUhpaWR2dGJLTjdzTUJDR29udXh6d2xDaTNOUDQ3
# clVqTmVISGFxVVUyYXI2NEM4TlRINGZDZE1wYjZpU053eU9kaHZFcHBLV0xrTVR5
# RXIzRkhiNVVkbnkxM3Y4NkEyZE5LU1hna282RDhMYXNxTExNRHBOaG1kZTdzT2Rs
# VHo1M1E3eXNhMUFZeUc4SENQUmsreWR0YldYSmRINFdCMjlMdEhpazc2My90WjJW
# Y0NZVUgvMmh2NnZFL0tlSzhOdHIybFZnMkcrOTExWHdLd3ZSZUFNdkppOEo0dGJB
# V1l2NXhYallOZVZuN2lZc24zU1ZMYU5RNmFZcHppcjRDSkpXUFNHa3dIMkpXeHp1
# Q3NPTlNlclo5R0JqdkdNdlZZYTNXSnNhdHpESWR4QjlKQ0tQUHlzSFJTWkZUam9a
# djBia1JldlBGSGwzbExuVUZ3akRVMGlwTGVBTm1RVnNwR2svaW5ZdUJyTFpFQVQr
# aUZoTkRvVlFKVDF3cElKWER6Q0hSWFphUElRaDgyL2hGZGZFb0tRdmg1Q01YWG90
# TEdNUE5JVnU2M0RjMTNlM0Q1aU5SKzZ2Kzd6Wm8yVUdwNTNSdStlbFRvdGdKaEVh
# OGFKTWlQdVpOT3hnMithejNRb1k2UUtNMFZDZC8rNU9GTUt3aFBab1FXbHJ1aVFI
# RCtxTU1ad2VuWUx0eW9JS1RNRjVsMVorQjNFTTl3bkg=
# -----END RING SIGNATURE-----

# ---------------------------------------
# Load signature bytes from PEM.
signFromPEM = PEMtoSign(pemBytes)

# Get KeyImage

# ---------------------------------------
# Verify signature.
result = VerifySignature(message, ringPubKeys, signFromPEM)
print("Result of verification (true):", result)
# Output:
# Result of verification (true): True

result = VerifySignature([ord(c) for c in "Hello fokls!"], ringPubKeys, signFromPEM)
print("Invalid verification (false):", result)
# Output:
# Invalid verification (false): False

# ---------------------------------------
# Get KeyImage - unique private key identifier.
keyImage = GetKeyImage(sign)
print("\nYour private key image:")
print(ToBase64(keyImage).decode())
# Output:
# Your private key image:
# pOcgOuwHYPTT8Ofi/+rqlVM0jxntrlWh1aUqmssQptE+5a7pyA3kOs//gPySO2aVXR45VZVnxqI+aWXbhijZfg==
```

#### Node.js

Wrapper for [Node.js](https://nodejs.org/) is ready in subfolder `wrappers/nodejs/lirisi/`.
Before usage, you have to make symlink to binary:

```
$ ln -s ../../lirisilib.so wrappers/nodejs/lirisi/lirisilib.so
```

Go to folder `nodejs`:

```
$ cd wrappers/nodejs
```

Before run example install necessary `node` packages:

```
$ npm install
```

Now, you can run example:

```
$ node example.js
```

Example of usage:

```javascript
const shuffle = require('shuffle-array')
const convertHex = require('convert-hex')
const lirisi = require('lirisi')

// Create your provate key.
const privateKey = lirisi.CreatePrivateKey()
console.log("Your private key:")
console.log(convertHex.bytesToHex(privateKey))
/*
Your private key:
2679fd46ca96602c21affaa48b3d4e13b902bb9494751c0a87271d2373e1364a
*/

// Extract public key.
const publicKey = lirisi.ExtractPublicKey(privateKey)
console.log("\nYour public key:")
console.log(Buffer.from(publicKey).toString("base64"))
/*
Your public key:
BPiG5WjNzwPARnp7oOIbvUl0HPANWIrUsC898xuwYuzlqlBPssGnpm9BUQmgrRm0aAvvOBTYJ4em6Wz76awzLyg=
*/

// Create the ring of fake public keys.
const ring = lirisi.CreateRingOfPublicKeys(9)

// Append your public key.
const ringBytes = ring.concat(publicKey)

// Shuffle ring randomly
const size = lirisi.GetPubKeyBytesSize()
let pubsRing = []
for (let i = 0; i < ringBytes.length / size; i++) {
    const n = i * size
    pubsRing.push(ringBytes.slice(n, n + size))
}
shuffle(pubsRing)
// Concat shuffled keys into one array.
let ringPubs = []
for (let i = 0; i < pubsRing.length; i++) {
    ringPubs = ringPubs.concat(pubsRing[i])
}

// Serialize public keys for save.
let b64Pubs = []
for (let i = 0; i < pubsRing.length; i++) {
    b64Pubs.push(Buffer.from(pubsRing[i]).toString("base64"))
}
console.log("\nRing of public keys:")
console.log(b64Pubs.join("\n"))
/*
Ring of public keys:
BAB0waZBNlIT8n5OgATUJSGeJX6lQxBK+S/WfVsA1UWMDO7yNsbKkdCvFuJYFImfyD0pOv4SWOhlrbkIVslIO+M=
BLp801LIVSIcOMVoEpfb1PfP19JwI+6wfl4Gy/fppwjEy5ISoShL0OyBSL3QxTDKSPx6dVug3fJPU183uYDjLF4=
BDGMfeZKpud/RznARmLdDWo00NoNF7Cxqo7q2LNBHNJxP2YkQiVM0Fb/iXxtKIwapxOAB1FUkk7ElIO52mTg69E=
BPypBiipURq3YXeqkAsDpRQC9+i0mE9iqSzeG86MJ45Z3emIfIiSA119ZnTqemiUJRqg/ei/is9SBJRvpHSFEwA=
BGp6cefHI9nhIHlce4aF98h/4yaq3zQYCwQp+Ff91FNJdD9fbH/JVzJfLZnJ3xV6sXWRy/bmqWFK8giV1bMi/jM=
BC/c7mXhJrP9mrwvcBmk78innrYw5WrhuNV/+Vam+Lglx7VE33xnVlwNzqcHvPuzxjRnp4YQT6SJWnX2PVn7kcU=
BFWqAoa+hoY2bfPZE/TSDBIhIUA+rZYNc8rlnmta1oaANcPdRNXv2QWq3rmBn/RwvYHsHfpXO21qLIUu7mzPLCg=
BJhZiBf2XiIUIP88TcBP0EwYeEuGW0ek4KtT/MIIlTkpm9SomdRWZPLvHnPy2Yvm3wpMo+jucN7CPMRjhuYUIm4=
BPiG5WjNzwPARnp7oOIbvUl0HPANWIrUsC898xuwYuzlqlBPssGnpm9BUQmgrRm0aAvvOBTYJ4em6Wz76awzLyg=
BJd6+bzj43l1dZmkg1ygYSkBRysHOCW+Dx1XH21gADS8bC1bO9qgTcwjE9FyDr7BhJ5AzCZgkce+9nfb6wMK6so=
*/

// Prepare message to sign.
const message = "Hello world!"

// Make signature.
const sign = lirisi.CreateSignature(message, ringPubs, privateKey)
const pem = lirisi.SignToPEM(sign)
console.log("\nSignature in PEM:")
console.log(Buffer.from(pem).toString())
/*
Signature in PEM:
-----BEGIN RING SIGNATURE-----
KeyImage: Xs6oCbGa1pt0wqVjXZQHLqiLXNNJKa/1VNWeEf6oSSFnzGmOWQ6xhQZirle0aMYP77brws5a71CxUhxHzEeFCA==

QklJQm9GN09xQW14bXRhYmRNS2xZMTJVQnk2b2kxelRTU212OVZUVm5oSCtxRWto
Wjh4cGpsa09zWVVHWXE1WHRHakdEKysyNjhMT1d1OVFzVkljUjh4SGhRZ1U5aG5T
N25iTmp0TlNwenAvTk9UbTlFcmliMVlsNHNvK2VJWlU2VXhDanhkclc2c1hWMVp1
aFp4dXpSMDFjQWp1UHY0NC96cm0yeUNXaWJVSXlSVExWd2dONnZHWVdhWGpqNzdh
aGJtNm1OcVE2aTFVMS9BTXRmaGk5N2ZrUzVvYTAvL012UllVWHdwWGxvMTh3RXgw
Y0hFMytCMGh0VS9lOXFxcEtsNWFpVW1xRmp3Y0J5ZUtldXh1UGN3MmQ4bWYxM2p4
TEpmekVNaGp6MTFEWXNaZzNVTWh6c2pUVFRMd015czI3MXM2bFhyM3lIQlAycXl2
L0dVdXliclE2S3FvOVBOdHVVYkxDbW5wdEl5MkNTRTlhbkJjSytjQkdqWkpQYTBn
UDJ1Ym5Kd3diZDRMWnhEcEpzazRlR3RjL2RlNHpiTWdBT3Y1WHlOTkNWQTRvdjND
cWM0MnM3eVNXM3lQellaZG8rSFN0NWFIV0tCTlRzWmJFVlM2K1oyNmdzM3RKVTQ4
WVJDSzg5WDdVdXEzeFFZV2lGeW5weWxvc0Q5Ynh0a0FMOWFWLzVRQkwzYmVHY0xJ
UHBxSGtMSzB5UUFCcFpEbHVlSWFYUzlhQnltOVpkZkU=
-----END RING SIGNATURE-----
*/

const signFromPEM = lirisi.PEMtoSign(pem)

// Verify signature.
const result = lirisi.VerifySignature(message, ringPubs, signFromPEM)
console.log("Result of verification (1):", result)
/*
Result of verification (1): 1
*/

const failed = lirisi.VerifySignature("foo", ringPubs, signFromPEM)
console.log("Invalid verification (0):", failed)
/*
Invalid verification (0): 0
*/

// Get KeyImage - unique private key identifier.
const keyImage = lirisi.GetKeyImage(signFromPEM)
console.log("\nKeyImage:", Buffer.from(keyImage).toString("base64"))
/*
KeyImage: rVCx0N999oGof35UnuNC35RcYfTpEUD7ORupIQDV+yLrpC7CbDMGPPPRzK6HpnjS/apWP5Grb9qWsOuevW1ixw==
*/
```

### Project status

This project is still under development. For real usage wait for version v1.0.0.

### License

See [LICENSE](/LICENSE).
