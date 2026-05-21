# LMSGI_UD07_López_Alonso_Marco

## 1. Generación del Informe Dinámico (QWeb XML):
Para esta primera parte del proyecto voy a diseñar y programar la estructura XML de una plantilla de informe de factura personalizada para "WillmanTech S.L." utilizando la sintaxis de QWeb.
Y para esta primera parte voy a tener en cuenta los siguientes requisistos:
1. Sintaxis estructurada: El documento debe estar bien formado e incluir espacios de nombres correctos si fuera necesario.
2. Lógica embebida QWeb: Debe hacer uso de directivas de iteración (t-foreach) para desglosar dinámicamente las líneas de la factura (invoice_line_ids).
3. Lógica condicional: Debe implementar una condición (t-if) para ocultar la columna de "Descuento" en caso de que ninguna de las líneas de detalle de la factura lo contenga.
4. Data Binding: Utilizar la directiva t-field para mostrar de forma limpia campos como el número de factura (doc.name), fecha de emisión (doc.date) y el total neto (doc.amount_total).


## 2. Interoperabilidad de Datos (Extracción JSON/XML)
Ahora para esta segunda parte del trabajo voy a hacer que las facturas de "WillmanTech S.L." permitan incorporarse a la plataforma de la Agencia Tributaria o se sincronicen con sistemas contables externos de manera asíncrona y para eso voy a definir las estructuras de intercambio de datos.

## 3. Manual Técnico de Explotación - WillmanTech S.L.

**Normativa de referencia:** ISO/IEC/IEEE 26514:2022
**Sistema:** ERP/CRM WillmanTech
**Versión:** 1.0

### 1. Introducción y Arquitectura

El presente manual describe los procedimientos de operación, seguridad y despliegue para la infraestructura ERP/CRM de WillmanTech S.L. La topología lógica del sistema está basada en contenedores bajo un entorno **Docker Compose**, garantizando escalabilidad, modularidad y aislamiento de procesos.

La arquitectura activa se divide en dos módulos principales integrados:
* **Contenedor Web/App:** Ejecuta el motor del ERP y el servidor HTTP.
* **Contenedor SGBD:** Almacenamiento persistente operado mediante PostgreSQL.

### 2. Guía de Instalación y Reinstalación

Para inicializar el entorno productivo desde cero, el servidor anfitrión debe tener instalados Docker y Docker Compose.

1. Clonar el repositorio del proyecto en el servidor de explotación.
2. Definir las credenciales en el fichero de variables de entorno `.env`:
   - `POSTGRES_USER`: Usuario del SGBD.
   - `POSTGRES_PASSWORD`: Credencial cifrada.
   - `POSTGRES_DB`: Nombre de la base de datos de producción.
3. Ejecutar el comando para levantar el entorno de contenedores en segundo plano:
   `docker-compose up -d`
4. El sistema automatizará la descarga de imágenes, la creación de redes internas y el mapeo de puertos hacia la aplicación web (puerto estándar 8069).

### 3. Seguridad y Control de Acceso

La gestión de la información se rige por un esquema de control de acceso basado en roles (RBAC) para minimizar la superficie de exposición:

* **Administrador:** Acceso irrestricto a la configuración técnica, terminal de Docker, instalación de módulos y mantenimiento del SGBD.
* **Contable:** Permisos de escritura y lectura sobre transacciones financieras, emisión de facturas UBL y generación de reportes tributarios.
* **Comercial (Ventas):** Acceso acotado a la visualización del CRM, clientes asignados y estado de sus propias facturas. No puede alterar estados contables.

**Políticas de contraseñas:** Los usuarios deben establecer contraseñas de al menos 12 caracteres, mezclando alfanuméricos y símbolos, obligando a una rotación periódica cada 90 días.

### 4. Procedimiento de Backup y Restauración

La integridad de los datos de WillmanTech S.L. se asegura mediante volcados diarios del SGBD y de los volúmenes estáticos.

**Comando de Respaldo (Backup):**
Se utiliza la utilidad `pg_dump` dentro del contenedor del SGBD para generar un fichero comprimido personalizado.
`docker exec -t contenedor_db_postgres pg_dump -U $POSTGRES_USER -d $POSTGRES_DB -F c -f /var/lib/postgresql/data/backups/willmantech_backup_$(date +%F).dump`

**Comando de Restauración (Recovery):**
`docker exec -t contenedor_db_postgres pg_restore -U $POSTGRES_USER -d $POSTGRES_DB -1 /var/lib/postgresql/data/backups/willmantech_backup.dump`

### 5. Flujo Operativo de Facturación e Informes

El proceso operativo para generar la representación impresa de una factura en el sistema sigue un pipeline de transformación de datos de tres fases:

1. **Gestión de Datos (Interfaz):** El usuario valida un borrador de factura en la interfaz web. El sistema registra las transacciones en PostgreSQL.
2. **Transformación (HTML -> QWeb):** Al solicitar la impresión, el sistema invoca el motor QWeb. Este inyecta los datos de la base de datos dentro de la plantilla XML base (`report_invoice_willmantech.xml`), evaluando condiciones (ej. existencia de descuentos) y renderizando un documento HTML estructurado dinámicamente.
3. **Renderizado Final (wkhtmltopdf -> PDF):** El motor del ERP envía el documento HTML renderizado y la hoja de estilos subyacente a la utilidad binaria `wkhtmltopdf`. Esta librería interpreta el DOM y el CSS, imprimiendo visualmente el documento y devolviendo el archivo PDF final, que se descarga automáticamente en el equipo del usuario.
