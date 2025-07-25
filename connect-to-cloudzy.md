# Setting Up SSH Public Key Authentication for Cloudzy VPS

Here's a clear, step-by-step explanation of how to set up SSH key authentication to securely log in to your Cloudzy Linux VPS:

## Why Use SSH Keys?
- More secure than passwords (resistant to brute force attacks)
- No need to remember/type passwords for each login
- Required by many cloud providers for security compliance

## Step-by-Step Setup Guide

### **Step 1: Generate SSH Key Pair on Your Local Machine**

1. Open terminal (Linux/macOS) or Git Bash/PuTTY (Windows)
2. Run this command:
   ```bash
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```
   - `-t rsa`: Uses RSA algorithm
   - `-b 4096`: Creates 4096-bit key (more secure than default 2048)
   - `-C`: Adds a comment (usually your email)

### **Step 2: Specify Key Location and Passphrase**

1. You'll see:
   ```
   Enter file in which to save the key (/home/username/.ssh/id_rsa):
   ```
   - Press Enter to use default location
   - OR specify custom path like `/home/username/.ssh/cloudzy_vps_key`

2. Next, set a passphrase (recommended for extra security):
   ```
   Enter passphrase (empty for no passphrase):
   ```
   - Type a strong passphrase (you'll need this to use the key)

### **Step 3: Verify Key Generation**

You should see output like:
```
Your identification has been saved in /home/username/.ssh/id_rsa
Your public key has been saved in /home/username/.ssh/id_rsa.pub
```

### **Step 4: Copy Public Key to Cloudzy VPS**

**Method A: Using ssh-copy-id (easiest)**
```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub username@your-cloudzy-ip
```
(Enter your VPS password when prompted)

**Method B: Manual Copy (if ssh-copy-id unavailable)**
1. View your public key:
   ```bash
   cat ~/.ssh/id_rsa.pub
   ```
2. Copy the entire output (starts with `ssh-rsa...`)

3. SSH into your VPS:
   ```bash
   ssh username@your-cloudzy-ip
   ```

4. On the VPS:
   ```bash
   mkdir -p ~/.ssh
   echo "PASTE_YOUR_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
   chmod 600 ~/.ssh/authorized_keys
   ```

### **Step 5: Test Key-Based Login**

1. Try logging in:
   ```bash
   ssh -i ~/.ssh/id_rsa username@your-cloudzy-ip
   ```
   - If you set a passphrase, you'll be prompted for it

2. If successful, you'll be logged in without password prompt

### **Step 6: Disable Password Authentication (Optional but Recommended)**

1. On your VPS, edit SSH config:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. Change these lines:
   ```
   PasswordAuthentication no
   PubkeyAuthentication yes
   ```

3. Restart SSH service:
   ```bash
   sudo systemctl restart sshd
   ```

## PuTTY Users (Windows)

If using PuTTY instead of OpenSSH:

1. Generate keys using PuTTYgen
2. Save private key as `.ppk` file
3. In PuTTY:
   - Connection > SSH > Auth
   - Browse to select your private key
   - Save session for future use

## Troubleshooting Tips

- **Permission denied**? Ensure:
  - `~/.ssh` has 700 permissions
  - `authorized_keys` has 600 permissions
  - Private key has 600 permissions locally

- **Not working**? Check logs:
  ```bash
  sudo tail -f /var/log/auth.log
  ```
