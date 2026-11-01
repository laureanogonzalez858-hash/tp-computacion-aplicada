# Trabajo práctico integrador — Computación Aplicada

**Alumno:** Gonzalez Laureano

Utilicé Debian (`TPServer`) para publicar la página de alumnos y preparar los respaldos. La página y el logo funcionaron desde Windows.

## Configuración y respaldos

La interfaz puente `enp0s8` tiene la dirección fija `192.168.1.35/24` y la puerta de enlace `192.168.1.1`. Accedí por SSH como `tpadmin` mediante el puerto `2223` de Windows, reenviado al `22` de la VM. Diagrama: `diagrama_red.svg`.

Apache tiene configurado `/www_dir` como directorio del sitio. El disco adicional es de 10 GiB, con dos particiones Linux tipo 83: aproximadamente 3 GB para `/www_dir` y 6 GB para `/backup_dir`. Ambas usan ext4 y están configuradas en `fstab` mediante sus etiquetas.

El script `/opt/scripts/backup_full.sh` recibe origen y destino, ofrece ayuda con `-help` y comprueba los montajes. Cron está configurado para respaldar `/var/log` diariamente a las 00:00 y `/www_dir` los lunes, miércoles y viernes a las 23:00. Las pruebas manuales dieron código 0; revisé la integridad y el contenido de los comprimidos.

## Archivos incluidos

| Archivo | Contenido |
|---|---|
| `root.tar.gz`, `etc.tar.gz` | Archivos de root y configuración del sistema |
| `opt.tar.gz` | Script de respaldo |
| `proc.tar.gz` | Captura parcial de información del sistema |
| `www_dir.tar.gz` | Página y logo |
| `backup_dir.tar.gz` | Copias de los respaldos de la web y los logs |
| `var.tar.gz.part-00`, `01`, `02` | `/var` dividido en tres partes |

## Aclaraciones

Para esta entrega se excluyeron credenciales, claves privadas, hashes de contraseñas, historiales, contenido de logs y datos crudos de MariaDB. Por eso estas copias **no permiten restaurar el sistema completo**; la aceptación de estas exclusiones queda pendiente del docente. En los comprimidos reconstruidos también se normalizaron los propietarios, las fechas y otros metadatos; `opt.tar.gz` y `proc.tar.gz` se conservaron sin cambios.

`proc.tar.gz` contiene una captura parcial: `particion` es una copia de `partitions` dentro del comprimido. No se creó un archivo en el `/proc` real. Esta alternativa también queda pendiente de confirmación del docente.

La copia de `index.php` requiere la variable `TP_DB_PASSWORD`; si falta, responde con error 503. Estos cambios afectan únicamente los archivos de entrega, no la VM.

Falta comprobar una ejecución automática del respaldo, el acceso SSH de root mediante clave, la versión de PHP y dejar registrada la carga de `db.sql`.

## Comprobación de los archivos

En Debian, `sha256sum -c SHA256SUMS` comprueba los archivos de esta carpeta. Para reunir las partes sin sobrescribir un archivo existente:

```bash
(
  set -eC
  for n in 00 01 02; do
    dd if="var.tar.gz.part-$n" status=none || exit 1
  done > var.tar.gz
  gzip -t var.tar.gz
  sha256sum var.tar.gz
)
```

El SHA-256 esperado de `var.tar.gz` es:

```text
c2340ea0aba3ddb6271f1ab10c989fc788a7ed67749ca55e5ddcfefea752880b
```

Estos comandos reconstruyen y verifican el comprimido; no extraen ni restauran su contenido.
