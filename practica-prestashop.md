---
description: >-
  En esta práctica se usará docker y docker compose para lanzar un servidor
  apache con Prestashop.
---

# Práctica Prestashop

**DATOS DE LA MÁQUINA PARA SU REVISIÓN:**

* **IP:** [**http://54.234.127.140/**](http://54.234.127.140/)
* **USUARIO PRESTASHOP:** admin@admin.com
* **CLAVE PRESTASHOP:** adminpassword
* **ACCESO PHPMYADMIN:** [**http://54.234.127.140:8080/**](http://54.234.127.140:8080/)
* **SERVIDOR MYSQL:** mysql
* **USUARIO PHPMYADMIN:** pst\_user
* **CLAVE PHPMYADMIN:** pst\_password

Esta práctica es muy parecida a la anterior, pero se sustituirá el CMS wordpress por prestashop que está pensado para lanzar una tienda online. Igual que la anterior, se usará docker y docker-compose por lo que el primer paso será instalar ambas aplicaciones:

```text
$ sudo apt update
$ sudo apt install docker.io
$ sudo apt install docker-compose
```

Una vez hecho esto, se importará el archivo de docker-compose incluído en el directorio de este repositorio, que tiene el siguiente contenido:

```text
version: '3.4'


services:
  mysql:
    image: mysql:5.7.28
    ports: 
      - 3306:3306
    environment: 
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
    volumes: 
      - mysql_data:/var/lib/mysql
    networks:
      - backend-network
    restart: always

  phpmyadmin:
    image: phpmyadmin
    ports:
      - 8080:80
    depends_on:
      - "mysql"
    environment: 
      - PMA_ARBITRARY=1
    networks:
      - backend-network
      - frontend-network
    restart: always

  prestashop:
    image: prestashop/prestashop
    ports: 
      - 80:80
    depends_on:
      - "mysql"
    environment: 
      - DB_NAME=${DB_NAME}
      - DB_USER=${DB_USER}
      - DB_PASSWORD=${DB_PASSWORD}
    volumes:
      - prestashop_data:/var/www/html
    networks:
      - backend-network
      - frontend-network
    restart: always

volumes:
  mysql_data:
  prestashop_data:

networks:
  backend-network:
  frontend-network:
```

Es prácticamente idéntico pero en este caso la imagen de wordpress desaparece para dejar paso a prestashop. Las imágenes de phpmyadmin y prestashop dependen de mysql como se puede observar en las cláusulas "depends\_on", por lo que este servicio será el que se ejecute en primer lugar.

También será necesario el archivo con el contenido de las variables de entorno referenciadas:

```text
MYSQL_ROOT_PASSWORD=root
MYSQL_DATABASE=prestashop
MYSQL_USER=pst_user
MYSQL_PASSWORD=pst_password
DB_NAME=mysql
DB_USER=pst_user
DB_PASSWORD=pst_password
```

Estando en la misma carpeta que estos dos archivos, se ejecutará docker-compose:

```text
$ sudo docker-compose up -d
```

Y se probará a acceder a la IP pública de la máquina donde debería saltar la instalación de prestashop:

![](https://raw.githubusercontent.com/ivanmp-lm/IAW/master/.gitbook/assets/image%20(28).png)

Esto indica que el servidor apache con prestashop está funcionando correctamente. Se instalará para comprobar que también funciona la base de datos.

Antes de esto, y ya que se ha creado un usuario específico para prestashop en mysql, se accederá al docker de mysql con el siguiente comando:

```text
$ sudo docker exec -it IDDOCKER /bin/bash
```

Donde "IDDOCKER" serán los dos primeros caracteres de la ID del contenedor \(obtenida con el comando "sudo docker ps"\). Dentro del contenedor mysql se accederá a la base de datos y se le darán privilegios al usuario con los siguientes comandos:

```text
$ mysql -u root -p
mysql> GRANT ALL PRIVILEGES ON prestashop.* TO 'pst_user'.'%';
mysql> FLUSH PRIVILEGES;
```

Acto seguido, en el penúltimo paso de la instalación de prestashop se introducirá la siguiente información:

![](https://raw.githubusercontent.com/ivanmp-lm/IAW/master/.gitbook/assets/image%20(23).png)

Tras la instalación se accederá dentro del contenedor docker de prestashop de la misma forma que se ha hecho anteriormente con mysql y se eliminará la carpeta "install" por motivos de seguridad. Tras esto, prestashop estará instalado:

![](https://raw.githubusercontent.com/ivanmp-lm/IAW/master/.gitbook/assets/image%20(25).png)

Para iniciar sesión se utilizará el correo electrónico y contraseña especificados al inicio del documento. Para finalizar la práctica, se comprobará también que phpMyAdmin funciona correctamente:

![](https://raw.githubusercontent.com/ivanmp-lm/IAW/master/.gitbook/assets/image%20(30).png)
