# Incident 03 — Wrong directory permissions

## Суть

Пользователь `user1` входит в группу `supportteam`, но не может зайти в директорию `/opt/support-share`:

```bash
sudo -u user1 bash -c 'cd /opt/support-share'
```

```text
bash: line 1: cd: /opt/support-share: Permission denied
```

## Причина

У директории отсутствует execute-бит `x`:

```text
drw-rwS--- 2 root supportteam /opt/support-share
```

Большая `S` означает, что setgid-бит установлен, но execute-бит для группы отсутствует.

Для директории `x` означает право прохода внутрь директории. Без `x` пользователь не может зайти в директорию и обращаться к объектам внутри неё.

## Исправление

Вернуть корректные права директории:

```bash
chmod 3770 /opt/support-share
```

## Проверка

```bash
ls -ld /opt/support-share
sudo -u user1 bash -c 'cd /opt/support-share && pwd'
```

После исправления `user1` снова может зайти в директорию.
