Порядок
- Установить зависимости
- Скачать с users.v8.1c.ru пакеты для нужного релиза платформы:
  - Клиент 1С:Предприятия (64-bit) для DEB-based Linux-систем
  - Cервер 1С:Предприятия (64-bit) для DEB-based Linux-систем
- Скачать драйвер usb-ключа
- Распаковать все архивы, удалить пакеты тонкого клиента и nls 
- Установить все пакеты

Работаем в каталоге ~/temp, дистрибутивы 1С качаем руками.

Для успешной установки ttf-mscorefonts-installer необходимо для основного репозитория помимо main добавить contrib non-free:
В /etc/apt/sources.list найти "deb http:/.../debian/ stretch main" и добавить "contrib non-free"

```
cd ~/temp
sudo apt install net-tools imagemagick libwebkitgtk-1.0-0 libc6-i386 \
libgsf-bin libgsf-1-common ttf-freefont ttf-mscorefonts-installer
wget ftp://ftp.etersoft.ru/pub/Etersoft/HASP/stable/x86_64/Debian/9/haspd_7.60-eter1debian_amd64.deb
tar -xvzf *.tar.gz
rm *thin*.deb
rm *nls*.deb
sudo dpkg -i *.deb
sudo systemctl start srv1cv83
sudo systemctl start haspd
```
