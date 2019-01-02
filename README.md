# openssl_enc_dec
Simple walkthrough for encrypting and decrypting a file with openssl


---
### GENERATE KEYS
Assume the recipient is creating the keys and certificate and provides the public key and/or certificate to the sender, keeping the private key private.

1. Generate private key. Use a 4 to 1023 character passphrase.

```openssl genrsa -aes256 -out privkey.pem 2048```

2. Generate public key. You'll be prompted for the private key passphrase.

```openssl rsa -pubout -in privkey.pem -out pubkey.pem -outform PEM```

3. Generate a certificate, if desired. You'll be prompted for the private key passphrase.

```openssl req -new -x509 -key privkey.pem -out certificate.pem```



---
### ENCRYPT FILE
Assume the sender performing these operations has the recipient's public key and/or certificate as well as the file they wish to encrypt.

1. Normally, you would get the public key or certificate from the recipient. In this exercise, we perform the encryption as the sender using the public key we created above and assume we do not have access to the private key until the decryption phase. The following command extracts the public key from the certificate file if the public key was not provided to the sender.

```openssl x509 -pubkey -noout -in certificate.pem  > pubkeyextracted.pem```

2. Generate a random key / password file (use a new key every time). Use HEX key format because base64 includes newlines which may affect the reading of the key file.

```openssl rand -hex -out key.bin 64```

3. Encrypt the large file with the random key

```openssl enc -aes-256-cbc -salt -in largefile.pdf -out largefile.pdf.enc -pass file:./bin.key -k "1"```

4. Encrypt the random key with the public key

```openssl rsautl -encrypt -inkey pubkey.pem -pubin -in key.bin -out key.bin.enc```

5. You can safely send the key.bin.enc and the largefile.pdf.enc to the other party. You may want to sign the two files with your own key.	



---
### DECRYPT FILE

1. Decrypt the random key using the private key file that corresponds to the public key used to encrypt the file

```openssl rsautl -decrypt -inkey privkey.pem -in key.bin.enc -out key.bin```

2. Decrypt the large file using the random key

```openssl enc -d -aes-256-cbc -in largefile.pdf.enc -out largefile.pdf -pass file:./bin.key -k "1"```



---
### OVERALL PROCESS VISUALIZATION
![Process](https://github.com/cbaker730/openssl_enc_dec/blob/master/enc_dec.jpeg "Process")


TODO: Make a better process visualization
