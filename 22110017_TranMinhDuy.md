# Task 1: Public-key based authentication 
**Question 1**: 
Implement public-key based authentication step-by-step with openssl according the following scheme.
![alt text](image-1-1.png)

**Answer 1**:

**Step1: Create RSA key Pair** 
- On the client:

```
openssl genpkey -algorithm RSA -out client_private_key.pem
openssl rsa -pubout -in client_private_key.pem -out client_public_key.pem
```
![alt text](image-2.png)
- On the server:

```
openssl genpkey -algorithm RSA -out server_private_key.pem
openssl rsa -pubout -in server_private_key.pem -out server_public_key.pem
```
![alt text](image-3.png)
**Step 2: Send Challenge Message from Server to Client**


- On Server create challenge message:


```
echo "This is a challenge message" > challenge.txt
```
![alt text](image-4.png)
- Encrypt the challenge message with the client's public key:

```
openssl pkeyutl -encrypt -inkey client_public_key.pem -pubin -in challenge.txt -out encrypted_challenge.bin
```
![alt text](image-5.png)
- Send the file encrypted_challenge.bin to the client.

**Step 3: Client Decrypts the Challenge Message**
On the Client:

- Decrypt the encrypted_challenge.bin file:
```
openssl pkeyutl -decrypt -inkey client_private_key.pem -in encrypted_challenge.bin -out decrypted_challenge.txt
```
![alt text](image-6.png)
- View the decrypted content:
```
cat decrypted_challenge.txt
```
![alt text](image-7.png)

**Step 4: Client Signs the Decrypted Challenge**

On the Client:
- Sign the file decrypted_challenge.txt:
```
openssl dgst -sha256 -sign client_private_key.pem -out signed_challenge.bin decrypted_challenge.txt
```
![alt text](image-8.png)
- Send the file signed_challenge.bin to the server.

**Step 5: Server Verifies the Signature**

On the Server:
- Verify the signature:
```
openssl dgst -sha256 -verify client_public_key.pem -signature signed_challenge.bin decrypted_challenge.txt
```
![alt text](image-9.png)

The output is: "Verified OK", which shows that the authentication is successful.

# Task 2: Encrypting large message 
Create a text file at least 56 bytes.
**Question 1**:
Encrypt the file with aes-256 cipher in CFB and OFB modes. How do you evaluate both cipher as far as error propagation and adjacent plaintext blocks are concerned. 

**Answer 1**:

**Step 1. Create a Text File**

Create a file with more than 56 bytes:
```
echo "This is a sample text file for encryption testing. It is more than 56 bytes long." > large_text_file.txt
```
![alt text](image-10.png)
***2. Generate AES Key and Initialization Vector (IV)***

- Generate a random 256-bit key for AES-256 and Generate a random 128-bit IV :
```
openssl rand -hex 32 > aes_key.txt
```
```
openssl rand -hex 16 > aes_iv.txt
```
![alt text](image-12.png)
**Step 3: Encrypt with AES-256-CFB Mode and AES-256-OFB Mode**

Encrypt the file (AES-256-CFB Mode):
```
openssl enc -aes-256-cfb -in large_text_file.txt -out encrypted_cfb.bin -K $(cat aes_key.txt) -iv $(cat aes_iv.txt)
```
Encrypt the file (AES-256-OFB Mode):
```
openssl enc -aes-256-ofb -in large_text_file.txt -out encrypted_ofb.bin -K $(cat aes_key.txt) -iv $(cat aes_iv.txt)
```
![alt text](image-13.png)



**Question 2**:
Modify the 8th byte of encrypted file in both modes (this emulates corrupted ciphertext).
Decrypt corrupted file, watch the result and give your comment on Chaining dependencies and Error propagation criteria.

**Answer 2**:





