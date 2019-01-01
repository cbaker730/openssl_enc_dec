# openssl_enc_dec
Simple walkthrough for encrypting and decrypting a file with openssl

### GENERATE KEYS

1 Generate keys and cert / pem

```openssl genrsa -aes256 -out privkey.pem 2048```

2 Generate public key

```openssl rsa -pubout -in privkey.pem -out pubkey.pem -outform PEM```

3 Generate certificate

```openssl req -new -x509 -key privkey.pem -out certificate.pem```




### ENCRYPT FILE

1 Normally, you would get the public key or certificate from the other party. In this exercise, we perform the encryption ourselves using the public key we created above and assume we do not have access to the private key. If a certificate you can extract the public key using this command:

```openssl x509 -pubkey -noout -in certificate.pem  > pubkeyextracted.pem```

2 Generate the random key / password file for every file (use a new key every time). The key format is HEX because the base64 format adds newlines. 

```openssl rand -hex -out key.bin 64```

3 Encrypt the large file with the random key

```openssl enc -aes-256-cbc -salt -in largefile.pdf -out largefile.pdf.enc -pass file:./bin.key -k "1"```

4 Encrypt the random key with the public key

```openssl rsautl -encrypt -inkey pubkey.pem -pubin -in key.bin -out key.bin.enc```

5 You can safely send the key.bin.enc and the largefile.pdf.enc to the other party. You may want to sign the two files with your own key.	




### DECRYPT FILE

1 Decrypt the random key using the private key file that corresponds to the public key used to encrypt the file

```openssl rsautl -decrypt -inkey privkey.pem -in key.bin.enc -out key.bin```

2 Decrypt the large file using the random key

```openssl enc -d -aes-256-cbc -in largefile.pdf.enc -out largefile.pdf -pass file:./bin.key -k "1"```
