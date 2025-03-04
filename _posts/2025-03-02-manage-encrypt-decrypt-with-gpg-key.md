---
title: Manage, Encrypt and Decrypt with gpg Keys
author:
  name: 
  link: 
date: 2025-03-02 10:35:00 -0400
categories: [Security]
tags: [gpg]
render_with_liquid: false
---

Managing, encrypting, and decrypting with a GPG key involves using the `gpg` command-line tool to handle your keys, secure data, and recover it. Below, I’ll break this into three sections: **Managing Keys**, **Encrypting**, and **Decrypting**. I’ll assume you’ve already created a GPG key pair (if not, refer to my earlier response on creating a key).

---

### 1. Managing GPG Keys

Managing keys includes listing, editing, adding subkeys, exporting, importing, and revoking them. Here are common tasks:

#### List Keys
- **Public keys** in your keyring:
  ```bash
  gpg --list-keys
  ```
  Example output:
  ```
  pub   rsa4096 2025-03-03 [SC] [expires: 2027-03-03]
        1234567890ABCDEF1234567890ABCDEF12345678
  uid           [ultimate] Alice Smith <alice@example.com>
  sub   rsa4096 2025-03-03 [E]
  ```
- **Secret keys** (private keys):
  ```bash
  gpg --list-secret-keys
  ```
  Example:
  ```
  sec   rsa4096 2025-03-03 [SC] [expires: 2027-03-03]
        1234567890ABCDEF1234567890ABCDEF12345678
  ssb   rsa4096 2025-03-03 [E]
  ```

#### Edit a Key
To modify a key (e.g., add subkeys, change expiration, or passphrase):
```bash
gpg --edit-key 1234567890ABCDEF1234567890ABCDEF12345678
```
- Inside the prompt:
  - `adduid`: Add a new user ID.
  - `addkey`: Add a subkey (e.g., for signing or encryption).
  - `expire`: Change expiration date.
  - `passwd`: Change passphrase.
  - `save`: Save changes and exit.

Example: Add a signing subkey:
```
gpg> addkey
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection? 4
What keysize do you want? (2048) 4096
Key is valid for? (0) 2y
gpg> save
```

#### Export Keys
- **Public key**:
  ```bash
  gpg --armor --export 1234567890ABCDEF1234567890ABCDEF12345678 > alice_public.asc
  ```
- **Private key** (includes subkeys):
  ```bash
  gpg --armor --export-secret-keys 1234567890ABCDEF1234567890ABCDEF12345678 > alice_private.asc
  ```

#### Import Keys
- **Public key**:
  ```bash
  gpg --import alice_public.asc
  ```
- **Private key**:
  ```bash
  gpg --import alice_private.asc
  ```

#### Revoke a Key
If a key is compromised:
1. Import the revocation certificate (created during key generation):
   ```bash
   gpg --import 1234567890ABCDEF1234567890ABCDEF12345678.rev
   ```
2. Publish the revocation:
   ```bash
   gpg --keyserver hkps://keys.openpgp.org --send-keys 1234567890ABCDEF1234567890ABCDEF12345678
   ```

---

### 2. Encrypting with a GPG Key

Encryption secures data so only the intended recipient (with the private key) can decrypt it.

#### Encrypt a File
1. Identify the recipient’s public key:
   ```bash
   gpg --list-keys
   ```
   Example: Bob’s key is `9876543210FEDCBA9876543210FEDCBA98765432`.

2. Encrypt the file for Bob:
   ```bash
   gpg --armor --encrypt --recipient 9876543210FEDCBA9876543210FEDCBA98765432 myfile.txt
   ```
   - `--armor`: ASCII output (creates `myfile.txt.asc`).
   - `--recipient` (or `-r`): Specifies the recipient’s key.
   - Output: `myfile.txt.asc` (or `.gpg` without `--armor`).

3. Optionally, encrypt for multiple recipients:
   ```bash
   gpg --armor --encrypt -r 9876543210FEDCBA9876543210FEDCBA98765432 -r alice@example.com myfile.txt
   ```

#### Encrypt and Sign
To encrypt and sign in one step (proving it’s from you):
```bash
gpg --armor --encrypt --sign -r 9876543210FEDCBA9876543210FEDCBA98765432 myfile.txt
```
- You’ll be prompted for your passphrase.

#### Encrypt a Message
To encrypt text directly:
```bash
gpg --armor --encrypt -r 9876543210FEDCBA9876543210FEDCBA98765432
```
Type your message, then press `Ctrl+D` (Unix) or `Ctrl+Z` (Windows).

---

### 3. Decrypting with a GPG Key

Decryption recovers the original data using your private key.

#### Decrypt a File
If someone encrypted a file for you (using your public key):
```bash
gpg --decrypt myfile.txt.asc > myfile.txt
```
- You’ll be prompted for your passphrase.
- Output: The decrypted content is written to `myfile.txt`.

#### Decrypt and Verify
If the file was signed and encrypted:
```bash
gpg --decrypt myfile.txt.asc > myfile.txt
```
- GPG will verify the signature if the signer’s public key is in your keyring.
- Example output:
  ```
  gpg: encrypted with 4096-bit RSA key, ID FEDCBA9876543210, created 2025-03-03
       "Alice Smith <alice@example.com>"
  gpg: Signature made Mon 03 Mar 2025 12:00:00 PM EST
  gpg:                using RSA key 1234567890ABCDEF1234567890ABCDEF12345678
  gpg: Good signature from "Alice Smith <alice@example.com>" [ultimate]
  ```

#### Decrypt Without Saving
To view the decrypted content without saving:
```bash
gpg --decrypt myfile.txt.asc
```

---

### Additional Tips
- **Passphrase Caching**: Use `gpg-agent` to avoid repeated passphrase prompts:
  ```bash
  gpg-agent --daemon
  ```
- **Key Trust**: If verifying a signature fails, ensure you trust the signer’s key:
  ```bash
  gpg --edit-key 1234567890ABCDEF1234567890ABCDEF12345678
  gpg> trust
  ```
- **Symmetric Encryption**: Encrypt without keys (using a passphrase only):
  ```bash
  gpg --armor --symmetric myfile.txt
  ```
