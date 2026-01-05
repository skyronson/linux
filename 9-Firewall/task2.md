# Открываем firewald

## 1. Удалите iptables и установите firewalld
```
sudo /sbin/service iptables stop
sudo chkconfig iptables off
sudo apt-get install firewalld -y
sudo systemctl enable --now firewalld
```
## 2. Попробуйте так-же проверить возможность подключения по ssh
![img.png](images/1.png)

Возможность осталась.

## 3. Если её нет то откройте порт
![img.png](images/2.png)

## 4. Выведите список открытых портов с помощью firewall-cmd
![img.png](images/3.png)

## 5. Можно ли там добавить порты по названию сервиса?
Да, можно добавлять сервисы по названию, например ssh, http, samba.

`firewall-cmd --add-service=ssh`

## 6. На вашей Локальной виртуальной машине попробуйте подключиться к серверу samba из предыдущих заданий
![img.png](images/4.png)

## 7. Если не получилось то откройте нужные порты

## 9. Сделайте так чтобы изменения были постоянными
![img.png](images/5.png)

