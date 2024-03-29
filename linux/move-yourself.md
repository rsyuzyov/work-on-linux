## Вводная
...  
Тут важно понимать с самого начала: внешний вид любого дистрибутива определяется набором программ, именуемых окружением рабочего стола, сиречь desktop environment.  
Штука в том, что в любой момент одной командой ~~брюки превращаются в элегантные шорты~~ любую из этих "шкурок" можно натянуть на любой дистрибутив. Да что там, можно все сразу натянуть и каждый день менять под настроение.  
Возникает резонный вопрос: а в чем тогда разница между дистрибутивами? Ответ сначала немного ошарашивает: практически ни в чем, то есть существенных отличий, кардинально меняющих подход, нет. Разница, как и дьявол, кроется в деталях...    
Ну ведь не просто так столько дистрибутивов народ наплодил, скажете вы, и будете правы.  
...  
Разнообразие есть благо и зло linux с момента его появления.  
Когда нужно решить какую-либо задачу, оказывается, что вариантов решения всегда не один и не два, а гораздо больше - очень высок риск надолго утонуть в сравнении и изучении различных способов...
...

## Ставим драйверы
Вообще обычно никакие драйверы ставить не надо, так как в ядре они есть почти для всего, но иногда все-таки приходится.
Драйверы обычно можно найти на гитхабе по названию модели.  
Инструкцию по установке читаем в README.md, чащего всего она примерно такая:
 ```
 make
 sudo make install
 ```
 либо более "кошерный" вариант со сборкой deb-пакета и последующей установкой:  
```
 make
 sudo checkinstall
 ```
 
На примере usb wifi-AC модуля D-Link DWA-171:  
Читаем README.md:  
```
mkdir ~/rtl8812au
cd ~/rtl8812au
git clone https://github.com/gnab/rtl8812au.git .
make
sudo insmod 8812au.ko
```
Драйверы всегда собираются под конкретное ядро. Это означает, что при обновлении ядра необходимо пересобрать драйвер.  
Для автоматизации процесса есть технология [DKMS](https://wiki.archlinux.org/index.php/Dynamic_Kernel_Module_Support_(%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9)):  
```
sudo apt install build-essential dkms 
sudo dkms add -m 8812au -v 4.2.2
sudo dkms build -m 8812au -v 4.2.2
sudo dkms install -m 8812au -v 4.2.2
```


[howto-faq](https://gist.github.com/rsyuzyov/05054c8df5ebe68cac45943104c24493)
