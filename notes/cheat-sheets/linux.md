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

### Problems adn fixes

- `Permission denied` → смотреть права в `ls -l`; `Operation not permitted` → это может только владелец или root;
- новый файл принадлежит тому, кто его создал: через `sudo` это root;
- группа файла не обязана совпадать с группой владельца (`root adm`);
- ловушки sudoers, которые молчат: `all` строчными (считается именем сервера), точка в имени файла в `sudoers.d` (файл пропускается), команда без аргументов (разрешены любые аргументы);
