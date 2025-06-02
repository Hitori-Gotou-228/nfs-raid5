# nfs-raid5
dnf install mdadm
mdadm --create /dev/md0 -l5 -n 3 /dev/sd{b,c,d}
mkfs.ext4 /dev/md0
mkdir /etc/mdadm
echo "DEVICE partitions" > /etc/mdadm.conf
mdadm --detail --scan | awk '/ARRAY/ {print}' >> /etc/mdadm.conf
mkdir /raid5
nano /etc/fstab
дописываем в конец файла
/dev/md0 /raid5 ext4 defaults 0 0
Выполняем mount -a, если ошибок не возникает, выполняем df -h, смотрим, смонтирован ли массив в папку /raid5

на сервере
dnf install nfs-utils -y
mkdir /raid5/nfs
chmod -R 777 /raid5/nfs 
в файл nano /etc/exports пишем
/raid5/nfs 192.168.200.0/28(rw,sync,no_root_squash)
systemctl enable --now nfs-utils
systemctl restart nfs-utils
systemctl restart nfs-server


на клиенте:
dnf install nfs-utils
systemctl enable --now nfs-utils
mkdir /mnt/nfs
chmod -R 777 /mnt/nfs
в файл nano /etc/fstab пишем
192.168.100.2:/raid5/nfs /mnt/nfs nfs defaults 0 0
mount -a
mount 192.168.100.2:/raid5/nfs /mnt/nfs
systemctl restart nfs-utils
dh -h 
В конце вывода должен быть виден смонтированный ресурс
