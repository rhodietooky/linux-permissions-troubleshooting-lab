# Linux Permissions Troubleshooting Lab

Учебный проект по диагностике прав доступа в Linux.

Проект моделирует типовые ситуации, с которыми может столкнуться инженер при работе с пользователями, группами, файлами и директориями.

## Цель проекта

Отработать на практике:

- пользователей и группы Linux;
- `chmod`;
- `chown`;
- `chgrp`;
- `ls -l`;
- `ls -ld`;
- `id`;
- `usermod`;
- `gpasswd`;
- права `r/w/x` на файлы и директории;
- диагностику ошибки `Permission denied`;
- setgid на общей директории;
- sticky bit;
- базовый ACL через `setfacl` и `getfacl`.

## Лабораторная схема

Группа:

```bash
supportteam
```

Пользователи:

```text
user1   — участник supportteam
user2   — участник supportteam
user3   — не входит в supportteam
auditor — отдельный пользователь для ACL-инцидента
```

Общая директория:

```bash
/opt/support-share
```

Базовая настройка директории:

```bash
sudo chown root:supportteam /opt/support-share
sudo chmod 3770 /opt/support-share
```

Права директории:

```bash
drwxrws--T 2 root supportteam 4096 May 17 10:21 /opt/support-share
```

Расшифровка:

```text
root:        rwx
supportteam: rwx + setgid
others:      ---
sticky bit:  включён
```

## Подготовка стенда

Создание группы:

```bash
sudo groupadd supportteam
```

Создание пользователей:

```bash
sudo useradd -m -s /bin/bash user1
sudo useradd -m -s /bin/bash user2
sudo useradd -m -s /bin/bash user3
sudo useradd -m -s /bin/bash auditor
```

Добавление `user1` и `user2` в группу `supportteam`:

```bash
sudo usermod -aG supportteam user1
sudo usermod -aG supportteam user2
```

Создание общей директории:

```bash
sudo mkdir -p /opt/support-share
sudo chown root:supportteam /opt/support-share
sudo chmod 3770 /opt/support-share
```

Проверка:

```bash
id user1
id user2
id user3
id auditor
ls -ld /opt/support-share
```

## Инциденты

### 01 — User is not in required group

Пользователь `user3` не входит в группу `supportteam` и получает `Permission denied` при попытке открыть `/opt/support-share`.

Файл:

```text
incidents/01-user-not-in-group.md
```

### 02 — Wrong file permissions

Пользователь `user2` входит в `supportteam`, но не может прочитать файл, потому что у группы нет права `r`.

Файл:

```text
incidents/02-wrong-file-permissions.md
```

### 03 — Wrong directory permissions

У директории отсутствует execute-бит `x`, из-за чего пользователь не может зайти в директорию, даже если входит в нужную группу.

Файл:

```text
incidents/03-wrong-directory-permissions.md
```

### 04 — Wrong owner or group

Директория имеет группу `root` вместо `supportteam`, поэтому пользователи из `supportteam` теряют доступ.

Файл:

```text
incidents/04-wrong-owner-or-group.md
```

### 05 — Missing setgid on shared directory

На общей директории отсутствует setgid-бит, поэтому новые файлы создаются с группой пользователя, а не с группой `supportteam`.

Файл:

```text
incidents/05-missing-setgid-on-shared-directory.md
```

### 06 — ACL extra user access

Пользователь `auditor` не входит в `supportteam`, но получает доступ к одному конкретному файлу через ACL без добавления в основную группу.

Файл:

```text
incidents/06-acl-extra-user-access.md
```

## Используемые команды

Основные команды проекта:

```bash
id
groups
ls -l
ls -ld
chmod
chown
chgrp
groupadd
useradd
usermod
gpasswd
setfacl
getfacl
```

## Ключевые выводы

### Права на файл

Для файла:

```text
r = прочитать содержимое файла
w = изменить содержимое файла
x = запустить файл как программу или скрипт
```

### Права на директорию

Для директории:

```text
r = посмотреть список файлов внутри директории
w = создавать, удалять и переименовывать файлы внутри директории
x = зайти в директорию и обращаться к объектам внутри неё
```

Важный момент: для директории `x` означает право прохода внутрь директории.

### Удаление файла

Удаление файла зависит не только от прав на сам файл, а от прав на родительскую директорию.

Чтобы удалить файл, пользователю нужны права `w+x` на директорию.

### setgid

`setgid` на директории нужен, чтобы новые файлы внутри наследовали группу директории.

Пример:

```bash
drwxrws--- root supportteam /opt/support-share
```

Файл, созданный внутри такой директории пользователем `user1`, получит группу `supportteam`.

### sticky bit

`sticky bit` на директории нужен, чтобы пользователи не могли удалять чужие файлы в общей директории.

Пример:

```bash
drwxrws--T root supportteam /opt/support-share
```

В такой директории пользователь может удалить свой файл, но не может удалить файл другого пользователя.

### ACL

ACL позволяет выдать доступ отдельному пользователю или группе без изменения базовой модели `owner/group/others`.

Пример:

```bash
setfacl -m u:auditor:--x /opt/support-share
setfacl -m u:auditor:r-- /opt/support-share/report-06.txt
```

Проверка:

```bash
getfacl /opt/support-share
getfacl /opt/support-share/report-06.txt
```

## Практический результат

В проекте отработаны типовые проблемы с правами доступа в Linux:

- пользователь не состоит в нужной группе;
- файл имеет неправильные права;
- директория имеет неправильные права;
- директория принадлежит неправильной группе;
- отсутствует setgid на общей директории;
- нужен точечный доступ через ACL.
