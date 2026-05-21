# Red y VLANs

## Equipos principales
- Mikrotik CCR2004
- pfSense
- Cisco Catalyst

## VLANs

| VLAN | Red | Función |
|---|---|---|
| VLAN 10 | 192.168.10.0/24 | Gestión |
| VLAN 20 | 192.168.20.0/24 | Servidores |
| VLAN 30 | 192.168.30.0/24 | Streaming |
| VLAN 40 | 192.168.40.0/24 | Invitados |
| VLAN 50 | 192.168.50.0/24 | Usuarios |

## Seguridad
- Firewall pfSense
- Segmentación VLAN
- Monitorización Zabbix
- Centralización logs Graylog
