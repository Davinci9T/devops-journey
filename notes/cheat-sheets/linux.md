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

## Day 4 - network

### Addresses and routes
- адреса интерфейсов: `ip a` - `inet x.x.x.x/24`, `dynamic` значит адрес от DHCP ;
- маршруты: `ip r` - `default via` это шлюз;
- DNS-серверы: `resolvectl dns`, сводка по интерфейсу: `networkctl status ens33 --no-pager`;
- есть ли связь: `ping -c 4 x.x.x.x` - `-c` сколько пакетов, без него бесконечно;
- спросить DNS: `dig ubuntu.com` - `status: NOERROR`, адрес в `ANSWER SECTION`;

### Ports
- кто слушает порты: `sudo ss -tulpn` → `t` TCP, `u` UDP, `l` слушающие, `p` процесс, `n` номера цифрами;
- `127.0.0.1` и `[::1]` - только с самого сервера, `0.0.0.0` и `[::]` - со всех адресов;
- проверить веб-сервер на самом сервере: `curl localhost`;
- проверить порт с Windows: `Test-NetConnection x.x.x.x -Port 80`→`TcpSucceeded: True`;

### nginx
- поставить: `sudo apt install nginx`;
- процессы: `ps -f -C nginx` - мастер от root, воркеры от `www-data`;

### Firewall: ufw
- поставить: `sudo apt install ufw`;
- сначала пустить ssh, потом включать: `sudo ufw allow 22/tcp` → `sudo ufw enable`;
-  открыть веб: `sudo ufw allow 80/tcp`;
-  посмотреть правила: `sudo ufw status`, подробно: `sudo ufw status verbose`;
-  после `enable` проверить новым входом во втором окне: текущая сессия может обмануть;

### Problems and Fixes
- служба на `127.0.0.1` снаружи недоступна, даже если порт открыт в ufw. Нужны две двери: служба слушает не только на петле, и файрвол пропускает;
- «слушает» видно в `ss` на сервере, «доступен» - только снаружи (Test-NetConnetction);
- `ping` ходит без портов: пинг есть, а порт может быть закрыт;
- отказ сразу - на порту никто не слушает, долго думает - пакеты режет файрвол;
- TTL в `ping` - сколько маршрутизаторов пакет ещё пройдёт, TTL в `dig` - сколько секунд ответ можно держать в кэше;

## Day 5 - Packages & Disks
### Packages: apt
- обновить списки пакетов: `sudo apt update` - `Hit` не изменилось, `Get` скачал новое;
- что можно обновить: `apt list --upgradable`;
- обновить: `sudo apt upgrade` - сначала прочитать сводку, потом `Y`;
- карточка пакета: `apt show nginx`, выход из просмотра - `q`;
- какая версия стоит и какая доступна: `apt policy nginx` - `Installed` и `Candidate`;
- откуда apt берет пакеты: `cat /etc/apt/sources.list.d/ubuntu/sources`;

### Space, Disks, Memory
- место на всех файловых системах: `df -h`, в одной: `df -h /data`;
- что сколько занимает: `sudo du  -h -d 1 -x`;
- файлы в папке с размерами: `ls -lh /`;
- диски, разделы и тома деревом: lsblk;
- пул LVM и свободное место в нем: `sudo vgs` - колонка `VFree`;
- память и подкачка: `free -h` - смотреть на `available`;

### Search and Configs
- найти слово во всех файлах папки, с номерами строк: `grep -rn listen /etc/nginx/`;
- открыть файл сразу на нужной строке: `sudo nano +22 (файл)`;
- проверить конфиг nginx: `sudo nginx -t`;
- сообщения ядра с этой загрузки: `journalctl -k -b`;
- искать сразу несколько слов: `journalctl -k -b | grep -i -e "memory": -e crashkernel`;

### New Disk → /data
1. найти диск: `lsblk` → новый `sdb` без разделов;
2. разместить: `sudo fdisk /dev/sdb` → `g` (таблица GPT), `n` (раздел, три раза Enter), `p`(проверить), `w` (записать);
3. файловая система: `sudo mkfs.ext4 /dev/sbd1` → записать `Filesystem UUID`;
4. подключить руками: `sudo mkdir /data` → `sudo mount /dev/sdb1/data` → `df -h /data`;
5. навсегда: `sudo nano /etc/fstab`, в конец строка:

   ```console
   UUID=<UUID> /data ext4 defaults 0 2
   ```
6. проверить до перезагрузки: `sudo findmnt --verify` → `sudo systemctl daemon-reaload` → `sudo mount -a`;
7. перезагрузить: `sudo reboot` → `df -h /data`, `systemctl status data.mount`;
- отключить: `sudo unmount /data` (если внутри `/data` - `target is busy`);

### fstab: 6 fields

```console
<что подключать>    <куда>    <тип>  <параметры>  <dump>  <pass>
UUID=<UUID>          /data    ext4    defaults      0      2
```
- `pass`: 1 - только корень, 2 - остальные, 0 - не проверять;
- UUID, а не `/путь`: имена `sdX` могут съехать, если добавить диск;

### Problems and Fixes
- сначала `nginx -t`, потом `restart`. `-t` проверяет синтаксис, а не смысл: результат смотреть в `ss`;
- `apt policy` и `apt list --upgradable` не ходят в интернет: сначала `sudo apt update`;
- `Not upgading yet due to phasing` - обновление раскрывают волнами, просто "подождать" когда дойдёт очередь;
- `du -d 1` показывает только папки: если сумма не сходится с итогом, искать большой файл через `ls -lh`;
- `mount` поверх непустой папки прячет её файлы до `umount`, но не удаляет;
- в fstab - UUID своей файловой системы: чужой UUID подключит не тот раздел;
- после правки fstab - `daemon-reload`: systemd делает из строк fstab юниты .mount;
- корень 15ГБ из 32: установщик отдал корню половину пула LVM, остальное свободно (`sudo vgs`);

## LVM, аварийная загрузка, журнал (неделя 2, выходные)

| Задача | Команда |
|---|---|
| Сколько свободно в пуле LVM | `sudo vgs` (колонка `VFree`) |
| Добавить корню 5 ГБ вместе с файловой системой | `sudo lvextend -r -L +5G /dev/ubuntu-vg/ubuntu-lv` |
| Диски и разделы с типом ФС и UUID | `lsblk -f` |
| Только ошибки журнала текущей загрузки | `journalctl -xb -p err --no-pager` |
| Журнал службы за отрезок времени | `journalctl -u ssh -S '2026-11-23 01:00' -U '2026-11-23 01:30'` |
