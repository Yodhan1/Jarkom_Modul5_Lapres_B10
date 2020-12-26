# Jarkom_Modul5_Lapres_B10
Lapres modul 5 Jarkom

## Topologi.sh
```
# Switch
uml_switch -unix switch1 > /dev/null < /dev/null &  #1   R BATU     S MALANG      S MOJOKERTO
uml_switch -unix switch2 > /dev/null < /dev/null &  #2   R BATU     C SIDOARJO
uml_switch -unix switch3 > /dev/null < /dev/null &  #3   R BATU     R SURABAYA
uml_switch -unix switch4 > /dev/null < /dev/null &  #4   R SURABAYA R KEDIRI
uml_switch -unix switch5 > /dev/null < /dev/null &  #5   R KEDIRI   C GRESIK 
uml_switch -unix switch6 > /dev/null < /dev/null &  #6   R KEDIRI   S PROBOLINGGO S MADIUN
    
    # Router
xterm -T SURABAYA -e linux ubd0=SURABAYA,jarkom umid=SURABAYA eth0=tuntap,,,10.151.74.45 eth1=daemon,,,switch3 eth2=daemon,,,switch4 mem=96M &
xterm -T BATU -e linux ubd0=BATU,jarkom umid=BATU eth0=daemon,,,switch3 eth1=daemon,,,switch1 eth2=daemon,,,switch2 mem=96M &   
xterm -T KEDIRI -e linux ubd0=KEDIRI,jarkom umid=KEDIRI eth0=daemon,,,switch4 eth1=daemon,,,switch5 eth2=daemon,,,switch6 mem=96M &

# Server
xterm -T MALANG -e linux ubd0=MALANG,jarkom umid=MALANG eth0=daemon,,,switch1 mem=128M &   
xterm -T MOJOKERTO -e linux ubd0=MOJOKERTO,jarkom umid=MOJOKERTO  eth0=daemon,,,switch1   mem=128M &
xterm -T MADIUN -e linux ubd0=MADIUN,jarkom umid=MADIUN eth0=daemon,,,switch6 mem=128M &
xterm -T PROBOLINGGO -e linux ubd0=PROBOLINGGO,jarkom umid=PROBOLINGGO eth0=daemon,,,switch6 mem=128M &

# Klien 
xterm -T GRESIK -e linux ubd0=GRESIK,jarkom umid=GRESIK eth0=daemon,,,switch2 mem=96M &
xterm -T SIDOARJO -e linux ubd0=SIDOARJO,jarkom umid=SIDOARJO eth0=daemon,,,switch5 mem=96M &


```
## Bye.sh
```
uml_mconsole MOJOKERTO halt
uml_mconsole MALANG halt
uml_mconsole BATU halt
uml_mconsole SURABAYA halt
uml_mconsole MADIUN halt
uml_mconsole PROBOLINGGO halt
uml_mconsole GRESIK halt
uml_mconsole SIDOARJO halt
uml_mconsole KEDIRI halt

```
## etc
```
nano /etc/network/interfaces
Router 
    SURABAYA
        auto eth0
        iface eth0 inet static
        address 10.151.74.46
        netmask 255.255.255.252
        gateway 10.151.74.45

        auto eth1
        iface eth1 inet static
        address 192.168.1.1
        netmask 255.255.255.252

        auto eth2
        iface eth2 inet static
        address 192.168.3.9
        netmask 255.255.255.252

    BATU
        auto eth0
        iface eth0 inet static
        address 192.168.1.2
        netmask 255.255.255.252

        auto eth1
        iface eth1 inet static
        address 10.151.83.89
        netmask 255.255.255.248

        auto eth2
        iface eth2 inet static
        address 192.168.0.1
        netmask 255.255.255.0

    KEDIRI
        auto eth0
        iface eth0 inet static
        address 192.168.3.10
        netmask 255.255.255.252

        auto eth1
        iface eth1 inet static
        address 192.168.2.1
        netmask 255.255.255.0

        auto eth2
        iface eth2 inet static
        address 192.168.3.1
        netmask 255.255.255.248

Server 
    MALANG
        auto eth0
        iface eth0 inet static
        address 10.151.83.90
        netmask 255.255.255.248
        gateway 10.151.83.89
    MOJOKERTO
        auto eth0
        iface eth0 inet static
        address 10.151.83.91
        netmask 255.255.255.248
        gateway 10.151.83.89
    MADIUN 
        auto eth0
        iface eth0 inet static
        address 192.168.3.2
        netmask 255.255.255.248
        gateway 192.168.3.1
    PROBOLINGGO 
        auto eth0
        iface eth0 inet static
        address 192.168.3.3
        netmask 255.255.255.248
        gateway 192.168.3.1

Client 
    GRESIK 
        auto eth0
        iface eth0 inet dhcp

    SIDOARJO 
        auto eth0
        iface eth0 inet dhcp

```
## Route
```
SURABAYA
    BATU
        route add -net 192.168.0.0 netmask 255.255.254.0 gw 192.168.1.2
        route add -net 10.151.83.88 netmask 255.255.255.248 gw 192.168.1.2
    KEDIRI
        route add -net 192.168.2.0 netmask 255.255.254.0 gw 192.168.3.10
BATU
    route add -net 0.0.0.0 netmask 0.0.0.0 gw 192.168.1.1
KEDIRI
    route add -net 0.0.0.0 netmask 0.0.0.0 gw 192.168.3.9

```
## dhcp
```
SURABAYA BATU KEDIRI{
    nano /etc/sysctl.conf
    net.ipv4.ip_forward=1
    sysctl -p

    apt-get install isc-dhcp-relay -y
    nano /etc/default/isc-dhcp-relay
    server 10.151.83.91
    interface <blank> 
}

MOJOKERTO{
    apt-get install isc-dhcp-server -y
    nano /etc/default/isc-dhcp-server
    INTERFACES="eth0"
    nano /etc/dhcp/dhcpd.conf
    subnet 10.151.83.0 netmask 255.255.255.0 {  
        option routers 10.151.83.91; 
    }
}

nano /etc/dhcp/dhcpd.conf
subnet 192.168.0.0 netmask 255.255.255.0 {
    option routers 192.168.0.1;
    option broadcast-address 192.168.0.255;
    option domain-name-servers 10.151.83.90;
}

subnet 192.168.2.0 netmask 255.255.255.0 {
    option routers 192.168.2.1;
    option broadcast-address 192.168.2.255;
    option domain-name-servers 10.151.83.90;
}
service isc-dhcp-server stop
service isc-dhcp-server start

service isc-dhcp-relay stop
service isc-dhcp-relay start
```
## no 1 
```
iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source 10.151.74.46 -s 192.168.0.0/16
```
