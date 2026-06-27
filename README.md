# Plataforma Distribuída de Monitoramento e Segurança de Redes
**Módulo de Orquestração e Deploy Automatizado (VM10)**

Projeto desenvolvido para a disciplina de Laboratório de Redes. 

Este repositório contém os códigos de Infraestrutura como Código (IaC) e Gerência de Configuração responsáveis pelo provisionamento, estruturação de redes e implantação de serviços de uma plataforma distribuída operando em ambiente de nuvem OpenStack.

A solução aqui documentada corresponde às atribuições do **Deploy Automatizado (VM10)**, o nó central de orquestração da arquitetura.


---

## Visão Geral da Arquitetura

A infraestrutura do laboratório é composta por múltiplas máquinas virtuais (VMs) com propósitos específicos, interligadas por switches virtuais (Open vSwitch) que segmentam o tráfego em múltiplas VLANs (Virtual LANs). 

A arquitetura exige que o provisionamento de hardware virtual e a instalação de softwares ocorram de forma padronizada e reproduzível. Para atingir este objetivo, o projeto adota uma abordagem de provisionamento declarativo e configuração imperativa centralizada.

A máquina **VM10** atua como o controlador de automação. Ela executa instruções contra a API do OpenStack para construir a topologia física e, subsequentemente, estabelece túneis criptografados via SSH com cada nó para injetar as configurações operacionais.

---

## Função do Deploy Automatizado (VM10)

As responsabilidades do módulo de Deploy Automatizado incluem:

1. **Provisionamento Computacional (Terraform):** Comunicação com a API de Identidade (Keystone) e Computação (Nova) do OpenStack para instanciar 10 máquinas virtuais, alocar recursos de processamento (vCPU/RAM), discos físicos virtuais e injetar chaves públicas de acesso.
2. **Topologia de Rede (Terraform):** Conexão das interfaces de rede das instâncias aos barramentos (redes lógicas) pré-configurados no OpenStack, definindo a rede de gerência primária (`labredes1`) e as redes de trânsito secundárias (`VLAN20_SERVER`, `TRUNK_INTER_SW`, etc.).
3. **Gerenciamento de Configuração (Ansible):** Execução de *playbooks* modulares que instalam pacotes do sistema (Ubuntu), configuram regras de roteamento do kernel Linux (via Netplan) e implantam serviços em contêineres Docker de forma idempotente.
4. **Padronização de Roteamento:** Configuração das tabelas de roteamento das VMs para que o tráfego de internet e produção seja direcionado para o IP da VM11 (Firewall/Roteador), garantindo que as políticas de segurança `nftables` sejam respeitadas.

---

## Especificações Técnicas da VM10

A máquina orquestradora opera com os seguintes parâmetros alocados no OpenStack:

* **Flavor (Perfil de Hardware):** `light.micro.medium`
* **Processamento:** 2 vCPUs
* **Memória RAM:** 4 GB
* **Armazenamento:** 8 GB
* **Rede de Operação:** `VLAN10_DEPLOY` (Endereço: 10.0.110.8/24)
* **Rede de Gerência (OOBM):** `labredes1` (Endereço: 192.168.10.145)

A interface principal (`ens7`) está configurada de forma estática através da ferramenta Netplan, garantindo persistência na conexão com a sub-rede de provisionamento.

---

## Estrutura deste Repositório

Para garantir o isolamento de responsabilidades, o repositório é segmentado em dois diretórios principais:

```text
.
├── terraform/
│   ├── main.tf           # Definição do provider OpenStack e autenticação
│   ├── variables.tf      # Mapeamento de instâncias, flavors e redes
│   ├── instances.tf      # Instruções de criação das VMs e geração do inventário
│   └── README.md         # Documentação específica de IaC
│
├── ansible/
│   ├── ansible.cfg       # Parâmetros de execução e localização do inventário
│   ├── hosts.ini         # Inventário dinâmico gerado pelo Terraform
│   ├── instalar_base.yml # Playbook de instalação de pacotes (Docker, OVS)
│   ├── configurar_*.yml  # Playbooks específicas de configuração por VM
│   └── README.md         # Documentação específica de Gerência de Configuração
│
└── README.md             # Este documento
