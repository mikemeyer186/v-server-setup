# vm-cloud-config

Procedure to configure a vm-cloud on a server with zsh/bash:

- creating ssh-key pair
- deactivating password login
- login with a SSH-key only
  
</br>

## 1. Create SSH-key pair on local machine

> [!NOTE]
> The ED25519 key pair is a modern alternative to RSA keys and should be used. 

#### create key

    ssh-keygen -t ed25519

#### define path to store the key

    ~/.ssh/id_ed25519

#### check if key pair is stored in defined path

    ls ~/.ssh

</br>

## 2. First login on V-Server

#### login with ssh command

    ssh <username>@<ip>

#### Confirm the question with "yes" to use the new fingerprint and enter server password
    Are you sure you want to continue connecting (yes/no/[fingerprint])? yes

#### Output on successful login 👍🏻
    Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-107-generic x86_64) ...

</br>

## 3. Add the public key to the VM

> [!IMPORTANT]
To copy the public key to the vm use the programm `ssh-copy-id` on local terminal:

    ssh-copy-id -i ~/.ssh/id_ed25519.pub username@ip

#### Output on successfully adding the public key on vm 👍🏻
```
Number of key(s) added: 1
Now try logging into the machine, with: "ssh 'username@ip'"
and check to make sure that only the key(s) you wanted were added.
```

#### try to login with ssh key without password

    ssh -i ~/.ssh/id_ed25519 <username>@<ip>

#### check storage path of key on server

    ls -al ~/.ssh

#### show content of file "authorized_keys"

> [!NOTE]
> The public key should stored in this file

    cat ~/.ssh/authorized_keys

</br>

## 4. Disable password-login

> [!IMPORTANT]
> Disable the password-login on V-Server, since it can be unsafe. The SSH-key pair is a safe authorization method.
> Using of `sudo` to edit the `sshd-config` with nano 

    sudo nano /etc/ssh/sshd_config

#### Remove the hastag before "PasswordAuthentication" and set the value to "no"

```
...
PasswordAuthentication no
...
```

#### restart ssh.service after changes are done

    sudo systemctl restart ssh.service

#### check if config on server works

first check (login works):

    ssh -i ~/.ssh/vm_da_ed25519 username@ip

double check (login denied):

    ssh -o PubkeyAuthentication=no username@ip

Expected output:
> Permission denied (publickey).

</br>

