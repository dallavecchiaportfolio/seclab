# 🧠 Laboratório de Segurança Cibernética com Segmentação de Rede via Software (Debian + iptables)

Este projeto documenta a criação de um ambiente de laboratório de cibersegurança funcional utilizando hardware acessível, rede segmentada logicamente e controle de tráfego via `iptables`, sem a necessidade de switches gerenciáveis.

---

## 📌 Objetivos

- Simular um ambiente de rede corporativa segmentada
- Realizar análises e testes ofensivos/defensivos em segurança
- Aprimorar práticas com Debian, roteamento, firewall e rede
- Usar apenas linha de comando (sem interfaces gráficas)

---

## 🧱 Topologia e Equipamentos

| Dispositivo         | Função                           |
|---------------------|----------------------------------|
| Debian Router        | Roteador/firewall central       |
| Raspberry Pi         | DNS local com PiHole            |
| Impressora HP        | Periférico de produção          |
| Debian Netbook       | Host de segurança defensiva     |
| Debian Laptop        | Coletor de logs ou monitoramento|
| Mini PC com Kali     | Ambiente ofensivo de pentesting |
| Laptop Windows 11    | Simulação de estação de trabalho|
| Switch não gerenciável | Backbone físico entre os hosts |

---

## 🧩 Segmentação Lógica (via IP)

A rede foi dividida em sub-redes distintas, simulando VLANs através de software:

| Sub-rede (CIDR)     | Função                       | Gateway sugerido   |
|---------------------|------------------------------|--------------------|
| `192.168.10.0/24`   | Admin, DNS, impressora        | `192.168.10.1`     |
| `192.168.20.0/24`   | Usuário Windows               | `192.168.20.1`     |
| `192.168.30.0/24`   | Hosts defensivos (Debian)     | `192.168.30.1`     |
| `192.168.40.0/24`   | Pentest (Kali)                | `192.168.40.1`     |

---

## ⚙️ Configuração do Debian como Roteador

### Ativar Encaminhamento de Pacotes

```bash
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

###

# Interface externa (com DHCP)
auto eth0
iface eth0 inet dhcp

# Interface interna (com IP fixo)
auto eth1
iface eth1 inet static
    address 192.168.10.1
    netmask 255.255.255.0

---

🌐 Interfaces de Rede (exemplo em /etc/network/interfaces)

###

# Interface externa (com DHCP)
auto eth0
iface eth0 inet dhcp

# Interface interna (com IP fixo)
auto eth1
iface eth1 inet static
    address 192.168.10.1
    netmask 255.255.255.0

---

📦 Servidor DHCP (isc-dhcp-server)

# Instale o serviço:
sudo apt install isc-dhcp-server

# Configuração em /etc/dhcp/dhcpd.conf:
subnet 192.168.10.0 netmask 255.255.255.0 {
    range 192.168.10.100 192.168.10.150;
    option routers 192.168.10.1;
    option domain-name-servers 192.168.10.2;
}

# Repita para outras redes, alterando faixas e gateways.

# Arquivo /etc/default/isc-dhcp-server:
INTERFACESv4="eth1"

---

🔥 Regras de iptables (exemplo)

###

# Limpa regras existentes
iptables -F
iptables -t nat -F

# NAT para acesso à Internet
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Permitir comunicação na LAN
iptables -A FORWARD -i eth1 -o eth1 -j ACCEPT

# Permitir saída da rede ofensiva (Kali) para a Internet
iptables -A FORWARD -s 192.168.40.0/24 -o eth0 -j ACCEPT

# Bloquear acesso da Kali para Admin/Usuários
iptables -A FORWARD -s 192.168.40.0/24 -d 192.168.10.0/24 -j DROP
iptables -A FORWARD -s 192.168.40.0/24 -d 192.168.20.0/24 -j DROP

# Permitir acesso ao PiHole a partir de outras redes
iptables -A FORWARD -d 192.168.10.2 -j ACCEPT

# Salvar regras com iptables-persistent:
sudo apt install iptables-persistent
sudo netfilter-persistent save

---

🔬 Testes e Diagnóstico

###

    ✅ ping entre hosts da mesma rede

    ✅ traceroute para identificar rotas

    ✅ tcpdump -i eth1 para inspeção de pacotes

    ✅ nmap para verificar portas acessíveis

---

📈 Expansões futuras

###

    IDS/IPS com Suricata ou Snort

    Redirecionamento para honeypots ou armadilhas

    Logs centralizados com rsyslog

    Proxy transparente com Squid

    VPN entre segmentos (com WireGuard)

---

📎 Referências

###

  iptables tutorial - https://wiki.debian.org/iptables

  DHCP server Debian - https://wiki.debian.org/DHCP_Server

  tcpdump cheatsheet - https://danielmiessler.com/study/tcpdump/




