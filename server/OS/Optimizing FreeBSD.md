Optimizing FreeBSD

FreeBSD is well established as a system for building intranet and Internet servers. It provides fairly reliable network services and efficient memory management.

Standard FreeBSD settings do not allow optimal use of hardware resources. Consider an example of the configuration of this OS to work as a platform for the Web server.

General settings

The basic tuning of FreeBSD is done by making changes to sysctl . All parameters are divided into two groups:

Parameters used only at boot time
Parameters used on the fly
Parameters used at boot time

To edit the settings that apply only at boot time, you need to make changes to the file /boot/loader.conf , and then reboot the OS.

# Отсутствие ограничений на размер зоны.
kern.ipc.nmbclusters=0


# После исчерпания не возможно выполнить повторную передачи пакетов.
net.inet.tcp.reass.maxsegments=2048


# Для избежания лимита "PV Entry"
vm.pmap.shpgperproc=400


# Позволяет принимать за одно прерывание до 4096 пакетов
hw.em.rxd=4096


# Позволяет передавать за одно прерывание до 4096 пакетов
hw.em.txd=4096


# Минимальная задержка между генерациями прерываний в мкс для приема данных
hw.em.rx_int_delay=100


# Минимальная задержка между генерациями прерываний в мкс для передачу данных
hw.em.tx_int_delay=100


# Максимальная задержка между генерациями прерываний в мкс для приема данных
hw.em.rx_abs_int_delay=1000


# Максимальная задержка между генерациями прерываний в мкс для передачи данных
hw.em.tx_abs_int_delay=1000
dev.em.rx_processing_limit=-1


# Настройка hostcache
net.inet.tcp.hostcache.hashsize=4096
net.inet.tcp.hostcache.bucketlimit=100
net.inet.tcp.hostcache.cachelimit=65536


# Настройка syncache
net.inet.tcp.syncache.hashsize=1024
net.inet.tcp.syncache.bucketlimit=100
net.inet.tcp.syncache.cachelimit=65536


# Таблица TCP control-block
net.inet.tcp.tcbhashsize=4096


# Настройка net.isr


# Очередь netisr
net.isr.defaultqlimit=4096


# Каждая очередь к отдельному ядру
net.isr.bindthreads=1


# Количество потоков netist (равно количеству ядер)
net.isr.maxthreads=8


# Размер очереди исходящий пакетов
net.link.ifqmaxlen=1024


# Модули


# Загружаем драйвер AHCI
ahci_load="YES"


# Асинхронное выполнение IO системных вызовов
aio_load="YES"
Parameters used "on the fly"

To change the parameters in an already loaded OS, run the following command:

sysctl param_name=value
# param_name - name of the parameter, value - value

Also, you should duplicate the changes to the /etc/sysctl.conf file , otherwise, after the reboot, the default value will be applied.

# Разрешает пользователям видеть только запущенные ими процессы
security.bsd.see_other_uids=0


# Количество сокетов
kern.ipc.maxsockets=204800


# Mbuf clusters
kern.ipc.nmbclusters=262144


# Запрещает вытеснение Wired памяти в Swap
kern.ipc.shm_use_phys=1


# Размер очереди для приема новых TCP соединений
kern.ipc.somaxconn=4096


# Определяет максимальное число дескрипторов файлов
kern.maxfiles=204800


# Максимальное число файлов на один процесс
kern.maxfilesperproc=200000


# Количество файловых дескрипторов закэшированых в памяти
kern.maxvnodes=256000


# не использовать трафик и прерывания
kern.random.sys.harvest.ethernet=0


# как источник энтропии для random'a
kern.random.sys.harvest.interrupt=0


# Попытаться синхронизировать диски при панике, для избежания
# fsck после перезагрузки
kern.sync_on_panic=1


# Для защиты от SMURF атак (ICMP echo request на broadcast адрес)
net.inet.icmp.bmcastecho=0


# Игнорировать пакеты icmp redirect
net.inet.icmp.drop_redirect=1


# Не отдавать по icmp маску своей подсети
net.inet.icmp.maskrepl=0


# Размер очереди IP-пакетов
# (Необходимо увеличивать если net.inet.ip.intr_queue_drops != 0)
net.inet.ip.intr_queue_maxlen=256


# Количество фрагментированных пакетов в очереди
net.inet.ip.maxfragpackets=1024


# Минимальный порт для исходящих соединений
net.inet.ip.portrange.first=1024


# Максимальный порт для исходящих соединений
net.inet.ip.portrange.last=65535


# Не использовать случайные порты для исх. соединений
net.inet.ip.portrange.randomized=0


# Не реагируем на ICMP редирект
net.inet.ip.redirect=0


# Отключение маршрутизации от источника
net.inet.ip.sourceroute=0
net.inet.ip.accept_sourceroute=0


# Не отвечать на все пакеты полученные на закрытый порт
net.inet.tcp.blackhole=2


# Отбрасывать пакеты с флагами SYN+FIN
net.inet.tcp.drop_synfin=1


# FIN_WAIT_2 state fast recycle
net.inet.tcp.fast_finwait2_recycle=1


# FIN_WAIT_2 таймаут
net.inet.tcp.finwait2_timeout=3000


# Время хранение hostcache
net.inet.tcp.hostcache.expire=1200


# Timeout for establishing syn
net.inet.tcp.keepinit=5000


# Максимально количество TIME_WAIT сокетов
net.inet.tcp.maxtcptw=65536


# Количество времени пребывания соединения в состоянии TIME_WAIT
# (в миллисекундах деленных на 2, 2 x 30000 MSL = 60 секунд)
net.inet.tcp.msl=5000


# Отключить автоподстройку receive буфера
net.inet.tcp.recvbuf_auto=0


# Размер receive буфера
net.inet.tcp.recvspace=65536


# Отключить автоподстройку send буфера
net.inet.tcp.sendbuf_auto=0


# Размер send буфера
net.inet.tcp.sendspace=131072


# Возможность перехода в "TCP SYN cookies" при переполнении syncache
net.inet.tcp.syncookies=1


# Отключаем использование TSO для сетевых карт
net.inet.tcp.tso=0


# Отбрасываются все UDP пакеты, адресованные закрытым портам
net.inet.udp.blackhole=1


# Размер UDP receive буфера
net.inet.udp.recvspace=32768


# Обрабатывать исходящие пакеты непосредственно при попытке отправки
net.isr.direct=1


# Размер исходящей очереди
net.route.netisr_maxqlen=1024


# При большом количестве файлов на сервере
vfs.ufs.dirhash_maxmem=100000000
Running applications

The next step in tuning the OS is to configure the network and launch the necessary applications in the /etc/rc.conf file .

# Задаем имя сервера
hostname="host.domain.com"


# Настраиваем сеть
ifconfig_em0="inet 172.16.1.5 netmask 255.255.255.0"
ifconfig_em1="inet 10.16.1.51 netmask 255.255.255.0"
defaultrouter="172.16.1.1"


# Запуск SSHD при загрузки
sshd_enable="YES"


# Запрещаем SSHD выполнять запросы к DNS
sshd_flags="-u0"


# Отключаем использование USB
usbd_enable="NO"


# Отключаем sendmail
sendmail_enable=NONE


# Запускаем ntpd при старте ОС
ntpd_enable="YES"


# Отключаем NFS
inetd_enable="NO"
portmap_enable="NO"
nfs_client_enable="NO"
nfs_reserved_port_only="NO"
nfs_server_enable="NO"


# Небольшая защита от DDOSа
tcp_drop_synfin="YES"
icmp_drop_redirect="YES"
icmp_log_redirect="NO"


# Запускаем систему журналирования
syslogd_enable="YES"


# Указываем syslog сохранять только локальные сообщений
# и не выполняет DNS запросов
syslogd_flags="-s -n"


# Включить fsck при загрузке
fsck_y_enable="YES"


# Выполнять во время загрузки ОС
background_fsck="NO"


# Сохраняем корки ядра
dumpdir="/home"
dumpdev="AUTO"


# Отключить SNMPD
snmpd_enable="NO"


# Настройка фаервола
firewall_enable="YES"
firewall_script="/etc/rc.ipfw"
And finally add the scripts to the /etc/rc.local file that you need to execute at boot time.
# Привязывание очередей прерываний сетевых карт к процессорным ядрам

# по мотивам <a href="http://dadv.livejournal.com/139366.html" target="_blank"> этой статьи</a>.
/usr/local/startup/cpuset-emigb.sh
Scripts

The firewall initialization script /etc/rc.ipfw :

#!/bin/sh

WAN="em0"
LAN="em1"
OPEN_PORT="80,443"
IPFW=`which ipfw`
RET=$?

if [ ${RET} -ne 0 ]; then
echo "IPFW not found."
exit ${RET}
fi

${IPFW} -f flush
${IPFW} -f table 1 flush

${IPFW} add 100 allow ip from any to any via lo0
${IPFW} add 110 deny ip from any to 127.0.0.0/8
${IPFW} add 120 deny ip from 127.0.0.0/8 to any
${IPFW} add 130 allow ip from any to any via ${LAN}
${IPFW} add 140 deny all from table\(1\) to me
${IPFW} add 150 deny all from any to any frag

${IPFW} add 160 allow carp from any to any via ${WAN}
${IPFW} add 180 allow tcp from any to any via ${WAN} established
${IPFW} add 200 check-state
${IPFW} add 210 allow tcp from me to any via ${WAN} setup keep-state
${IPFW} add 220 allow udp from me to any via ${WAN} keep-state
${IPFW} add 230 allow tcp from any to me dst-port ${OPEN_PORT} via ${WAN}
${IPFW} add 240 allow icmp from any to any icmptypes 0,8 via ${WAN}
${IPFW} add 65530 deny ip from any to any

exit 0
The script of binding interrupts of network card queues to processor cores /usr/local/startup/cpuset-emigb.sh :

#!/bin/sh
cpus=`sysctl -n kern.smp.cpus`
DRIVER=`pciconf -l | egrep 'em|igb' | sed -E 's/^([emigb]+).*/\1/' | head -n 1`
case "$DRIVER" in
em)
echo $DRIVER
i=1
vmstat -ai | egrep "em[0-9]:[a-z]x" | sed -E 's/^irq([0-9]+): (em[0-9]):([a-z]+).*/\1 \2 \3/' |
while read irq iface queue
do
cpu=$(( $i % $cpus ))
echo "Binding ${iface} queue ${queue} (irq ${irq}) -&gt; CPU${cpu}"
cpuset -l $cpu -x $irq
i=$(( $i + 1 ))
done
;;
igb)
echo $DRIVER
vmstat -ai | sed -E '/^irq.*que/!d; s/^irq([0-9]+): igb([0-9]+):que ([0-9]+).*/\1 \2 \3/' |
while read irq iface queue
do
cpu=$(( ($iface+$queue) % $cpus ))
echo "Binding igb${iface} queue ${queue} (irq ${irq}) -&gt; CPU${cpu}"
cpuset -l $cpu -x $irq
done
;;
esac
The most important

The above is an example of setting up a Web server platform for high-load work. Adjustment of parameters should be done periodically to adapt the OS to changes in loads.