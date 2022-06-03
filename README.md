# OTUS5_NFS
Homework

Основная часть:
- `vagrant up` должен поднимать 2 настроенных виртуальных машины
(сервер NFS и клиента) без дополнительных ручных деи?ствии?;
- на сервере NFS должна быть подготовлена и экспортирована
директория;
- в экспортированнои? директории должна быть поддиректория
с именем __upload__ с правами на запись в нее?;
- экспортированная директория должна автоматически монтироваться
на клиенте при старте виртуальнои? машины (systemd, autofs или fstab -
любым способом);
- монтирование и работа NFS на клиенте должна быть организована
с использованием NFSv3 по протоколу UDP;
- firewall должен быть включен и настроен как на клиенте,
так и на сервере.

Выполнение:

Сервер:
Установил необходимое ПО
yum install nfs-utils

Включил firewall и проверил, что он запущен
systemctl enable filewalld --now

Дал доступ необходимым сервисам через firewall и перезагрузил его
firewall-cmd --add-service="nfs3" \
--add-service="rpc-bind" \
--add-service="mountd" \
--permanent
firewall-cmd --reload

Включил nfs сервер
systemctl enable nfs --now

Проверил открытые порты
ss -tnplu

Создал папку /srv/share/upload, сменил владельца и права на неё
mkdir -p /srv/share/upload
chown -R nfsnobody:nfsnobody /srv/share
chmod 0777 /srv/share/upload

В файле /etc/exports указал, какие папки будут расшарены
cat << EOF > /etc/exports
/srv/share 192.168.50.11/32(rw,sync,root_squash)
EOF

Экспортировал директорию /srv/share/upload
exportfs -r

Клиент:
Установил необходимое ПО
yum install nfs-utils

Включил firewall и проверил, что он запущен
systemctl enable filewalld --now

Изменил /etc/fstab для автоматического монтирования
echo "192.168.50.10:/srv/share/ /mnt nfs vers=3,proto=udp,noauto,x-
systemd.automount 0 0" >> /etc/fstab

Перезагрузил службу для подмонтрирования директории
systemctl daemon-reload
systemctl restart remote-fs.target

Проверил успешное монтирование
mount | grep mnt

Сервер и клиент перезагрузил, создал по файлу на сервере и клиенте. Оба видят все созданные файлы.


Уничтожил обе машины
vagrant destroy -f

Дополнил Vagrantfile строчками для запуска shell скрипта при первом запуске
nfss.vm.provision "shell", path: "nfss_script.sh"
nfsc.vm.provision "shell", path: "nfsc_script.sh"

Создал скрипты с соответствующими названиями в репозитории и записал в них необходимые команды.
Командой vagrant up запустил обе машины, после запуска проверил отработку скриптов.