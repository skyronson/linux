# Пишем юниты

1. Создайте скрипт который создаёт папку заполняет её файлами ( имена 1-4 ) и записывает в них информацию
о текущей дате, версии ядра, имени компьютера и списке всех файлов в домашнем каталоге пользователя от которого выполняется скрипт (не забудьте сдлеать проверку на существование файлов и папок)
```bash

#!/bin/bash

set -euo pipefail

cd "$HOME"

WORK_DIR="$HOME/unit_test"

# Создаём папку, если не существует
mkdir -p "$WORK_DIR"

for i in 1 2 3 4; do
[ -f "$WORK_DIR/$i" ] || touch "$WORK_DIR/$i"
done

# 1 — текущая дата
date > "$WORK_DIR/1"
# 2 — версия ядра
uname -r > "$WORK_DIR/2"
# 3 — имя компьютера
hostname > "$WORK_DIR/3"
# 4 — список всех файлов в домашнем каталоге пользователя
find "$HOME" -maxdepth 1 -mindepth 1 -printf '%f\n' > "$WORK_DIR/4"
```
Делаем исполняемым:<br>
`chmod +x ~/script_2.sh`

![img.png](images/1.png)

2. Создайте юнит который будет вызывать этот скрипт при запуске. Проверьте

Файл юнита: `/etc/systemd/system/unit_test.service`
```
[Unit]
Description=Generate system report
[Service]
Type=oneshot
ExecStart=/usr/local/bin/unit_test.sh
[Install]
WantedBy=multi-user.target
```

Активация и проверка
```
sudo systemctl daemon-reload
sudo systemctl start unit-task.service
sudo systemctl status unit-task.service
```

3. Создайте таймер который будет вызывать выполнение одноимённого systemd юнита каждые 5 минут.

Файл таймера: `/etc/systemd/system/unit_test.timer`
```
[Unit]
Description=Run report every 5 minutes

[Timer]
OnBootSec=1min
OnUnitActiveSec=5min
Unit=unit_test.service
Persistent=true

[Install]
WantedBy=timers.target
```

Включим таймер:
```
sudo systemctl daemon-reload
sudo systemctl enable --now unit_test.timer
sudo systemctl list-timers unit_test.timer
```

4. От какого пользователя вызыаются юниты поумолчанию?

Для системных юнитов (/etc/systemd/system, /usr/lib/systemd/system) по умолчанию процессы запускаются от root, если в юните явно не указан флаг User

5. Создайте пользователя от имени которого будет выполняться ваш скрипт.

`sudo useradd -m -s /bin/bash unituser`
Скопировать скрипт этому пользователю:<br>
```
sudo cp ~/unit_task.sh /home/unituser/unit_test.sh
sudo chown unituser:unituser /home/unituser/unit_test.sh
sudo chmod +x /home/unituser/unit_test.sh
```

6. Дополните юнит информацией о пользователе от которого должен выплняться скрипт.

Обновленный файл юнита:
```
[Unit]
Description=Generate system report
[Service]
Type=oneshot
ExecStart=/usr/local/bin/unit_test.sh
User=unituser
Group=unituser
WorkingDirectory=/home/unituser
[Install]
WantedBy=multi-user.target
```

7. Дополните ваш скрипт так, что бы он независимо от местоположения всега выполнялся в домашней папке того кто его вызывает.

```
cd "$HOME"
WORKDIR="$HOME/unit_files"
```
$HOME всегда указывает на домашний каталог пользователя, под которым запущен процесс, поэтому скрипт работает в нужной домашней директории независимо от того, из какого каталога он был вызван.