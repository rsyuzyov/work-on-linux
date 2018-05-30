Apache httpd - веб-сервер, маленький, шустрый, простой и кроссплатформенный. Нафига нам IIS?

### Установка  
**debian:**  
```
sudo apt install apache2
```
**windows:**  
```
choco install apache-httpd --params "/installLocation:C:\Progra~1 /port:80 /serviceName:Apache" -y
```
Если порт 80 или 443 заняты, в процессе установки получм сообщение о невозможности открытия порта при запуске службы.  
В этом случае либо освобождаем порты (весьма вероятно, что уже установлен IIS и его нужно снести), либо меняем порты на нестандартные: в c:\Program Files\Apache24\conf\httpd.conf меняем  
```
Listen 80
```
на
```
Listen 5080
```
Для https: в C:\Program Files\Apache24\conf\extra\httpd-ahssl.conf меняем  
```
Listen 443
```
на
```
Listen 5443
```
Либо можно просто отключить ssl, поставив # перед строкой в c:\Program Files\Apache24\httpd.conf:
```
LoadModule ssl_module modules/mod_ssl.so
```


### Настройка
Публикация средствами 1С - для слабаков, публикуем вручную:

Добавляем в /etc/apache2/apache2.conf (C:\Program Files\Apache24\conf\httpd.conf):
```
```
Создаем vrd-файл следующего содержания и кладем в ...:
```
```

Перезапускаем apache:  
```
sudo systemctl restart apache2
```
или 
```
net stop Apache; net start Apache
```


### Авторизация средствами ОС
