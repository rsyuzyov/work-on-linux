Два хороших варианта: openvpn и wireguard.

Установка openvpn описана [здесь](https://pve.proxmox.com/wiki/OpenVPN_in_LXC)

Установка wireguard на debian 12:

```
echo 'deb http://deb.debian.org/debian bookworm-backports main contrib non-free non-free-firmware' > /etc/apt/sources.list.d/bookworm-backports.list
apt update
apt install wireguard wireguard-tools qrencode -y
apt install iptables -y
cd /etc/wireguard
wget https://raw.githubusercontent.com/burghardt/easy-wg-quick/master/easy-wg-quick
chmod +x ./easy-wg-quick
systemctl enable wg-quick@wghub
# initialization when adding first user
./easy-wg-quick firstuser
systemctl start wg-quick@wghub
```

Добавление пользователя:

```
./easy-wg-quick username-device
systemctl restart wg-quick@wghub
```

Передать настройку клиенту:  
Скопипастить содержимое файла, либо передать файл целиком:
/etc/wireguard/username-device.conf
Либо открыть /etc/wireguard/username-device.qrcode.txt и дать сфоткать пользователю через клиента

Удалить пользоателя:

```
rm wgclient_username_device\*
/etc/wireguard/wghub.conf: удалить раздел с данними пользователя
systemctl restart wg-quick@wghub
```

Аудит входов/выходов:  
https://stacktuts.com/how-to-log-network-activity-in-wireguard

WG UI:  
https://github.com/ngoduykhanh/wireguard-ui
Это UI несовместим с easy-wg-quick, поэтому либо то, либо другое.
По большому счету ui нафиг надо

=====================================  
Заметки

Сервер WG под Windows:  
https://habr.com/ru/articles/568450/  
Установить https://www.wiresock.net/sdc_download/921/?key=bekfpuvidq5x8ofg2u74j2lqk8zy9y  
Далее wg-quick-config -add -start

**Полезные ссылки:  
https://habr.com/ru/articles/527404/
https://habr.com/ru/articles/738890/  
