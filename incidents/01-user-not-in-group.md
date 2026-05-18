# Incident 01 — User is not in required group

## Суть

Пользователь `user3` не может открыть общую директорию `/opt/support-share`:

```bash
sudo -u user3 ls /opt/support-share
```

```text
ls: cannot open directory '/opt/support-share': Permission denied
```

## Причина

Директория `/opt/support-share` доступна только владельцу `root` и группе `supportteam`:

```text
drwxrws--T 2 root supportteam /opt/support-share
```

Пользователь `user3` не состоит в группе `supportteam`:

```text
uid=1005(user3) gid=1005(user3) groups=1005(user3)
```

Поэтому для него применяются права `others`, а у `others` доступа нет.

## Исправление

Добавить пользователя `user3` в группу `supportteam`:

```bash
sudo usermod -aG supportteam user3
```

## Проверка

```bash
id user3
sudo -u user3 ls /opt/support-share
```

После добавления в группу `supportteam` пользователь получает доступ к директории.
