---
tags: security-tutorials
---

# Digital Signature

**In this short demo you will:** 
- On how to create a digital signature.

First of all you will create a private key with the command below:

```
openssl genrsa -out private_key.pem 1024
```

Next, create a public key:

```
openssl rsa -in private_key.pem -out public_key.pem -outform PEM -pubout
```

Now, create a file and add some text in it.

```
openssl dgst -sha256 -sign private_key.pem -out /tmp/filename.sha256 test.txt
```

Then you need to encrypt the hash.

```
openssl base64 -in /tmp/filename.sha256 -out signature.sha256
```

Now, you will delete the file that has been created when you have generated the hash.

```
rm /tmp/filename.sha256
```

Finally, you can create the hash of the signature.

```
openssl base64 -d -in signature.sha256 -out /tmp/filename.sha256
```

and check whether the hash of the file and the hash of the signature have been created from the same public key.

```
openssl dgst -sha256 -verify public_key.pem -signature /tmp/filename.sha256 test.txt
```
