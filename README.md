# Laravel en CentOS7 con driver SQLServer

Guia rapida para instalar Laravel 5.8 en CentOS 7, utilizando Apache, PHP 7.2. y driver de SQLServer.

## Yum

Limpia el cache de Yum:

```
$| sudo yum clean all
```

Actualiza los paqueques del sistema:

```
$| sudo yum -y update
```

## Apache

Instala Apache:

```
$| sudo yum -y install httpd
```

Inicia Apache:

```
$ sudo systemctl start httpd
```

Habilita ejecución de Apache al inicio del sistema:

```
$ sudo systemctl enable httpd
```

Revisa el estado del servicio:

```
$ sudo systemctl status httpd
```

## Herramientas

Algunas herramientas necesarias:

```
$ sudo yum -y install wget vim net-tools unzip git 
```

## PHP 7.2

Instala PHP y modulos requeridos por Laravel y SQLServer:

```
$ wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
$ wget http://rpms.remirepo.net/enterprise/remi-release-7.rpm
$ rpm -Uvh remi-release-7.rpm epel-release-latest-7.noarch.rpm
$ yum install -y yum-utils
$ yum-config-manager --enable remi-php72
$ yum install -y php php-pdo php-xml php-pear php-devel php-common php-bcmath php-mbstring php-gd php-cli php-fpm re2c gcc-c++ gcc
```

Actualiza GCC para compilar los drivers PHP con PECL y PHP 7.2.

```
$ yum install -y centos-release-scl
$ yum-config-manager --enable rhel-server-rhscl-7-rpms
$ yum install -y devtoolset-7
$ scl enable devtoolset-7 bash
```

## SQL Server

Se indica a SELinux que permita conectar a apache a una BBDD a traves de SELinux:

```
$ setsebool -P httpd_can_network_connect_db 1
```

Instala driver ODBC y el CLI de SQLServer:

```
$ curl https://packages.microsoft.com/config/rhel/7/prod.repo > /etc/yum.repos.d/mssql-tools.repo
$ ACCEPT_EULA=Y yum install -y msodbcsql17 mssql-tools
$ yum remove unixODBC-utf16 unixODBC-utf16-devel #to avoid conflicts
$ ACCEPT_EULA=Y yum install -y msodbcsql17 mssql-tools
$ echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile
$ echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
$ source ~/.bashrc
$ yum install -y unixODBC-devel
 ```
 
Instala driver SQLServer para PHP:
 
 ```
$ pecl install sqlsrv
$ pecl install pdo_sqlsrv
$ echo extension=pdo_sqlsrv.so >> `php --ini | grep "Scan for additional .ini files" | sed -e "s|.*:\s*||"`/30-pdo_sqlsrv.ini
$ echo extension=sqlsrv.so >> `php --ini | grep "Scan for additional .ini files" | sed -e "s|.*:\s*||"`/20-sqlsrv.ini
 ```

## Composer

Descarga e instala composer mediante curl. Luego se habilita que el comando composer se ejecute de forma global:

```
$ sudo curl -sS https://getcomposer.org/installer | php
$ mv composer.phar /usr/bin/composer
chmod +x /usr/bin/composer
```

