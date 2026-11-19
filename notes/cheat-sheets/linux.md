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
