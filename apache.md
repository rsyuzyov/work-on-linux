Apache httpd - веб-сервер, маленький, шустрый, простой и кроссплатформенный. Нафига нам IIS?

### Установка  
**debian:**  
```
sudo apt install apache2
```
**windows:**  
```
choco install apache-httpd -y
```

### Настройка
Публикация средствами 1С - для слабаков, публикуем вручную:

Добавляем в /etc/apache2/apache2.conf (c:\program files\apache24\httpd.conf):
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
net stop ""; net start ""
```


### Авторизация средствами ОС
