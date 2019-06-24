# Laravel en CentOS7 con driver SQLServer

Guia rapida para instalar Laravel 5.8 en CentOS 7, utilizando Apache, PHP 7.2. y driver de SQLServer.

Pre-requisitos:
- Maquina local con cliente SSH instalado. (Putty por ejemplo)
- VPS con CentOS 7 corriendo.
- Usuario no root con privilegios sudo, por motivos de seguridad.

## SSH

Se ingresa al servidor vía SSH.

## Yum

Una vez dentro del servidor limpiamos el cache de Yum:

```
$| sudo yum clean all
```

Actualizamos los paqueques del sistema:

```
$| sudo yum -y update
```

## Herramientas

Instalamos algunas herramientas necesarias durante la instalacion

```
$| sudo yum -y install wget vim net-tools unzip git 
```

## Apache

Instalamos Apache:

```
$| sudo yum -y install httpd
```

Iniciamos Apache:

```
$| sudo systemctl start httpd
```

Habilitamos ejecución de Apache al inicio del sistema:

```
$| sudo systemctl enable httpd
```

Revisamos el estado del servicio:

```
$| sudo systemctl status httpd
```

Debe aparecer asi:

```
Active: active (running) 
```

## PHP 7.2

Instala PHP y modulos requeridos por Laravel y SQLServer:

```
$| sudo su
$| wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
$| wget http://rpms.remirepo.net/enterprise/remi-release-7.rpm
$| rpm -Uvh remi-release-7.rpm epel-release-latest-7.noarch.rpm
$| yum install -y yum-utils
$| yum-config-manager --enable remi-php72
$| yum install -y php php-pdo php-xml php-pear php-devel php-common php-bcmath php-mbstring php-gd php-cli php-fpm re2c gcc-c++ gcc
$| exit
```

Actualiza GCC para compilar los drivers PHP con PECL y PHP 7.2.

```
$| sudo yum install -y centos-release-scl
$| sudo yum-config-manager --enable rhel-server-rhscl-7-rpms
$| sudo yum install -y devtoolset-7
$| scl enable devtoolset-7 bash
```

Se verifica la version de php instalada:

```
$| php -v
```

El output:

```
PHP 7.2.19 (cli) (built: May 29 2019 11:04:13) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
```

Para revisar los modulos php instalados

```
$| php -m
```

## SQL Server

Se indica a SELinux que permita conectar a apache a una BBDD en CentOS:

```
$| sudo service httpd stop
$| sudo setsebool -P httpd_can_network_connect 1
$| sudo setsebool -P httpd_can_network_connect_db 1
$| sudo service httpd start 
```

Instala driver ODBC y el CLI de SQLServer:

```
$| sudo su
$| curl https://packages.microsoft.com/config/rhel/7/prod.repo > /etc/yum.repos.d/mssql-tools.repo
$| ACCEPT_EULA=Y yum install -y msodbcsql17 mssql-tools
$| exit
$| sudo remove unixODBC-utf16 unixODBC-utf16-devel #no es necesario en una instalacion fresca de CentOS
$| sudo ACCEPT_EULA=Y yum install -y msodbcsql17 mssql-tools
$| echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile
$| echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
$| source ~/.bashrc
$| sudo yum install -y unixODBC-devel
 ```
 
Instala driver SQLServer para PHP:
 
 ```
$| sudo pecl install sqlsrv
$| sudo pecl install pdo_sqlsrv
$| sudo su
$| echo extension=pdo_sqlsrv.so >> `php --ini | grep "Scan for additional .ini files" | sed -e "s|.*:\s*||"`/30-pdo_sqlsrv.ini
$| echo extension=sqlsrv.so >> `php --ini | grep "Scan for additional .ini files" | sed -e "s|.*:\s*||"`/20-sqlsrv.ini
$| exit
 ```

Verificamos la instalacion creando una base de datos de prueba utilizando la consola sqlcmd, ojo en los accesos 

```
$| sqlcmd -S tu_host -U sa -P tu_password -Q "CREATE DATABASE tu_base_datos_prueba;"
```

Luego creamos un archivo php de prueba que conecte al servidor sql:

```
$| vi /var/www/html/connect.php
```

Pegamos el siguiente codigo:

```php
<?php
    $serverName = "tu_host";
    $connectionOptions = array(
        "Database" => "tu_base_datos_prueba",
        "Uid" => "sa",
        "PWD" => "tu_password"
    );
    $conn = sqlsrv_connect($serverName, $connectionOptions);
    if($conn)
        echo "Conectado!"
?>
```

Ejecutamos el script:

```
php connect.php
```

El resultado debe ser:

```
Conectado!
```

## Composer

Descarga e instala composer mediante curl. Luego se habilita que el comando composer se ejecute de forma global:

```
$| sudo curl -sS https://getcomposer.org/installer | php
$| mv composer.phar /usr/bin/composer
```

Verificar instalacion de Composer:

```
$| composer -v
```

El output:

```
   ______
  / ____/___  ____ ___  ____  ____  ________  _____
 / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                    /_/
Composer version 1.8.6 2019-06-11 15:03:05

```

## Laravel

Se instala en la ruta "/var/www/html/":

```
$| cd /var/www/html/
```
Ahora se crea el proyecto laravel mediante composer create-project, el ultimo parametro indica el nombre del proyecto:
 
```
$| composer create-project --prefer-dist laravel/laravel nombreproyecto
```
 
Se modifican los permisos de ejecucion y lectura de las carpetas storage y bootstrap/cache. Si no se modifican laravel no corre:
 
```
$| chmod -R 775 /var/www/html/nombreproyecto
$| chown -R apache.apache /var/www/html/nombreproyecto
$| chmod -R 775 /var/www/html/nombreproyecto/storage/  
$| chmod -R 775 /var/www/html/nombreproyecto/bootstrap/cache
$| chcon -R -t httpd_sys_rw_content_t /var/www/html/testapp/storage/
$| chcon -R -t httpd_sys_rw_content_t /var/www/html/testapp/bootstrap/cache
```

Ahora se debe abrir y modificar el archivo de configuracion de Apache:

```
$| vi /etc/httpd/conf/httpd.conf
```

Primero se indica que la raiz del proyecto estara dentro de la carpeta public del proyecto laravel, se modifica la siguiente linea:

```
DocumentRoot "/var/www/html"
```

Por esta otra:
```
DocumentRoot "/var/www/html/nombreproyecto/public"
```

Luego se habilita el acceso al directorio "/var/www/html", se modifica la siguiente linea:

```
<Directory "/var/www/http"> 
   Options Indexes FollowSymLinks
   AllowOverride None
   Require all granted
</Directory>
```

Por esta otra:

```
<Directory "/var/www/http"> 
   Options Indexes FollowSymLinks
   AllowOverride All
   Require all granted
</Directory>
```

Se reinicia Apache para que los cambios tengan efecto:

```
$| sudo systemctl restart httpd
```

## Firewall

Se habilita puerto 80 en el firewall:

```
$| firewall-cmd --permanent --zone=public --add-port=80/tcp
```

Luego se reinicia el servicio:

```
$| firewall-cmd --reload
```

Ahora ingresando la IP del servidor en el browser, es posible visualizar la pagina de inicio de Laravel.
