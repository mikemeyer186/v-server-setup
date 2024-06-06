# vm-cloud-config

Configuration of V-Server

# 1. Create SSH-Key pair

The ED25519 key pair is a modern alternative to RSA keys

### create key

ssh-keygen -t ed25519

### define path to store the key

/Users/mike/.ssh/vm_da_ed25519

### check if key pair is stored in defined path

ls ~/.ssh

# 2. First login on v-server

To login on server use username and ip:

ssh username@ip

Confirm the following question with "yes" to use the new fingerprint and enter server password:
"Are you sure you want to continue connecting (yes/no/[fingerprint])? yes"

Output:
"Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-107-generic x86_64) ..."

# 3. Add the public key to the VM

To copy the public key to the VM us the programm ssh-copy-id in local terminal:

ssh-copy-id -i ~/.ssh/vm_da_ed25519.pub username@ip

Output:
"Number of key(s) added: 1

Now try logging into the machine, with: "ssh 'username@ip'"
and check to make sure that only the key(s) you wanted were added."

### try to login with ssh key without password

ssh -i ~/.ssh/vm_da_ed25519 username@ip

Output:
"Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-107-generic x86_64) ..."

### check storage path of key on server

ls -al ~/.ssh

### show content of file "authorized_keys

cat ~/.ssh/authorized_keys

# 4. Disable password-login

sudo nano /etc/ssh/sshd_config

### Remove the hastag before "PasswordAuthentication" and set the value to "no":

"...
PasswordAuthentication no
..."

### restart ssh.service

sudo systemctl restart ssh.service

### check if config on server works

first check:
ssh -i ~/.ssh/vm_da_ed25519 username@ip

double check:
ssh -o PubkeyAuthentication=no username@ip

Expected output:
... "Permission denied (publickey)."
