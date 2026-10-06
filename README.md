# Vagrant-DHCP
En este documento, documentare paso a paso el proceso de instalacion de dhcp mediante vagrant.

## Creacion de repositorio 
 En esta parte creamos las carpetas y iniciamos el git en esa carpeta.

```

alumnom@a209e:~/Vagrant-DHCP$ git init
ayuda: Usando 'master' como el nombre de la rama inicial. Este nombre de rama predeterminado
ayuda: está sujeto a cambios. Para configurar el nombre de la rama inicial para usar en todos
ayuda: de sus nuevos repositorios, reprimiendo esta advertencia, llama a:
ayuda: 
ayuda: 	git config --global init.defaultBranch <nombre>
ayuda: 
ayuda: Los nombres comúnmente elegidos en lugar de 'master' son 'main', 'trunk' y
ayuda: 'development'. Se puede cambiar el nombre de la rama recién creada mediante este comando:
ayuda: 
ayuda: 	git branch -m <nombre>
Inicializado repositorio Git vacío en /home/alumnom/Vagrant-DHCP/.git/

```
Aqui creamos el .gitignore
```
alumnom@a209e:~/Vagrant-DHCP$ touch .gitignore
alumnom@a209e:~/Vagrant-DHCP$ git add .gitignore
alumnom@a209e:~/Vagrant-DHCP$ git commit --allow-empty -m "Repo creation"
[master (commit-raíz) 5a6c405] Repo creation
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 .gitignore
alumnom@a209e:~/Vagrant-DHCP$ 

```

## Creacion de Vagrantfile
Dentro del Vagrantfile añadimos lo siguiente. Aqui debemos de crear la mac para las maquinas que lo necesiten, luego debemos de en "vitualbox_intnet"  poner **intnet**, y la ip poner la de la imagen.

Tambien debemos de añadir el fichero .vagrant al .gitignore.
```
echo ".vagrant/" >> .gitignore
```

```
Vagrant.configure("2") do |config|

  # =========================
  # SERVIDOR DHCP
  # =========================

  config.vm.define "srv" do |srv|

    srv.vm.box = "debian/bookworm64"

    # Adaptador público
    srv.vm.network "public_network" bridge:"enp4s0"

    # Red interna
    srv.vm.network "private_network",
      ip: "192.168.57.10",
      virtualbox__intnet: "intnet"

  end


  # =========================
  # CLIENTE C1
  # =========================

  config.vm.define "c1" do |c1|

    c1.vm.box = "debian/bookworm64"

    c1.vm.network "private_network",
      type: "dhcp",
      virtualbox__intnet: "intnet"

  end


  # =========================
  # PRINTER
  # =========================

  config.vm.define "printer" do |printer|

    printer.vm.box = "debian/bookworm64"

    printer.vm.network "private_network",
      type: "dhcp",
      mac: "080027AABBCC",
      virtualbox__intnet: "intnet"

  end

end
```
Despues de realizar el archivo, realizamos el vagrant up para arrancar las maquinas. Aqui podemos observar las maquinas funcionando medieante vargrant status, una vez usado el comando de vagrant up.
```
 vagrant status
Current machine states:

srv                       running (virtualbox)
c1                        running (virtualbox)
printer                   running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```
## Maquina virtual servidor

Ahora procederemos a entrar a la maquina servidor  mediante **"vagrant ssh nombremaquina"**. Una vez dentro usamos ip route para ver la asignaciones realizadas.
```
vagrant@bookworm:~$ ip route
default via 10.0.2.2 dev eth0 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 
10.209.0.0/16 dev eth1 proto kernel scope link src 10.209.69.1 
192.168.57.0/24 dev eth2 proto kernel scope link src 192.168.57.10 
vagrant@bookworm:~$ 


```

Ahora probamos la conectivdad de la red interna haciendo ping.

```
vagrant@bookworm:~$ ping -c 4 192.168.57.10
PING 192.168.57.10 (192.168.57.10) 56(84) bytes of data.
64 bytes from 192.168.57.10: icmp_seq=1 ttl=64 time=0.014 ms
64 bytes from 192.168.57.10: icmp_seq=2 ttl=64 time=0.081 ms
64 bytes from 192.168.57.10: icmp_seq=3 ttl=64 time=0.039 ms
64 bytes from 192.168.57.10: icmp_seq=4 ttl=64 time=0.057 ms

--- 192.168.57.10 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3463ms
rtt min/avg/max/mdev = 0.014/0.047/0.081/0.024 ms

```

## Instalacion del servidor DHCP

Ahora procederemos con la instalacion del servidor DHCP desde la maquina "srv", creada anteriormente. Primero deberemos realizar el comando ```sudo apt update``` y acontinuacion ``sudo apt install isc-dhcp-server``. Puede que de error al final de la instalacion, debido a que intenta arrancar el servidor sin estar configurado.

```
vagrant@bookworm:~$ systemctl status isc-dhcp-server
× isc-dhcp-server.service - LSB: DHCP server
     Loaded: loaded (/etc/init.d/isc-dhcp-server; generated)
     Active: failed (Result: exit-code) since Tue 2026-10-06 20:59:50 UTC; 9min ago
       Docs: man:systemd-sysv-generator(8)
    Process: 1748 ExecStart=/etc/init.d/isc-dhcp-server start (code=exited, status=1/FAILURE)
        CPU: 19ms
```

Una vez instalado procederemos a configurar la interfaz DHCP.
## Configuracion de la interfaz de DHCP
Para proceder con este paso debemos editar el siguinte archivo, mediante el comando ``sudo nano /etc/default/isc-dhcp-server``, una vez dentro bajamos al final y rellenamos las comillas de donde pone ( **INTERFACESv4=""**) y las sustituimos por nuestro adaptador **"eth2"**.

Una vez realizado esto modificamos el siguiente archivo mediante el comando ``sudo nano /etc/dhcp/dhcpd.conf``, y al final de este añadimos los siguiente. Y guardamos y salimos del archivo.

```
subnet 192.168.57.0 netmask 255.255.255.0 {
    range 192.168.57.25 192.168.57.50;

    default-lease-time 86400;
    max-lease-time 691200;

    option broadcast-address 192.168.57.255;
    option routers 192.168.57.2;
    option domain-name-servers 192.168.57.3, 4.4.4.4;
    option domain-name "TU NOMBRE.test";
}
```
Esto dira que de la red 192.168.57.0 asignara el rango de IPs que pongamos mediante  nuestro servidor DHCP y la mascara de subred y la direccion broadcast. Tambien asiganara un tiempo maximo de concesion para IP.

Para ver que todo funciona correctamente usamos el comando ``sudo dhcpd -t``, y nos mostrarara lo siguiente.
```
vagrant@bookworm:~$ sudo dhcpd -t
Internet Systems Consortium DHCP Server 4.4.3-P1
Copyright 2004-2022 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/
Config file: /etc/dhcp/dhcpd.conf
Database file: /var/lib/dhcp/dhcpd.leases
PID file: /var/run/dhcpd.pid
```


 Despues reiniciamos el servidor DHCP con el comando ``sudo systemctl restart isc-dhcp-server``, y usamos el comando ``sudo systemctl status isc-dhcp-server`` para ver que esta funcionado.

```
vagrant@bookworm:~$ systemctl status isc-dhcp-server
● isc-dhcp-server.service - LSB: DHCP server
     Loaded: loaded (/etc/init.d/isc-dhcp-server; generated)
     Active: active (running) since Tue 2026-10-06 21:23:29 UTC; 3s ago
       Docs: man:systemd-sysv-generator(8)
    Process: 1835 ExecStart=/etc/init.d/isc-dhcp-server start (code=exited, status=0/SUCCESS)
      Tasks: 1 (limit: 496)
     Memory: 4.0M
        CPU: 27ms
     CGroup: /system.slice/isc-dhcp-server.service
             └─1847 /usr/sbin/dhcpd -4 -q -cf /etc/dhcp/dhcpd.conf eth2
vagrant@bookworm:~$
```

## Maquina cliente

Ahora entramos a la maquina cliente, primero salimos de la maquina servidor con el comando ``exit`` y despues entramos al cliente mediante ``vagrant ssh c1``. Una vez dentro comprobamos que la IP es correcta que esta dentro del rango asignado.

```
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:00:91:46 brd ff:ff:ff:ff:ff:ff
    altname enp0s8
    inet 192.168.57.20/24 brd 192.168.57.255 scope global dynamic eth1
       valid_lft 295sec preferred_lft 295sec
    inet6 fe80::a00:27ff:fe00:9146/64 scope link
       valid_lft forever preferred_lft forever
```
Ahora probaremos a renovar la ip, para comprobar que funciona correctamente, mediante el comando ``sudo dhclient -r`` y despues usamos ``sudo dhclient`` y volvemos a usar el comando ``ip a``.

```
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:00:91:46 brd ff:ff:ff:ff:ff:ff
    altname enp0s8
    inet 192.168.57.20/24 brd 192.168.57.255 scope global dynamic eth1
       valid_lft 105sec preferred_lft 105sec
    inet 192.168.57.25/24 brd 192.168.57.255 scope global secondary dynamic eth1
       valid_lft 86398sec preferred_lft 86398sec
    inet6 fe80::a00:27ff:fe00:9146/64 scope link
       valid_lft forever preferred_lft forever
```
Una vez comprobado, hacemos ping al servidor para ver que funcione.
```
vagrant@bookworm:~$ ping -c 4 192.168.57.10
PING 192.168.57.10 (192.168.57.10) 56(84) bytes of data.
64 bytes from 192.168.57.10: icmp_seq=1 ttl=64 time=0.382 ms
64 bytes from 192.168.57.10: icmp_seq=2 ttl=64 time=0.249 ms
64 bytes from 192.168.57.10: icmp_seq=3 ttl=64 time=0.240 ms
64 bytes from 192.168.57.10: icmp_seq=4 ttl=64 time=0.221 ms

--- 192.168.57.10 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3714ms
rtt min/avg/max/mdev = 0.221/0.273/0.382/0.063 ms

```

## Comprobar el lease en el servidor

Para esto salimos de la maquina cliente y volvemos a entrar en la maquina servidor. Una vez dentro usamos el comando ``cat /var/lib/dhcp/dhcpd.leases`` para ver las asignaciones realizadas a la maquina c1.
```
vagrant@bookworm:~$ cat /var/lib/dhcp/dhcpd.leases
# The format of this file is documented in the dhcpd.leases(5) manual page.
# This lease file was written by isc-dhcp-4.4.3-P1

# authoring-byte-order entry is generated, DO NOT DELETE
authoring-byte-order little-endian;

server-duid "\000\001\000\0012X#O\010\000'\333\230\323";

lease 192.168.57.25 {
  starts 2 2026/10/06 21:42:38;
  ends 3 2026/10/07 21:42:38;
  cltt 2 2026/10/06 21:42:38;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 08:00:27:00:91:46;
  client-hostname "bookworm";
}
lease 192.168.57.25 {
  starts 2 2026/10/06 21:44:26;
  ends 3 2026/10/07 21:44:26;
  cltt 2 2026/10/06 21:44:26;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 08:00:27:00:91:46;
  uid "\377'\000\221F\000\001\000\0012X\033\301\010\000'\000\221F";
  client-hostname "bookworm";
}
lease 192.168.57.26 {
  starts 2 2026/10/06 21:44:35;
  ends 3 2026/10/07 21:44:35;
  cltt 2 2026/10/06 21:44:35;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 08:00:27:aa:bb:cc;
  uid "\377'\252\273\314\000\001\000\0012X\034B\010\000'\252\273\314";
  client-hostname "bookworm";
}
```

## Configuracion de impresora ip fija
Para hacer que la impresora reciba siempre misma ip 192.168.57.111 para eso  primero entramos a la impresora mediante ``vagrant ssh printer``. Una vez dentro usamos ``ip link`` y buscamos la direccion MAC una vez la tengamos accedemos de nuevo a la maquina servidor y dentro del archivo **dchpd.config** alfinal añadimos otra cosa nueva, que servira para dar la ip fija mediante la MAC de la impresora.
Para acceder a dicho archivo usamos ``sudo nano /etc/dhcp/dhcpd.conf``.

```
host printer {
    hardware ethernet 08:00:27:aa:bb:cc;
    fixed-address 192.168.57.111;
}
```
Una vez realizado el cambio usamos  ``sudo systemctl restart isc-dhcp-server``.

Ahora para ver que ha funcionado, nos dirigimos a la maquina impresora, liberamos la ip mediante ``sudo dhcpd -r`` y pedimos otra usando el comando ``sudo dhcpd `` y nos aseguramos de que nos hayan dado la IP deseada 192.168.57.111.

```
vagrant@bookworm:~$ sudo dhclient -r
vagrant@bookworm:~$ sudo dhclient
RTNETLINK answers: File exists
vagrant@bookworm:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:8d:c0:4d brd ff:ff:ff:ff:ff:ff
    altname enp0s3
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic eth0
       valid_lft 81608sec preferred_lft 81608sec
    inet6 fd17:625c:f037:2:a00:27ff:fe8d:c04d/64 scope global dynamic mngtmpaddr
       valid_lft 85965sec preferred_lft 13965sec
    inet6 fe80::a00:27ff:fe8d:c04d/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:aa:bb:cc brd ff:ff:ff:ff:ff:ff
    altname enp0s8
    inet 192.168.57.26/24 brd 192.168.57.255 scope global dynamic eth1
       valid_lft 84690sec preferred_lft 84690sec
    inet 192.168.57.100/24 brd 192.168.57.255 scope global secondary dynamic eth1
       valid_lft 86397sec preferred_lft 86397sec
    inet6 fe80::a00:27ff:feaa:bbcc/64 scope link
       valid_lft forever preferred_lft forever
vagrant@bookworm:~$
```

Ahora probamos la conectivida de la impresora hacia el servidor.

```
vagrant@bookworm:~$ ping -c 4 192.168.57.10
PING 192.168.57.10 (192.168.57.10) 56(84) bytes of data.
64 bytes from 192.168.57.10: icmp_seq=1 ttl=64 time=0.262 ms
64 bytes from 192.168.57.10: icmp_seq=2 ttl=64 time=0.235 ms
64 bytes from 192.168.57.10: icmp_seq=3 ttl=64 time=0.240 ms
64 bytes from 192.168.57.10: icmp_seq=4 ttl=64 time=0.287 ms

--- 192.168.57.10 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3062ms
rtt min/avg/max/mdev = 0.235/0.256/0.287/0.020 ms
vagrant@bookworm:~$
```

## Aplicaciond de la configuracion
Para que la configuracion se aplique siempre debemos de copiar la carpeta **dhcpd.conf** al repo, para que asi podamos desde el VagrantFile usar esa configuracion.

Para ello usaremos fuera de las maquin el siguiente comando, esto copiara el archivo en el  repo.
```
vagrant ssh srv -c "sudo cat /etc/dhcp/dhcpd.conf" > dhcpd.conf
```

Una vez tengamos el archivo en el repo modificaremos el Vagrantfile para que ejecute la configuracion, tambien añadiremos el cambio que hicimos a **interfacesv4**. Añadiremos lo siguiente dentro del **srv**
```
       # Instalar y configurar DHCP
    srv.vm.provision "shell", inline: <<-SHELL
      apt update
      apt install -y isc-dhcp-server

      cp /vagrant/dhcpd.conf /etc/dhcp/dhcpd.conf

      sed -i 's/^INTERFACESv4=.*/INTERFACESv4="eth2"/' /etc/default/isc-dhcp-server

      dhcpd -t
      systemctl enable isc-dhcp-server
      systemctl restart isc-dhcp-server
    SHELL
    
```

Esto realizara la instalacion del servidor dhcp y despues copiara el archivo de configuracion del repo.

