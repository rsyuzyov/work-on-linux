# Контроллер домена на linux

Если необходимо развернуть домен Active Directory с нуля - делаем по официальной инструкции:
https://wiki.samba.org/index.php/Setting_up_Samba_as_an_Active_Directory_Domain_Controller

Если же необходимо поднять дополнительный контроллер в существем лесу, то... тоже делаем по официальной инструкции:
https://wiki.samba.org/index.php/Joining_a_Samba_DC_to_an_Existing_Active_Directory

Поскольку второй вариант более часто востребован, ниже приведена выжимка:  
Если samba будет версии ниже 4.11, то нужно понизить версию схемы с 69 (2012R2) до 47 (2008R2):
```
Import-Module ActiveDirectory
Set-ADForestMode -Identity domain.local -ForestMode Windows2008Forest
Get-ADForest
Set-ADDomainMode -Identity domain.local -DomainMode Windows2008Domain
Get-ADDomain
```
Установка samba, ntp-сервера:  
```
apt install acl attr samba winbind libpam-winbind libnss-winbind krb5-config krb5-user dnsutils python3-setproctitle chrony -y
```
Удаление конфигурации:  
```
rm /etc/samba/smb.conf
```
Ввод в домен как DC:  
```
kinit username
samba-tool domain join domain.local DC
```
Включение службы:  
```
#rm /etc/systemd/system/samba-ad-dc.service && systemctl daemon-reload
systemctl enable samba-ad-dc
reboot
```
Настройка репликации SYSVOL через robocopy: 
На виндовом контроллере создаем задание в планировщике:
```
Program/script:               C:\Windows\System32\Robocopy.exe
Add arguments (optional):     \\DC1\SYSVOL\samdom.example.com\ C:\Windows\SYSVOL\domain\ /mir /sec
```
Настройка обновления записей в DNS через DHCP:
https://wiki.samba.org/index.php/Configure_DHCP_to_update_DNS_records

** Ссылки:
https://wiki.samba.org/index.php/Setting_up_Samba_as_an_Active_Directory_Domain_Controller
https://wiki.samba.org/index.php/Joining_a_Samba_DC_to_an_Existing_Active_Directory
https://it-lux.ru/debian-10-ustanovka-samba-ad/
https://habr.com/ru/post/450572/ на альте
http://www.lissyara.su/articles/freebsd/programms/bind+ad/ BIND как DNS сервер в Active Directory

