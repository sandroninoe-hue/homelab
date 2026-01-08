# Guía de Comunicación Inter-Redes (Routing Puro con WireGuard)

## Configuración WireGuard
```ini
[Interface] 
Address = 10.0.1.1/24 
ListenPort = 10001 
PrivateKey = <YOUR_PRIVATE_KEY>

# Nota: ausencia de la regla "nat" 
PostUp = iptables -A FORWARD -i %i -o eth0 -j ACCEPT; iptables -A FORWARD -i eth0 -o %i -m state --state ESTABLISHED,RELATED -j ACCEPT 
PostDown = iptables -D FORWARD -i %i -o eth0 -j ACCEPT; iptables -D FORWARD -i eth0 -o %i -m state --state ESTABLISHED,RELATED -j ACCEPT 

[Peer] 
# laptop 
PublicKey = <PEER_PUBLIC_KEY>
AllowedIPs = 10.0.1.4/32, 192.168.10.0/24
```

## Arquitectura

Esta arquitectura permite la comunicación transparente entre una VM de origen (en Fedora/Libvirt) y una VM de destino (en Proxmox/GNS3) a través de un túnel WireGuard, manteniendo las IPs originales en todo el trayecto.

## 1. El Origen: Peer (VM en Laptop)

La VM vive en una red aislada dentro de la laptop (ej. `192.168.10.90`).

- **Gateway de la VM:** Debe ser la IP de la interfaz puente de la laptop (`192.168.10.1`).
- **Función:** La VM confía en la laptop para sacar cualquier paquete que no sea de su propia subred.

## 2. El Tránsito: La Laptop (Host & Gateway VPN)

La laptop actúa como un router inteligente. Recibe el tráfico de la VM y lo encapsula en la VPN.

- **Configuración `wg1` (AllowedIPs):** Para alcanzar el destino, el archivo de configuración en la laptop debe incluir la IP o red de destino en su lista de permitidos.
  - `AllowedIPs = 10.0.1.0/24, 192.168.100.90/32` (o la red completa `.100.0/24`).
- **IP Forwarding:** Es obligatorio que la laptop tenga el reenvío de paquetes activo (`net.ipv4.ip_forward = 1`) para pasar el tráfico de la interfaz de Libvirt (`virbr1`) a la interfaz de la VPN (`wg1`).
```bash
# Habilitar IP Forwarding temporalmente
sudo sysctl -w net.ipv4.ip_forward=1

# Habilitar IP Forwarding permanentemente
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## 3. El Pivote: Servidor WireGuard (`.96`)

Es el punto de entrada a la red remota.

- **Acceso Local:** El servidor debe estar físicamente (o virtualmente) en la misma red que el destino (`192.168.100.0/24`).
- **AllowedIPs del Peer:** En su configuración de Peers, el servidor debe autorizar no solo la IP de la VPN de la laptop, sino también la red de las VMs que vienen detrás.
  - `AllowedIPs = 10.0.1.4/32, 192.168.10.0/24`

## 4. El Destino: GNS3 VM / Destino Final (`.90`)

El paquete llega aquí con su IP de origen original (`192.168.10.90`).

- **Ruta Estática:** El destino no sabe qué es la red `10.0.1.0` o `192.168.10.0`. Por ello, necesita una regla permanente:
```bash
# Agregar ruta estática
ip route add 192.168.10.0/24 via 192.168.100.96
```

**Lógica:** "Para responder a la red VPN, entrégale el paquete al servidor WireGuard (`.96`)".

## Diagrama de Flujo
```
VM Origen (192.168.10.90)
        ↓
Laptop Gateway (192.168.10.1) + WireGuard Client (10.0.1.4)
        ↓ [Túnel WireGuard]
Servidor WireGuard (192.168.100.96 + 10.0.1.1)
        ↓
VM Destino (192.168.100.90)
```

## Verificación

### En la Laptop
```bash
# Verificar IP Forwarding
sysctl net.ipv4.ip_forward

# Verificar estado de WireGuard
sudo wg show

# Verificar tabla de routing
ip route
```

### En el Servidor WireGuard
```bash
# Verificar peers conectados
sudo wg show

# Verificar reglas de iptables
sudo iptables -L FORWARD -v -n
```

### En la VM Destino
```bash
# Verificar ruta estática
ip route | grep 192.168.10.0
```

## Troubleshooting

- **No hay conectividad:** Verificar que `ip_forward` esté habilitado en ambos extremos.
- **Paquetes no regresan:** Asegurarse de que la ruta estática en el destino apunte correctamente al servidor WireGuard.
- **AllowedIPs incorrectos:** Revisar que ambas configuraciones incluyan todas las subredes necesarias.
