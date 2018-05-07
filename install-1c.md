- Установить зависимости
- Скачать с users.v8.1c.ru пакеты для нужного релиза платформы:
  - Клиент 1С:Предприятия (64-bit) для DEB-based Linux-систем
  - Cервер 1С:Предприятия (64-bit) для DEB-based Linux-систем
- Скачать драйвер usb-ключа
- Распаковать все архивы, удалить пакеты тонкого клиента и nls 
- Установить все пакеты

Работаем в каталоге ~/temp, на примере платформы 8.3.10.2466

cd ~/temp
sudo apt install net-tools imagemagick libwebkitgtk-1.0-0 libc6-i386 libgsf-bin libgsf-1-common ttf-freefont ttf-mscorefonts-installer
wget ftp://ftp.etersoft.ru/pub/Etersoft/HASP/stable/x86_64/Debian/8/haspd_7.40-eter10debian_amd64.deb
rm *thin*.deb
rm *nls*.deb
sudo dpkg -i *.deb
systemctl start srv1cv83
systemctl start haspd
