# Movimiento de Datos
## 1. Realiza una exportación del esquema de SCOTT usando Oracle Data Pump con las siguientes condiciones:
### - Exporta tanto la estructura de las tablas como los datos de las mismas.

### - Excluye la tabla BONUS y los departamentos con menos de dos empleados.
### - Realiza una estimación previa del tamaño necesario para el fichero de exportación.
### - Programa la operación para dentro de 2 minutos.
### - Genera un archivo de log en el directorio raíz.

Creamos un directorio en la ruta /opt/oracle/mov_datos y le asignamos al usuario y grupo `oracle:oinstall` los permisos de escritura y lectura. Este directorio lo usaremos como destino para la exportación de datos.

```bash
debian@agb:~$ sudo mkdir /opt/oracle/mov_datos

debian@agb:~$ sudo chown oracle:oinstall /opt/oracle/mov_datos/
```

Crearemos un directorio de Oracle Data Pump llamado `DATA_PUMP_ALEX` en la ruta `/opt/oracle/mov_datos/`. Este directorio será la ubicación de trabajo para las operaciones de exportación que vayamos a hacer.

```sql
SQL> CREATE DIRECTORY DATA_PUMP_ALEX AS '/opt/oracle/mov_datos/';

Directorio creado.
```

Concedemos permisos de lectura y escritura en el directorio `DATA_PUMP_ALEX` al usuario SCOTT. Esto le permitirá a SCOTT realizar operaciones de exportación e importación de datos en ese directorio.

```sql
SQL> GRANT READ,WRITE ON DIRECTORY DATA_PUMP_ALEX TO SCOTT;

Concesion terminada correctamente.
```

Otorgamos el privilegio `DATAPUMP_EXP_FULL_DATABASE` al usuario SCOTT le permite realizar exportaciones completas de la base de datos utilizando Oracle Data Pump.

```sql
SQL> GRANT DATAPUMP_EXP_FULL_DATABASE TO SCOTT;

Concesion terminada correctamente.
```

Este es el comando de exportación utilizando la opción `ESTIMATE_ONLY=yes` para calcular el tamaño estimado del archivo de exportación sin realizar la exportación real de datos esta estimación se guarda en el archivo de registro `estimacion.log`.

Se estimaron los tamaños de los datos de las tablas "DEPT" y "EMP", resultando en 64 KB cada una.
La estimación total del tamaño de exportación fue de 128 KB.

```bash
debian@agb:~$ expdp SCOTT/TIGER DIRECTORY=DATA_PUMP_ALEX CONTENT=ALL LOGFILE=estimacion.log SCHEMAS=SCOTT ESTIMATE_ONLY=yes

Export: Release 19.0.0.0.0 - Production on Sat Mar 2 16:03:34 2024
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production

Advertencia: Las operaciones de Oracle Data Pump no se necesitan normalmente cuando se conecta a la raiz o al elemento inicial de una base de datos del contenedor.

Iniciando "SCOTT"."SYS_EXPORT_SCHEMA_01":  SCOTT/******** DIRECTORY=DATA_PUMP_ALEX CONTENT=ALL LOGFILE=estimacion.log SCHEMAS=SCOTT ESTIMATE_ONLY=yes 
Estimacion en curso mediante el metodo BLOCKS...
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/TABLE_DATA
.  estimado "SCOTT"."DEPT"                                 64 KB
.  estimado "SCOTT"."EMP"                                  64 KB
Estimacion total mediante el metodo BLOCKS: 128 KB
El trabajo "SCOTT"."SYS_EXPORT_SCHEMA_01" ha terminado correctamente en Sab Mar 2 16:03:41 2024 elapsed 0 00:00:05
```

Este comando realiza la exportación real de los datos del esquema SCOTT con las siguientes condiciones:
- Se excluye la tabla "BONUS" y los departamentos con menos de dos empleados utilizando la cláusula EXCLUDE y la cláusula QUERY.
- Guarda el archivo de datos exportado como `scott.dmp`.
- Registra la actividad en el archivo `scott.log`.

```bash
debian@agb:~$ expdp SCOTT/TIGER DIRECTORY=DATA_PUMP_ALEX DUMPFILE=scott.dmp CONTENT=ALL LOGFILE=scott.log SCHEMAS=SCOTT EXCLUDE=TABLE:"IN('BONUS')" QUERY=DEPT:'"WHERE DEPTNO IN (SELECT DEPTNO FROM EMP GROUP BY DEPTNO HAVING COUNT(*) >= 2)"'
 
Export: Release 19.0.0.0.0 - Production on Sat Mar 2 16:17:38 2024
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production

Advertencia: Las operaciones de Oracle Data Pump no se necesitan normalmente cuando se conecta a la raiz o al elemento inicial de una base de datos del contenedor.

Iniciando "SCOTT"."SYS_EXPORT_SCHEMA_01":  SCOTT/******** DIRECTORY=DATA_PUMP_ALEX DUMPFILE=scott.dmp CONTENT=ALL LOGFILE=scott.log SCHEMAS=SCOTT EXCLUDE=TABLE:IN('BONUS') QUERY=DEPT:"WHERE DEPTNO IN (SELECT DEPTNO FROM EMP GROUP BY DEPTNO HAVING COUNT(*) >= 2)" 
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/TABLE_DATA
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/INDEX_STATISTICS
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
Procesando el tipo de objeto SCHEMA_EXPORT/STATISTICS/MARKER
Procesando el tipo de objeto SCHEMA_EXPORT/DEFAULT_ROLE
Procesando el tipo de objeto SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/TABLE
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/COMMENT
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/INDEX/INDEX
. . "SCOTT"."DEPT"                              5.976 KB       2 filas exportadas
. . "SCOTT"."DUMMY"                             5.054 KB       1 filas exportadas
. . "SCOTT"."EMP"                                 8.5 KB       7 filas exportadas
. . "SCOTT"."SALGRADE"                          5.953 KB       5 filas exportadas
La tabla maestra "SCOTT"."SYS_EXPORT_SCHEMA_01" se ha cargado/descargado correctamente
******************************************************************************
El juego de archivos de volcado para SCOTT.SYS_EXPORT_SCHEMA_01 es:
  /opt/oracle/mov_datos/scott.dmp
El trabajo "SCOTT"."SYS_EXPORT_SCHEMA_01" ha terminado correctamente en Sab Mar 2 16:18:14 2024 elapsed 0 00:00:35
```

 Mostramos una lista de archivos en el directorio `/opt/oracle/mov_datos/` que incluye los archivos de registro `estimacion.log` y `scott.log`, asímismo el archivo de datos exportado `scott.dmp`.

```bash
root@agb:/opt/oracle/mov_datos# ls -hil
total 400K
415157 -rw-r--r-- 1 oracle oinstall  821 mar  2 16:03 estimacion.log
415158 -rw-r----- 1 oracle oinstall 392K mar  2 16:18 scott.dmp
415156 -rw-r--r-- 1 oracle oinstall 1,8K mar  2 16:18 scott.log
```

## 2. Importa el fichero obtenido anteriormente usando Oracle Data Pump pero en un usuario distinto de la misma base de datos.

En primer lugar crearemos un usuario `SCOTTY` con privilegios para conectarse y tener recursos sobre la base de datos, espacio ilimitado y permisos de lectura y escritura sobre el directorio creado antes.

```sql
SQL> CREATE USER SCOTTY IDENTIFIED BY SCOTTY;

Usuario creado.

SQL> GRANT CONNECT, RESOURCE TO SCOTTY;

Concesion terminada correctamente.

SQL> GRANT UNLIMITED TABLESPACE TO SCOTTY;

Concesion terminada correctamente.

SQL> GRANT READ,WRITE ON DIRECTORY DATA_PUMP_ALEX TO SCOTTY;

Concesion terminada correctamente.
```

Usaremos el comando `impdp` para importar los datos del archivo de la exportación anterior en el esquema de `SCOTTY` con el dump del directorio al que le hemos dado acceso, ademas generará un archivo de log en el mismo, y remapeando el origen con el esquema de `SCOTTY` a pesar del error que salta se importa con éxito.

```bash
debian@agb:~$ impdp SCOTTY/SCOTTY DIRECTORY=DATA_PUMP_ALEX DUMPFILE=scott.dmp LOGFILE=scotty.log SCHEMAS=SCOTT REMAP_SCHEMA=SCOTT:SCOTTY

Import: Release 19.0.0.0.0 - Production on Mon Mar 4 13:16:01 2024
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production

Advertencia: Las operaciones de Oracle Data Pump no se necesitan normalmente cuando se conecta a la raiz o al elemento inicial de una base de datos del contenedor.

La tabla maestra "SCOTTY"."SYS_IMPORT_SCHEMA_01" se ha cargado/descargado correctamente
Iniciando "SCOTTY"."SYS_IMPORT_SCHEMA_01":  SCOTTY/******** DIRECTORY=DATA_PUMP_ALEX DUMPFILE=scott.dmp LOGFILE=scotty.log SCHEMAS=SCOTT REMAP_SCHEMA=SCOTT:SCOTTY 
Procesando el tipo de objeto SCHEMA_EXPORT/DEFAULT_ROLE
ORA-31685: El tipo de objeto DEFAULT_ROLE:"SCOTTY" ha fallado debido a privilegios insuficientes. El sql que ha fallado es:
 ALTER USER "SCOTTY" DEFAULT ROLE ALL

Procesando el tipo de objeto SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/TABLE
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/TABLE_DATA
. . "SCOTTY"."DEPT"                             5.976 KB       2 filas importadas
. . "SCOTTY"."DUMMY"                            5.054 KB       1 filas importadas
. . "SCOTTY"."EMP"                                8.5 KB       7 filas importadas
. . "SCOTTY"."SALGRADE"                         5.953 KB       5 filas importadas
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
Procesando el tipo de objeto SCHEMA_EXPORT/STATISTICS/MARKER
El trabajo "SCOTTY"."SYS_IMPORT_SCHEMA_01" ha terminado con 1 error(es) en Lun Mar 4 13:16:14 2024 elapsed 0 00:00:12

root@agb:/opt/oracle/mov_datos# ls -l
total 404
-rw-r--r-- 1 oracle oinstall    821 mar  2 16:03 estimacion.log
-rw-r----- 1 oracle oinstall 401408 mar  2 16:18 scott.dmp
-rw-r--r-- 1 oracle oinstall   1815 mar  2 16:18 scott.log
-rw-r--r-- 1 oracle oinstall   1498 mar  4 13:16 scotty.log

```

Nos conoectamos con el usuario `SCOTTY` y probamos a ver que se han importado los departamentos con mas de dos empleados, los empleados, asimismo las tablas `dummy` y `salgrade` y se ha excluido la tabla `BONUS`.

```sql
debian@agb:~$ sqlplus SCOTTY/SCOTTY

SQL*Plus: Release 19.0.0.0.0 - Production on Mon Mar 4 13:19:23 2024
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle.  All rights reserved.

Hora de Ultima Conexion Correcta: Lun Mar 04 2024 13:17:50 +01:00

Conectado a:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.3.0.0.0

SQL> select * from dept;

    DEPTNO DNAME	  LOC
---------- -------------- -------------
	10 ACCOUNTING	  NEW YORK
	30 SALES	  CHICAGO

SQL> select * from emp;

     EMPNO ENAME      JOB	       MGR HIREDATE	   SAL	     COMM
---------- ---------- --------- ---------- -------- ---------- ----------
    DEPTNO
----------
      7499 ALLEN      SALESMAN	      7698 20/02/81	  1600	      300
	30

      7521 WARD       SALESMAN	      7698 22/02/81	  1250	      500
	30

      7654 MARTIN     SALESMAN	      7698 28/09/81	  1250	     1400
	30


     EMPNO ENAME      JOB	       MGR HIREDATE	   SAL	     COMM
---------- ---------- --------- ---------- -------- ---------- ----------
    DEPTNO
----------
      7698 BLAKE      MANAGER	      7839 01/05/81	  2850
	30

      7782 CLARK      MANAGER	      7839 09/06/81	  2450
	10

      7839 KING       PRESIDENT 	   17/11/81	  5000
	10


     EMPNO ENAME      JOB	       MGR HIREDATE	   SAL	     COMM
---------- ---------- --------- ---------- -------- ---------- ----------
    DEPTNO
----------
      7844 TURNER     SALESMAN	      7698 08/09/81	  1500		0
	30


7 filas seleccionadas.

SQL> select * from salgrade;

     GRADE	LOSAL	   HISAL
---------- ---------- ----------
	 1	  700	    1200
	 2	 1201	    1400
	 3	 1401	    2000
	 4	 2001	    3000
	 5	 3001	    9999

SQL> select * from dummy;

     DUMMY
----------
	 0

SQL> select * from bonus;
select * from bonus
              *
ERROR en linea 1:
ORA-00942: la tabla o vista no existe
```

## 3. Realiza una exportación de la estructura de todas las tablas de la base de datos usando el comando expdp de Oracle Data Pump probando al menos cinco de las posibles opciones que ofrece dicho comando y documentándolas adecuadamente.

Crearemos un usuario nuevo para este ejercicio con los privilegios anteriores y con el permiso de exportación, este esquema tendrá diversas tablas sobre una empresa de camiones.

```sql
SQL> CREATE USER EJ3 IDENTIFIED BY EJ3;

Usuario creado.

SQL> GRANT CONNECT, RESOURCE TO EJ3;

Concesion terminada correctamente.

SQL> GRANT UNLIMITED TABLESPACE TO EJ3;

Concesion terminada correctamente.

SQL> GRANT READ,WRITE ON DIRECTORY DATA_PUMP_ALEX TO EJ3;

Concesion terminada correctamente.

SQL> GRANT DATAPUMP_EXP_FULL_DATABASE TO EJ3;

Concesion terminada correctamente.
```

Realizamos una exportacion del esquema del usuario `EJ3`, indicamos:
- **DIRECTORY=DATA_PUMP_ALEX:** donde se almacenarán los archivos del volcado generados por la exportación, tanto el dump como el archivo de log.
- **DUMPFILE=ej3.dmp:** nombre del archivo del dump.
- **SCHEMAS=EJ3:** esquema del que se van a exportar los datos.
- **LOGFILE=ej3.log:** nombre del archivo de logs.
- **EXCLUDE=INDEX:** se excluirán los índices.
- **COMPRESSION=ALL:** los datos exportados se comprimirán.
- **PARALLEL=4:** la exportación usará 4 hilos o subprocesos par agilizarla.
- **JOB_NAME=EXPORTEJ3:** el trabajo de exportación tendrá un nombre.
- **CONTENT=ALL:** se exportará todo el contenido del esquema, tabla, datos y metadatos...

```bash
debian@agb:~$ expdp EJ3/EJ3 DIRECTORY=DATA_PUMP_ALEX DUMPFILE=ej3.dmp SCHEMAS=EJ3 LOGFILE=ej3.log EXCLUDE=INDEX COMPRESSION=ALL PARALLEL=4 JOB_NAME=EXPORTEJ3 CONTENT=ALL

Export: Release 19.0.0.0.0 - Production on Mon Mar 4 21:50:06 2024
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production

Advertencia: Las operaciones de Oracle Data Pump no se necesitan normalmente cuando se conecta a la raiz o al elemento inicial de una base de datos del contenedor.

Iniciando "EJ3"."EXPORTEJ3":  EJ3/******** DIRECTORY=DATA_PUMP_ALEX DUMPFILE=ej3.dmp SCHEMAS=EJ3 LOGFILE=ej3.log EXCLUDE=INDEX COMPRESSION=ALL PARALLEL=4 JOB_NAME=EXPORTEJ3 CONTENT=ALL 
Procesando el tipo de objeto SCHEMA_EXPORT/DEFAULT_ROLE
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
Procesando el tipo de objeto SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/TABLE_DATA
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/COMMENT
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/TABLE
Procesando el tipo de objeto SCHEMA_EXPORT/STATISTICS/MARKER
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/CONSTRAINT/REF_CONSTRAINT
. . "EJ3"."CAMION"                              5.195 KB       7 filas exportadas
. . "EJ3"."CAMION_CONDUCTOR"                    5.187 KB       7 filas exportadas
. . "EJ3"."CIUDAD"                              5.265 KB       8 filas exportadas
. . "EJ3"."CIUDAD_CAMION"                       5.187 KB       7 filas exportadas
. . "EJ3"."CONDUCTOR"                           6.242 KB       9 filas exportadas
. . "EJ3"."INGRESOS"                            4.898 KB       8 filas exportadas
. . "EJ3"."LINEAS"                              5.609 KB       8 filas exportadas
. . "EJ3"."PAGO"                                4.890 KB       8 filas exportadas
. . "EJ3"."PARQUE"                              5.421 KB       8 filas exportadas
. . "EJ3"."PEDIDO"                              4.984 KB       9 filas exportadas
. . "EJ3"."REMOLQUE"                            5.421 KB      24 filas exportadas
. . "EJ3"."REMOLQUE_CAMION"                     5.195 KB       7 filas exportadas
. . "EJ3"."REMOLQUE_CISTERNA"                   5.070 KB       8 filas exportadas
. . "EJ3"."REMOLQUE_FRIGORIFICO"                5.078 KB       8 filas exportadas
. . "EJ3"."REMOLQUE_NORMAL"                     4.960 KB       8 filas exportadas
. . "EJ3"."REMOLQUE_PARQUE"                     5.195 KB       8 filas exportadas
La tabla maestra "EJ3"."EXPORTEJ3" se ha cargado/descargado correctamente
******************************************************************************
El juego de archivos de volcado para EJ3.EXPORTEJ3 es:
  /opt/oracle/mov_datos/ej3.dmp
El trabajo "EJ3"."EXPORTEJ3" ha terminado correctamente en Lun Mar 4 21:50:38 2024 elapsed 0 00:00:30
```

Ahora para probar la importación crearemos un nuevo usuario.

```sql
SQL> CREATE USER EJ3IMP IDENTIFIED BY EJ3IMP;

Usuario creado.

SQL> GRANT CONNECT, RESOURCE TO EJ3IMP;

Concesion terminada correctamente.

SQL> GRANT UNLIMITED TABLESPACE TO EJ3IMP;

Concesion terminada correctamente.

SQL> GRANT READ,WRITE ON DIRECTORY DATA_PUMP_ALEX TO EJ3IMP;

Concesion terminada correctamente.
```

Y realizaremos la importación en el esquema del usuario creado antes.

```bash
debian@agb:~$ impdp EJ3IMP/EJ3IMP DIRECTORY=DATA_PUMP_ALEX DUMPFILE=ej3.dmp LOGFILE=ej3imp.log SCHEMAS=EJ3 REMAP_SCHEMA=EJ3:EJ3IMP

Import: Release 19.0.0.0.0 - Production on Mon Mar 4 21:58:11 2024
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production

Advertencia: Las operaciones de Oracle Data Pump no se necesitan normalmente cuando se conecta a la raiz o al elemento inicial de una base de datos del contenedor.

La tabla maestra "EJ3IMP"."SYS_IMPORT_SCHEMA_01" se ha cargado/descargado correctamente
Iniciando "EJ3IMP"."SYS_IMPORT_SCHEMA_01":  EJ3IMP/******** DIRECTORY=DATA_PUMP_ALEX DUMPFILE=ej3.dmp LOGFILE=ej3imp.log SCHEMAS=EJ3 REMAP_SCHEMA=EJ3:EJ3IMP 
Procesando el tipo de objeto SCHEMA_EXPORT/DEFAULT_ROLE
ORA-31685: El tipo de objeto DEFAULT_ROLE:"EJ3IMP" ha fallado debido a privilegios insuficientes. El sql que ha fallado es:
 ALTER USER "EJ3IMP" DEFAULT ROLE ALL

Procesando el tipo de objeto SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/TABLE
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/TABLE_DATA
. . "EJ3IMP"."CAMION"                           5.195 KB       7 filas importadas
. . "EJ3IMP"."CAMION_CONDUCTOR"                 5.187 KB       7 filas importadas
. . "EJ3IMP"."CIUDAD"                           5.265 KB       8 filas importadas
. . "EJ3IMP"."CIUDAD_CAMION"                    5.187 KB       7 filas importadas
. . "EJ3IMP"."CONDUCTOR"                        6.242 KB       9 filas importadas
. . "EJ3IMP"."INGRESOS"                         4.898 KB       8 filas importadas
. . "EJ3IMP"."LINEAS"                           5.609 KB       8 filas importadas
. . "EJ3IMP"."PAGO"                             4.890 KB       8 filas importadas
. . "EJ3IMP"."PARQUE"                           5.421 KB       8 filas importadas
. . "EJ3IMP"."PEDIDO"                           4.984 KB       9 filas importadas
. . "EJ3IMP"."REMOLQUE"                         5.421 KB      24 filas importadas
. . "EJ3IMP"."REMOLQUE_CAMION"                  5.195 KB       7 filas importadas
. . "EJ3IMP"."REMOLQUE_CISTERNA"                5.070 KB       8 filas importadas
. . "EJ3IMP"."REMOLQUE_FRIGORIFICO"             5.078 KB       8 filas importadas
. . "EJ3IMP"."REMOLQUE_NORMAL"                  4.960 KB       8 filas importadas
. . "EJ3IMP"."REMOLQUE_PARQUE"                  5.195 KB       8 filas importadas
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/CONSTRAINT/REF_CONSTRAINT
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
Procesando el tipo de objeto SCHEMA_EXPORT/STATISTICS/MARKER
El trabajo "EJ3IMP"."SYS_IMPORT_SCHEMA_01" ha terminado con 1 error(es) en Lun Mar 4 21:58:30 2024 elapsed 0 00:00:19
```

Comprobamos que se ha importado correctamente, conectándonos con el usuario y realizando algunas consultas.

```sql
SQL> connect EJ3IMP    
Introduzca la contraseña: 
Conectado.
SQL> select * from camion;

MATRICU FECHA_AL MARCA	      PESO_MAX
------- -------- ---------- ----------
9630KVF 05/10/11 EUROSTAR	 25000
7465IBL 28/12/15 IVECO		 20000
8484BKB 30/06/18 SCANIA 	 30000
2615KSA 11/08/10 VOLVO		 35000
9832BHT 21/05/09 NISSAN 	 18500
7530MPO 19/11/08 IVECO		 22000
9137JYE 05/10/08 IVECO		 18500

7 filas seleccionadas.

SQL> select * from ciudad;

CODIG NOMBRE		   COMUNIDADAUTONOMA		  CODIG
----- -------------------- ------------------------------ -----
12345 Madrid		   Madrid			  28001
23456 Barcelona 	   Cataluna			  08001
34567 Valencia		   Comunidad Valenciana 	  46001
45678 Sevilla		   Andalucia			  41004
56789 Zaragoza		   Aragon			  50003
67890 Malaga		   Andalucia			  29001
78901 Bilbao		   Pais Vasco			  48001
89012 Murcia		   Region De Murcia		  30002

8 filas seleccionadas.
```
## 4. Intenta realizar operaciones similares de importación y exportación con las herramientas proporcionadas con MySQL desde línea de comandos, documentando el proceso.

Realizaremos exportaciones usando el comando de `mysqldump`, con ello podremos realizar:
- Exportaciones completas.
- Solo estrucutra y sin datos (`--no-data`).
- Tabla en concreto (Conductor).
- Todas las bases de datos (`--all-databases`).

```bash
debian@agb:~/exp_mysql$ mysqldump -u alex -p camiones  > exp_camiones.sql
Enter password: 

debian@agb:~/exp_mysql$ mysqldump -u alex -p --no-data camiones > camiones_estructura.sql
Enter password: 

debian@agb:~/exp_mysql$ mysqldump -u alex -p camiones Conductor > camiones_conductor.sql
Enter password: 

debian@agb:~/exp_mysql$ mysqldump -u root -p --all-databases > todas_bd.sql
Enter password: 

debian@agb:~/exp_mysql$ ls -hil
total 2,6M
415169 -rw-r--r-- 1 debian debian 4,0K mar  4 22:12 camiones_conductor.sql
415171 -rw-r--r-- 1 debian debian  17K mar  4 22:12 camiones_estructura.sql
415151 -rw-r--r-- 1 debian debian  27K mar  4 22:09 exp_camiones.sql
415173 -rw-r--r-- 1 debian debian 2,5M mar  4 22:13 todas_bd.sql
```

Como bien vemos en el siguiente cuadro, los archivos son `.sql` y contienen todas las ordenes que se necesitan para importarse.

```bash
debian@agb:~/exp_mysql$ cat camiones_conductor.sql 
-- MariaDB dump 10.19  Distrib 10.11.4-MariaDB, for debian-linux-gnu (x86_64)
--
-- Host: localhost    Database: camiones
-- ------------------------------------------------------
-- Server version	10.11.4-MariaDB-1~deb12u1-log

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8mb4 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Table structure for table `Conductor`
--

DROP TABLE IF EXISTS `Conductor`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `Conductor` (
  `codigo` varchar(5) NOT NULL,
  `nombre` varchar(20) DEFAULT NULL,
  `apellido1` varchar(20) DEFAULT NULL,
  `apellido2` varchar(20) DEFAULT NULL,
  `dni` varchar(9) NOT NULL,
  `calle` varchar(20) DEFAULT NULL,
  `num_calle` varchar(5) DEFAULT NULL,
  `provincia` varchar(20) DEFAULT NULL,
  `poblacion` varchar(20) DEFAULT NULL,
  `telefono` varchar(9) NOT NULL,
  `correo` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`codigo`),
  UNIQUE KEY `uc_dni` (`dni`),
  CONSTRAINT `c_codigocond` CHECK (`codigo` regexp '[0-9]{5}'),
  CONSTRAINT `c_dni` CHECK (`dni` regexp '[0-9]{8}[A-Z]$'),
  CONSTRAINT `ck_dni` CHECK (cast(`dni` as char charset binary) = ucase(`dni`)),
  CONSTRAINT `c_calle` CHECK (cast(`calle` as char charset binary) = ucase(`calle`)),
  CONSTRAINT `c_telefono` CHECK (`telefono` regexp '^[679][0-9]{8}'),
  CONSTRAINT `c_correo` CHECK (`correo` regexp '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+.[A-Za-z]{2,}$'),
  CONSTRAINT `c_numcalle` CHECK (`num_calle` regexp '[0-9]{5}')
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `Conductor`
--

LOCK TABLES `Conductor` WRITE;
/*!40000 ALTER TABLE `Conductor` DISABLE KEYS */;
INSERT INTO `Conductor` VALUES
('00158','Sofia','Valassi','Moreno','85003495A','COCO','00115','Barcelona','Sitges','621685420','sofiavalasi-moreno@hotmail.cat'),
('08951','Jose','Fernandez','Murillo','85210684A','MAYOR','00025','Madrid','Madrid','695201597','josefermur@hotmail.es'),
('15974','Maria','Gutierrez','Nunez','82206485A','LOS ROSALES','00008','Sevilla','Dos Hermanas','635792048','mariaguati08@hotmail.com'),
('32058','Cesar','Paredes','Espinar','48620159W','SA TAULERA','00018','Islas Baleares','Palma','971520384','cesarpeesp-432@gmail.com'),
('45068','Maite','Antunez','Gonzalez','52300168X','CONSOLACION','00148','Sevilla ','Alcala De Guadaira','752156308','maiteangon8@hotmail.com'),
('52014','Juan','Perez','Rodriguez','63201598D','LA JARA','00025','Cadiz','Jerez De La Frontera','789201564','juanprodri025@gmail.es'),
('54321','Natalia','Rodriguez','Garcia','87654321B','SAN FRANCISCO','00021','Sevilla','Sevilla','971562018','natalyrodricia-234@outlook.org'),
('65204','Alex','Martin','Nunez','41578964D','QUEVEDO','00011','Sevilla','Los Palacios','658912044','alexmarnun435@outlook.com'),
('95320','Almudena','Garcia','Camacho','25301477D','FELIPE II','00015','Sevilla','Dos Hermanas','785220159','almugar@hotmail.com');
/*!40000 ALTER TABLE `Conductor` ENABLE KEYS */;
UNLOCK TABLES;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed on 2024-03-04 22:12:56
```


Probaremos también la importación, pero antes crearemos una base de datos donde importarla, un usuario con permisos sobre ella.

```sql
MariaDB [(none)]> CREATE DATABASE camionesimp;
Query OK, 1 row affected (0,000 sec)

MariaDB [(none)]> CREATE USER 'aleximp'@'localhost' IDENTIFIED BY 'aleximp';
Query OK, 0 rows affected (0,002 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON camionesimp.* TO 'aleximp'@'localhost';
Query OK, 0 rows affected (0,001 sec)
```

Y para realizar la importación lo haremos con el siguiente comando, como si de iniciar sesión se tratará pero agrgando al final `< /ruta/al/archivo.sql`.

```bash
debian@agb:~/exp_mysql$ mysql -u aleximp -p camionesimp < camiones_estructura.sql 
Enter password: 
```

Comprobaremos que se ha importado correctamente solo la estructura y no contienen filas de datos.

```sql
debian@agb:~$ mysql -u aleximp -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 47
Server version: 10.11.4-MariaDB-1~deb12u1-log Debian 12

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> use camionesimp;
Database changed

MariaDB [camionesimp]> show tables;
+-----------------------+
| Tables_in_camionesimp |
+-----------------------+
| Camion                |
| Camion_Conductor      |
| Ciudad                |
| Ciudad_Camion         |
| Conductor             |
| Ingresos              |
| Lineas                |
| Pago                  |
| Parque                |
| Pedido                |
| Remolque              |
| Remolque_Camion       |
| Remolque_Cisterna     |
| Remolque_Frigorifico  |
| Remolque_Normal       |
| Remolque_Parque       |
+-----------------------+
16 rows in set (0,000 sec)

MariaDB [camionesimp]> select * from Camion;
Empty set (0,000 sec)

MariaDB [camionesimp]> select * from Pedido;
Empty set (0,000 sec)
```

## 5. Intenta realizar operaciones similares de importación y exportación con las herramientas proporcionadas con Postgres desde línea de comandos, documentando el proceso.

Con Postgres usaremos el comando `pg_dump` donde podemos realizar exportaciones:
- Objetos de la base de datos sin dueño (lo será quien lo importe más adelante).
- Completas.
- Tablas en concreto.
- Todas las bases de datos.

```bash
postgres@agb:~/exp_psql$ pg_dump -U alex camiones -h localhost --no-owner > exp_camiones.sql
Contraseña: 

postgres@agb:~/exp_psql$ pg_dump -U alex -h localhost -s camiones > camiones_estructura.sql
Contraseña: 

postgres@agb:~/exp_psql$ pg_dump -U alex -h localhost --no-owner -t Conductor camiones > camiones_conductor.sql
Contraseña: 

postgres@agb:~/exp_psql$ pg_dumpall -U postgres > todas_bd.sql

postgres@agb:~/exp_psql$ ls -hil
total 180K
415209 -rw-r--r-- 1 postgres postgres 2,7K mar  4 23:14 camiones_conductor.sql
415178 -rw-r--r-- 1 postgres postgres  38K mar  4 23:14 camiones_estructura.sql
415208 -rw-r--r-- 1 postgres postgres  40K mar  4 23:10 exp_camiones.sql
415210 -rw-r--r-- 1 postgres postgres  93K mar  4 23:14 todas_bd.sql
```

Probaremos a importar una, pero antes crearemos un usuario y una base de datos de la que sea propietario.

```sql
postgres=# CREATE USER aleximp WITH PASSWORD 'aleximp';
CREATE ROLE
postgres=# CREATE DATABASE aleximp WITH OWNER = aleximp;
CREATE DATABASE
```

Una vez creados, importaremos al igual que con Mysql como si iniciáramos sesión pero indicando el archivo del `.sql` que queramos importar.

```bash
postgres@agb:~/exp_psql$ psql -U aleximp -d aleximp -h localhost < exp_camiones.sql
Contraseña para usuario aleximp: 
SET
SET
SET
SET
SET
 set_config 
------------
 
(1 fila)

SET
SET
SET
SET
CREATE SCHEMA
COMMENT
CREATE EXTENSION
COMMENT
CREATE FUNCTION
COMMENT
CREATE FUNCTION
CREATE FUNCTION
COMMENT
CREATE FUNCTION
COMMENT
CREATE PROCEDURE
CREATE PROCEDURE
CREATE FUNCTION
CREATE PROCEDURE
CREATE PROCEDURE
SET
SET
CREATE TABLE
...
```

Entonces una vez importado el archivo sql, entraremos con el usuario para comprobar que se ha importado la estructura de la base de datos, que ahora el usuario es el dueño y que no contienen filas de datos.

```sql
postgres@agb:~/exp_psql$ psql -U aleximp -h localhost
Contraseña para usuario aleximp: 
psql (15.5 (Debian 15.5-0+deb12u1))
Conexión SSL (protocolo: TLSv1.3, cifrado: TLS_AES_256_GCM_SHA384, compresión: desactivado)
Digite «help» para obtener ayuda.

aleximp=> \d
                Listado de relaciones
 Esquema |        Nombre        |   Tipo    |  Dueño  
---------+----------------------+-----------+---------
 public  | camion               | tabla     | aleximp
 public  | camion_conductor     | tabla     | aleximp
 public  | ciudad               | tabla     | aleximp
 public  | ciudad_camion        | tabla     | aleximp
 public  | conductor            | tabla     | aleximp
 public  | ingresos             | tabla     | aleximp
 public  | lineas               | tabla     | aleximp
 public  | mi_tabla             | tabla     | aleximp
 public  | mi_tabla_id_seq      | secuencia | aleximp
 public  | pago                 | tabla     | aleximp
 public  | parque               | tabla     | aleximp
 public  | pedido               | tabla     | aleximp
 public  | remolque             | tabla     | aleximp
 public  | remolque_camion      | tabla     | aleximp
 public  | remolque_cisterna    | tabla     | aleximp
 public  | remolque_frigorifico | tabla     | aleximp
 public  | remolque_normal      | tabla     | aleximp
 public  | remolque_parque      | tabla     | aleximp
(18 filas)

aleximp=> select * from camion;
 matricula | fecha_alta | marca | peso_max 
-----------+------------+-------+----------
(0 filas)
```

Crearemos otro donde importaremos el dump del que solo deberia contener la tabla de `Conductores`

```sql
postgres=# CREATE USER alexcond WITH PASSWORD 'alexcond';
CREATE ROLE
postgres=# CREATE DATABASE alexcond WITH OWNER = alexcond;
CREATE DATABASE
```

Realizamos la importación.

```bash
postgres@agb:~/exp_psql$ psql -U alexcond -d alexcond -h localhost < camiones_conductor.sql 
Contraseña para usuario alexcond: 
SET
SET
SET
SET
SET
 set_config 
------------
 
(1 fila)

SET
SET
SET
SET
SET
SET
CREATE TABLE
COPY 9
ALTER TABLE
ALTER TABLE
```

Entramos con el usuario creado y comprobamos que solo tiene la tabla de `conductor` e incluso con los datos.

```sql
postgres@agb:~/exp_psql$ psql -U alexcond -h localhost
Contraseña para usuario alexcond: 
psql (15.5 (Debian 15.5-0+deb12u1))
Conexión SSL (protocolo: TLSv1.3, cifrado: TLS_AES_256_GCM_SHA384, compresión: desactivado)
Digite «help» para obtener ayuda.

alexcond=> \d
         Listado de relaciones
 Esquema |  Nombre   | Tipo  |  Dueño   
---------+-----------+-------+----------
 public  | conductor | tabla | alexcond
(1 fila)

alexcond=> select * from conductor;
 codigo |  nombre  | apellido1 | apellido2 |    dni    |     calle     | num_calle |   provincia    |      poblacion       | telefono  |             correo             
--------+----------+-----------+-----------+-----------+---------------+-----------+----------------+----------------------+-----------+--------------------------------
 15974  | Maria    | Gutierrez | Nunez     | 82206485A | LOS ROSALES   | 00008     | Sevilla        | Dos Hermanas         | 635792048 | mariaguati08@hotmail.com
 95320  | Almudena | Garcia    | Camacho   | 25301477D | FELIPE II     | 00015     | Sevilla        | Dos Hermanas         | 785220159 | almugar@hotmail.com
 52014  | Juan     | Perez     | Rodriguez | 63201598D | LA JARA       | 00025     | Cadiz          | Jerez De La Frontera | 789201564 | juanprodri025@gmail.es
 65204  | Alex     | Martin    | Nunez     | 41578964D | QUEVEDO       | 00011     | Sevilla        | Los Palacios         | 658912044 | alexmarnun435@outlook.com
 00158  | Sofia    | Valassi   | Moreno    | 85003495A | COCO          | 00115     | Barcelona      | Sitges               | 621685420 | sofiavalasi-moreno@hotmail.cat
 32058  | Cesar    | Paredes   | Espinar   | 48620159W | SA TAULERA    | 00018     | Islas Baleares | Palma                | 971520384 | cesarpeesp-432@gmail.com
 45068  | Maite    | Antunez   | Gonzalez  | 52300168X | CONSOLACION   | 00148     | Sevilla        | Alcala De Guadaira   | 752156308 | maiteangon8@hotmail.com
 08951  | Jose     | Fernandez | Murillo   | 85210684A | MAYOR         | 00025     | Madrid         | Madrid               | 695201597 | josefermur@hotmail.es
 54321  | Natalia  | Rodriguez | Garcia    | 87654321B | SAN FRANCISCO | 00021     | Sevilla        | Sevilla              | 971562018 | natalyrodricia-234@outlook.org
(9 filas)
```


## 6. Exporta los documentos de una colección de MongoDB que cumplan una determinada condición e impórtalos en otra base de datos.

Tenemos un usuario y una base de datos llamada `camiones`, sobre está realizaremos una exportación de los documentos de la colección `Remolque` que cumplan la condición de que sean del modelo FORD.

```
vagrant@bullseye:~$ mongosh -u alex -p alex --authenticationDatabase camiones
Current Mongosh Log ID:	65e72e68b210e08a2f4cb57e
Connecting to:		mongodb://<credentials>@127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&authSource=camiones&appName=mongosh+2.1.5
Using MongoDB:		7.0.5
Using Mongosh:		2.1.5

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

------
   The server generated these startup warnings when booting
   2024-03-05T14:30:08.289+00:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
   2024-03-05T14:30:09.342+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
   2024-03-05T14:30:09.342+00:00: vm.max_map_count is too low
------

Enterprise test> use camiones
switched to db camiones
Enterprise camiones> show collections
camiones
Ciudad
Parque
Remolque
Remolque_Cisterna
Remolque_Frigorifico
Remolque_Normal
tablas
Enterprise camiones>  db.Remolque.find();
[
  {
    _id: ObjectId('65e732f2d4705c3d7ee241e6'),
    matricula: '1234ABC',
    modelo: 'FORD',
    peso: 11000,
    codigo_parque: '10000'
  },
  {
    _id: ObjectId('65e732f2d4705c3d7ee241e7'),
    matricula: '2345BCD',
    modelo: 'TOYOTA',
    peso: 11500,
    codigo_parque: '10001'
  },
  {
    _id: ObjectId('65e732f2d4705c3d7ee241e8'),
    matricula: '3456CDE',
    modelo: 'CHEVROLET',
    peso: 11300,
    codigo_parque: '10002'
  },
  {
    _id: ObjectId('65e732f2d4705c3d7ee241e9'),
    matricula: '4567DEF',
    modelo: 'RAM',
    peso: 11500,
    codigo_parque: '10003'
  },
  {
    _id: ObjectId('65e732f2d4705c3d7ee241ea'),
    matricula: '9012IJK',
    modelo: 'CHEVROLET',
    peso: 11000,
    codigo_parque: '10000'
  },
  {
    _id: ObjectId('65e732f2d4705c3d7ee241eb'),
    matricula: '0123JKL',
    modelo: 'RAM',
    peso: 11500,
    codigo_parque: '10001'
  },
  {
    _id: ObjectId('65e732f2d4705c3d7ee241ec'),
    matricula: '1234KLM',
    modelo: 'DODGE',
    peso: 12500,
    codigo_parque: '10002'
  },
  {
    _id: ObjectId('65e732f2d4705c3d7ee241ed'),
    matricula: '2345LMN',
    modelo: 'JEEP',
    peso: 11600,
    codigo_parque: '10003'
  },
  {
    _id: ObjectId('65e732f2d4705c3d7ee241ee'),
    matricula: '9524CFD',
    modelo: 'FORD',
    peso: 11000,
    codigo_parque: '10000'
  },
  {
    _id: ObjectId('65e732f2d4705c3d7ee241ef'),
    matricula: '9524ASF',
    modelo: 'TOYOTA',
    peso: 10500,
    codigo_parque: '10001'
  },
  {
    _id: ObjectId('65e732f2d4705c3d7ee241f0'),
    matricula: '7845OTN',
    modelo: 'CHEVROLET',
    peso: 11300,
    codigo_parque: '10002'
  },
  {
    _id: ObjectId('65e732f2d4705c3d7ee241f1'),
    matricula: '2514BGF',
    modelo: 'RAM',
    peso: 11500,
    codigo_parque: '10003'
  }
]
```

Para realizar la exportación usaremos el comando `mongodump` donde indicaremos:

- **--db camiones:** base de datos a exportar.

- **-u alex -p alex --authenticationDatabase camiones:** credenciales de usuario contraseña y base de datos de autenticación.

- **--collection Remolque:** exportación solo de la coleccion indicada.

- **--query '{"modelo": "FORD"}':** consulta que filtra los documentos a exportar, en este caso los que sean del modelo FORD.

- **--out /home/vagrant/:** ubicación de donde se almacenarán los archivos de la exportación.

```
vagrant@bullseye:~$ mongodump --db camiones -u alex -p alex --authenticationDatabase camiones --collection Remolque --query '{"modelo": "FORD"}' --out /home/vagrant/
2024-03-05T15:17:23.517+0000	writing camiones.Remolque to /home/vagrant/camiones/Remolque.bson
2024-03-05T15:17:23.518+0000	done dumping camiones.Remolque (2 documents)
vagrant@bullseye:~$ ls -hil camiones/
total 8.0K
1179690 -rw-r--r-- 1 vagrant vagrant 194 Mar  5 15:17 Remolque.bson
1179689 -rw-r--r-- 1 vagrant vagrant 554 Mar  5 15:17 Remolque.metadata.json
```

Para importarla usaremos el comando `mongorestore` :

- **--db camionesford:** base de datos donde se importaran los datos.
- **/home/vagrant/camiones:** ubicación del directorio que tiene almacenados los archivos de la exportación realizada.

```bash
vagrant@bullseye:~$ mongorestore --db camionesford /home/vagrant/camiones
2024-03-05T15:38:30.631+0000	The --db and --collection flags are deprecated for this use-case; please use --nsInclude instead, i.e. with --nsInclude=${DATABASE}.${COLLECTION}
2024-03-05T15:38:30.631+0000	building a list of collections to restore from /home/vagrant/camiones dir
2024-03-05T15:38:30.631+0000	reading metadata for camionesford.Remolque from /home/vagrant/camiones/Remolque.metadata.json
2024-03-05T15:38:30.656+0000	restoring camionesford.Remolque from /home/vagrant/camiones/Remolque.bson
2024-03-05T15:38:30.667+0000	finished restoring camionesford.Remolque (2 documents, 0 failures)
2024-03-05T15:38:30.667+0000	no indexes to restore for collection camionesford.Remolque
2024-03-05T15:38:30.667+0000	2 document(s) restored successfully. 0 document(s) failed to restore.
```

Accederemos a la base de datos `camionesimp` donde solo habrá una colección `Remolque` que contendrá solo los documentos que cumpla con la condición de que el modelo sea FORD.

```yaml
vagrant@bullseye:~$ mongosh -u alex -p alex --authenticationDatabase camiones
Current Mongosh Log ID:	65e73d115bb8d7dd4d819138
Connecting to:		mongodb://<credentials>@127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&authSource=camiones&appName=mongosh+2.1.5
Using MongoDB:		7.0.5
Using Mongosh:		2.1.5

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

------
   The server generated these startup warnings when booting
   2024-03-05T14:30:08.289+00:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
   2024-03-05T14:30:09.342+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
   2024-03-05T14:30:09.342+00:00: vm.max_map_count is too low
------

Enterprise test> use camionesford
switched to db camionesford
Enterprise camionesford> show collections
Remolque
Enterprise camionesford> db.Remolque.find()
[
  {
    _id: ObjectId('65e732f2d4705c3d7ee241e6'),
    matricula: '1234ABC',
    modelo: 'FORD',
    peso: 11000,
    codigo_parque: '10000'
  },
  {
    _id: ObjectId('65e732f2d4705c3d7ee241ee'),
    matricula: '9524CFD',
    modelo: 'FORD',
    peso: 11000,
    codigo_parque: '10000'
  }
]
```

## 7. SQL*Loader es una herramienta que sirve para cargar grandes volúmenes de datos en una instancia de ORACLE. Exportad los datos de una base de datos completa desde MariaDB a texto plano con delimitadores y emplead SQL*Loader para realizar el proceso de carga de dichos datos a una instancia ORACLE. Debéis documentar todo el proceso, explicando los distintos ficheros de configuración y de log que tiene SQL*Loader.

En primer lugar crearemos un entorno de trabajo donde almacenaremos las exportaciones de datos de cada una de las tablas.

```
debian@agb:/var/lib/mysql$ sudo mkdir tablascsv
```

Entraremos como root en Mariadb y con el siguiente comando realizaremos las exportaciones en formato de texto plano de los datos de cada una de las tablas.

Seleccionaremos todas las columnas de la tabla, indicaremos la ruta del archivo e indicaremos que los campos estarán separados por `,` comas.

```sql
SELECT * INTO OUTFILE '/ruta/datos.csv' FIELDS TERMINATED BY ',' FROM TABLA;
```

```sql
debian@agb:~$ sudo mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 33
Server version: 10.11.4-MariaDB-1~deb12u1-log Debian 12

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> use camiones
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [camiones]> SELECT * INTO OUTFILE '/var/lib/mysql/tablascsv/Camion.csv' FIELDS TERMINATED BY ',' FROM Camion;
Query OK, 8 rows affected (0,001 sec)

MariaDB [camiones]> SELECT * INTO OUTFILE '/var/lib/mysql/tablascsv/Camion.csv' FIELDS TERMINATED BY ',' FROM Camion;
Query OK, 8 rows affected (0,000 sec)

MariaDB [camiones]> 
MariaDB [camiones]> SELECT * INTO OUTFILE '/var/lib/mysql/tablascsv/Camion_Conductor.csv' FIELDS TERMINATED BY ',' FROM Camion_Conductor;
Query OK, 8 rows affected (0,000 sec)

MariaDB [camiones]> 
MariaDB [camiones]> SELECT * INTO OUTFILE '/var/lib/mysql/tablascsv/Ciudad.csv' FIELDS TERMINATED BY ',' FROM Ciudad;
Query OK, 8 rows affected (0,000 sec)

MariaDB [camiones]> 
MariaDB [camiones]> SELECT * INTO OUTFILE '/var/lib/mysql/tablascsv/Ciudad_Camion.csv' FIELDS TERMINATED BY ',' FROM Ciudad_Camion;
Query OK, 8 rows affected (0,000 sec)

MariaDB [camiones]> 
MariaDB [camiones]> SELECT * INTO OUTFILE '/var/lib/mysql/tablascsv/Conductor.csv' FIELDS TERMINATED BY ',' FROM Conductor;
Query OK, 9 rows affected (0,000 sec)

MariaDB [camiones]> 
MariaDB [camiones]> SELECT * INTO OUTFILE '/var/lib/mysql/tablascsv/Ingresos.csv' FIELDS TERMINATED BY ',' FROM Ingresos;
Query OK, 8 rows affected (0,000 sec)

MariaDB [camiones]> 
MariaDB [camiones]> SELECT * INTO OUTFILE '/var/lib/mysql/tablascsv/Lineas.csv' FIELDS TERMINATED BY ',' FROM Lineas;
Query OK, 8 rows affected (0,000 sec)

MariaDB [camiones]> 
MariaDB [camiones]> SELECT * INTO OUTFILE '/var/lib/mysql/tablascsv/Pago.csv' FIELDS TERMINATED BY ',' FROM Pago;
Query OK, 8 rows affected (0,000 sec)

MariaDB [camiones]> 
MariaDB [camiones]> SELECT * INTO OUTFILE '/var/lib/mysql/tablascsv/Parque.csv' FIELDS TERMINATED BY ',' FROM Parque;
Query OK, 8 rows affected (0,000 sec)

MariaDB [camiones]> 
MariaDB [camiones]> SELECT * INTO OUTFILE '/var/lib/mysql/tablascsv/Pedido.csv' FIELDS TERMINATED BY ',' FROM Pedido;
Query OK, 8 rows affected (0,000 sec)

MariaDB [camiones]> 
MariaDB [camiones]> SELECT * INTO OUTFILE '/var/lib/mysql/tablascsv/Remolque.csv' FIELDS TERMINATED BY ',' FROM Remolque;
Query OK, 24 rows affected (0,000 sec)

MariaDB [camiones]> 
MariaDB [camiones]> SELECT * INTO OUTFILE '/var/lib/mysql/tablascsv/Remolque_Camion.csv' FIELDS TERMINATED BY ',' FROM Remolque_Camion;
Query OK, 8 rows affected (0,000 sec)

MariaDB [camiones]> 
MariaDB [camiones]> SELECT * INTO OUTFILE '/var/lib/mysql/tablascsv/Remolque_Cisterna.csv' FIELDS TERMINATED BY ',' FROM Remolque_Cisterna;
Query OK, 8 rows affected (0,000 sec)

MariaDB [camiones]> 
MariaDB [camiones]> SELECT * INTO OUTFILE '/var/lib/mysql/tablascsv/Remolque_Frigorifico.csv' FIELDS TERMINATED BY ',' FROM Remolque_Frigorifico;
Query OK, 8 rows affected (0,000 sec)

MariaDB [camiones]> 
MariaDB [camiones]> SELECT * INTO OUTFILE '/var/lib/mysql/tablascsv/Remolque_Normal.csv' FIELDS TERMINATED BY ',' FROM Remolque_Normal;
Query OK, 8 rows affected (0,000 sec)

MariaDB [camiones]> 
MariaDB [camiones]> SELECT * INTO OUTFILE '/var/lib/mysql/tablascsv/Remolque_Parque.csv' FIELDS TERMINATED BY ',' FROM Remolque_Parque;
Query OK, 8 rows affected (0,001 sec)
```

Una vez hecho para cada una de las tablas tendremos todos los datos almacenados.

```
debian@agb:/var/lib/mysql/tablascsv$ ls -hil
total 64K
415863 -rw-r--r-- 1 mysql mysql 272 mar  5 18:02 Camion_Conductor.csv
415859 -rw-r--r-- 1 mysql mysql 277 mar  5 18:02 Camion.csv
415868 -rw-r--r-- 1 mysql mysql 272 mar  5 18:02 Ciudad_Camion.csv
415865 -rw-r--r-- 1 mysql mysql 252 mar  5 18:02 Ciudad.csv
415869 -rw-r--r-- 1 mysql mysql 986 mar  5 18:02 Conductor.csv
415870 -rw-r--r-- 1 mysql mysql  96 mar  5 18:02 Ingresos.csv
415871 -rw-r--r-- 1 mysql mysql 504 mar  5 18:02 Lineas.csv
415872 -rw-r--r-- 1 mysql mysql  96 mar  5 18:02 Pago.csv
415873 -rw-r--r-- 1 mysql mysql 403 mar  5 18:02 Parque.csv
415874 -rw-r--r-- 1 mysql mysql 216 mar  5 18:02 Pedido.csv
415876 -rw-r--r-- 1 mysql mysql 288 mar  5 18:02 Remolque_Camion.csv
415877 -rw-r--r-- 1 mysql mysql 226 mar  5 18:02 Remolque_Cisterna.csv
415875 -rw-r--r-- 1 mysql mysql 701 mar  5 18:02 Remolque.csv
415878 -rw-r--r-- 1 mysql mysql 174 mar  5 18:02 Remolque_Frigorifico.csv
415879 -rw-r--r-- 1 mysql mysql 136 mar  5 18:02 Remolque_Normal.csv
415880 -rw-r--r-- 1 mysql mysql 272 mar  5 18:02 Remolque_Parque.csv
```

Ahora pasamos a Oracle, donde previamente crearemos un usuario, con privielgios y espacio ilimitado, además que crearemos la estrucutra de la base de datos, creando las tablas con sus debidas restricciones y que estarán en principio vacias.

```sql
SQL> CREATE USER sqlloaderalex IDENTIFIED BY sqlloaderalex;

Usuario creado.

SQL> GRANT CONNECT, RESOURCE TO sqlloaderalex;

Concesion terminada correctamente.

SQL> GRANT UNLIMITED TABLESPACE TO sqlloaderalex;

Concesion terminada correctamente.

CREATE TABLE Camion (
	matricula varchar2(7),
	fecha_alta date NOT NULL,
	marca varchar2(10) DEFAULT 'IVECO',
	peso_max number(7,2) DEFAULT 18000.00,
CONSTRAINT pk_camion PRIMARY KEY (matricula),
CONSTRAINT nn_peso CHECK (peso_max IS NOT NULL),
CONSTRAINT c_matricula CHECK (REGEXP_LIKE(matricula, '[0-9]{4}[A-Z]{3}')),
CONSTRAINT ck_matricula CHECK (matricula=UPPER (matricula)),
CONSTRAINT c_marca CHECK (marca=UPPER (marca)),
CONSTRAINT c_fechacam CHECK (fecha_alta BETWEEN to_date ('2000-01-01', 'YYYY-MM-DD') AND to_date ('2080-12-31', 'YYYY-MM-DD')),
CONSTRAINT c_peso_max CHECK (peso_max>18000.00),
CONSTRAINT ck_peso_max CHECK (peso_max BETWEEN 18000.00 AND 35000.00)
);

CREATE TABLE Conductor (
	codigo varchar2(5),
	nombre varchar2(20),
	apellido1 varchar2(20),
	apellido2 varchar2(20),
	dni varchar2(9) NOT NULL,
	calle varchar2(20),
	num_calle varchar2(5),
	provincia varchar2(20),
	poblacion varchar2(20),
	telefono varchar2(9) NOT NULL,
	correo varchar(100),
CONSTRAINT pk_conductor PRIMARY KEY (codigo),
CONSTRAINT uc_dni UNIQUE (dni),
CONSTRAINT c_codigocond CHECK (REGEXP_LIKE(codigo, '[0-9]{5}')),
CONSTRAINT c_nombre CHECK (nombre=initcap (nombre)),
CONSTRAINT c_apellido1 CHECK (apellido1=initcap (apellido1)),
CONSTRAINT c_apellido2 CHECK (apellido2=initcap (apellido2)),
CONSTRAINT c_dni CHECK (REGEXP_LIKE(dni, '[0-9]{8}[A-Z]$')),
CONSTRAINT ck_dni CHECK (dni=UPPER (dni)),
CONSTRAINT c_calle CHECK (calle=UPPER (calle)),
CONSTRAINT c_provincia_condc CHECK (provincia=initcap (provincia)),
CONSTRAINT c_poblacion CHECK (poblacion=initcap (poblacion)),
CONSTRAINT c_telefono CHECK (REGEXP_LIKE(telefono, '^[679][0-9]{8}')),
CONSTRAINT c_correo CHECK (REGEXP_LIKE(correo, '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'))
);

CREATE TABLE Pedido (
	num_pedido varchar2(5),
	fecha date,
	cif varchar2(9) NOT NULL,
CONSTRAINT pk_pedido PRIMARY KEY (num_pedido),
CONSTRAINT uc_cif UNIQUE (cif),
CONSTRAINT c_fechacpes CHECK (fecha BETWEEN to_date ('01-01-2000', 'dd-mm-yyyy') AND to_date ('31-12-2080', 'dd-mm-yyyy')),
CONSTRAINT c_numpedi CHECK (REGEXP_LIKE(num_pedido, '[0-9]{5}')),
CONSTRAINT ck_cif CHECK (cif=UPPER (cif)),
CONSTRAINT c_cif CHECK (REGEXP_LIKE(cif, '[0-9]{8}[A-Z]$'))
);

CREATE TABLE Ingresos (
	num_ingreso varchar2(5),
	num_pedido varchar2(5),
CONSTRAINT pk_ingresos PRIMARY KEY (num_ingreso),
CONSTRAINT fk_ingresos foreign key (num_pedido) references Pedido (num_pedido),
CONSTRAINT ck_num_ing CHECK (num_ingreso=UPPER (num_ingreso)),
CONSTRAINT c_num_ing CHECK (REGEXP_LIKE(num_ingreso, '[0-9]{3}[A-Z]{2}'))
);

CREATE TABLE Pago (
	num_pago varchar2(5),
	codigo_conductor varchar2(5),
CONSTRAINT pk_pago PRIMARY KEY (num_pago),
CONSTRAINT fk_pago foreign key (codigo_conductor) references Conductor (codigo),
CONSTRAINT ck_num_pag CHECK (num_pago=UPPER (num_pago)),
CONSTRAINT c_num_pag CHECK (REGEXP_LIKE(num_pago, '[0-9]{3}[A-Z]{2}'))
);

CREATE TABLE Ciudad (
	codigo varchar2(5),
	nombre varchar2(20),
	comunidadautonoma varchar2(30),
	codigo_postal varchar2(5) NOT NULL,
CONSTRAINT pk_ciudad PRIMARY KEY (codigo),
CONSTRAINT uc_cp UNIQUE (codigo_postal),
CONSTRAINT c_codigopciu CHECK (REGEXP_LIKE(codigo_postal, '[0-9]{5}')),
CONSTRAINT c_codigociu CHECK (REGEXP_LIKE(codigo, '[0-9]{5}')),
CONSTRAINT c_nombreciud CHECK (nombre=initcap (nombre)),
CONSTRAINT c_comunidadautonoma CHECK (comunidadautonoma NOT IN ('Islas Baleares','Santa Cruz De Tenerife','Las Palmas','Ceuta','Melilla')),
CONSTRAINT c_comunidadautonoma_mays CHECK (comunidadautonoma=initcap (comunidadautonoma))
);

CREATE TABLE Parque (
	codigo varchar2(5),
	calle varchar2(50),
	num_calle varchar2(3),
	capacidad number(3),
	codigo_ciudad varchar2(5),
	nombre varchar2(20),
CONSTRAINT pk_parque PRIMARY KEY (codigo),
CONSTRAINT c_codigoparq CHECK (REGEXP_LIKE(codigo, '[0-9]{5}')),
CONSTRAINT c_nombrecalle CHECK (calle=initcap (calle)),
CONSTRAINT c_num_calle CHECK (REGEXP_LIKE(num_calle, '[0-9]{3}')),
CONSTRAINT fk_parque foreign key (codigo_ciudad) references Ciudad (codigo)
);

CREATE TABLE Remolque (
	matricula varchar2(7),
	modelo varchar2(10),
	peso number(7,2) DEFAULT 10000.00,
	codigo_parque varchar2(5),
CONSTRAINT pk_remolque PRIMARY KEY (matricula),
CONSTRAINT c_modelorem CHECK (modelo=UPPER (modelo)),
CONSTRAINT c_matricularem CHECK (REGEXP_LIKE(matricula, '[0-9]{4}[A-Z]{3}')),
CONSTRAINT ck_matricularem CHECK (matricula=UPPER (matricula)),
CONSTRAINT fk_codigo_parque foreign key (codigo_parque) references Parque (codigo),
CONSTRAINT c_peso CHECK (peso>10000)
);

CREATE TABLE Remolque_Cisterna (
	matricula_remolque varchar2(7),
	capacidad number(7,2),
	tipo_mercancia varchar2(12),
CONSTRAINT fk_remolque_cisterna foreign key (matricula_remolque) references Remolque (matricula),
CONSTRAINT ck_tipo_mercancia CHECK (tipo_mercancia=UPPER (tipo_mercancia)),
CONSTRAINT c_tipo_mercancia CHECK (tipo_mercancia IN ('PELIGROSO','NO PELIGROSO')),
CONSTRAINT c_capacidadremcis CHECK (capacidad BETWEEN 2000 AND 20000)
);

CREATE TABLE Remolque_Frigorifico (
	matricula_remolque varchar2(7),
	capacidad number(7,2),
	rango_temperatura number(3,1),
CONSTRAINT fk_remolque_frigorifico foreign key (matricula_remolque) references Remolque (matricula),
CONSTRAINT c_capacidadremfrig CHECK (capacidad BETWEEN 2000 AND 20000),
CONSTRAINT c_temp CHECK (rango_temperatura BETWEEN -30 AND 10)
);

CREATE TABLE Remolque_Normal (
	matricula_remolque varchar2(7),
	capacidad number(7,2),
CONSTRAINT fk_remolque_normal foreign key (matricula_remolque) references Remolque (matricula),
CONSTRAINT c_capacidadremnorm CHECK (capacidad BETWEEN 2000 AND 20000)
);

CREATE TABLE Lineas (
	codigo varchar2(5),
	volumen_carga number(7,2),
	fecha_partida date,
	fecha_destino date,
	codigo_ciudad_destino varchar2(5),
	codigo_ciudad_origen varchar2(5),
	matricula_remolque varchar2(7),
	num_pedido varchar2(5),
CONSTRAINT pk_lineas PRIMARY KEY (codigo),
CONSTRAINT fk_lineas_cd foreign key (codigo_ciudad_destino) references Ciudad (codigo),
CONSTRAINT fk_lineas_co foreign key (codigo_ciudad_origen) references Ciudad (codigo),
CONSTRAINT fk_lineas_mremol foreign key (matricula_remolque) references Remolque (matricula),
CONSTRAINT fk_lineas_npedido foreign key (num_pedido) references Pedido (num_pedido),
CONSTRAINT C_volumen_carga CHECK (volumen_carga BETWEEN 16000.00 AND 25000.00),
CONSTRAINT c_codigolin CHECK (REGEXP_LIKE(codigo, '[0-9]{5}')),
CONSTRAINT c_fechas CHECK(fecha_destino>fecha_partida)
);

CREATE TABLE Ciudad_Camion (
	camion_matricula varchar2(7),
	codigo_ciudad varchar2(5),
	fecha date,
	hora varchar2(8),
CONSTRAINT pk_ciudad_camion PRIMARY KEY (camion_matricula,codigo_ciudad),
CONSTRAINT fk_ciudad_camion_m foreign key (camion_matricula) references Camion (matricula),
CONSTRAINT fk_ciudad_camion_c foreign key (codigo_ciudad) references Ciudad (codigo),
CONSTRAINT c_fechacc CHECK (fecha BETWEEN to_date ('01-01-2000', 'dd-mm-yyyy') AND to_date ('31-12-2080', 'dd-mm-yyyy'))
);

CREATE TABLE Remolque_Parque (
	matricula_remolque varchar2(7),
	codigo_parque varchar2(5),
	fecha date,
	hora varchar2(8),
CONSTRAINT pk_remolque_parque PRIMARY KEY (matricula_remolque,codigo_parque),
CONSTRAINT fk_remolque_parque foreign key (matricula_remolque) references Remolque (matricula),
CONSTRAINT fk_remolque_parque_c foreign key (codigo_parque) references Parque (codigo),
CONSTRAINT c_fecharp CHECK (fecha BETWEEN to_date ('01-01-2000', 'dd-mm-yyyy') AND to_date ('31-12-2080', 'dd-mm-yyyy'))
);

CREATE TABLE Camion_Conductor (
	camion_matricula varchar2(7),
	codigo_conductor varchar2(5),
	fecha date,
	hora varchar2(8),
CONSTRAINT pk_camion_conductor PRIMARY KEY (camion_matricula,codigo_conductor),
CONSTRAINT fk_camion_conductor foreign key (camion_matricula) references Camion (matricula),
CONSTRAINT fk_camion_conductor2 foreign key (codigo_conductor) references Conductor (codigo),
CONSTRAINT c_fechacamcon CHECK (fecha BETWEEN to_date ('01-01-2000', 'dd-mm-yyyy') AND to_date ('31-12-2080', 'dd-mm-yyyy'))
);

CREATE TABLE Remolque_Camion (
	camion_matricula varchar2(7),
	remolque_matricula varchar2(7),
	fecha date,
	hora varchar2(8),
CONSTRAINT pk_Remolque_Camion PRIMARY KEY (camion_matricula,remolque_matricula),
CONSTRAINT fk_Remolque_Camion foreign key (camion_matricula) references Camion (matricula),
CONSTRAINT fk_Remolque_Camion2 foreign key (remolque_matricula) references Remolque (matricula),
CONSTRAINT c_fechacamrem CHECK (fecha BETWEEN to_date ('01-01-2000', 'dd-mm-yyyy') AND to_date ('31-12-2080', 'dd-mm-yyyy'))
);
```

Copiaremos todos los datos a una ruta algo más accesible para Oracle.

```
debian@agb:~/tablascsv$ sudo cp /var/lib/mysql/tablascsv/* .
debian@agb:~/tablascsv$ ls -hila
total 72K
415153 drwxr-xr-x  2 alex   alex   4,0K mar  5 18:35 .
 39903 drwx------ 10 debian debian 4,0K mar  5 17:52 ..
415881 -rw-r--r--  1 root   root    272 mar  5 18:35 Camion_Conductor.csv
415883 -rw-r--r--  1 root   root    277 mar  5 18:35 Camion.csv
415884 -rw-r--r--  1 root   root    272 mar  5 18:35 Ciudad_Camion.csv
415885 -rw-r--r--  1 root   root    252 mar  5 18:35 Ciudad.csv
415886 -rw-r--r--  1 root   root    986 mar  5 18:35 Conductor.csv
415887 -rw-r--r--  1 root   root     96 mar  5 18:35 Ingresos.csv
415888 -rw-r--r--  1 root   root    504 mar  5 18:35 Lineas.csv
415889 -rw-r--r--  1 root   root     96 mar  5 18:35 Pago.csv
415890 -rw-r--r--  1 root   root    403 mar  5 18:35 Parque.csv
415891 -rw-r--r--  1 root   root    216 mar  5 18:35 Pedido.csv
415892 -rw-r--r--  1 root   root    288 mar  5 18:35 Remolque_Camion.csv
415893 -rw-r--r--  1 root   root    226 mar  5 18:35 Remolque_Cisterna.csv
415894 -rw-r--r--  1 root   root    701 mar  5 18:35 Remolque.csv
415895 -rw-r--r--  1 root   root    174 mar  5 18:35 Remolque_Frigorifico.csv
415896 -rw-r--r--  1 root   root    136 mar  5 18:35 Remolque_Normal.csv
415897 -rw-r--r--  1 root   root    272 mar  5 18:35 Remolque_Parque.csv
```

Y para importar los datos con SQL*Loader primero de todo deberemos crear un archivo de control de carga, donde indicaremos:
- **LOAD DATA:** el comando que ejecuta la carga de datos.
- **INFILE '/home/debian/tablascsv/Camion.csv':** ubicación del archivo de datos en texto plano.
- **INTO TABLE camion:** indica en que tabla se cargarán los datos.
- **FIELDS TERMINATED BY ',':** indica que los campos de los datos están separados por comas.
- **TRAILING NULLCOLS:** las columnas no especificadas se cargarán como calor NULL.
- La última linea define los tipos de datos de la columna, en este caso especificamos un formato de fecha y de número con decimales.

```bash
debian@agb:~/tablascsv$ sudo nano camion.ctl
debian@agb:~/tablascsv$ cat camion.ctl 
LOAD DATA
INFILE '/home/debian/tablascsv/Camion.csv'
INTO TABLE camion
FIELDS TERMINATED BY ','
TRAILING NULLCOLS
(matricula, fecha_alta DATE "YYYY-MM-DD", marca, peso_max "to_number(:peso_max, '99999.99')")
```

Ahora sí con el siguiente comando cargaremos los datos indicando el archivo de control y una ruta donde irá el archivo de log, para revisiones posteriores.

```bash
debian@agb:~/tablascsv$ sqlldr sqlloaderalex/sqlloaderalex control=/home/debian/tablascsv/camion.ctl log=/home/debian/tablascsv/logs/camion.log

SQL*Loader: Release 19.0.0.0.0 - Production on Tue Mar 5 19:28:19 2024
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

Path used:      Conventional
Commit point reached - logical record count 8

Table CAMION:
  8 Rows successfully loaded.

Check the log file:
  /home/debian/tablascsv/logs/camion.log
for more information about the load.
```

Asimismo podemos ver el archivo de log que ha creado, donde nos muestra las ubicaciones de los archivos usados, los números de filas que cargará, los errores, una breve descripción en tabla de las columnas y tipo son, detalles del tiempo uso de CPU...

```bash
debian@agb:~/tablascsv$ cat logs/camion.log 

SQL*Loader: Release 19.0.0.0.0 - Production on Tue Mar 5 19:28:19 2024
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

Control File:   /home/debian/tablascsv/camion.ctl
Data File:      /home/debian/tablascsv/Camion.csv
  Bad File:     /home/debian/tablascsv/Camion.bad
  Discard File:  none specified
 
 (Allow all discards)

Number to load: ALL
Number to skip: 0
Errors allowed: 50
Bind array:     250 rows, maximum of 1048576 bytes
Continuation:    none specified
Path used:      Conventional

Table CAMION, loaded from every logical record.
Insert option in effect for this table: INSERT
TRAILING NULLCOLS option in effect

   Column Name                  Position   Len  Term Encl Datatype
------------------------------ ---------- ----- ---- ---- ---------------------
MATRICULA                           FIRST     *   ,       CHARACTER            
FECHA_ALTA                           NEXT     *   ,       DATE YYYY-MM-DD      
MARCA                                NEXT     *   ,       CHARACTER            
PESO_MAX                             NEXT     *   ,       CHARACTER            
    SQL string for column : "to_number(:peso_max, '99999.99')"


Table CAMION:
  8 Rows successfully loaded.
  0 Rows not loaded due to data errors.
  0 Rows not loaded because all WHEN clauses were failed.
  0 Rows not loaded because all fields were null.


Space allocated for bind array:                 258000 bytes(250 rows)
Read   buffer bytes: 1048576

Total logical records skipped:          0
Total logical records read:             8
Total logical records rejected:         0
Total logical records discarded:        0

Run began on Tue Mar 05 19:28:19 2024
Run ended on Tue Mar 05 19:28:19 2024

Elapsed time was:     00:00:00.04
CPU time was:         00:00:00.01
```

Para cada una de las tablas crearemos un archivo de control de carga de datos.

```bash
debian@agb:~/tablascsv$ cat conductor.ctl 
LOAD DATA
INFILE '/home/debian/tablascsv/Conductor.csv'
INTO TABLE conductor
FIELDS TERMINATED BY ','
TRAILING NULLCOLS
(codigo, nombre, apellido1, apellido2, dni, calle, num_calle, provincia, poblacion, telefono, correo)

debian@agb:~/tablascsv$ cat pedido.ctl 
LOAD DATA
INFILE '/home/debian/tablascsv/Pedido.csv'
INTO TABLE pedido
FIELDS TERMINATED BY ','
TRAILING NULLCOLS
(num_pedido, fecha DATE "YYYY-MM-DD", cif)

debian@agb:~/tablascsv$ cat ingresos.ctl 
LOAD DATA
INFILE '/home/debian/tablascsv/Ingresos.csv'
INTO TABLE ingresos
FIELDS TERMINATED BY ','
TRAILING NULLCOLS
(num_ingreso, num_pedido)

debian@agb:~/tablascsv$ cat pago.ctl 
LOAD DATA
INFILE '/home/debian/tablascsv/Pago.csv'
INTO TABLE pago
FIELDS TERMINATED BY ','
TRAILING NULLCOLS
(num_pago, codigo_conductor)

debian@agb:~/tablascsv$ cat ciudad.ctl 
LOAD DATA
INFILE '/home/debian/tablascsv/Ciudad.csv'
INTO TABLE ciudad
FIELDS TERMINATED BY ','
TRAILING NULLCOLS
(codigo, nombre, comunidadautonoma, codigo_postal)

debian@agb:~/tablascsv$ cat parque.ctl 
LOAD DATA
INFILE '/home/debian/tablascsv/Parque.csv'
INTO TABLE parque
FIELDS TERMINATED BY ','
TRAILING NULLCOLS
(codigo, calle, num_calle, capacidad, codigo_ciudad, nombre)

debian@agb:~/tablascsv$ cat remolque.ctl 
LOAD DATA
INFILE '/home/debian/tablascsv/Remolque.csv'
INTO TABLE remolque
FIELDS TERMINATED BY ','
TRAILING NULLCOLS
(matricula, modelo, peso "to_number(:peso, '99999.99')", codigo_parque)

debian@agb:~/tablascsv$ cat remolque_cisterna.ctl 
LOAD DATA
INFILE '/home/debian/tablascsv/Remolque_Cisterna.csv'
INTO TABLE remolque_cisterna
FIELDS TERMINATED BY ','
TRAILING NULLCOLS
(matricula_remolque, capacidad "to_number(:capacidad, '99999.99')", tipo_mercancia)

debian@agb:~/tablascsv$ cat remolque_frigorifico.ctl 
LOAD DATA
INFILE '/home/debian/tablascsv/Remolque_Frigorifico.csv'
INTO TABLE remolque_frigorifico
FIELDS TERMINATED BY ','
TRAILING NULLCOLS
(matricula_remolque, capacidad "to_number(:capacidad, '99999.99')", rango_temperatura "to_number(:rango_temperatura, '999.9')")

debian@agb:~/tablascsv$ cat remolque_normal.ctl 
LOAD DATA
INFILE '/home/debian/tablascsv/Remolque_Normal.csv'
INTO TABLE remolque_normal
FIELDS TERMINATED BY ','
TRAILING NULLCOLS
(matricula_remolque, capacidad "to_number(:capacidad, '99999.99')")

debian@agb:~/tablascsv$ cat lineas.ctl 
LOAD DATA
INFILE '/home/debian/tablascsv/Lineas.csv'
INTO TABLE lineas
FIELDS TERMINATED BY ','
TRAILING NULLCOLS
(codigo, volumen_carga "to_number(:volumen_carga, '99999.99')", fecha_partida DATE "YYYY-MM-DD", fecha_destino DATE "YYYY-MM-DD", codigo_ciudad_destino, codigo_ciudad_origen, matricula_remolque, num_pedido)

debian@agb:~/tablascsv$ cat ciudad_camion.ctl 
LOAD DATA
INFILE '/home/debian/tablascsv/Ciudad_Camion.csv'
INTO TABLE ciudad_camion
FIELDS TERMINATED BY ','
TRAILING NULLCOLS
(camion_matricula, codigo_ciudad, fecha DATE "YYYY-MM-DD", hora)

debian@agb:~/tablascsv$ cat remolque_parque.ctl 
LOAD DATA
INFILE '/home/debian/tablascsv/Remolque_Parque.csv'
INTO TABLE remolque_parque
FIELDS TERMINATED BY ','
TRAILING NULLCOLS
(matricula_remolque, codigo_parque, fecha DATE "YYYY-MM-DD", hora)

debian@agb:~/tablascsv$ cat camion_conductor.ctl 
LOAD DATA
INFILE '/home/debian/tablascsv/Camion_Conductor.csv'
INTO TABLE camion_conductor
FIELDS TERMINATED BY ','
TRAILING NULLCOLS
(camion_matricula, codigo_conductor, fecha DATE "YYYY-MM-DD", hora)

debian@agb:~/tablascsv$ cat remolque_camion.ctl 
LOAD DATA
INFILE '/home/debian/tablascsv/Remolque_Camion.csv'
INTO TABLE remolque_camion
FIELDS TERMINATED BY ','
TRAILING NULLCOLS
(camion_matricula, remolque_matricula, fecha DATE "YYYY-MM-DD", hora)
```

Y deberemos de cargarlas cada una.

```bash
sqlldr sqlloaderalex/sqlloaderalex control=/home/debian/tablascsv/conductor.ctl log=/home/debian/tablascsv/logs/conductor.log

sqlldr sqlloaderalex/sqlloaderalex control=/home/debian/tablascsv/pedido.ctl log=/home/debian/tablascsv/logs/pedido.log

sqlldr sqlloaderalex/sqlloaderalex control=/home/debian/tablascsv/ingresos.ctl log=/home/debian/tablascsv/logs/ingresos.log

sqlldr sqlloaderalex/sqlloaderalex control=/home/debian/tablascsv/pago.ctl log=/home/debian/tablascsv/logs/pago.log

sqlldr sqlloaderalex/sqlloaderalex control=/home/debian/tablascsv/ciudad.ctl log=/home/debian/tablascsv/logs/ciudad.log

sqlldr sqlloaderalex/sqlloaderalex control=/home/debian/tablascsv/parque.ctl log=/home/debian/tablascsv/logs/parque.log

sqlldr sqlloaderalex/sqlloaderalex control=/home/debian/tablascsv/remolque.ctl log=/home/debian/tablascsv/logs/remolque.log

sqlldr sqlloaderalex/sqlloaderalex control=/home/debian/tablascsv/remolque.ctl log=/home/debian/tablascsv/logs/remolque.log

sqlldr sqlloaderalex/sqlloaderalex control=/home/debian/tablascsv/remolque_cisterna.ctl log=/home/debian/tablascsv/logs/remolque_cisterna.log

sqlldr sqlloaderalex/sqlloaderalex control=/home/debian/tablascsv/remolque_frigorifico.ctl log=/home/debian/tablascsv/logs/remolque_frigorifico.log

sqlldr sqlloaderalex/sqlloaderalex control=/home/debian/tablascsv/remolque_normal.ctl log=/home/debian/tablascsv/logs/remolque_normal.log

sqlldr sqlloaderalex/sqlloaderalex control=/home/debian/tablascsv/lineas.ctl log=/home/debian/tablascsv/logs/lineas.log

sqlldr sqlloaderalex/sqlloaderalex control=/home/debian/tablascsv/ciudad_camion.ctl log=/home/debian/tablascsv/logs/ciudad_camion.log

sqlldr sqlloaderalex/sqlloaderalex control=/home/debian/tablascsv/remolque_parque.ctl log=/home/debian/tablascsv/logs/remolque_parque.log

sqlldr sqlloaderalex/sqlloaderalex control=/home/debian/tablascsv/camion_conductor.ctl log=/home/debian/tablascsv/logs/camion_conductor.log

sqlldr sqlloaderalex/sqlloaderalex control=/home/debian/tablascsv/remolque_camion.ctl log=/home/debian/tablascsv/logs/remolque_camion.log
```

Finalmente accederemos con el usuario creado anteriormente, y comprobaremos que los datos se han cargado con éxito.

```sql
SQL> connect sqlloaderalex/sqlloaderalex
Conectado.
SQL> select * from camion;

MATRICU FECHA_AL MARCA	      PESO_MAX
------- -------- ---------- ----------
2615KSA 11/08/10 VOLVO		 35000
5941KBF 15/02/08 IVECO		 18500
7465IBL 28/12/15 IVECO		 20000
7530MPO 19/11/08 IVECO		 22000
8484BKB 30/06/18 SCANIA 	 30000
9137JYE 05/10/08 IVECO		 18500
9630KVF 05/10/11 EUROSTAR	 25000
9832BHT 21/05/09 NISSAN 	 18500

8 filas seleccionadas.

SQL> select * from ciudad;

CODIG NOMBRE		   COMUNIDADAUTONOMA		  CODIG
----- -------------------- ------------------------------ -----
12345 Madrid		   Madrid			  28001
23456 Barcelona 	   Cataluna			  08001
34567 Valencia		   Comunidad Valenciana 	  46001
45678 Sevilla		   Andalucia			  41004
56789 Zaragoza		   Aragon			  50003
67890 Malaga		   Andalucia			  29001
78901 Bilbao		   Pais Vasco			  48001
89012 Murcia		   Region De Murcia		  30002

8 filas seleccionadas.

SQL> select * from pago;

NUM_P CODIG
----- -----
005EZ 00158
006FZ 08951
001AZ 15974
007GZ 32058
002BZ 52014
008HZ 54321
003CZ 65204
004DZ 65204

8 filas seleccionadas.

SQL> select * from remolque_camion;

CAMION_ REMOLQU FECHA	 HORA
------- ------- -------- --------
2615KSA 8468UTT 05/09/22 12:00:00
5941KBF 1234ABC 01/05/22 08:00:00
7465IBL 9524ASF 03/07/22 10:00:00
7530MPO 3456MNO 07/11/22 14:00:00
8484BKB 7845OTN 04/08/22 11:00:00
9137JYE 9524CFD 08/12/22 15:00:00
9630KVF 6789FGH 02/06/22 09:00:00
9832BHT 4567DEF 06/10/22 13:00:00

8 filas seleccionadas.
```