# Check LDAP Groups Members 🔍

Este script de Bash permite auditar grupos en un servidor **LDAP**, identificando usuarios que están listados como miembros de un grupo pero que ya no existen en el directorio (miembros huérfanos).

## 🚀 Propósito
En entornos LDAP, a veces los usuarios son eliminados de la unidad organizativa principal (`ou=People`), pero sus referencias permanecen en los grupos (`ou=Group`). Este script automatiza la detección de estas inconsistencias y genera archivos **LDIF** para corregirlas fácilmente.


## 🛠️ Características
- **Detección Dual:** Identifica huérfanos tanto por DN (`member`) como por UID (`memberUid`).
- **Modo Individual o Masivo:** Puede auditar un solo grupo o todos los grupos del sistema (excluyendo grupos privados).
- **Generación de Fix:** Crea automáticamente un archivo `.ldif` listo para ser usado con `ldapmodify`.
- **Logs:** Registra toda la actividad en `/var/log/check_ldap.log`.

## 📋 Requisitos
- Un servidor LDAP operativo.
- Herramientas de cliente LDAP instaladas (`ldapsearch`, `ldapmodify`).
- Permisos de escritura en `/var/log/` para el archivo de log.

## ⚙️ Configuración
Antes de ejecutarlo, edita las variables en la sección de `CONFIGURACIÓN` del script:
- `SERVIDOR`: URL de tu servidor LDAP.
- `BASE_DN`: El DN base de tu organización.
- `ADMIN_DN`: DN del administrador para realizar las modificaciones.

## 📖 Uso
Dar permisos de ejecución:
```bash
chmod +x check_ldap_groups_members
```

Ejecutar para un grupo específico:
```bash
./check_ldap_groups_members nombre_del_grupo
```

Ejecutar para todos los grupos: 
```bash
./check_ldap_groups_members all
```

## ⚙️ Gestión de Miembros Huérfanos (.ldif)
Cuando el script detecta inconsistencias, genera archivos con extensión `.ldif` (*LDAP Data Interchange Format*). Estos archivos contienen las instrucciones necesarias para limpiar el directorio de forma segura.

* **Nombre del archivo:** `borrar_huerfanos_[nombre_del_grupo].ldif`
* **Contenido:** Incluye operaciones de tipo `delete` para cada entrada huérfana detectada.
* **Aplicación de cambios:** Para ejecutar la limpieza, se debe utilizar el comando `ldapmodify`. El script mostrará al final la sintaxis exacta, que suele ser:

```bash
ldapmodify -x -H ldap://ip-servidor -D "cn=admin,ou=People,dc=instituto,dc=extremadura,dc=es" -W -f borrar_huerfanos_nombre.ldif
