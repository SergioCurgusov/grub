1) Включить отображение меню Grub.

Комментируем строку, скрывающую меню:

sed -i '/GRUB_TIMEOUT_STYLE/s/^/#/' /etc/default/grub

ставим задержку для выбора пункта меню в 10 секунд:

sed -i 's/'$(cat /etc/default/grub | grep GRUB_TIMEOUT=)'/GRUB_TIMEOUT=10/g' /etc/default/grub

Обновляем конфигурацию загрузчика и перезагружаемся для проверки:

update-grub
reboot

2) Попасть в систему без пароля несколькими способами.

a) Первый способ.

При загрузке меню grub нажимаем e.

В конце строки, начинающейся с linux, добавляем init=/bin/bash и нажимаем сtrl-x для загрузки системы.

Рутовая файловая система монтируется в режиме Read-Only. Перемонтируем в режим записи:
mount -o remount,rw /

Чтобы понять, что всё заработало, создадим пользователя и после перезагрузки попробуем в него зайти:
useradd user
passwd user

Перезагружаемся, заходим успешно под созданным пользовалем.

b) Второй способ.

В меню выбираем Advanced options… и в появившемся меню выбираем строку с recovery mode в названии.
Система прогрузилась. Нажимаем Enter.
Графическое меню как в методичке у меня не подгрузилось, подгрузилась консоль. Рутовая файловая система монтируется в режиме Read-Only. Перемонтируем в режим записи. Делаю как в предыдущем примере:
mount -o remount,rw /

Для проверки удаляем пользователя user.
userdel user

Перезагружаемся. Проверяем, войти под user не можем. Под нашим пользователем вводим команду:
getent group | user
users:x:100:

Т.е. пользователя user нет.

3) Установить систему с LVM, после чего переименовать VG.

Выводим список Volume Gruop
root@server:/home/sergio# vgs
  VG        #PV #LV #SN Attr   VSize   VFree 
  ubuntu-vg   1   1   0 wz--n- <23.00g 11.50g

Переименовываем ubuntu-vg в ubuntu-otus, как в задании

root@server:/home/sergio# vgrename ubuntu-vg ubuntu-otus
  Volume group "ubuntu-vg" successfully renamed to "ubuntu-otus"

В /boot/grub/grub.cfg меняем старое название на новое. Но нозвания задаются по-другому, не так, как в выводе команды vgs. Автоматизируем вычисление новых названий и меняем grub.cfg:
STALO=$(ls /dev/mapper | grep otus)
BYLO=$(awk ' /root=\/dev\/mapper\//{print $3}' /boot/grub/grub.cfg | head -1 | sed 's/root=\/dev\/mapper\///g')
sed -i 's/'$BYLO'/'$STALO'/g' /boot/grub/grub.cfg

Перезагружаемся, у нас всё работает.