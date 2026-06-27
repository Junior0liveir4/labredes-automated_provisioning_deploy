# 🖥️ Configuração da VM Orquestradora (VM10)

Este documento detalha o processo de inicialização e preparação do sistema operacional Ubuntu 24.04 LTS na máquina VM10 (Deploy Automatizado).

O objetivo desta fase é estabelecer a conectividade de rede com o ambiente de provisionamento e instalar os binários necessários (OpenStack CLI, Terraform, Ansible e Docker) para que a VM10 atue como o nó controlador de toda a infraestrutura.

---

## 🔗 1. Configuração das Interfaces de Rede

A VM10 foi provisionada com duas interfaces de rede físicas virtuais:

* `ens3`: Conectada à rede de gerência (labredes1), recebendo endereçamento via DHCP.
* `ens7`: Conectada à rede interna de provisionamento (VLAN10_DEPLOY), exigindo configuração de IP estático.

### 1.1. Ativação Temporária em Memória

Para estabelecer acesso imediato à rede interna de deploy, os seguintes comandos foram enviados diretamente ao kernel do sistema operacional:

```
ip link set ens7 up
```
**Função do comando:** Altera o estado administrativo da interface de rede ens7 de "DOWN" para "UP".
Isso instrui o kernel do Linux a ativar a placa de rede para o envio e recebimento de quadros ethernet na Camada 2.


```
ip addr add 10.0.110.8/24 dev ens7
```
**Função do comando:** Vincula o endereço IPv4 estático `10.0.110.8`, com a máscara de sub-rede `/24 (255.255.255.0)`, à interface `ens7`.
Esta configuração ocorre na memória volátil e permite o roteamento imediato de pacotes IP, mas não sobrevive a uma reinicialização do sistema.

### 1.2. Configuração de Persistência (Netplan)

Para tornar a configuração de rede permanente, o utilitário padrão do Ubuntu (Netplan) foi utilizado. O arquivo de configuração de rede foi editado com as diretivas estruturais.

Abra o arquivo de configuração:
```
nano /etc/netplan/99-cloud-init.yaml
```

Insira a seguinte estrutura de dados:
```
network:
  version: 2
  ethernets:
    ens3:
      match:
        macaddress: "mac da sua vm correposndente a interface ens3"
      dhcp4: true
      set-name: "ens3"
      mtu: 1450

    ens7:
      match:
        macaddress: "mac da sua vm correposndente a interface ens7"
      addresses:
        - 10.0.110.8/24
      dhcp4: true
      set-name: "ens7"
      mtu: 1450
```


Aplique as configurações:
```
netplan apply
```
**Função do comando:** O utilitário lê a estrutura de dados declarada no arquivo YAML, compila as informações e aplica o estado desejado ao renderizador de rede do sistema (neste caso, o systemd-networkd). Isso regrava as tabelas de roteamento e endereçamento de forma definitiva no disco.

---

## ⚙️ 2. Atualização do Sistema Base

Antes de proceder com a instalação de novos softwares, os pacotes do sistema operacional foram atualizados para garantir estabilidade e segurança.
```
apt update && apt upgrade -y
```

**Função do comando:** `apt update`: Consulta os servidores remotos configurados no sistema e atualiza a lista local de metadados de softwares disponíveis.

`apt upgrade -y`: Compara as versões dos softwares instalados localmente com a lista de metadados atualizada. Realiza o download e a substituição dos binários desatualizados.

`-y`: suprime os prompts de confirmação, aprovando automaticamente as substituições.
