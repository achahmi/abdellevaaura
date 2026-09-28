# Configurar un servidor DNS en Ubuntu Server

Aquí explicaré cómo configurar un servidor DNS en una máquina UBUNTU SERVER 26.04.1

## 1. Actualizar el sistema

Primero, después de haber creado la máquina y configurado el nombre, empezamos por hacer un `sudo apt update`:

```bash
sudo apt update
```

## 2. Configurar la red (Netplan)

Después entré en el archivo de configuración de red de Netplan:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Y dejé esta configuración:

![Configuración de Netplan](imagenes/netplan-configuracion.png)

### Por qué lo configuré de esta manera

La máquina tiene **dos adaptadores de red**:

- **Primer adaptador (`enp0s3`), en modo NAT:** lo uso para que el servidor tenga salida a Internet (por ejemplo, para descargar paquetes). Como el NAT de VirtualBox reparte la IP automáticamente, tuve que ponerlo en **DHCP** (`dhcp4: true`).
- **Segundo adaptador (`enp0s8`), en modo Red interna:** es la red por la que el servidor DNS dará servicio a los demás equipos. Aquí no hay DHCP (`dhcp4: false`), así que le puse una **IP fija**: `192.168.6.125/24`.

### Problema con `routes` y cómo lo arreglé

Al principio tenía puesta en el segundo adaptador una **ruta por defecto** (`routes` → `to: default` → `via: 192.168.6.1`). Eso me daba problemas por dos motivos:

1. **Esa puerta de enlace no existe.** La red interna es una red aislada donde no hay ningún router en `192.168.6.1`. Con esa ruta, el servidor mandaba el tráfico hacia Internet a una dirección que no responde, y por eso me quedaba **sin conexión** (por ejemplo, `apt update` no funcionaba).
2. **Había dos rutas por defecto a la vez.** La primera venía del adaptador NAT (que sí tiene Internet y la recibe por DHCP) y la segunda era la que puse yo a mano. Al tener dos, el sistema no sabía cuál usar y a veces elegía la que no funcionaba.

Lo arreglé simplemente **quitando el bloque `routes`** (lo comenté con `#`). Así la única salida a Internet es la del adaptador NAT, que es la que funciona, y la red interna se usa solo para comunicarse dentro de la red local.

![Bloque routes comentado](imagenes/netplan-routes-comentado.png)
