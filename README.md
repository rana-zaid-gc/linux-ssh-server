# Linux Server Setup with SSH Key Authentication

Setting up a remote Linux server and configuring it to allow SSH connections using **two separate SSH key pairs**, an `~/.ssh/config` alias for quick access, and `fail2ban` to protect against brute-force attacks.

> https://roadmap.sh/projects/ssh-remote-server-setup

## Goal

- Set up a remote Linux server
- Create two SSH key pairs and add both to the server
- Connect using either key
- Configure an SSH alias for one-word connection
- (Stretch) Install and configure `fail2ban`

## Server

A remote Linux server (Ubuntu) running on AWS EC2, with a security group allowing inbound SSH (port 22). Default user: `ubuntu`.

## Steps Taken

### 1. Created two SSH key pairs (on the local machine)

```bash
ssh-keygen -t ed25519 -f ~/.ssh/aws_key1 -C "aws-key1"
ssh-keygen -t ed25519 -f ~/.ssh/aws_key2 -C "aws-key2"
```

This produced two private keys (`aws_key1`, `aws_key2`) and their matching public keys (`aws_key1.pub`, `aws_key2.pub`).

> The private keys are kept secure on the local machine and are **never** shared or committed.

### 2. Added both public keys to the server

Connected to the server with the initial key, then appended both public keys to the server's authorized keys file:

```bash
echo '<aws_key1.pub contents>' >> ~/.ssh/authorized_keys
echo '<aws_key2.pub contents>' >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

### 3. Connected using both keys

Verified that **both** keys grant access independently:

```bash
ssh -i ~/.ssh/aws_key1 ubuntu@<server-ip>
ssh -i ~/.ssh/aws_key2 ubuntu@<server-ip>
```

Both log in successfully without a password.

### 4. Configured an SSH alias

Added an entry to `~/.ssh/config` on the local machine:

```
Host myserver
    HostName <server-ip>
    User ubuntu
    IdentityFile ~/.ssh/aws_key1
```

Now the server can be reached with a single short command:

```bash
ssh myserver
```

### 5. Installed and configured fail2ban (stretch goal)

```bash
sudo apt update
sudo apt install -y fail2ban
sudo systemctl enable --now fail2ban
```

Verified it is active and protecting SSH:

```bash
sudo systemctl status fail2ban
sudo fail2ban-client status sshd
```

`fail2ban` monitors authentication logs and temporarily bans IP addresses that make too many failed login attempts, mitigating brute-force attacks.

## Concepts Learned

- **SSH key authentication** — how public/private key pairs work
- **Managing multiple keys** on a single server via `authorized_keys`
- **SSH config aliases** — simplifying connections with `~/.ssh/config`
- **Server hardening basics** — using `fail2ban` to block brute-force attempts
- **Key security** — never exposing private keys

## Security Note

Private keys are never included in this repository. Only the steps are documented. Public keys are safe to share; private keys must always remain protected on the local machine.

---






<img width="907" height="242" alt="image" src="https://github.com/user-attachments/assets/1dd0e0d2-8b5e-40d2-a295-280414fd2f68" />
