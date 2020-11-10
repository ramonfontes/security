---
tags: security-tutorials
---

# How to Create a Public/Private Key Pair 

**In this short demo you will:** 
- Learn on how to create a public/private key pair
- and encrypt/decrypt a file


First you have to create a private key with the commands below:

```
openssl genrsa -out private_key.pem 1024
```

Since the private key has been created let's create our public key:

```
openssl rsa -in private_key.pem -out public_key.pem -outform PEM -pubout
```

Now create a file called `encrypt.txt` and put some text in it. Then, encrypt `encrypt.txt` and send the encrypted data to a new file called `encrypt.dat`

```
openssl rsautl -encrypt -inkey public_key.pem -pubin -in encrypt.txt -out encrypt.dat
```

The command below can be used to decrypt the `*.dat` file


```
openssl rsautl -decrypt -inkey private_key.pem -in encrypt.dat -out decrypt.txt
```
