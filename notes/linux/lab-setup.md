# lab01 setup

## VM
- VMware Workstation 17, Ubuntu Server;
- CPU 2 core / RAM 4 GB/ disk 32 GB/ network Bridge;
- sudo apt update && sudo apt upgrade -y;

## SSH keys
- создал ключ PSh: `ssh-keygen -t ed25519`;
- создал на Linux привило для профиля: 
    `mkdir -p ~/.ssh
    chmod 700 ~/.ssh
    chmod 600 ~/.ssh/authorized_keys`
- закинул ключ на ВМ с Windows: `Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub` -> `sudo nano ~/.ssh/authorized_keys`;
- ssh config на Windows: 
Host lab01
    HostName x.x.x.x
    User x
    IdentityFile ~/.ssh/id_ed25519 
    создал файл конфиг чтобы заходить на вм просто по имени ssh lab01;

## SSH hardening
- sudo nano /etc/ssh/sshd_config.d/00-hardening.conf
  ```
  PasswordAuthentication no
  PermitRootLogin no
  ```
  ctrl-o, ctrl-x
  ```
  Проверить: `sudo sshd -t`;
  Применить: `sudo systemctl restart ssh`
  вход только по ключу и запрет входа под root;
- Проверка с Windows: `ssh -o PubkeyAuthentication=no lab01` → должно быть `Permission denied (publickey)`
## Updates, timezone
- `timedatectl set-timezone Asia/Tashkent`;
## Snapshots
- создал clean-base для отката в самое начало;
## Problems and fixes
- ловушка с 50-cloud-init.conf: не сработало включение параметра "PermitRootLogin no" в файле по пути `sudo nano /etc/ssh/sshd_config`;
