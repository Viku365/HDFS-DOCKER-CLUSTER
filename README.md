# HDFS Docker Cluster Setup

Este repositorio contiene los archivos necesarios para levantar un clúster HDFS usando Docker. A continuación, se presenta una guía paso a paso basada en los problemas que encontramos y los procesos de configuración realizados para que funcione adecuadamente.

## Requisitos previos
- Docker y Docker Compose instalados en tu sistema.
- Asegúrate de tener acceso a una red estable.

## Paso 1: Clonar el repositorio
Clona este repositorio en tu máquina local ejecutando el siguiente comando:

```bash
$ git clone https://github.com/Viku365/hdfs-docker-cluster.git
$ cd hdfs-docker-cluster
```

## Paso 2: Configurar Docker Network
Asegúrate de que todos los contenedores que forman parte del clúster estén en la misma red. Revisa el archivo `docker-compose.yml` y verifica la sección de redes:

```yaml
networks:
  hadoop_network:
    driver: bridge
```

Todos los servicios deben estar conectados a `hadoop_network` para garantizar la comunicación.

## Paso 3: Verificar la Configuración del Metastore
Para configurar el metastore de Hive correctamente, actualiza el archivo `hive-site.xml`. Asegúrate de que los siguientes parámetros sean correctos:

- URL de conexión de la base de datos:
  ```xml
  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://metastore-host:3306/metastore_db?createDatabaseIfNotExist=true</value>
  </property>
  ```
- Credenciales de la base de datos:
  ```xml
  <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>metastore_user</value>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>metastore_password</value>
  </property>
- Driver JDBC:
  ```xml
  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.cj.jdbc.Driver</value>
  </property>
  ```

## Paso 4: Verificar la Red entre Contenedores
Para verificar que los contenedores se estén comunicando adecuadamente:

1. Ejecuta un contenedor interactivo para acceder al contenedor de Hive Metastore y verifica si puedes hacer ping al contenedor de MySQL:
   ```bash
   docker exec -it <hive-metastore-container-id> bash
   ping metastore-host
   ```

2. Si `ping` falla, verifica la configuración de red en `docker-compose.yml`.

## Paso 5: Asegurarse de que el Servicio de Base de Datos Esté Funcionando
Asegúrate de que la base de datos MySQL esté levantada y contenga la base de datos requerida.

- Verifica los logs del contenedor MySQL:
  ```bash
  docker logs <mysql-metastore-container-id>
  ```

- Conecta manualmente al contenedor para verificar que la base de datos exista:
  ```bash
  docker exec -it <mysql-metastore-container-id> mysql -u metastore_user -p
  SHOW DATABASES;
  ```

## Paso 6: Verificar los Logs de Hive Metastore
Es importante revisar los logs del metastore para diagnosticar problemas:

```bash
docker logs <hive-metastore-container-id>
```

Busca errores relacionados con la conexión a la base de datos, dependencias faltantes, o problemas de configuración.

## Paso 7: Instalar y Configurar el Driver JDBC
Asegúrate de que el driver JDBC esté disponible en el classpath de Hive. Para esto, puedes descargar el archivo `.jar` correspondiente y montarlo en el contenedor de Hive:

```yaml
volumes:
  - ./mysql-connector-java.jar:/opt/hive/lib/mysql-connector-java.jar
```

## Paso 8: Ejecutar las Migraciones del Metastore
Si el metastore no ha sido inicializado correctamente, debes ejecutar `schematool` para crear las tablas necesarias:

```bash
schematool -initSchema -dbType mysql
```

Reemplaza `mysql` con el tipo de base de datos que estés utilizando.

## Paso 9: Configurar URI de Hive Metastore
Finalmente, configura el URI del metastore en `hive-site.xml` para que HiveServer2 pueda conectarse correctamente:

```xml
<property>
  <name>hive.metastore.uris</name>
  <value>thrift://hive-metastore-host:9083</value>
</property>
```

Asegúrate de que el puerto `9083` esté abierto y accesible.

## Problemas Comunes
- **Namenode en Modo Seguro**: Si el Namenode está en modo seguro, asegúrate de que todos los DataNodes estén en funcionamiento y que se hayan registrado suficientes bloques.
- **Problemas de Conexión al Metastore**: Verifica que el `hive-site.xml` esté bien configurado y que no haya problemas de red entre los contenedores.

## Conclusión
Siguiendo estos pasos, deberías poder levantar un clúster HDFS funcional con Hive Metastore. Si encuentras problemas adicionales, verifica los logs de cada componente para identificar y solucionar errores.

Si tienes alguna pregunta o sugerencia, no dudes en crear un issue en este repositorio.

