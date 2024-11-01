#### Создание дополнительного контроллера в существующем домене
```
apt install acl attr samba winbind libpam-winbind libnss-winbind krb5-config krb5-user dnsutils python3-setproctitle -y
kinit kamushkin
samba-tool domain join bwc.local DC
```

#### Настройка DHCP-сервера
Установка: и настройка:  
```
apt install isc-dhcp-server
nano /etc/default/isc-dhcp-server:
INTERFACES="eth0"

nano /etc/dhcp/dhcpd.conf:
option domain-name "ag.local";
option domain-name-servers 192.168.88.2, 192.168.88.3;

default-lease-time 600;
max-lease-time 7200;
authoritative;
ddns-update-style none;

subnet 192.168.88.0 netmask 255.255.255.0 {
  range 192.168.88.151 192.168.88.180;
  option routers 192.168.88.1;
}
```

#### Настройка авторегистрации в DNS клиентов DHCP
Создание пользователя для работы с DNS
```
kinit kamushkin
samba-tool user create dhcpduser --random-password --description='Unprivileged user for DNS updates via ISC DHCP server'
samba-tool user setexpiry dhcpduser --noexpiry
samba-tool group addmembers DnsAdmins dhcpduser
```

Генерация keytab-файла для пользователя dhcpuser
```
samba-tool domain exportkeytab --principal=dhcpduser /etc/dhcpduser.keytab
#chown XXXX:XXXX /etc/dhcpduser.keytab
#Replace 'XXXX:XXXX' with the user & group that dhcpd runs as on your distro
chmod 400 /etc/dhcpduser.keytab
```
Установка скрипта dhcp-dyndns.sh  
```
...
```

Добавление обработчиков событий выдачи адреса и истечения срока действия  
```
...
```

#### Примеры команд
```
samba-tool dns add srv-dc1 ag.local rsyuzyov A 192.168.88.192 -U syuzyov%pass  
samba-tool dns query srv-dc1 ag.local @ A -U syuzyov%pass  
wbinfo -u все пользователи  
tdbdump - утилита для дампа tdbф-файлов  
```

#### Ссылки
[Создание контроллера нового домена](https://wiki.samba.org/index.php/Setting_up_Samba_as_an_Active_Directory_Domain_Controller)  
[Создание дополнительного контроллера в существующем домене](https://wiki.samba.org/index.php/Joining_a_Samba_DC_to_an_Existing_Active_Directory)  
[Настройка авторегистрации в DNS клиентов DHCP](https://wiki.samba.org/index.php/Configure_DHCP_to_update_DNS_records)
[Администрирование DNS](https://wiki.samba.org/index.php/DNS_Administration#Administering_DNS_on_Windows)  
[Ввод в домен рабочей станции](https://wiki.samba.org/index.php/Setting_up_Samba_as_a_Domain_Member)

