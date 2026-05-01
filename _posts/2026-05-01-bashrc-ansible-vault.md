---
title: "Ansible Vault"
date: 2026-05-01
categories: [ansible, DevOps & Automation, AWS]
tags: [ansible, AWS IAM, Ansible Vault, bashrc]

---

# .bashrc or Ansible Vault

🔹 What is .bashrc (very simple)

.bashrc is just a configuration file for your terminal (Bash shell).

It runs automatically every time you open a terminal
You usually store:
environment variables
shortcuts (aliases)
PATH settings

👉 Example inside .bashrc:
export AWS_ACCESS_KEY_ID="ABC123"
export AWS_SECRET_ACCESS_KEY="XYZ456"

This means:

“Every time I open terminal, these variables are loaded automatically.”

Why using .bashrc for AWS keys is risky

Even though it works, it’s not secure:

1.Plain text
 Anyone who accesses your machine can read it
2. Can be exposed accidentally
 If you upload your dotfiles (GitHub), your keys leak
3. Process visibility
 In some cases, environment variables can be seen by other processes/users

Why Ansible Vault is better

Instead of plain text, Vault does this:

🔐 Encrypts your secrets (AES-256)
📁 Stores them safely in a file
🔑 Requires a password to decrypt

👉 Example:

aws_access_key: "ABC123"

Becomes:

$ANSIBLE_VAULT;1.1;AES256
8392asdjkhaskjdh...

👉 Meaning:

Even if someone sees the file, they can’t read anything
