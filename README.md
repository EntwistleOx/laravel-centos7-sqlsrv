# Laravel en CentOS7 con driver SQLServer

Guia rapida para instalar Laravel 5.8 en CentOS 7, utilizando Apache, PHP 7.2. y driver de SQLServer.

## Yum

Limpia el cache de Yum:

```
$ sudo yum clean all
```

Actualiza los paqueques del sistema:

```
$ sudo yum -y update
```

## Apache

Instala Apache:

```
$ sudo yum -y install httpd
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

```
