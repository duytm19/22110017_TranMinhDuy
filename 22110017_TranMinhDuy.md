# Lab #2, 22110017, Tran Minh Duy, INSE330380E_02FIE
# Task 1: Public-key based authentication 
**Question 1**: 
Implement public-key based authentication step-by-step with openssl according the following scheme.
![image-1-1](https://github.com/user-attachments/assets/2ef43c3a-e259-4f13-aa11-37a82db1d36a)


**Answer 1**:

**Step1: Create RSA Key Pair** 
- On the client, create private key and public key:

```
openssl genpkey -algorithm RSA -out client_private_key.pem
openssl rsa -pubout -in client_private_key.pem -out client_public_key.pem
```
![image-2](https://github.com/user-attachments/assets/b2ca56e0-20a7-43b8-905b-3d7d6fc0f704)
- On the server, create private key and public key :

```
openssl genpkey -algorithm RSA -out server_private_key.pem
openssl rsa -pubout -in server_private_key.pem -out server_public_key.pem
```
![image-3](https://github.com/user-attachments/assets/fbf31572-de7b-430e-9a23-02c4e06051c8)
**Step 2: Send Message from Server to Client**

- On the Server, create message with the content: "This is a challenge message":
```
echo "This is a challenge message" > challenge.txt
```
![image-4](https://github.com/user-attachments/assets/7cafd170-7e16-4e03-ba92-e788d25de47a)
- Encrypt the message with the public key of the client:
```
openssl pkeyutl -encrypt -inkey client_public_key.pem -pubin -in challenge.txt -out encrypted_challenge.bin
```
![image-5](https://github.com/user-attachments/assets/73207212-b701-4a96-9dea-65ea58b3ce43)
- Show encrypted_challenge.bin
![image](https://github.com/user-attachments/assets/9eb92059-cd30-4117-a1a9-a81e4bbc1157)


**Step 3: Client Decrypts the Challenge Message**
On the Client:

- Decrypt the encrypted_challenge.bin file:
```
openssl pkeyutl -decrypt -inkey client_private_key.pem -in encrypted_challenge.bin -out decrypted_challenge.txt
```
![image-6](https://github.com/user-attachments/assets/c68e2242-f5c5-407d-aaee-632071cd0198)
- View the decrypted content:
```
xxd decrypted_challenge.txt
```
![image](https://github.com/user-attachments/assets/a3e94ff9-5ff8-4e80-a2c1-7b39ec5e1b1b)

**Step 4: The client signs the decrypted message**

On the Client:
- Sign the file decrypted_challenge.txt:
```
openssl dgst -sha256 -sign client_private_key.pem -out signed_challenge.bin decrypted_challenge.txt
```
![image-8](https://github.com/user-attachments/assets/47350dbd-76cd-4eaf-8b8a-084b71f993e6)
![image](https://github.com/user-attachments/assets/a796fb4d-aa6a-4c93-b929-0e87c10d8bf7)

**Step 5: Server verifies the signature**

On the Server, we will verify the signature:
```
openssl dgst -sha256 -verify client_public_key.pem -signature signed_challenge.bin decrypted_challenge.txt
```
![image-9](https://github.com/user-attachments/assets/d6c497c8-fe91-4bdd-9e7f-778d5765a634)

The output is: "Verified OK", which shows that the authentication is successful.

# Task 2: Encrypting large message 
Create a text file at least 56 bytes.
**Question 1**:
Encrypt the file with aes-256 cipher in CFB and OFB modes. How do you evaluate both cipher as far as error propagation and adjacent plaintext blocks are concerned. 

**Answer 1**:

## Step 1: Create a text file named `message.txt`:

- The text file is at least 56 bytes, so I will use python to create

```
python -c "print '0123456789' * 10" > message.txt
```

- Check file size, I will use command:
```
Get-Item message.txt | Select-Object Name, Length
```
 
![image-16](https://github.com/user-attachments/assets/fb4e05c5-4013-4654-8a4b-179067e10d82)


The size of file is 102


## Step 2: Encrypt file with AES-256 in CFB and OFB modes:

Key of AES-256 is `e2ca81a858b089013e9a1bf12f0d74c431760ec75f35353a838096e8c9c82f6f` and iv is `bcc02b3544d458e14ec545817f18b3d3`

- Encrypt `message.txt` with `CFB mode`
```
openssl enc -aes-256-cfb -e -in message.txt -out message-cfb.txt -K e2ca81a858b089013e9a1bf12f0d74c431760ec75f35353a838096e8c9c82f6f -iv bcc02b3544d458e14ec545817f18b3d3
```
- To show message-cfb.txt, I will use git bash
```
xxd message-cfb.txt
```
![image-17](https://github.com/user-attachments/assets/f16152a0-5383-4801-8f17-c6ce311b4929)

- Encrypt `message.txt` with `OFB mode`
```
openssl enc -aes-256-ofb -e -in message.txt -out message-ofb.txt -K e2ca81a858b089013e9a1bf12f0d74c431760ec75f35353a838096e8c9c82f6f -iv bcc02b3544d458e14ec545817f18b3d3
```
- Show the `message-ofb.txt`
![image-18](https://github.com/user-attachments/assets/7b8fa4e5-0347-48bf-aa01-b9974275d0ec)


**Evaluate the outcome**

CFB Mode:

- Errors in one bit can propagate, impacting both the current plaintext block and the subsequent blocks.

- Adjacent plaintext blocks are interdependent, so a change in one plaintext block affects the subsequent ciphertext blocks

OFB Mode:

- Errors are confined to the affected bit within the current block and do not propagate further.

- Plaintext blocks are encrypted separately, ensuring changes in one block have no impact on others.

**Question 2**:
Modify the 8th byte of encrypted file in both modes (this emulates corrupted ciphertext).
Decrypt corrupted file, watch the result and give your comment on Chaining dependencies and Error propagation criteria.

**Answer 2**:

## Step 1: Convert file to hex and modify the 8th byte:
**Convert file message-cfb from binary to hex**
```
xxd message-cfb.txt > message-cfb.hex
```
In message-cfb, the 8th byte of encrypted file is `45`, so I will replace it by `40`

![image-19](https://github.com/user-attachments/assets/b94628cb-7f15-4b59-a03b-31e10db2aeeb)


After modify the encrypted file, I will convert that file back to binary

```
xxd -r message-cfb.hex > message-cfb.txt
```
**Modify the 8th byte in OFB mode**

We will do the same as above
![image-20](https://github.com/user-attachments/assets/82031855-1470-4fb1-a2dd-cb8ef7671cfe)


## Step 2: Decrypt altered files:

We will decrypt `message-cfb.txt` into `message-cfb-dec.txt`
```
openssl enc -aes-256-cfb -d -in message-cfb.txt -out message-cfb-dec.txt -K e2ca81a858b089013e9a1bf12f0d74c431760ec75f35353a838096e8c9c82f6f -iv bcc02b3544d458e14ec545817f18b3d3
```
We will decrypt `message-ofb.txt` into `message-ofb-dec.txt`
```
openssl enc -aes-256-ofb -d -in message-ofb.txt -out message-ofb-dec.txt -K e2ca81a858b089013e9a1bf12f0d74c431760ec75f35353a838096e8c9c82f6f -iv bcc02b3544d458e14ec545817f18b3d3
```

## Step 4: Show the result after we modify files:
```
cat message-cfb-dec.txt
```
```
cat message-ofb-dec.txt
```
![image-21](https://github.com/user-attachments/assets/47430d7e-900a-47f3-bda9-7e76c824c21c)


In summary, CFB mode operates such that the encryption of each plaintext block depends on the previous ciphertext block. If a single bit in the ciphertext becomes corrupted, it impacts the corresponding bit in the decrypted plaintext block and also the corresponding bit in the next plaintext block. 

This means the error propagation is limited to the next block and does not extend beyond it. Consequently, in this scenario, the corruption will influence the 8th byte in the decrypted plaintext as well as the corresponding bit in the following block, while the rest of the decrypted plaintext remains unaffected. In my case, the resulting incorrect plaintext:

![image-23](https://github.com/user-attachments/assets/31fe292f-3a17-444c-98f2-036edda94fd8)


On the other hand, in OFB mode, a keystream is generated independently of the plaintext and is used to encrypt it. If a single bit in the ciphertext becomes corrupted, it affects only the corresponding bit in the decrypted plaintext block. 

Since the keystream generation remains unaffected by the corruption, there is no error propagation to subsequent blocks. In my case, the resulting incorrect plaintext:
![image-22](https://github.com/user-attachments/assets/dd06fbf2-0593-4111-966d-7a8ce98bb5d8)

It should be `7`, not `2`
