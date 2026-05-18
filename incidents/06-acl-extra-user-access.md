# Incident 06 — ACL extra user access

## Суть

Пользователь `auditor` не входит в группу `supportteam`, но ему нужно дать доступ на чтение только к одному файлу:

```bash
sudo -u auditor cat /opt/support-share/report-06.txt
```

До настройки ACL доступ запрещён:

```text
cat: /opt/support-share/report-06.txt: Permission denied
```

## Причина

Директория `/opt/support-share` закрыта для `others`:

```text
drwxrws--T 2 root supportteam /opt/support-share
```

Файл также закрыт для `others`:

```text
-rw-rw---- 1 user1 supportteam /opt/support-share/report-06.txt
```

Пользователь `auditor` не входит в группу `supportteam`, поэтому обычные права `owner/group/others` не дают ему доступ.

## Исправление

Дать `auditor` право прохода к директории:

```bash
setfacl -m u:auditor:--x /opt/support-share
```

Дать `auditor` право чтения конкретного файла:

```bash
setfacl -m u:auditor:r-- /opt/support-share/report-06.txt
```

## Проверка

```bash
getfacl /opt/support-share
getfacl /opt/support-share/report-06.txt
sudo -u auditor cat /opt/support-share/report-06.txt
```

После настройки ACL пользователь `auditor` может прочитать конкретный файл, но не получает доступ ко всей группе `supportteam`.
