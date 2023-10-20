# Configuración del Host

El primer paso es configurar el host. Esto se puede lograr a través de los siguientes comandos:

```bash
sudo apt install resolvconf
sudo nano /etc/resolvconf/resolv.conf.d/head
sudo systemctl status resolvconf.service
sudo systemctl restart systemd-resolved.service
```
# Respaldo de la Base de Datos

Para respaldar la base de datos, se puede utilizar el siguiente comando:

```bash
psql.exe -h db.server-inamhi.com -U postgres -d bandahm -a -f D:\Proyectos\Inamhi\respaldodb\bandahm_data.sql
```

# Exportación de Datos
La exportación de datos se puede realizar con el siguiente comando:
```bash
pg_dump.exe -U postgres -h localhost -p 5432 --data-only -F c -b -v -f "D:\Proyectos\Inamhi\respaldodb\bandahm_data.backup" bandahm
```
# Restauración de la Base de Datos
La restauración se puede realizar a través de los siguientes comandos:

```bash
pg_dump.exe --format=plain --no-owner --file=D:\Proyectos\Inamhi\respaldodb\bandahm_data.sql postgres://postgres:postgres@localhost:5432/bandahm
pg_restore.exe -U postgres -h db.server-inamhi.com -p 5432 -d bandahm "D:\Proyectos\Inamhi\respaldodb\bandahm_data.backup"
psql -U postgres -h db.server-inamhi.com -p 5432 -d bandahmf < "D:\Proyectos\Inamhi\respaldodb\bandahm_data.backup"
```
# Exportación de la Estructura de la Base de Datos
Se puede exportar la estructura de la base de datos con el siguiente comando:

```bash
pg_dump -U postgres -d bandahm --schema-only --file="D:\Proyectos\Inamhi\respaldodb\bandahm.sql"
```

# Distribución de Tablas y Creación de Llaves Foráneas
Para distribuir las tablas y crear llaves foráneas, se deben seguir los siguientes pasos:
1.- Separar todas las llaves foráneas en otro archivo:

```bash
D:\Proyectos\Inamhi\respaldodb\foreign.sql
```
2.- Analizar la base de datos y su estructura para generar la distribución de todas las tablas:
```bash
SELECT create_distributed_table('administrativo.provincias');
SELECT create_distributed_table('administrativo.cantones');
```

3.- En caso de problemas de compatibilidad con llaves foráneas, crear tablas referenciales:
```bash
SELECT create_reference_table('administrativo.provincias');
SELECT create_reference_table('administrativo.cantones');
```

4.- Ahora se puede realizar el alter para agregar una llave foránea:
```bash
ALTER TABLE administrativo.cantones
    ADD CONSTRAINT cant_fk FOREIGN KEY (id_provincia)
    REFERENCES administrativo.provincias(id_provincia);
```

# Preparación del Ambiente para Migración
Con los datos ya preparados, se procede a preparar el ambiente para la migración, a través de los siguientes pasos:
1.- Exportación de datos de la base de datos:
```bash
pg_dump.exe -U postgres -h localhost -p 5432 --data-only -F c -b -v -f "D:\Proyectos\Inamhi\respaldodb\bandahm_data.backup" bandahm
```
2.- Creación de la base de datos en el servidor y los nodos:
```bash
sudo -i -u postgres  createdb bandahm;
sudo -i -u postgres psql -d bandahm -c "CREATE EXTENSION citus;"
```
3.- Configuración de nodos:
```bash
sudo -i -u postgres psql -d bandahm -c "SELECT * from citus_set_coordinator_host('db.server-inamhi.com',5432);"
```
4.- Adición de nodos al servidor:
```bash
sudo -i -u postgres psql -d bandahm -c "SELECT * from citus_add_node('db.nodo1-inamhi.com',5432);"
sudo -i -u postgres psql -d bandahm -c "SELECT * from citus_add_node('db.nodo2-inamhi.com',5432);"
```
# Carga de Base de Datos y Migración de Datos
Para cargar la base de datos sin llaves foráneas, distribuir esquemas de tablas, cargar llaves foráneas y migrar los datos, se utilizan los siguientes comandos:

```bash
psql.exe -U postgres -h db.server-inamhi.com -p 5432 -d pruebasdb < "D:\Proyectos\Inamhi\respaldodb\bandahm.sql"
psql.exe -U postgres -h db.server-inamhi.com -p 5432 -d pruebasdb < "D:\Proyectos\Inamhi\respaldodb\distributed_table.sql"
psql.exe -U postgres -h db.server-inamhi.com -p 5432 -d pruebasdb < "D:\Proyectos\Inamhi\respaldodb\foreign.sql"
pg_restore.exe -U postgres -h db.server-inamhi.com -p 5432 -d pruebasdb "D:\Proyectos\Inamhi\respaldodb\bandahm_data.backup"
```






