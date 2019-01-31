Apache httpd - веб-сервер, маленький, шустрый, простой и кроссплатформенный. Нафига нам IIS?  
  
**Внимание!**  
В коде фрагменты типа "{...}" необходимо заменить на реальные значения:
###### 
| Параметр | Описание |
| --- | --- |
| {PublicationName} | имя корневой ссылки, http://localhost/{PublicationName} |
| {PublicationCatalog} | каталог, в котором будет находиться файл настроек публикации нашей базы. Может быть асболютно любой, часто это apache/htdocs/{PublicationName} |
 | {ConnectionString} | строка соединения с ИБ, которую мы видим в стартере 1С внизу, под списокм баз. Важный момент: кавычки должны быть заменены на &quot;, то есть `File="c:\1c\ib\myib";` -> `File=&quot;c:\1c\ib\myib&quot;;` |  

Команды приведены для 64-празрядных версий Apache и 1С, при работе с 32-разрядными версиями заменаяем адреса:  
`C:\Program Files\` -> `C:\Program Files (x86)\`  
`/opt/1C/v8.3/x86_64/` -> `/opt/1C/v8.3/i386/`  

### Установка  
| **debian** | **windows** |
| ---------- | ----------- |
| `sudo apt install apache2` | `choco install apache-httpd --params "/installLocation:'C:\Program Files' /port:80 /serviceName:Apache" -y` |  

### Ошибки при установке  
Наиолее частая ошибка: порты 80 или 443 уже кем-то заняты.  
Весьма вероятно, что уже установлен IIS ~~и его нужно снести к ежам~~.  
В этом случае либо освобождаем порты, либо меняем их на нестандартные:  
Для http: в C:\Program Files\Apache24\conf\httpd.conf меняем `Listen 80` на `Listen 5080`  
Для https: в C:\Program Files\Apache24\conf\extra\httpd-ahssl.conf меняем `Listen 443` на `Listen 5443`  
Либо можно просто отключить ssl, поставив # перед строкой в c:\Program Files\Apache24\httpd.conf:
`LoadModule ssl_module modules/mod_ssl.so` -> `#LoadModule ssl_module modules/mod_ssl.so`
  
### Публикация вручную  
Публикация средствами 1С - для слабаков, публикуем вручную. Открываем конфиг:  
######  
| **debian** | **windows** |
| ---------- | ----------- |
| `/etc/apache2/apache2.conf` | `C:\Program Files\Apache24\conf\httpd.conf` |

Добавляем обработчик для 1С:  
######  
| **debian** | **windows** |
| ---------- | ----------- |
| `LoadModule _1cws_module "/opt/1C/v8.3/x86_64/wsap24.so"` | `LoadModule _1cws_module "C:/Program Files/1cv8/{PlatformVersion}/bin/wsap24.dll"` |  

Стандартно 1С при публикации вставляет свою запись перед строкой загрузки модуля slotmem_plain_module, можно найти ее и вставить туда же.   
  
Где-нибудь после окончания секции `<Directory "${SRVROOT}/cgi-bin">` вставляем описание приложения:  
```
# 1c publication
Alias "/{PublicationName}" "{PublicationCatalog}/"
<Directory "{PublicationCatalog}/">
    AllowOverride All
    Options None
    Require all granted
    SetHandler 1c-application
    ManagedApplicationDescriptor "{PublicationCatalog}/default.vrd"
</Directory>

```
  
Создаем файл default.vrd по адресу {PublicationCatalog} следующего содержания:  
```
<?xml version="1.0" encoding="UTF-8"?>
<point xmlns="http://v8.1c.ru/8.2/virtual-resource-system"
		xmlns:xs="http://www.w3.org/2001/XMLSchema"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		base="/{PublicationName}"
		ib="{ConnectionString}">
	<ws publishExtensionsByDefault="true" />
	<httpServices publishExtensionsByDefault="true"/>
	<standardOdata enable="true"
			reuseSessions="autouse"
			sessionMaxAge="20"
			poolSize="10"
			poolTimeout="5"/>
</point>
```

Перезапускаем apache:  
######  
| **debian** | **windows** |
| --- | --- |
| `sudo systemctl restart apache2` | `net stop Apache; net start Apache` |  

### Публикация с помощью webinst  
Каждый раз так утруждаться тяжело, поэтому можно заменить все одной строкой:  
######  
| **debian** | **windows** |
| --- | --- |
| `/opt/1C/v8.3/x86_64/webinst -apache24 -wsdir {PublicationName} -dir "{PublicationCatalog}" -connstr "Srvr=myserver;Ref=mybase;" -confPath /etc/apache2/httpd.conf` | `"C:\Program Files\1cv8\{PlatformVersion}\bin\webinst.exe" -apache24 -wsdir {PublicationName} -dir "{PublicationCatalog}" -connstr "Srvr=myserver;Ref=mybase;" -confPath "C:\Program Files\Apache24\conf\httpd.conf"` |  

### Авторизация средствами ОС
...  

