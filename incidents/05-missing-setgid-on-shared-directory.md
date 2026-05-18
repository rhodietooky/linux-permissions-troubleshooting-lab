# Incident 05 — Missing setgid on shared directory

## Суть

Новый файл в общей директории `/opt/support-share` создаётся с группой пользователя `user1`, а не с группой `supportteam`:

```text
-rw-rw---- 1 user1 user1 /opt/support-share/report-05.txt
```

Из-за этого `user2`, хотя и входит в `supportteam`, не может прочитать файл:

```bash
sudo -u user2 cat /opt/support-share/report-05.txt
```

```text
cat: /opt/support-share/report-05.txt: Permission denied
```

## Причина

На директории отсутствует setgid-бит:

```text
drwxrwx--T 2 root supportteam /opt/support-share
```

Без setgid новые файлы наследуют основную группу пользователя, который их создал.

В данном случае файл получил группу `user1`, а не `supportteam`.

## Исправление

Вернуть setgid на директорию:

```bash
chmod g+s /opt/support-share
```

Для уже созданного файла отдельно исправить группу:

```bash
chgrp supportteam /opt/support-share/report-05.txt
```

## Проверка

```bash
ls -ld /opt/support-share
ls -l /opt/support-share/report-05.txt
sudo -u user2 cat /opt/support-share/report-05.txt
```

После исправления файл принадлежит группе `supportteam`, и `user2` может его прочитать.
