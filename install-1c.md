sudo apt install net-tools imagemagick libwebkitgtk-1.0-0 libc6-i386 libgsf-bin libgsf-1-common ttf-freefont ttf-mscorefonts-installer

Скачать с users.v8.1c.ru пакеты для нужного релиза платформы в ~/temp/:
- Клиент 1С:Предприятия (64-bit) для DEB-based Linux-систем
- Cервер 1С:Предприятия (64-bit) для DEB-based Linux-систем

cd ~/temp
rm *thin*.deb
rm *nls*.deb
sudo dpkg -i *.deb

Управление сервером 1С:
systemctl status srv1cv83
systemctl enable srv1cv83
systemctl start srv1cv83

