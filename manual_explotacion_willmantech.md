# MANUAL TÉCNICO DE EXPLOTACIÓN (ISO/IEC/IEEE 26514)
1. Introducción y Arquitectura
Módulos Activos: sale (gestión comercial), account (facturación y asientos contables) y base (directorio relacional de clientes/proveedores).

Topología Lógica: Arquitectura de microservicios aislada mediante Docker Compose. El servidor de aplicaciones se comunica internamente con el motor de base de datos a través de una red virtual privada (bridge).

[Cliente Web] ──(Puerto 8069)──► [Contenedor Odoo] ──(Puerto 5432)──► [Contenedor PostgreSQL]
2. Guía de Instalación y Reinstalación
Pasos críticos para levantar o resetear el entorno desde la terminal del host:

Bash
# 1. Reset limpio / Reinstalación (Borra contenedores y volúmenes previos)
docker-compose down --volumes

# 2. Levantar el entorno desde cero en segundo plano
docker-compose up -d
Variables de Entorno Críticas: HOST=db, USER=odoo_admin, PASSWORD=Secured_DB_Pass_2026! (mapeadas en el contenedor de Odoo para enlazar con el SGBD).

Dependencias del SGBD: Imagen oficial de postgres:15-alpine con persistencia de datos vinculada a un volumen local del host (odoo-db-data).

3. Seguridad y Control de Acceso
Aplicación del Principio de Menor Privilegio (POLP) mediante control de accesos basado en roles (RBAC):

Matriz de Roles:

Administrador: Acceso total (CRUD), configuración de variables y modo desarrollador activo.

Contable: CRUD en Facturación (account). Bloqueado para modificar configuraciones del sistema.

Comercial: CRUD en Ventas y Clientes propios. Acceso denegado a balances contables.

Políticas de Seguridad: Contraseñas de mínimo 12 caracteres (alfanuméricos y símbolos), desconexión automática por inactividad tras 15 minutos y tráfico web obligatoriamente cifrado mediante TLS 1.3 / HTTPS.

4. Procedimiento de Backup y Restauración
Copia de Seguridad (Base de Datos + Almacén de Archivos):
Bash
# Backup del SGBD Relacional (PostgreSQL)
docker exec -t willmantech_erp_db pg_dump -U odoo_admin -d postgres -F c > /var/backups/db_willmantech.dump

# Backup del Almacén Asociado (Filestore: logos, firmas y adjuntos)
docker cp willmantech_erp_app:/var/lib/odoo/.local/share/Odoo/filestore/ /var/backups/filestore/
Restauración Crítica (Disaster Recovery):
Bash
# 1. Limpieza y restauración de la Base de Datos
docker exec -i willmantech_erp_db dropdb -U odoo_admin --if-exists postgres
docker exec -i willmantech_erp_db createdb -U odoo_admin postgres
docker exec -i willmantech_erp_db pg_restore -U odoo_admin -d postgres -v < /var/backups/db_willmantech.dump

# 2. Inyección del Filestore y reinicio del servicio
docker cp /var/backups/filestore/. willmantech_erp_app:/var/lib/odoo/.local/share/Odoo/filestore/
docker-compose restart
5. Flujo Operativo de Facturación e Informes
Flujo en Interfaz de Usuario:
El usuario crea el documento en Facturación > Facturas > Crear.

Al seleccionar el cliente, el sistema autocompeta los campos fiscales vía RPC.

Se cargan las líneas de servicio. Al pulsar "Confirmar", la factura pasa de borrador (Draft) a publicado (Posted) y genera su número secuencial único.

Pipeline de Renderizado PDF (Didáctico):
Cuando el usuario pulsa "Imprimir", los datos de PostgreSQL siguen este camino de transformación:

[Datos de Factura] ──► [Motor QWeb] ──► [Código HTML5] ──► [wkhtmltopdf] ──► [Archivo PDF]
Compilación QWeb: Odoo toma la plantilla XML, ejecuta la lógica embebida (los bucles t-foreach de las líneas y el condicional t-if del descuento) e inyecta los datos de la base de datos.

Generación HTML: El motor genera un documento de hipertexto HTML5 plano con todas las reglas de estilo CSS aplicadas.

Conversión Gráfica (wkhtmltopdf): El ERP envía el código HTML al binario wkhtmltopdf. Este componente levanta un navegador virtual invisible (Headless WebKit), "dibuja" la página internamente respetando fuentes y márgenes, y la compila en un archivo PDF binario que se envía al navegador del usuario.