# Práctica 3b. Autoridades certificadoras (CA) y Public Key Infrastructure (PKI) y Apache TLS

## Contexto
Autoridades certificadoras son las responsables de emitir certificados digitales para verificar identidades en internet (servidores, personas, conexiones).
El uso de estos certificados permiten conexiones confiables, además de autenticidad y no repudio.
TLS (Transport Layer Security) es un protocolo de seguridad basado en criptografía asimétrica que establece un canal seguro entre dos hosts
    
En esta práctica configuraremos nuestro servidor web (Apache) para establecer conexiones seguras mediante el protocolo HTTPS.
El certificado digital que usaremos deberá estar firmado por una Autoridad Certificadora que crearemos y configuraremos.

## Links
* [Autoridades certificadorase](https://devopscube.com/create-self-signed-certificates-openssl/)
  
* [Crear una autoridad certificadora - Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-and-configure-a-certificate-authority-on-ubuntu-22-04)
* [Configurar TLS en Apache](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-20-04)
* [Configurar NFS](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-22-04)


## Objetivos
* Entender el papel de una CA y el proceso de creación de certificados digitales
* Crear y configurar una CA mediante la utilidad *easy-rsa*
* Crear par de claves para crear una petición de certificado
* Saber firmar peticiones de certificados digitles(CSR)
* Compartir ficheros(certificados) en la red mediante NFS
* Configurar el protocolo https del servidor web apache para transmisiones seguras.
* Verificar y documentar de forma clara, concisa y completa los pasos llevados a cabo para la finalización de la práctica.

## Desarrollo

### Pasos previos
* Configurar VM con ip pública de aula
    Comentamos la linea " config.vm.network "private_network", ip: "192.168.56.201" ". Depues de esto añadimos las siguientes  lineas:
    ``` shell
    config.vm.network "public_network",
    :auto_config => true,
    :bridge => "eno1",
    :ip => "192.168.82.111"

    ```
    Luego ya hacemos "vagrant up".

* Configurar NFS Server en cada máquina
    Instlamos el nfs-server con el comando *apt isntall nfs-kernel-server*.
    Creamos las carpetas que usaremos para subir y recoger nuestros certificados en /net/pki ademas debemos cambiar los permisos a nobody no group.
    ``` shell
    sudo chown nobody:nogroup /net/pki/
    ```
    Dentro de estas carpertas creamos los links de las carpetas issued y reqs  (se crean como si fueran externas).
    ``` shell
    sudo ln -s nfs://192.168.82.111:/home/vagrant/easy-rsa/pki/ /net/pki/
    ```
    Y en nuestra maquina los links de estas carpetas a la nuestra
    ```
    sudo ln -s ~/easy-rsa/pki/issued /net/pki/issued
    ```

    Ahora reiniciamos el servicio nfs (comando systemctl restart nfs-kernel-server).

    Ahora para contactar con nuestro servidor CA para que pueda firmar nuestras requests. Creamos una carpeta en /nfs
    ``` shell
    sudo mkdir -p /nfs/vinicio-server
    ```
    Y ahora montamos su carpeta /net/pki en nuestro fichero /nfs/vinicio-server
    ``` shell
    sudo mount 192.168.82.112:/net/pki /nfs/vinicio-server/
    ```
    Para qeu se monte automaticamente modificasmos los archivo /etc/fstab
    ``` shell
    192.168.82.112:/net/pki/        /nfs/vinicio-server/    nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
    ```
    Otorgamso lso permisos que se requieran a las carpetas:
    ```
    sudo chmod 755 /net/pki/issued/
    sudo chmod 775 /net/pki/reqs/
    ```

    Ahora copiamso el ca.crt a la carpeta pki y hacemos el coamndo para actualizar la lista de certificadores ca.
    ```
    sudo cp ~/easy-rsa/pki/ca.crt /net/pki/
    sudo cp ./ca.crt /usr/local/share/ca-certificates/
    sudo update-ca-certificates
    ```

    Despues instalamos openssl para poder pasar los reqs y recoger los ya firmados. Se instala con el comando "sudo apt install openssl".
    Ahroa generamos el rsa .key para pedir que el CA nos firme.
    ```
    openssl genrsa -out jorge-srv.key
    
   
    ```
    Ahora creamos el archivo .req para pedir al CA que los firme.
    ``` 
    openssl req -new -key jorge-srv.key -out jorge-srv.req
    openssl req -in jorge-srv.req -noout -subject
    ```
    Ahora copiamos el .req a la carpeta de red para qeu llegue al ca y asi lo firme. (damos premisos 777 -R a la carpeta /nfs/vinicio-server)
    ```
    sudo cp jorge-srv.req /nfs/vinicio-server/reqs
    ```
    Ahora el CA lo firmará y el cliente lo podra recuperar en issued.

* Configurar NFS Client a CA_TEAM y CA_CENTRAL

### Pasos CA
* Crear y configurar Certification Authority (CA Server)

>[!NOTE]
> Obtenemos (ca.crt y ca.key) Explicar que es cada fichero

* Compartir certificado digital de la autoridad certificadora (ca.crt) 


### Pasos solicitante
* Instalar  certificado digital de la autoridad certificadora en el sistema
* Crear solicitud de certificado (CSR)
* Enviar solicitud a CA
* Firmada la solictud configurar apache para uso de HTTPS 

>[!WARNING]
> Verificar y demostrar que sompos capaces de establecer conexiones seguras HTTPS con nuestro servidor web y que el certificado de nuestro servidor web está firmado por la CA de clase.