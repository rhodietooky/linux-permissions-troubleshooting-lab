# Incident 04 — Wrong owner or group

## Суть

Пользователь `user1` входит в группу `supportteam`, но не может открыть директорию `/opt/support-share`:

```bash
sudo -u user1 ls /opt/support-share
```

```text
ls: cannot open directory '/opt/support-share': Permission denied
```

## Причина

У директории неправильная группа:

```text
drwxrws--T 2 root root /opt/support-share
```

Пользователь `user1` входит в `supportteam`, но директория принадлежит группе `root`.

Поэтому права группы применяются к группе `root`, а для `user1` фактически работают права `others`, у которых доступа нет.

## Исправление

Вернуть директории правильную группу:

```bash
chown root:supportteam /opt/support-share
```

## Проверка

```bash
ls -ld /opt/support-share
sudo -u user1 ls /opt/support-share
```

После исправления `user1` снова получает доступ к директории.
