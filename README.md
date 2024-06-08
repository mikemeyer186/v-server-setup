# v-server-setup

Procedure to configure a vm-cloud on a server with zsh/bash:

- creating ssh-key pair
- deactivating password login
- login with a SSH-key only
- installing nginx
- installing alternativ index file
- creating aliases and identities
- setting up Git and GitHub
  
</br>

## 1. Create SSH-key pair on local machine

> [!NOTE]
> The ED25519 key pair is a modern alternative to RSA keys and should be used. 

### Create key

    ssh-keygen -t ed25519

### Define path to store the key

    ~/.ssh/id_ed25519

### Check if key pair is stored in defined path

    ls ~/.ssh

</br>

## 2. First login on V-Server from local machine

### Login with ssh command

    ssh <username>@<ip>

### Confirm the question with "yes" to use the new fingerprint and enter server password
    Are you sure you want to continue connecting (yes/no/[fingerprint])? yes

### Output on successful login 👍🏻
    Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-107-generic x86_64) ...

</br>

## 3. Add the public key to the VM from local maschine

> [!IMPORTANT]
> To copy the public key to the vm use the programm `ssh-copy-id` on local terminal:

    ssh-copy-id -i ~/.ssh/id_ed25519.pub username@ip

### Output on successfully adding the public key on vm 👍🏻
```
Number of key(s) added: 1
Now try logging into the machine, with: "ssh 'username@ip'"
and check to make sure that only the key(s) you wanted were added.
```

### Try to login with ssh key without password

    ssh -i ~/.ssh/id_ed25519 <username>@<ip>

### Check storage path of key on server

    ls -al ~/.ssh

### Show content of file `authorized_keys`

> [!NOTE]
> The public key must be stored in this file

    cat ~/.ssh/authorized_keys

</br>

## 4. Disable password-login on server

> [!IMPORTANT]
> Disable the password-login on V-Server, since it can be unsafe. The SSH-key pair is a safe authorization method.

> [!CAUTION]
> Before deactivating password login, make sure that the login on server works well with password

### Edit the `sshd-config` with nano 

    sudo nano /etc/ssh/sshd_config

### Remove the hastag before `PasswordAuthentication` and set the value to `no`

```
...
PasswordAuthentication no
...
```

### Restart ssh.service after changes are done

    sudo systemctl restart ssh.service

### Check if config on server works

First check (login works ✅):

    ssh -i ~/.ssh/id_ed25519 <username>@<ip>

Double check (permission denied ❌):

    ssh -o PubkeyAuthentication=no <username>@<ip>

Expected output, if everything works well 👍🏻:

    Permission denied (publickey).

</br>

## 4. Installing of nginx on server

### First update the packetmanager `apt` and then install `nginx`

    sudo apt update
    sudo apt install nginx -y

### Check status of `nginx.service`

    systemctl status nginx.service     

### Check if nginx webserver is active by connecting to ip with webbrowser

    http://<ip-adress>

![nginx](https://github.com/mikemeyer186/vm-cloud-config/assets/112903209/d7be6337-9e82-4a38-9eec-379986804bca)

</br>

## 5. Installing alternative html content on webserver

### Check the default html content in `index.nginx-debian.html`

    cat /var/www/html/index.nginx-debian.html

### Create a directory with the command `mkdir` in `/var/www/`

    sudo mkdir /var/www/alternatives

### Create a html file in this directory

    sudo touch /var/www/alternatives/alternate-index.html

### Check if the new html file is created correctly

    ls /var/www/alternatives

 Expected output, if everything works well 👍🏻:

    alternate-index.html

### Configure the nginx settings

1. Create the file:

```
sudo nano /etc/nginx/sites-enabled/alternatives
```

2. Paste the follwing code in the file:

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

### Create the alternative html file

    sudo nano /var/www/alternatives/alternate-index.html

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

### Testing of configuration (default `nginx.conf`)

    sudo nginx -t


### Restart nginx and check if everything is running

```
sudo service nginx restart
systemctl status nginx.service
```

### Check if the webserver can be connected with webbrowser

    http://<ip-adress>:8081

> [!NOTE]
> Trying to connect to a non existing page should show 404 error (default nginx)

</br>

## 6. Creating aliases on local machine for easier login on server

### Create an new alias

    alias da_connect="ssh -i ~/.ssh/id_ed25519 <username>@<ip>"

### Login with alias

    da_connect

> [!TIP]
> To search for created aliases use `alias | grep <keyword>`

> [!IMPORTANT]
> To make the alias avilable for future sessions, store it in `.zshrc`

</br>

## 7. Creating identity keys on local machine for easier login on server

### Create config file, if it doesn't exists

    touch ~/.ssh/config

### Edit the config file and add the identity key

    nano ~/.ssh/config 

Paste the following config (replace the right path to ssh key):
```
Host <ip-adress>
  User <username>
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_ed25519
```

### Login with identity key

    ssh <hostname>

> [!TIP]
> Aliases and identities can be combined

</br>

## 8. Setup of Git and GitHub on server

### Create a SSH-key pair on server

    ssh-keygen -t ed25519

### Copy public key from created file

    cat ~/.ssh/git/id_ed25519.pub

> [!IMPORTANT]
> Create a new SSH key entry in github account settings and paste the public key

### Configure github username and email on server

    git config --global user.name "<username>"
    git config --global user.email "<email address>"

### Check if everything is correct in config

    git config --list

### Clone this repository from GitHub

    git clone git@github.com:mikemeyer186/vm-cloud-config.git

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
