# Incident 02 — Wrong file permissions

## Суть

Пользователь `user2` входит в группу `supportteam`, но не может прочитать файл `/opt/support-share/report-02.txt`:

```bash
sudo -u user2 cat /opt/support-share/report-02.txt
```

```text
cat: /opt/support-share/report-02.txt: Permission denied
```

## Причина

Файл принадлежит группе `supportteam`, но у группы нет права на чтение:

```text
-rw------- 1 user1 supportteam /opt/support-share/report-02.txt
```

Права `600` означают:

```text
user1:       rw-
supportteam: ---
others:      ---
```

Поэтому `user2`, несмотря на членство в `supportteam`, не может прочитать файл.

## Исправление

Дать группе `supportteam` право на чтение:

```bash
chmod 640 /opt/support-share/report-02.txt
```

## Проверка

```bash
ls -l /opt/support-share/report-02.txt
sudo -u user2 cat /opt/support-share/report-02.txt
```

После исправления `user2` может прочитать содержимое файла.
