
# Шарим


## 1. Установите пакет samba
```
sudo apt-get update
sudo apt-get install -y
```
![img.png](images/1.png)

## 2. Что такое общая папка, зачем оно может быть нужно?
Общая папка (share) — это каталог на сервере, к которому можно получить доступ по сети с других компьютеров через протокол SMB/CIFS и передавать файлы туда-обратно с заданием прав, групп и т.д.

## 3. Создайте общую папку без пароля с правами только на чтение файлов
```
mkdir -p /srv/samba/public_ro
chown root:root /srv/samba/public_ro
chmod 755 /srv/samba/public_ro
```
Открыаем конфиг samba:
`vi /etc/samba/smb.conf`

Добавляем в конец:
```
[public_ro]
   path = /srv/samba/public_ro
   browseable = yes
   read only = yes
   guest ok = yes
   writable = no
   guest only = yes
```

И перезапускаем:
```
systemctl restart smb
```

## 4. Создайте общую папку с паролем с правами на чтение и запись
Создаем папку:
```
mkdir -p /srv/samba/private_rw
chown sambauser:sambauser /srv/samba/private_rw
chmod 770 /srv/samba/private_rw
```

Создаем юзера:
```
useradd -m -s /bin/bash sambauser
passwd sambauser
smbpasswd -a sambauser
```

Меняем конфиг:
```
[private_rw]
	path = /srv/samba/private_rw
        browseable = yes
        read only = no
        valid users = sambauser
```

Перезагружаем:
`systemctl restart smb`

![img.png](images/3.png)

## 5. Создайте общую папку с доступом для какой-то группы с полными правами
Создадим группу и добавим в неё `sambauser`

```
groupadd sambagroup
usermod -aG sambagroup sambauser
mkdir -p /srv/samba/group_1
chown root:sambagroup /srv/samba/group_1
chmod 2770 /srv/samba/group_1
```

Меняем конфиг:
```
[group_1]
   path = /srv/samba/group_1
   browseable = yes
   read only = no
   guest ok = no
   valid users = @sambagroup
   writable = yes
   force group = sambagroup
   create mask = 0660
   directory mask = 2770
```
И перезагружаем:
`systemctl restart smb`

## 6. Создайте общую папку в которой у одной группы будет полный доступ, а у другой только доступ на чтение. Третья группа не должна иметь к ней доступа
Создаем группы и пользователей для каждой из них:
```
groupadd group_rw
groupadd group_ro
groupadd group_deny

useradd -m -s /bin/bash rwuser
useradd -m -s /bin/bash rouser
useradd -m -s /bin/bash denieduser

passwd rwuser
passwd rouser
passwd denieduser

usermod -aG group_rw rwuser
usermod -aG group_ro rouser
usermod -aG group_deny denieduser

smbpasswd -a rwuser
smbpasswd -a rouser
smbpasswd -a denieduser
```

Создаем общую папку:

```
mkdir -p /srv/samba/three_groups
chown root:group_rw /srv/samba/three_groups
chmod 2770 /srv/samba/three_groups
```

Прописываем в конфиге:
```
[three_groups]
      path = /srv/samba/three_groups
      browseable = yes
      # По умолчанию только чтение через Samba
      read only = yes
      # но группе group_rw разрешим запись
      write list = @group_rw
      # Доступ только двум группам. group_deny не входит
      valid users = @group_rw @group_ro
      force group = group_rw
      create mask = 0664
      directory mask = 2775
```

И перезагружаем:
`systemctl restart smb`

