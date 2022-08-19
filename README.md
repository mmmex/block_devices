
1. Запустить первый раз vagrant up завершится с ошибкой SATA controller уже существует
2. Запустить второй раз vagrant up. Будут созданы контроллеры и диски. Диски будут присоеденены к контроллерам.
3. Выполнить вход в консоль vagrant ssh
4. Скрипт сборки конфигурации программного RAID10:
```
yum install mdadm -y
mdadm --create /dev/md/raid10 --raid-devices=4 /dev/sdb /dev/sdc /dev/sdd /dev/sde --level=10
mkdir /etc/mdadm
echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
mdadm --detail --scan | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
parted /dev/md/raid10 -s mklabel gpt
for i in $(seq 1 5) ; do parted /dev/md/raid10 -s mkpart primary ${i}00Mib $((i+1))00Mib ; done
```
5. Проверка содержания конфигурации RAID10 при старте ОС: [Файл конфигурации](/mdadm.conf)
```
cat /etc/mdadm/mdadm.conf
```
6. Результат:
```
parted /dev/md/raid10 -s print

Model: Linux Software RAID Array (md)
Disk /dev/md/raid10: 2143MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start  End    Size   File system  Name     Flags
 1      105MB  210MB  105MB               primary
 2      210MB  315MB  105MB               primary
 3      315MB  419MB  105MB               primary
 4      419MB  524MB  105MB               primary
 5      524MB  629MB  105MB               primary
```
