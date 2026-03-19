# PostgreSQL 

Repositório responsável pela infraestrutura de banco de dados do ambiente. Utiliza o PostgreSQL otimizado para baixo consumo de recursos, servindo múltiplas aplicações (ex: n8n, web, Nextcloud) através de bancos de dados e usuários isolados.

O projeto foi pensado para rodar em um RaspberryPi e portanto a persistência de dados é configurada diretamente em um disco externo para preservar a vida útil do armazenamento principal (SD Card) e garantir melhor performance de I/O.

## Pré-requisitos

1. **Docker e Docker Compose** instalados no host.   
2. **Rede Externa:** O container se conecta à rede Docker chamada `backend`. Se ela ainda não existir, crie-a executando:   
   ```bash
   docker network create backend
   ```
3. Ponto de Montagem: Um disco externo montado em /mnt/storage.
   
4. Preparação do Host   
O PostgreSQL em imagens Alpine roda sob o UID 999. Para evitar erros de permissão (Permission denied) ao inicializar o banco de dados no volume mapeado (bind), o diretório físico precisa ser criado e ter sua propriedade ajustada no host antes de subir o container.

Execute os comandos abaixo no servidor:
```Bash
# Cria o diretório de persistência de dados
sudo mkdir -p /mnt/storage/postgres_data

# Transfere a propriedade do diretório para o UID do PostgreSQL
sudo chown -R 999:999 /mnt/storage/postgres_data
```

5. Configuração e Execução   
  5.1. Clone este repositório no seu servidor.   
  5.2. Copie o arquivo de exemplo das variáveis de ambiente: `cp .env_example .env`   
  5.3. Edite o arquivo .env para definir a senha do usuário root do banco.   
  5.4. Suba a infraestrutura em background:   
```Bash
docker compose up -d
```

6. Gerenciamento de Bancos e Usuários   
Como boa prática de arquitetura e segurança, cada aplicação deve ter seu próprio banco de dados e usuário com privilégios restritos.
Para acessar o terminal do PostgreSQL e provisionar novos acessos, utilize:

```Bash
docker exec -it postgres psql -U postgres
```

7. Exemplo de provisionamento para uma nova aplicação:
   
```SQL
CREATE DATABASE nome_da_app_db;
CREATE USER nome_da_app_user WITH ENCRYPTED PASSWORD 'senha_da_app';
GRANT ALL PRIVILEGES ON DATABASE nome_da_app_db TO nome_da_app_user;
ALTER DATABASE nome_da_app_db OWNER TO nome_da_app_user;
```

8. Estrutura do Volume   
O `docker-compose.yml` utiliza o driver local com a opção bind para mapear o volume de forma absoluta no host. Isso garante que os dados do banco não fiquemno SDCard local em `/var/lib/docker/volumes` e permaneçam no HD externo. Isso evita que o SDCard acabe degradando cedo por alta utilização.
