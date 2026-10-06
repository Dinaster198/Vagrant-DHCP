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