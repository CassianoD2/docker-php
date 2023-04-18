# Docker
Imagens docker para serviços AWS e desenvolvimento local.

## Usando em desenvolvimento local

Você terá disponível um container php-fpm, nginx, mysql e redis.

Clone este repositório no mesmo diretório onde estão seus projetos. 
Exemplo:

```
|-- raiz
    |-- docker-env
    |-- projeto
    |-- projeto1
```

#### Configurando um ou mais projetos

1 - Para cada um deles devemos criar um arquivo de configuração do nginx. Exemplo:

Arquivo: `raiz/docker-env/nginx/projeto.conf`

```
server {
    listen 80;
    client_max_body_size 108M;
    access_log /var/log/nginx/access.log;

    # ---------------------------
    # Personalizar aqui
    server_name projeto.local;
    root /app/projeto/public;
    # ---------------------------

    index index.php;
    location / {
        try_files $uri /index.php$is_args$args;
    }

    if (!-e $request_filename) {
        rewrite ^.*$ /index.php last;
    }

    location ~ \.php$ {
        fastcgi_pass apps:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PHP_VALUE "error_log=/var/log/nginx/php_errors.log";
        fastcgi_buffers 16 16k;
        fastcgi_buffer_size 32k;
        fastcgi_read_timeout 240;
        include fastcgi_params;
    }  
}
```

2 - Agora precisamos criar e configurar o arquivo `docker-env/docker-compose.yml`. Faça uma cópia do arquivo
`docker-compose-example.yml`

Encontre a configuração do `webserver` e edite as partes informadas abaixo.
```yml
webserver:
    volumes:
      - ../:/app
      # Adicione aqui abaixo os arquivos de configuração do nginx dos seus projetos.
      - ./nginx/projeto.conf:/etc/nginx/conf.d/projeto.conf
    hostname: projeto.local # Adicione aqui o projeto default a ser usado
```

Encontre a configuração do `apps` e edite as partes informadas abaixo.
```yml
apps:
    extra_hosts:
      - "host.docker.internal:host-gateway"
      # Adicione aqui abaixo no caso de usar mais de um projeto o dns local e coloque o IP do container do webserver: 172.10.0.101
      #- "projeto.local:172.10.0.101"
```

3 - Alterar a versão do PHP

Encontre a configuração do `apps` e edite as partes informadas abaixo. Os aquivos Dockerfile disponíveis estão com o sufixo
de acordo com a versão do PHP dentro do diretório `docker-env/php/`.
```yml
apps:
    build:
      context: ./php
      dockerfile: Dockerfile_php80 # Use o arquivo de acordo com a versão php pretendida.
```

4 - Adicione os namespaces configurados no arquivo `hosts` nos respectivos arquivos caminhos conforme seu sistema operacional.

linux: `/etc/hosts` <br>
window: `C:\windows\system32\drivers\etc\hosts`

```
127.0.0.1   localhost
127.0.0.1   projeto-apps            # importante para o xdebug
127.0.0.1   projeto.local
127.0.0.1   projeto1.local
```

<br>

## Containers

Navegue com o seu terminal até o diretório `docker-env`

**Iniciando containers**

```
docker-compose up -d

# Caso remova ou adicione um novo projeto

docker-compose up --build -d
```

**Desligando containers**

Navegue com o seu terminal até o diretório `docker-env`
```
docker-compose down
```

<br>

## xdebug 3

`xdebug.client_host=http://projeto-apps` <br>
`xdebug.port=9003` <br>
`xdebug.idekey=DOCKER_projeto` <br>

#### Configuração para phpstorm
Em breve.

#### Configuração para vscode
A chave `pathMappings` deve conter o caminho para o projeto dentro do container Docker. No arquivo abaixo o exemplo é `"/app/projeto"` mas no projeto1 seria `"/app/projeto1"`.

`.vscode/launch.json`

```JSON
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Xdebug 3 with Docker",
            "type": "php",
            "request": "launch",
            "port": 9003,
            "pathMappings": {
                "/app/projeto": "${workspaceFolder}"
            },
            "ignore": ["**/vendor/**/*.php"],
            "xdebugSettings": {
                "max_children": 10000,
                "max_data": 10000,
                "show_hidden": 1
            }
        }
    ]
}
```