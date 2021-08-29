# Statyczna konfiguracja sieci

//TODO miejsce na obrazek topologia-adresacja

## Sprawdzenie adresów mac switch'ów

### EthernetSwitch-1

```sh
Ethernetswitch-1> mac
Port       Mac                VLAN
Ethernet1  0c:4f:a4:71:67:00  1
Ethernet0  0c:4f:a4:06:57:00  1
```

### EthernetSwitch-2

```sh
Ethernetswitch-2> mac
Port       Mac                VLAN
Ethernet0  0c:4f:a4:64:23:02  1
Ethernet1  0c:4f:a4:be:16:00  1
```

## Wyłączenie auto konfiguracji IPv6 na hostach

Komenda wpisana w obu hostach:

```sh
sudo sysctl -w net.ipv6.conf.all.forwarding=0
```

## Konfiguracja domeny A

### Router-A

```sh
vyos@vyos# set interfaces ethernet eth0 address 2001:db8:a::1/64
vyos@vyos# commit
```

### Host-1

```sh
gns3@box:~$ sudo ip -6 addr add 2001:db8:a::2/64 dev eth0
gns3@box:~$ sudo ip -6 route add default via 2001:db8:a::1
```

### Rezultat - komunikacja między Host-1 a Router-A

![](img/1.png)

## Konfiguracja domeny B

### Router-B

```sh
vyos@vyos# set interfaces ethernet eth2 address 2001:db8:b::1/64
vyos@vyos# commit
```

### Host-2

```sh
gns3@box:~$ sudo ip -6 addr add 2001:db8:b::2/64 dev eth0
gns3@box:~$ sudo ip -6 route add default via 2001:db8:b::1
```

### Rezultat - komunikacja między Host-1 a Router-A

![](img/2.png)

## Konfiguracja domeny C

### Adresy

//TODO dopisać komentarz o zmienionej masce z /48 na /64

#### Router-A

```sh
vyos@vyos# configure
vyos@vyos# set interfaces ethernet eth0 address 2001:db8:c:1::1/64
vyos@vyos# set interfaces ethernet eth2 address 2001:db8:c:4::1/64
vyos@vyos# commit
```

#### Router-B

```sh
vyos@vyos# configure
vyos@vyos# set interfaces ethernet eth0 address 2001:db8:c:3::2/64
vyos@vyos# set interfaces ethernet eth1 address 2001:db8:c:7::2/64
vyos@vyos# commit
```

#### Router-C

```sh
vyos@vyos# configure
vyos@vyos# set interfaces ethernet eth0 address 2001:db8:c:1::2/64
vyos@vyos# set interfaces ethernet eth1 address 2001:db8:c:5::2/64
vyos@vyos# set interfaces ethernet eth2 address 2001:db8:c:2::1/64
vyos@vyos# commit
```

#### Router-D

```sh
vyos@vyos# configure
vyos@vyos# set interfaces ethernet eth0 address 2001:db8:c:2::2/64
vyos@vyos# set interfaces ethernet eth1 address 2001:db8:c:3::1/64
vyos@vyos# set interfaces ethernet eth2 address 2001:db8:c:6::1/64
vyos@vyos# commit
```

#### Router-E

```sh
vyos@vyos# configure
vyos@vyos# set interfaces ethernet eth0 address 2001:db8:c:4::2/64
vyos@vyos# set interfaces ethernet eth1 address 2001:db8:c:5::1/64
vyos@vyos# commit
```

#### Router-F

```sh
vyos@vyos# configure
vyos@vyos# set interfaces ethernet eth0 address 2001:db8:c:6::2/64
vyos@vyos# set interfaces ethernet eth1 address 2001:db8:c:7::1/64
vyos@vyos# commit
```

### Konfiguracja najkrótszej ścieżki

#### Router-A

```sh
vyos@vyos# set protocols static route6 2001:db8:b::/64 next-hop 2001:db8:c:1::2
vyos@vyos# commit
```

#### Router-B

```sh
vyos@vyos# set protocols static route6 2001:db8:a::/64 next-hop 2001:db8:c:3::1
vyos@vyos# commit
```

#### Router-C

```sh
vyos@vyos# set protocols static route6 2001:db8:a::/64 next-hop 2001:db8:c:1::1
vyos@vyos# set protocols static route6 2001:db8:b::/64 next-hop 2001:db8:c:2::2
vyos@vyos# commit
```

#### Router-D

```sh
vyos@vyos# set protocols static route6 2001:db8:a::/64 next-hop 2001:db8:c:2::1
vyos@vyos# set protocols static route6 2001:db8:b::/64 next-hop 2001:db8:c:3::2
vyos@vyos# commit
```

#### Rezultaty

##### Tablice routingu

###### Router-A

![](img/3.png)

###### Router-B

![](img/4.png)

###### Router-C

![](img/5.png)

###### Router-D

![](img/6.png)

##### Komunikacja między Hostami

###### Host-1 --> Host-2

![](img/7.png)

###### Host-1 --> Host-2

![](img/8.png)

### Konfiguracja najdłuższej ścieżki

#### Usunięcie poprzedniej ścieżki

##### Router-A

```sh
vyos@vyos# delete protocols static route6 2001:db8:b::/64
vyos@vyos# commit
```

##### Router-B

```sh
vyos@vyos# delete protocols static route6 2001:db8:a::/64
vyos@vyos# commit
```

##### Router-C

Usunęliśmy tylko ruch w stronę domeny A, ponieważ ruch w stronę domeny B, będzie taki sam dla obu ścieżek.

```sh
vyos@vyos# delete protocols static route6 2001:db8:a::/64
vyos@vyos# commit
```

##### Router-D

Usunęliśmy tylko ruch w stronę domeny B, ponieważ ruch w stronę domeny A, będzie taki sam dla obu ścieżek.

```sh
vyos@vyos# delete protocols static route6 2001:db8:b::/64
vyos@vyos# commit
```

#### Router-A

```sh
vyos@vyos# set protocols static route6 2001:db8:b::/64 next-hop 2001:db8:c:4::2
vyos@vyos# commit
```

#### Router-B

```sh
vyos@vyos# set protocols static route6 2001:db8:a::/64 next-hop 2001:db8:c:7::1
vyos@vyos# commit
```

#### Router-C

```sh
vyos@vyos# set protocols static route6 2001:db8:a::/64 next-hop 2001:db8:c:5::1
vyos@vyos# commit
```

#### Router-D

```sh
vyos@vyos# set protocols static route6 2001:db8:b::/64 next-hop 2001:db8:c:6::2
vyos@vyos# commit
```

#### Router-E

```sh
vyos@vyos# configure
vyos@vyos# set protocols static route6 2001:db8:a::/64 next-hop 2001:db8:c:4::1
vyos@vyos# set protocols static route6 2001:db8:b::/64 next-hop 2001:db8:c:5::2
vyos@vyos# commit
```

#### Router-F

```sh
vyos@vyos# configure
vyos@vyos# set protocols static route6 2001:db8:a::/64 next-hop 2001:db8:c:6::1
vyos@vyos# set protocols static route6 2001:db8:b::/64 next-hop 2001:db8:c:7::2
vyos@vyos# commit
```

#### Rezultaty

##### Tablice routingu

###### Router-A

![](img/9.png)

###### Router-B

![](img/10.png)

###### Router-C

![](img/11.png)

###### Router-D

![](img/12.png)

###### Router-E

![](img/13.png)

###### Router-F

![](img/14.png)

##### Komunikacja między Hostami

###### Host-1 --> Host-2

![](img/15.png)

###### Host-2 --> Host-1

![](img/16.png)

# Dynamiczna konfiguracja sieci

## Wyłączenie auto konfiguracji IPv6 na hostach

Komenda wpisana w obu hostach:

```sh
sudo sysctl -w net.ipv6.conf.all.forwarding=0
```

## Wygenerowanie adresów interfejsów 

//TODO opisać EUI64 i SLAAC

### Router-A

```sh
vyos@vyos# configure
vyos@vyos# set interfaces ethernet eth0 ipv6 address eui64 2001:db8:a::/64
vyos@vyos# set interfaces ethernet eth1 ipv6 address eui64 2001:db8:c:1::/64
vyos@vyos# set interfaces ethernet eth2 ipv6 address eui64 2001:db8:c:4::/64
vyos@vyos# commit
```

### Router-B

```sh
vyos@vyos# configure
vyos@vyos# set interfaces ethernet eth2 ipv6 address eui64 2001:db8:b::/64
vyos@vyos# set interfaces ethernet eth0 ipv6 address eui64 2001:db8:c:3::/64
vyos@vyos# set interfaces ethernet eth1 ipv6 address eui64 2001:db8:c:7::/64
vyos@vyos# commit
```

### Router-C
```sh
vyos@vyos# configure
vyos@vyos# set interfaces ethernet eth0 ipv6 address eui64 2001:db8:c:1::/64
vyos@vyos# set interfaces ethernet eth1 ipv6 address eui64 2001:db8:c:5::/64
vyos@vyos# set interfaces ethernet eth2 ipv6 address eui64 2001:db8:c:2::/64
vyos@vyos# commit
```

### Router-D
```sh
vyos@vyos# configure
vyos@vyos# set interfaces ethernet eth0 ipv6 address eui64 2001:db8:c:2::/64
vyos@vyos# set interfaces ethernet eth1 ipv6 address eui64 2001:db8:c:3::/64
vyos@vyos# set interfaces ethernet eth2 ipv6 address eui64 2001:db8:c:6::/64
vyos@vyos# commit
```

### Router-E
```sh
vyos@vyos# configure
vyos@vyos# set interfaces ethernet eth0 ipv6 address eui64 2001:db8:c:4::/64
vyos@vyos# set interfaces ethernet eth1 ipv6 address eui64 2001:db8:c:5::/64
vyos@vyos# commit
```

### Router-F
```sh
vyos@vyos# configure
vyos@vyos# set interfaces ethernet eth0 ipv6 address eui64 2001:db8:c:6::/64
vyos@vyos# set interfaces ethernet eth1 ipv6 address eui64 2001:db8:c:7::/64
vyos@vyos# commit
```

### Rezultaty

#### Globalne adresy IPv6

##### Router-A

###### `show interfaces ethernet eth<x>`

![](img/17.png)

##### Router-B

###### `show interfaces ethernet eth<x>`

![](img/18.png)

##### Router-C

###### `show interfaces ethernet eth<x>`

![](img/19.png)

##### Router-D

###### `show interfaces ethernet eth<x>`

![](img/20.png)

##### Router-E

###### `show interfaces ethernet eth<x>`

![](img/21.png)

##### Router-F

###### `show interfaces ethernet eth<x>`

![](img/22.png)

## Rozgłoszenie wiadomości RA

### Router-A

```sh
vyos@vyos# set interfaces ethernet eth0 ipv6 router-advert send-advert true
vyos@vyos# set interfaces ethernet eth0 ipv6 router-advert min-interval 8
vyos@vyos# set interfaces ethernet eth0 ipv6 router-advert max-interval 12
vyos@vyos# set interfaces ethernet eth0 ipv6 router-advert prefix 2001:db8:a::/64
vyos@vyos# commit
```

Chcieliśmy dać 9-11, ale `MinRtrAdvInterval must be no greater than 3/4 MaxRtrAdvInterval`

### Router-B

```sh
vyos@vyos# set interfaces ethernet eth0 ipv6 router-advert send-advert true
vyos@vyos# set interfaces ethernet eth0 ipv6 router-advert min-interval 8
vyos@vyos# set interfaces ethernet eth0 ipv6 router-advert max-interval 12
vyos@vyos# set interfaces ethernet eth0 ipv6 router-advert prefix 2001:db8:b::/64
vyos@vyos# commit
```

### Rezultaty

#### Wygenerowanie globalnych adresów IPv6 hostów

##### Host-1: `gns3@bpx:~$ ip -6 addr show dev eth0`

![](img/23.png)

##### Host-2: `gns3@box:~$ ip -6 addr show dev eth0`

![](img/25.png)

#### Możliwość zpingowania z hosta interfejsów routerów w ich domenie

##### Host-1

![](img/24.png)

##### Host-2

![](img/26.png)

## Konfiguracja OSPFv3 na routerach

### Wyłączenie przekazywania IPv6 

Na wszelki wypadek na każdym routerze wykonujemy komendę:

```sh
vyos@vyos# delete system ipv6 disable-forwarding
```

Lecz okazało nie być to konieczne, ponieważ na każdym routerze dostaliśmy komunikat `Nothing to delete (the specified node does not exist)`.

### Nadanie router-id routerom OSPFv3

Użyjemy do tego adresów loopback, które zdefiniujemy według schematu `Router-A ==> 1.1.1.1`, `Router-B ==> 2.2.2.2` itd.

#### Router-A

```sh
vyos@vyos# set interfaces loopback lo address 1.1.1.1/32
vyos@vyos# set protocols ospfv3 parameters router-id 1.1.1.1
vyos@vyos# commit
```
#### Router-B

```sh
vyos@vyos# set interfaces loopback lo address 2.2.2.2/32
vyos@vyos# set protocols ospfv3 parameters router-id 2.2.2.2
vyos@vyos# commit
```

#### Router-C

```sh
vyos@vyos# set interfaces loopback lo address 3.3.3.3/32
vyos@vyos# set protocols ospfv3 parameters router-id 1.1.1.1
vyos@vyos# commit
```

#### Router-D

```sh
vyos@vyos# set interfaces loopback lo address 4.4.4.4/32
vyos@vyos# set protocols ospfv3 parameters router-id 4.4.4.4
vyos@vyos# commit
```

#### Router-E

```sh
vyos@vyos# set interfaces loopback lo address 5.5.5.5/32
vyos@vyos# set protocols ospfv3 parameters router-id 5.5.5.5
vyos@vyos# commit
```

#### Router-F

```sh
vyos@vyos# set interfaces loopback lo address 6.6.6.6/32
vyos@vyos# set protocols ospfv3 parameters router-id 6.6.6.6
vyos@vyos# commit
```

### Dodanie wszystkich interfejsów do `Area 0`

#### Router-A

```sh
vyos@vyos# set protocols ospfv3 area 0.0.0.0 interface eth0
vyos@vyos# set protocols ospfv3 area 0.0.0.0 interface eth1
vyos@vyos# set protocols ospfv3 area 0.0.0.0 interface eth2
vyos@vyos# commit
```

#### Router-B

```sh
vyos@vyos# set protocols ospfv3 area 0.0.0.0 interface eth0
vyos@vyos# set protocols ospfv3 area 0.0.0.0 interface eth1
vyos@vyos# set protocols ospfv3 area 0.0.0.0 interface eth2
vyos@vyos# commit
```

#### Router-C

```sh
vyos@vyos# set protocols ospfv3 area 0.0.0.0 interface eth0
vyos@vyos# set protocols ospfv3 area 0.0.0.0 interface eth1
vyos@vyos# set protocols ospfv3 area 0.0.0.0 interface eth2
vyos@vyos# commit
```

#### Router-D

```sh
vyos@vyos# set protocols ospfv3 area 0.0.0.0 interface eth0
vyos@vyos# set protocols ospfv3 area 0.0.0.0 interface eth1
vyos@vyos# set protocols ospfv3 area 0.0.0.0 interface eth2
vyos@vyos# commit
```

#### Router-E

```sh
vyos@vyos# set protocols ospfv3 area 0.0.0.0 interface eth0
vyos@vyos# set protocols ospfv3 area 0.0.0.0 interface eth1
vyos@vyos# commit
```

#### Router-F

```sh
vyos@vyos# set protocols ospfv3 area 0.0.0.0 interface eth0
vyos@vyos# set protocols ospfv3 area 0.0.0.0 interface eth1
vyos@vyos# commit
```

### Ustalenie prefiksów rozgłaszanych podsieci IPv6

#### Router-A

```sh
vyos@vyos# set protocols ospfv3 area 0.0.0.0 range 2001:db8:a::/64
vyos@vyos# set protocols ospfv3 area 0.0.0.0 range 2001:db8:c:1::/64
vyos@vyos# set protocols ospfv3 area 0.0.0.0 range 2001:db8:c:4::/64
vyos@vyos# commit
```

#### Router-B

```sh
vyos@vyos# set protocols ospfv3 area 0.0.0.0 range 2001:db8:b::/64
vyos@vyos# set protocols ospfv3 area 0.0.0.0 range 2001:db8:c:3::/64
vyos@vyos# set protocols ospfv3 area 0.0.0.0 range 2001:db8:c:7::/64
vyos@vyos# commit
```

#### Router-C

```sh
vyos@vyos# set protocols ospfv3 area 0.0.0.0 range 2001:db8:c:1::/64
vyos@vyos# set protocols ospfv3 area 0.0.0.0 range 2001:db8:c:2::/64
vyos@vyos# set protocols ospfv3 area 0.0.0.0 range 2001:db8:c:5::/64
vyos@vyos# commit
```

#### Router-D

```sh
vyos@vyos# set protocols ospfv3 area 0.0.0.0 range 2001:db8:c:2::/64
vyos@vyos# set protocols ospfv3 area 0.0.0.0 range 2001:db8:c:3::/64
vyos@vyos# set protocols ospfv3 area 0.0.0.0 range 2001:db8:c:6::/64
vyos@vyos# commit
```

#### Router-E

```sh
vyos@vyos# set protocols ospfv3 area 0.0.0.0 range 2001:db8:c:4::/64
vyos@vyos# set protocols ospfv3 area 0.0.0.0 range 2001:db8:c:5::/64
vyos@vyos# commit
```

#### Router-F

```sh
vyos@vyos# set protocols ospfv3 area 0.0.0.0 range 2001:db8:c:6::/64
vyos@vyos# set protocols ospfv3 area 0.0.0.0 range 2001:db8:c:7::/64
vyos@vyos# commit
```

### Rozgłoszenie prefiksów

Na każdym routerze używamy komendy:

```sh
vyos@vyos# set protocols ospfv3 redistribute connected
vyos@vyos# commit
```

### Rezultaty

#### Komunikacja między hostami

##### Host-1 --> Host-2

![](img/27.png)

##### Host-2 --> Host-1

![](img/28.png)

##### Router-A --> Host-2

Na hostach nie ma polecenie `traceroute6` dla IPv6, dlatego użyjemy go z routerów bezpośrednio połączonych do hostów.

![](img/29.png)

#### Router-B --> Host-1

![](img/30.png)

//TODO wypisać jak przebiega ścieżka Znaczy no wiadomo jak xd

### Zmiana konfiguracji sieci

Zmienimy ścieżkę między hostami z `A-C-D-B` na `A-E-C-D-B`

//TODO opisać kilka sposobów na uzyskanie takiego celu

W tym celu zmienimy koszt łącza `A-C` na `3` za pomocą komend:

```sh
Router-A: vyos@vyos# set interfaces ethernet eth1 ipv6 ospfv3 cost 3
Router-A: vyos@vyos# commit
```

```sh
Router-C: vyos@vyos# set interfaces ethernet eth0 ipv6 ospfv3 cost 3
Router-C: vyos@vyos# commit
```

Wystarczy wartość `3` ponieważ domyślnie łącza mają ustawione metryki równe `1`. Co do wartości `2` nie możemy być pewni, ponieważ wtedy powstał by nam remis: 2 łącza z metrykami `1` lub jedno łącze z metryką `2`.

#### Rezultaty

##### Nowa ścieżka zaobserwowana `traceroute`'em

###### Router-A --> Host-2

![](img/31.png)

###### Router-B --> Host-1

![](img/32.png)

//TODO dopisać wnioski, porównać dwie konfiguracje

//TODO dodać rozdziały do tego wszystkiego!!!