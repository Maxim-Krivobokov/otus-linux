Домашнее задание
1. Попасть в систему без пароля несколькими способами
2. Установить систему с LVM, после чего переименовать VG
3. Добавить модуль в initrd
4. (*) Сконфигурировать систему без отдельного раздела с /boot, а
только с LVM



### Выполнение 

1. через GUI. во время загрузки жмем Е, попадает в параметры  
Несколько вариантов действия:
* в строке linux16 добавляем init=/bin/sh, для продолжения Ctrl-X
    * результат - (после отображения своего модуля с пингвином) - Kernel Panic
````
Kernel panic - not syncing: Attempted to kill init! exitcode=0x00000000
````

* в строке linux16 добавляем rd.break, для продолжения Ctrl-X. будет загрузка в Emergency mode
    * результат - система загружается в обычном режиме, запрашивает пароль

* в строке linux16 изменяем ro на rw init=/sysroot/bin/sh, для продолжения Ctrl-X
     * результат - загрузка зависает на пингвине. проверил на другой ВМ. Зависла намного раньше
```
random: crng init done
```

### Исправление ошибок
* в параметрах при загрузке и в grub.cfg удаляем параметр  cconsole=ttyS0,115200n8
* значение параметра console=tty0 меняем на console=tty1


#### метод входа через добавление init=/bin/sh
    * видим "строку приглашения" sh-4.2
    * при попытке записи на диск видим сообщение - Read Only file system
    * для полноценного использования ФС перемонтируем ее в режиме чтения-записи
````
mount -o remount,rw /
# ключ -о обозначает перечисление опций, remount - перемонтирование, rw = режим read/write, / - path 
# при успешном выполнениии ничего не выводится
````
   * после этого запись на диск возможна 

   * изменил файл /boot/grub2/grub.cfg , изменив параметры console=ttyX, вызывающие баг virtualbox

* если нажать ctrl-D в терминале, то процесс /bin/sh закроется, и система "упадет" в kernel panic

* процесс загрузки очень короткий. Systemd функционирует, но вот команду shutdown /bin/sh не понимает. 

* доутспно все окружение. приложения, например, текстовый редактор nano

#### rd.break 
* дописал в той же строке linux16 rd.break. Система грузится в emergency mode. 
* отличие от предыдущего случая - попадаем не в свою ФС, нужно удет произвести chroot /sysroot
* работают команды "через" emergency shell,  в /bin сокращенный перечень команд  нет директории /home, т.к файловая система "на наша"
  * в "местной" папке /etc нет, например, таблицы fstab

* перемонтируем ФС и попадаем в нее
````
mount -o remount,rw /sysroot

chroot /sysroot

# замена пароля root
passwd root
touch /.autorelabel
````

* замечание - на стенде с HW1 (обновление ядра), через ядро 5 версии и rd.break запускался dracut emergency shell. Процесс загрузки был очень длительным. перемонтировани ене удавалось, система ругалась на отсуствтие /etc/fstab

#### Способ с init=/sysroot/bin/sh
* изменив параметр ro на rw, получаем сразу файловую систему в режиме чтения-записи
* попадаем в тот же emergency mode
* для  работы нужно сменить директорию рута, далее опять можно менять пароль, и.т.д

### переименование VG  
(протоколируем через script)
* после входа в систему смотрим список виртуальных групп VG

* переименовываем группу
```
vgs

vgrename VolGroup00 OtusRoot
```

* нужно изменить файлы /etc/fstab /etc/default/grub /boot/grub2/grub.cfg. В них заменить  имя VolGroup00 на OtusRoot

* пересоздать образ initrd, чтобы он знал новое имя Volume group
```
mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
```

* после перезагрузки (система запустилась - успех!) проверяем командой vgs обновленный volume group
###  Добавление модуля в initrd

* Для добавление своего модуля создадим директорию в /usr/lib/dracut/modules.d/01test
```
mkdir /usr/lib/dracut/modules.d/01test
```
* в нее помещены два скрипта, module-setup.sh и test.sh. Первый устанавливает модуль и вызывает второй скритп. Второй рисует пингвина.  Им присвоены права для исполнения

* пересобрать образ initrd
```
mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
# альтернатива:
dracut -f -v
```

* просмотр установленных в образ модулей
```
lsinitrd -m /boot/initramfs-$(uname -r).img | grep test
```

* проверка: либо во время перезагрузки убрать опции rghb и quiet в строке linux16, либо удалить их в /boot/grub2/grub.cfg

* во время перезагрузки в GUI видим пингвина. 