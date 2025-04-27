# Projeto AdGuard Home com Docker Compose

Este repositório fornece um ambiente Docker Compose para provisionar um container AdGuard Home com configuração personalizada via variáveis de ambiente.

## Pré-requisitos

- Docker e Docker Compose instalados na máquina.  
- Rede Docker externa chamada `network-share` já criada:

```bash
# Criar a rede externa se ainda não existir
docker network create --driver bridge network-share --subnet=<SUBNET_DOCKER>
```

- Arquivo `.env` baseado no exemplo fornecido (`.env.example`).

## Estrutura de arquivos

```plaintext
├── docker-compose.yml    # Definição dos serviços Docker
├── .env.example          # Exemplo de variáveis de ambiente
└── README.md             # Este arquivo de documentação
```

## Configuração das variáveis de ambiente

Copie o template e preencha os valores:

```bash
cp .env.example .env
```

No arquivo `.env`, ajuste:

```bash
# Nome do container AdGuard Home
ADGUARD_CONTAINER=<NOME_DO_CONTAINER_ADGUARD>

# IP fixo na rede Docker
IPV4_ADGUARD=<IP_DA_REDE_DOCKER>

# Portas
PORTS_DNS_TCP=53:53/tcp    # DNS TCP
PORTS_DNS_UDP=53:53/udp    # DNS UDP
PORTS_WEB=<PORTA>:80       # Interface Web

# Volumes
VOL01=<PASTA_DO_ADGUARD>/work:/opt/adguardhome/work
VOL02=<PASTA_DO_ADGUARD>/conf:/opt/adguardhome/conf

# Date-time
VOL03=/etc/timezone:/etc/timezone:ro
VOL04=/etc/localtime:/etc/localtime:ro

# Rede externa
SUBNET=<SUBNET_DOCKER>
```

## Subindo o serviço

Para iniciar o container em segundo plano:

```bash
docker compose up -d
```

Verifique se o container está em execução:

```bash
docker ps | grep ${ADGUARD_CONTAINER}
```

Acompanhe os logs:

```bash
docker logs -f ${ADGUARD_CONTAINER}
```

## Parando e removendo containers

Para parar e remover os recursos criados:

```bash
docker compose down
```

## Explicação do `docker-compose.yml`

```yaml
services:
  adguard-home:
    image: adguard/adguardhome:latest
    container_name: ${ADGUARD_CONTAINER}    # Nome do container definido no .env
    restart: unless-stopped
    ports:
      - "${PORTS_DNS_TCP}"  # Porte TCP do DNS
      - "${PORTS_DNS_UDP}"  # Porte UDP do DNS
      - "${PORTS_WEB}"      # Interface Web
    volumes:
      - ${VOL01}            # Dados de trabalho do AdGuard Home
      - ${VOL02}            # Configurações do AdGuard Home
      - ${VOL03}            # Timezone do sistema
      - ${VOL04}            # Localtime do sistema
    networks:
      network-share:
        ipv4_address: ${IPV4_ADGUARD}  # IP fixo na rede externa

networks:
  network-share:
    external: true
    ipam:
      config:
        - subnet: ${SUBNET}             # Faixa de IPs da rede externa
```

## Observações

- Substitua `<SUBNET_DOCKER>` e `<IP_DA_REDE_DOCKER>` pelos valores da sua infraestrutura Docker.  
- Os diretórios mapeados em `VOL01` e `VOL02` devem existir e ter permissão de leitura/gravação.  
- O AdGuard Home armazenará dados de trabalho em `/opt/adguardhome/work` e as configurações em `/opt/adguardhome/conf`.  
- Utilize `restart: unless-stopped` para manter o serviço ativo após reinicializações do servidor.  

---

Qualquer dúvida ou sugestão, abra uma issue ou envie uma contribuição!  
