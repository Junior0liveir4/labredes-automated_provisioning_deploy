# 🤖 Gerenciamento de Configuração (Ansible)

![Ansible](../images/ansible.png)

Este diretório contém as *Playbooks* responsáveis por automatizar a configuração do sistema operacional e dos serviços de rede nas instâncias provisionadas pelo Terraform.
O Ansible é a ferramenta responsável por acessar remotamente as outras VMs da topologia e injetar configurações em seus sistemas operacionais.
Baixar e instalar o Ansible:
```
apt install -y ansible
```
**Função do comando:** Realiza o download do motor de execução do Ansible e de suas bibliotecas dependentes (baseadas em Python) a partir do repositório oficial do Ubuntu. O binário ansible passa a ficar disponível no caminho de execução ($PATH) do sistema.

---

## 🛠 1. Configuração do Ambiente Ansible

Antes de executar as tarefas, configuramos o comportamento do Ansible para o ambiente de laboratório:

Preparar o ambiente do Ansible:
```
nano ~/ansible/labredes/ansible.cfg
```
```ini
[defaults]
host_key_checking = False
inventory = hosts.ini
```
**Estrutura do Arquivo:** `host_key_checking = False`: Desabilita a verificação rigorosa da chave SSH (known_hosts). Essencial em laboratórios onde as VMs são destruídas e recriadas frequentemente, evitando erros de "Host Key Changed".

`inventory = hosts.ini`: Define o arquivo de inventário gerado pelo Terraform como a fonte padrão de endereços IP e credenciais das VMs.

---

## 🧪 2. Teste de Automação de Rede

Usando o modelo do arquivo de criação de VM feito anteriormente no tutorial de instalação do Terraform, execute o comando abaixo para criar a vm-teste:
```
terraform plan
terraform apply -auto-approve
```
Agora, crie o Playbook que contém a sequência de tarefas que o Ansible executará na máquina:
```
nano ~/ansible/labredes/instalar_ovs.yml
```
```
- name: Instalar e Configurar o Open vSwitch
  hosts: vm-teste
  become: yes
  gather_facts: no
  
  vars:
    ansible_ssh_common_args: '-o StrictHostKeyChecking=no'

  tasks:
    - name: Aguardar o boot da VM e o SSH ficar disponivel
      wait_for_connection:
        delay: 5
        timeout: 300

    - name: Coletar informacoes do sistema
      setup:

    - name: Atualizar o cache de pacotes
      apt:
        update_cache: yes

    - name: Instalar o pacote do Open vSwitch
      apt:
        name: openvswitch-switch
        state: present

    - name: Garantir que o servico inicie junto com o sistema
      service:
        name: openvswitch-switch
        state: started
        enabled: yes
```
Executar a automação:
```
ansible-playbook instalar_ovs.yml
```
Para confirmar que o teste foi bem-sucedido, conecte-se à instância e verifique o serviço com o comando:
```
systemctl status openvswitch-switch
```
Limpeza do Ambiente:
```
terraform destroy -auto-approve
```
