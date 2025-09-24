# 🔐 WireGuard en Proxmox (LXC)

Guía rápida y reproducible para desplegar un contenedor **WireGuard** en **Proxmox VE** usando los **community-scripts**.

---

## 🚀 Instalación

En el **host Proxmox**, ejecuta el siguiente comando:

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/wireguard.sh)"
```

---

## ⚙️ Configuración del servidor

Dentro del contenedor, edita el archivo de configuración:

```bash
nano /etc/wireguard/wg0.conf
```

### Configuración del servidor (`wg0.conf`):

```ini
[Interface]
PrivateKey = <clave_privada_de_esta_maquina>
Address = 10.0.0.1/24
ListenPort = 10000

# Reglas de NAT y reenvío
PostUp   = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = <clave_publica_peer>
AllowedIPs = 10.0.0.4/32
```

### Configuración del cliente/peer:

```ini
[Interface]
PrivateKey = <clave_privada_peer>
Address = 10.0.0.4/32

[Peer]
PublicKey = <clave_publica_servidor>
AllowedIPs = 10.0.0.0/24, 192.168.100.90/32
Endpoint = <ip_publica_servidor>:<puerto>
PersistentKeepalive = 25
```

---

## 🔧 Comandos de gestión

### Levantar la interfaz:
```bash
sudo wg-quick up wg0
```

### Bajar la interfaz:
```bash
sudo wg-quick down wg0
```

### Habilitar inicio automático:
```bash
sudo systemctl enable wg-quick@wg0
```

---

## ✅ Verificación

### Ver interfaces activas:
```bash
wg show
```

### Probar conectividad desde el peer:
```bash
ping 10.0.0.1
```

---

## 📌 Notas importantes

- **Puerto personalizado**: Usa un puerto distinto si tienes otros servicios basados en WireGuard (ej: NetBird).
- **Interfaz LXC**: En contenedores LXC, la interfaz puede aparecer como `eth0@if13`, pero en la configuración debes usar `eth0`.
- **Firewall**: Asegúrate de que el puerto configurado esté abierto en el firewall del host Proxmox.
- **Generación de claves**: 
  ```bash
  # Generar clave privada
  wg genkey | tee privatekey | wg pubkey > publickey
  ```

---

## 🔍 Solución de problemas

### Ver logs del servicio:
```bash
journalctl -u wg-quick@wg0 -f
```

### Verificar rutas:
```bash
ip route show
```

### Verificar iptables:
```bash
iptables -t nat -L POSTROUTING -v
```