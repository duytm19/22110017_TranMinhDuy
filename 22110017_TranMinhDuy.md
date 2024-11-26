# Task 1: Public-key based authentication 
**Question 1**: 
Implement public-key based authentication step-by-step with openssl according the following scheme.
![image-1-1](https://github.com/user-attachments/assets/2ef43c3a-e259-4f13-aa11-37a82db1d36a)


**Answer 1**:

**Step1: Create RSA key Pair** 
- On the client:

```
openssl genpkey -algorithm RSA -out client_private_key.pem
openssl rsa -pubout -in client_private_key.pem -out client_public_key.pem
```
![image-2](https://github.com/user-attachments/assets/b2ca56e0-20a7-43b8-905b-3d7d6fc0f704)
- On the server:

```
openssl genpkey -algorithm RSA -out server_private_key.pem
openssl rsa -pubout -in server_private_key.pem -out server_public_key.pem
```
![image-3](https://github.com/user-attachments/assets/fbf31572-de7b-430e-9a23-02c4e06051c8)
**Step 2: Send Challenge Message from Server to Client**


- On Server create challenge message:


```
echo "This is a challenge message" > challenge.txt
```
![image-4](https://github.com/user-attachments/assets/7cafd170-7e16-4e03-ba92-e788d25de47a)
- Encrypt the challenge message with the client's public key:

```
openssl pkeyutl -encrypt -inkey client_public_key.pem -pubin -in challenge.txt -out encrypted_challenge.bin
```
![image-5](https://github.com/user-attachments/assets/73207212-b701-4a96-9dea-65ea58b3ce43)
- Send the file encrypted_challenge.bin to the client.

**Step 3: Client Decrypts the Challenge Message**
On the Client:

- Decrypt the encrypted_challenge.bin file:
```
openssl pkeyutl -decrypt -inkey client_private_key.pem -in encrypted_challenge.bin -out decrypted_challenge.txt
```
![image-6](https://github.com/user-attachments/assets/c68e2242-f5c5-407d-aaee-632071cd0198)
- View the decrypted content:
```
cat decrypted_challenge.txt
```
![image-7](https://github.com/user-attachments/assets/7d716356-fcb5-42d3-a4d2-fc359545d46b)

**Step 4: Client Signs the Decrypted Challenge**

On the Client:
- Sign the file decrypted_challenge.txt:
```
openssl dgst -sha256 -sign client_private_key.pem -out signed_challenge.bin decrypted_challenge.txt
```
![image-8](https://github.com/user-attachments/assets/47350dbd-76cd-4eaf-8b8a-084b71f993e6)
- Send the file signed_challenge.bin to the server.

**Step 5: Server Verifies the Signature**

On the Server:
- Verify the signature:
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

**Step 1. Create a Text File**

Create a file with more than 56 bytes:
```
echo "This is a sample text file for encryption testing. It is more than 56 bytes long." > large_text_file.txt
```
![image-10](https://github.com/user-attachments/assets/f06eb130-a36a-43a8-9255-e26bf424fcc3)
***2. Generate AES Key and Initialization Vector (IV)***

- Generate a random 256-bit key for AES-256 and Generate a random 128-bit IV :
```
openssl rand -hex 32 > aes_key.txt
```
```
openssl rand -hex 16 > aes_iv.txt
```
![image-12](https://github.com/user-attachments/assets/9aeb27d0-6cbe-422a-81af-e5a7a37a120e)
**Step 3: Encrypt with AES-256-CFB Mode and AES-256-OFB Mode**

Encrypt the file (AES-256-CFB Mode):
```
openssl enc -aes-256-cfb -in large_text_file.txt -out encrypted_cfb.bin -K $(cat aes_key.txt) -iv $(cat aes_iv.txt)
```
Encrypt the file (AES-256-OFB Mode):
```
openssl enc -aes-256-ofb -in large_text_file.txt -out encrypted_ofb.bin -K $(cat aes_key.txt) -iv $(cat aes_iv.txt)
```
![image-13](https://github.com/user-attachments/assets/a2a4e866-fd39-4ba1-9224-de7e30674577)



**Question 2**:
Modify the 8th byte of encrypted file in both modes (this emulates corrupted ciphertext).
Decrypt corrupted file, watch the result and give your comment on Chaining dependencies and Error propagation criteria.

**Answer 2**:





