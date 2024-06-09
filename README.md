# V-Server setup

Procedure to configure a VM-cloud on a server with zsh:

- creating ssh-key pair
- deactivating password login
- login with a SSH-key only
- installing nginx
- installing alternativ index file
- creating aliases and identities
- setting up Git and GitHub
  
</br>

## 1. Creating SSH-key pair on local machine

> [!NOTE]
> The ED25519 key pair is a modern alternative to RSA keys and should be used.

</br>

Creating a key pair with `ssh-keygen` command and defining the path in `~/.ssh/id_ed25519`:

```
ssh-keygen -t ed25519
```

Checking, if key pair is stored correctly in defined path:

```
ls ~/.ssh
```

</br>

## 2. First login on v-server from local machine

Login with `ssh` command followed by username and host ip:

```
ssh <username>@<ip>
```

Confirmation to use the new fingerprint and enter server password:

```
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

</br>

## 3. Adding the public key to the VM from local maschine

> [!IMPORTANT]
> Using the programm `ssh-copy-id` on local terminal to copy the public key to the vm:

```
ssh-copy-id -i ~/.ssh/id_ed25519.pub <username>@<ip>
```

Output on successfully adding the public key on vm 👍🏻:
```
Number of key(s) added: 1
Now try logging into the machine, with: "ssh 'username@ip'"
and check to make sure that only the key(s) you wanted were added.
```

Login by using the SSH-key:

```
ssh -i ~/.ssh/id_ed25519 <username>@<ip>
```


Checking path of key on server. In the directory `~/.ssh/` the file `authorized_keys` is stored:

```
ls -al ~/.ssh
```

Checking content of file `authorized_keys`:

> [!NOTE]
> The public key from client must be stored in this file.

```
cat ~/.ssh/authorized_keys
```

</br>

## 4. Deactivating password-login on server

> [!IMPORTANT]
> Password-login on a server can be unsafe. The SSH-key pair is a safe authorization method.

> [!CAUTION]
> Before deactivating password login, it has to be ensured that the login on server works well with password!

</br>

Editing the `sshd-config` with nano:

```
sudo nano /etc/ssh/sshd_config
```

Removing of the hastag before `PasswordAuthentication` and setting value to `no`

```
...
PasswordAuthentication no
...
```

Restarting `ssh.service` after changes are done:

```
sudo systemctl restart ssh.service
```

### Checking, if config on server works well

First check (login works ✅):

```
ssh -i ~/.ssh/id_ed25519 <username>@<ip>
```

Double check (permission should be denied ❌):

```
ssh -o PubkeyAuthentication=no <username>@<ip>
```

Expected output, if the password authentication is deactiveated correctly 👍🏻:

```
Permission denied (publickey).
```

</br>

## 4. Installing of nginx on server

Updating the package manager `apt` and then install `nginx`:

```
sudo apt update
sudo apt install nginx -y
```

Checking status of `nginx.service`:

```
systemctl status nginx.service     
```

Checking, if nginx webserver is active by connecting to ip with webbrowser:

```
http://<ip-adress>
```

![nginx](https://github.com/mikemeyer186/vm-cloud-config/assets/112903209/d7be6337-9e82-4a38-9eec-379986804bca)

</br>

## 5. Installing alternative html content on webserver

Checking the default html content in `index.nginx-debian.html`:

```
cat /var/www/html/index.nginx-debian.html
```

Creating a directory with the command `mkdir` in `/var/www/`:

```
sudo mkdir /var/www/alternatives
```

Creating a html file in this directory:

```
sudo touch /var/www/alternatives/alternate-index.html
```

Checking, if the new html file is created correctly:

```
ls /var/www/alternatives
```

Configuring of the nginx settings:

1. Creating the file `alternatives` in following directive:

```
sudo nano /etc/nginx/sites-enabled/alternatives
```

2. Adding the follwing code in this file:

```nginx
server { 
        listen 8081;                          # Server listens on port 8081
        listen [::]:8081;                     # Server listens on IPv6 connections

        root /var/www/alternatives;           # Root directory of webserver
        index alternate-index.html;           # Default file on initial laoding in root directory

        location / { 
                try_files $uri $uri/ =404;    # If files could not be found in root directory, 404 will be returned
        }
}

```

Editing the alternative html file:

```
sudo nano /var/www/alternatives/alternate-index.html
```

Example html:

```html
<!doctype html>
<html>
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Nginx Webserver</title>
</head>
<body>
        <h1>Welcome to my Nginx Webserver</h1>
        <p>This webserver is configured on Ubuntu Server!</p>
        </hr>
        <p>You can follow my instructions of the configuration in my github repository</p>
        <a href="https://github.com/mikemeyer186/vm-cloud-config"
           target="_blank">https://github.com/mikemeyer186/vm-cloud-config</a>
</body>
</html>
```

</br>

> [!NOTE]
> See my alternative `index.html` in my [repository](https://github.com/mikemeyer186/vm-cloud-config/blob/main/index.html).

</br>

Testing of configuration (default `nginx.conf`):

```
sudo nginx -t
```

Expected output, if configuaration works well 👍🏻:

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Restarting nginx and check if everything is running:

```
sudo service nginx restart
systemctl status nginx.service
```

Checking if the webserver can be connected with webbrowser:

```
http://<ip-adress>:8081
```

![alternative-index](https://github.com/mikemeyer186/vm-cloud-config/assets/112903209/c22bd09d-a7f6-48a8-b184-c68cc2cd74f5)

> [!NOTE]
> Trying to connect to a non existing page should show 404 error (default nginx).

</br>

## 6. Creating aliases on local machine for easier login on server

Creating a new alias:

```
alias da_connect="ssh -i ~/.ssh/id_ed25519 <username>@<ip>"
```

Login with alias:

```
da_connect
```

> [!TIP]
> Searching of created aliases with `alias | grep <keyword>`.

> [!IMPORTANT]
> To make the alias avilable for future sessions, they have to be stored in `.zshrc`.

</br>

## 7. Creating identities on local machine for easier login on server

Creating config file, if it doesn't exist:

```
touch ~/.ssh/config
```

Editing the config file and add the identity:

```
nano ~/.ssh/config 
```

Adding the following configuration (correct path to ssh key):

```
Host <ip-adress>
  User <username>
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_ed25519
```

Login with identity:

```
ssh <hostname>
```

> [!TIP]
> Aliases and identities can be combined.

</br>

## 8. Setup of Git and GitHub on server

Creating a SSH-key pair on server:

```
ssh-keygen -t ed25519
```

Copying the public key from created file for GitHub:

```
cat ~/.ssh/git/id_ed25519.pub
```

> [!IMPORTANT]
> New SSH key entry in GitHub account settings with created public key from server is needed!

Configuration of GitHub username and email on server (same as on GitHub):

```
git config --global user.name "<username>"
git config --global user.email "<email>"
```

Check if everything is correct in config:

```
git config --list
```

Creating ssh config file, if it doesn't exists:

```
touch ~/.ssh/config
```

Adding the following identity to the ssh config file (right path to ssh key):

```
Host github.com
  User git
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/git/id_ed25519
```

Testing, if connection to GitHub server works well:

```
ssh -T git@github.com
```

Expected output, if connection works 👍🏻:

```
Hi <username>! You've successfully authenticated, but GitHub does not provide shell access.
```

Cloning of this repository from GitHub:

```
git clone git@github.com:mikemeyer186/vm-cloud-config.git
```

Output, if everything works 👍🏻:

```
Cloning into 'vm-cloud-config'...
remote: Enumerating objects: 36, done.
remote: Counting objects: 100% (36/36), done.
remote: Compressing objects: 100% (32/32), done.
remote: Total 36 (delta 8), reused 11 (delta 1), pack-reused 0
Receiving objects: 100% (36/36), 10.50 KiB | 1.31 MiB/s, done.
Resolving deltas: 100% (8/8), done.
```
