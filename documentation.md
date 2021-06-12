#Zadanie 1
##Charakterystyka sieci:
* usługa DHCP do nadawania adresów w sposób automatyczny dla wszystkich stacji roboczych
* serwer i drukarka posiadają stałe adresy celem zminimalizowania potrzeby rekonfiguracji ustawień klientów
* translacja nazw domenowych:
    - erp.mojaorganizacja.pl
    - drukarka.mojaorganizacja.pl
    - router.mojaorganizacja.pl
* połączenie wszystkich urządzeń z siecią Internet za pomocą bramy NAT
* wykorzystanie podsieci rozmiaru /22
##Adresy sieci IP:
| net | netmask | addr_min | addr_max | hosts |
|-------------|-------|----------|----------|---------------|
|66.170.200.0|255.255.252.0|66.170.200.1|66.170.203.254|1024|
##Wykorzystane oprogramowanie:
* Projekt sieci: CISCO PACKET TRACER
* Konfiguracja prototypu rozwiązania: VirtualBox / maszyny wirtualne Alpine Linux
* Konfiguracja NAT: iptables
* Konfiguracja DNS: dnsmasq
* Konfiguracja DHCP: ISC DHCP
##Kluczowa konfiguracja
* ###konfiguracja NAT z iptables
  - router posiada dwa interfejsy:
    - eth0 - połączenie z siecią WAN
    - eth1 - połączenie z siecią lokalną, adres: 66.170.200.1
  - dla interfejsu eth0 została ustanowiona maskarada, w celu translacji adresów.<br>polecenia: 
    ```
    iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
    echo 1 > /proc/sys/net/ipv4/ip_forward 
    ```
  
* ###konfiguracja DHCP
    
  | subnet | netmask | range | router | domain-name-servers |
  |--------|---------|-------|--------|---------------------|
  |66.170.200.0|255.255.252.0|66.170.200.10<br>66.170.200.254|66.170.200.1|66.170.200.2|
  Rezerwacje adresów:
  
  | fixed-address | example-mac-address | host_name |
  |---------------|---------------------|-----------|
  | 66.170.200.11 | 08:00:27:E1:02:E8 | printer | 
  | 66.170.200.10 | 08:00:27:A2:E8:65 | server |
  plik /etc/dhcpd.conf:
  ````
  subnet 66.170.200.0 netmask 255.255.252.0 {
    range 66.170.200.10 66.170.200.254;
    option routers 66.170.200.1;
    host server {
      fixed-address 6.170.200.10;<br>
      hardware ethernet 08:00:27:a2:e8:65;
    }
    host printer {
      fixed-address 66.170.200.11<br>
      hardware ethernet 08:00:27:e1:02:e8;
    }
  }
  ```

* ###konfiguracja DNS:
  - adres serwera DNS: 66.170.200.2
  - konfiguracja pliku /etc/hosts:
  
  | domain-name | ip address |
  |-------------|------------|
  | erp.mojaorganizacja.pl | 66.170.200.10 |
  | drukarka.mojaorganizacja.pl | 66.170.200.11 |
  | router.mojaorganizacja.pl | 66.170.200.1 |
  
* ###konfiguracja interfejsów sieciowych:
  | device name | interfejs | ip address |
  |-------------|-----------|------------|
  | router | eth1 LAN | 66.170.200.1 |
  |  | eth0 | WAN |
  | dhcp / dns server | eth0 | 66.170.200.2 |
