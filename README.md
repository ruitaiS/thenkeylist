
# AES-GCM Encryption and Decryption

This document explains how the encryption and decryption process works in the provided code using AES-GCM (Advanced Encryption Standard - Galois/Counter Mode).

---

## Requirements

To run the encryption and decryption code, ensure the following dependencies are installed on your system:

#### **1. Install Python 3 and `pip` (if not already installed)**
   ```bash
   sudo apt update
   sudo apt install python3 python3-pip
   ```
   
#### **2. Install the pycryptodome library**
To install the `pycryptodome` library, use one of the following methods:
- **Via apt** (recommended for system-wide installation):
```bash
sudo apt install python3-pycryptodome
```
- **Via pip** (for virtual environments or the latest version):
```bash
pip install pycryptodome
```
#### **3. If you encounter an error like `externally-managed-environment` while installing with pip, you can override it using the following command**
```bash
pip install pycryptodome --break-system-packages
```
---
## Running the code

(TODO)
User configurable parameters are at the end of param_handler.py

encrypt.py handles encryption, decrypt.py handles decryption

-p specifies the password, and is required for both

-f flag reads from a file, eg:
-m flag reads directly off the terminal string
(either -f or -m must be used)

examples:
```bash
python encrypt.py -f foo.txt -p password123
python encrypt.py -m "hello world!" -p password123

python decrypt.py -f foo.enc -p password123
python decrypt.py -m o92UunLpDmuhPY/gULkZC4ih7PLqXksWQKuqEc5Rth27mFo3poMQnG8tHbNuLxRAIfwX8ntrerpEsfUZ -p password123
```

Recommend clearing the terminal history when you're done. In Bash, the command is:
```bash
history -c && clear
```

## Common Errors and Causes
(TODO)


## Details on the Encryption Process

### 1. **Input Parameters**
- **Password**: A user-provided passphrase
- **Plaintext**: The message to encrypt

### 2. **Key Derivation**
The password is converted into a cryptographic key using a **Key Derivation Function (KDF)**:
- **Salt**: A random 16-byte value is generated (`get_random_bytes(16)`).
- **KDF (PBKDF2)**: Combines the password and salt using a secure hash function (SHA-256 by default) and performs multiple iterations (100,000 by default). This ensures the derived key is unique and strong.

**Output**:
- A **256-bit key** (32 bytes) is derived.

### 3. **Encryption with AES**
AES-GCM mode is used for encryption:
- **GCM (Galois/Counter Mode)**: Provides encryption and built-in integrity checks (via the authentication tag).

Key steps:
1. **Nonce**: A random, unique 16-byte value is generated automatically (`cipher.nonce`).
2. **Encrypt and Digest**: The plaintext is encrypted into ciphertext, and an authentication tag is produced to ensure the ciphertext's integrity.

**Output**:
- **Ciphertext**: The encrypted version of the plaintext.
- **Tag**: The authentication tag, used during decryption to verify data integrity.

### 4. **Encoding the Encrypted Data**
The salt, nonce, ciphertext, and tag are concatenated into a single binary string in the following order:
1. **Salt** (16 bytes)
2. **Nonce** (16 bytes)
3. **Tag** (16 bytes)
4. **Ciphertext** (variable length, same as plaintext)

This combined binary string is then **Base64-encoded** to ensure it can be safely transmitted as a single string.

**Output**:
- A single Base64-encoded string representing all components of the encrypted data.

---

## Why This Works
1. **Salt**:
- Ensures that even if the same password is reused, the derived key will be different for each encryption.
- The salt is not a secret and is typically transmitted alongside the ciphertext. Its purpose is to ensure the key derived from a password is unique for each encryption.
- Even if the attacker knows the salt, they cannot derive the key without knowing the password.

2. **Nonce**:
- Ensures that the same plaintext encrypted with the same key produces different ciphertexts.
- The nonce (Number Used Once) is also not a secret but must be unique for each encryption operation with the same key.
- Knowing the nonce doesn’t help the attacker decrypt the ciphertext because the security of AES-GCM relies on the uniqueness of the nonce, not its secrecy.

3. **Authentication Tag**:
- Protects against tampering by ensuring that decryption will fail if the ciphertext or nonce has been altered.
- The tag ensures the integrity and authenticity of the ciphertext. An attacker knowing the tag doesn’t allow them to tamper with or forge valid ciphertext because the tag is tied to the specific ciphertext, key, and nonce.

4. **AES-GCM**: Provides robust encryption and built-in integrity checks, making it secure for most applications.

Even if an attacker knows the exact format of the output, (eg. can identify which parts of the Base64-encoded string correspond to the salt, nonce, tag, and ciphertext), this knowledge doesn’t provide any meaningful way to reverse the encryption or forge data.


---

## Decryption Workflow
The decryption process reverses the encryption:
1. Decode the Base64-encoded string to retrieve the binary data.
2. Split the binary data into its components:
   - Salt (16 bytes)
   - Nonce (16 bytes)
   - Tag (16 bytes)
   - Ciphertext (remaining bytes)
3. Use the salt to regenerate the same cryptographic key from the password.
4. Recreate the AES-GCM cipher using the key and nonce.
5. Decrypt the ciphertext and verify its authenticity using the tag.

---

### **Important Note**
Ensure you always:
- Use a **new, random salt** for each encryption.
- Never reuse the **same nonce** across multiple encryptions with the same key.

#### Why is this important?
1. **Reusing the same salt** allows attackers to detect patterns in the derived keys, making your encryption vulnerable to **dictionary** or **rainbow table attacks**.

2. **Reusing the nonce with the same key** can lead to **nonce reuse attacks**, where attackers may recover plaintexts, deduce relationships between messages, or compromise the key itself.

#### What about tag?
Users don’t need to manage the tag manually, as it is directly handled by the AES-GCM encryption process.


Both the **salt** and **nonce** are essential for maintaining the confidentiality and integrity of encrypted data. Always generate fresh random values for each encryption operation.

