# 🐳 Instalação do Docker (Motor de Contêineres)

![Docker](images/docker.png)

O Docker é necessário para criar, isolar e executar serviços empacotados em ambientes controlados.
```
curl -fsSL [https://get.docker.com](https://get.docker.com) | sudo sh
```

**Função do comando:** `curl`: faz o download do script oficial de instalação automatizada do Docker;

`| sudo sh`: executa esse script com privilégios de superusuário. O script encarrega-se de adicionar os repositórios do Docker, baixar os binários do daemon (dockerd) e do cliente CLI (docker), e iniciar o serviço em segundo plano.
```
sudo usermod -aG docker $USER
```
**Função do comando:** Modifica as propriedades do usuário atual logado no sistema (`$USER`), anexando-o (`-aG`) ao grupo de segurança denominado docker. Isso concede ao usuário a permissão de leitura e escrita no soquete do sistema (`/var/run/docker.sock`), permitindo executar comandos do Docker sem a necessidade de prefixá-los com sudo.

---
## 1. Validação do Ambiente de Execução
Após a instalação, é necessário validar se o serviço está íntegro e se as permissões de usuário estão corretas.

Teste de Versionamento:
```
docker --version
```
**Função do comando:** Exibe a versão instalada do Docker. Serve como o primeiro teste de sanidade para confirmar que o binário foi instalado com sucesso no sistema.

Teste de Hello World (Container Mínimo):
```
docker run hello-world
```
**Função do comando:** Realiza o download de uma imagem mínima (`hello-world`) do Docker Hub, instancia um container a partir dela e executa o comando padrão da imagem.
Se a mensagem de sucesso for exibida no terminal, confirma-se que o daemon está ativo, o usuário possui permissão de execução e há conectividade com a internet para baixar imagens.

---

## 💡 Comandos Úteis do Docker

**🐳 Comandos de Ciclo de Vida de Containers**
* `docker run -d --name <nome> <imagem>`: Cria e inicia um novo container em segundo plano (modo detached). A flag --name atribui um identificador amigável ao container para facilitar consultas futuras.

* `docker stop <nome_ou_id>`: Envia um sinal de interrupção (SIGTERM) para o processo principal dentro do container, interrompendo sua execução de forma controlada.

* `docker start <nome_ou_id>`: Reinicia um container que já foi criado anteriormente, mas que se encontra em estado parado.

* `docker rm -f <nome_ou_id>`: Força a remoção de um container do sistema. A flag -f (force) é utilizada para remover o container mesmo que ele esteja em execução, limpando os recursos ocupados.

**🛠 Comandos de Diagnóstico e Inspeção**
* `docker ps`: Lista todos os containers que estão ativos no momento, exibindo informações como ID, imagem, tempo de atividade e portas mapeadas.

* `docker ps -a`: Lista todos os containers existentes no sistema, independentemente de estarem rodando ou parados.

* `docker logs -f <nome_ou_id>`: Exibe o fluxo de saída padrão (stdout/stderr) do container em tempo real. A flag -f (follow) mantém o terminal aberto observando novos logs conforme a aplicação escreve.

* `docker inspect <nome_ou_id>`: Retorna um objeto JSON contendo toda a configuração interna de um container (redes, volumes, variáveis de ambiente, estado de execução).

**📦 Comandos de Gerenciamento de Imagens**
* `docker pull <imagem:tag>`: Baixa uma imagem específica de um registro remoto (como o Docker Hub) para o seu armazenamento local.

* `docker images`: Lista todas as imagens armazenadas localmente no host que podem ser utilizadas para instanciar containers.

* `docker build -t <nome:tag> .`: Constrói uma nova imagem a partir de um arquivo Dockerfile presente no diretório atual. A flag -t (tag) nomeia a imagem resultante.

**⚙️ Comandos de Orquestração (Docker Compose)**
* `docker compose up -d`: Lê o arquivo docker-compose.yml, cria e inicia todos os serviços nele definidos em segundo plano. Ele verifica automaticamente se as imagens já existem ou se precisam ser baixadas/construídas.

* `docker compose down`: Para e remove todos os containers, redes e volumes criados pelo arquivo docker-compose.yml, devolvendo o sistema ao estado inicial.

* `docker compose config`: Valida a sintaxe e a lógica do arquivo docker-compose.yml, exibindo a configuração final processada antes da execução.
