# Configuración de phpMyAdmin para trabajar con MySQL en XAMPP y MySQL Workbench

Este repositorio explica cómo configurar phpMyAdmin para trabajar simultáneamente con dos instalaciones de MySQL: la incluida en XAMPP (configurada en el puerto 3307) y MySQL Workbench (puerto estándar 3306).

##Propuesta1.txt PLanteada verificar

## Problema

Cuando tienes instalados tanto XAMPP como MySQL Workbench en el mismo sistema, suelen producirse conflictos porque ambos intentan usar el puerto 3306 por defecto. La solución es modificar el puerto de uno de ellos (en este caso, XAMPP a 3307) y configurar phpMyAdmin para que pueda conectarse a ambos servidores.

## Verificación de puertos

Primero, verifica que ambos servidores están activos y escuchando en sus respectivos puertos:


```bash
netstat -ano | findstr 3306
netstat -ano | findstr 3307
```

## Configuración del archivo my.ini de XAMPP

Para cambiar el puerto de MySQL en XAMPP:

1. Abrir el archivo `C:/xampp/mysql/bin/my.ini`
2. Buscar la línea que dice `port=3306` y cambiarla a `port=3307`
3. Guardar el archivo y reiniciar el servicio de MySQL en XAMPP

## Configuración de phpMyAdmin (config.inc.php)

Este es el archivo de configuración que permite a phpMyAdmin conectarse a ambos servidores:

```php
<?php
/* 
 * Clave para la encriptación de cookies
 */
$cfg['blowfish_secret'] = 'algoaleatorioysupersecreto123456';

/*
 * Configuración de servidores
 */
$i = 0;

/* Servidor #1 - MySQL XAMPP (3307) */
$i++;
$cfg['Servers'][$i]['verbose'] = 'MySQL XAMPP (3307)';
$cfg['Servers'][$i]['host'] = 'localhost';
$cfg['Servers'][$i]['port'] = '3307';
$cfg['Servers'][$i]['socket'] = '';
$cfg['Servers'][$i]['connect_type'] = 'tcp';
$cfg['Servers'][$i]['extension'] = 'mysqli';
$cfg['Servers'][$i]['auth_type'] = 'cookie';
$cfg['Servers'][$i]['user'] = 'root';
$cfg['Servers'][$i]['password'] = '';
$cfg['Servers'][$i]['AllowNoPassword'] = true;

/* Servidor #2 - MySQL Workbench (3306) */
$i++;
$cfg['Servers'][$i]['verbose'] = 'MySQL Workbench (3306)';
$cfg['Servers'][$i]['host'] = '127.0.0.1';
$cfg['Servers'][$i]['port'] = '3306';
$cfg['Servers'][$i]['socket'] = '';
$cfg['Servers'][$i]['connect_type'] = 'tcp';
$cfg['Servers'][$i]['extension'] = 'mysqli';
$cfg['Servers'][$i]['auth_type'] = 'cookie';
$cfg['Servers'][$i]['user'] = 'root';
$cfg['Servers'][$i]['password'] = 'Aroman1984';
$cfg['Servers'][$i]['AllowNoPassword'] = false;

/* Configuraciones generales */
$cfg['Lang'] = '';
$cfg['ForceSSL'] = false;
$cfg['PmaAbsoluteUri'] = '';
$cfg['CookieSameSite'] = 'Lax';
$cfg['CookieSecure'] = false;
?>
```

## Solución a problemas comunes

### Error de cookie de sesión

Si aparece el error: "Failed to set session cookie. Maybe you are using HTTP instead of HTTPS to access phpMyAdmin", asegúrate de incluir:

```php
$cfg['ForceSSL'] = false;
$cfg['CookieSecure'] = false;
```

### Error de acceso denegado para usuario 'pma'

Este error ocurre cuando phpMyAdmin está configurado para usar características avanzadas pero no puede acceder al usuario de control.

Soluciones:
1. **Solución recomendada**: Eliminar las referencias al usuario de control y características avanzadas (como se muestra en la configuración de arriba)
2. O bien, crear el usuario 'pma' en MySQL:

```sql
CREATE USER 'pma'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON phpmyadmin.* TO 'pma'@'localhost';
FLUSH PRIVILEGES;
```

### Errores de conexión con el segundo servidor

Si puedes conectarte al servidor de XAMPP pero no al de MySQL Workbench:

1. Verifica que el servidor de MySQL Workbench está activo
2. Comprueba las credenciales de acceso
3. Asegúrate de usar '127.0.0.1' en lugar de 'localhost' para el segundo servidor
4. Confirma que el usuario 'root' de MySQL Workbench tiene los permisos correctos:

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'tu_contraseña';
FLUSH PRIVILEGES;
```

## Consejos adicionales

- **Problema persistente**: Si los problemas persisten, prueba con una configuración mínima eliminando todas las características avanzadas
- **Archivos temporales**: Asegúrate de que phpMyAdmin tenga acceso de escritura al directorio temporal
- **Comprobación directa**: Verifica que puedes conectarte directamente a ambos servidores usando el cliente MySQL en línea de comandos:

```bash
# Conexión a XAMPP MySQL
mysql -h localhost -P 3307 -u root -p

# Conexión a MySQL Workbench
mysql -h 127.0.0.1 -P 3306 -u root -p
```

## Referencias

- [Documentación oficial de phpMyAdmin](https://www.phpmyadmin.net/docs/)
- [Configuración de XAMPP](https://www.apachefriends.org/docs/)
- [MySQL Workbench Manual](https://dev.mysql.com/doc/workbench/en/)
