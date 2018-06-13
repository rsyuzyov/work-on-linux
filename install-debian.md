План  
Делаем базовый образ ВМ: диск 20G, 1vCPU, память 1G. При установке:  
- руссккий язык  
- lvm, все на один диск  
- без рабочего стола  
- сервер ssh, системные утилиты  

Создать файл ответов  

Добавить contib, non-free  
Добавить backports  
update && dist-upgrade  

install sudo, mc, htop, iotop, iftop  

Если нужна графика: стаим на венду xming, в putty для соединения указываем настройки х11: localhost:0.0 [(инструкция)](http://www.geo.mtu.edu/geoschem/docs/putty_install.html)  
Если нужна графика из-под sudo: в sudoers: Defaults env_keep="XAUTHORIZATION XAUTHORITY TZ PS2 PS1 PATH LS_COLORS KRB5CCNAME HOSTNAME HOME DISPLAY COLORS"  [(инструкция)](https://askubuntu.com/questions/414785/cant-open-gedit-as-root)  

Делаем копию диска  

Дальше hostname, static ip, поднятие ролей  

плейбуки надо или не выносить моск?
