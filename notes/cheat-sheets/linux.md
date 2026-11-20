# Linux cheat sheet

Неделя 2 - «жить в консоли». Формат: задача → команда → пометка.

## Day 2 - users & permissions

### Users and groups

- кто я и в каких группах: `id`;
- то же про другого пользователя: `id deploy`, `groups deploy`;
- создать пользователя без пароля: `sudo adduser --disabled-password deploy`;
- где записаны пользователи и группы: `/etc/passwd`, `/etc/group`;
- откуда берется содержимое новой домашней папки: `/etc/skel`

### Permissions

- права файла: `ls -l файл`;
- права папки: `ls -ld папка`;
- со скрытыми файлами: `ls -la`;
- сменить владельца и группу: `sudo chown deploy:deploy файл`;
- сменить права: `chmod 700 ~/.ssh`

### sudo

- что мне можно через sudo: `sudo -l`;
- создать правило только через: `sudo visudo -f /etc/sudoers.d/deploy`

```console
deploy ALL=(root) NOPASSWD: /usr/bin/apt update
```

### Problems adn fixes

- `Permission denied` → смотреть права в `ls -l`; `Operation not permitted` → это может только владелец или root;
- новый файл принадлежит тому, кто его создал: через `sudo` это root;
- группа файла не обязана совпадать с группой владельца (`root adm`);
- ловушки sudoers, которые молчат: `all` строчными (считается именем сервера), точка в имени файла в `sudoers.d` (файл пропускается), команда без аргументов (разрешены любые аргументы);

## Day 3 - processes & systemd

### Processes
- все процессы: `ps aux`
- только первые строки: `ps aux | head -n 5`;
- процессы пользователя: `ps -u deploy`;
- кто родитель: `ps -f` → в колонке `PPID` номер (`PID`) родителя;
- мой терминал: `tty`; в колонке `TTY`:`pts/N` - ssh-сессия, `?` - без терминала (службы);
- завершить процесс: `sudo kill <PID>` - сигнал TERM, вежливая просьба выйти;
- живой монитор: `htop` - hot keys → `F5` дерево, `Shift+M` по памяти, `Shift+H` потоки, `q` выход;

### Services (systemd)
- состояние службы: `systemctl status nginx`;
- что сейчас работает: `systemctl list-units --type=service --state=running`;
- остановить / запустить / перезапустить: `sudo systemctl stop nginx, start, restart`;
- автозапуск при запуске: `sudo systemctl enable heartbeat`;
- показать юнит-файл: `systemctl cat heartbeat`;
- перечитать юнит-файлы после правки: `sudo systemctl daemon-reload`;
- журнал службы: `journalctl -u heartbeat -n 20` - последние 20 строк, `-f` - следить вживую;
- перезагрузить сервер: `sudo reboot`;

### My serice: heartbeat
- скрипт `/usr/local/bin/heartbeat.sh`, права `sudo chmod 755 /usr/local/bin/heartbeat.sh`

```console
#!/bin/bash
while true; do echo "hello world"; sleep 60; done
```
- юнит `/etc/systemd/system/heartbeat.service`:
```console
[Unit]
Description=Hello World Cycle

[Service]
ExecStart=/usr/local/bin/heartbeat.sh

[Install]
WantedBy=multi-user.target
```
- запуск: `sudo systemctl daemon-reload`→`sudo systemctl enable heartbeat`→`sudo systemctl start heratbeat`;

### Problems and Fixes
- поправил `.service`→`daemon-relaod`, потом `restart`; поправил скрипт → только: `restart heartbeat.service`; если забыл - systemctl напомнит `change on disk`;
- `enabled` / `disabled` - автозапуск, `active` / `inactive` - работает ли сейчас: два разных переключателя;
- запуск по пути (`usr/local/bin/heartbeat.sh) требует права `x` даже у root, для `bash` хватает `r`;
- `kill` шлёт сигнал процессу по PID, а службой управляет `systemctl`;
- проверить статус службы, которую менял;
- в `htop` память смотреть в `RES`, а не в `VIRT`; зеленые строки - потоки, их память не складывать;
- `%CPU` в `ps` - среднее за жизнь процесса, в `htop` - прямо сейчас;
