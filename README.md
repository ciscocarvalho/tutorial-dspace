# Tutorial DSpace

Este é tutorial de como instalar o DSpace. Esse procedimento funciona para as
versões 7, 8 e 9, podendo mudar apenas as versões de dependências específicas.

## Instalando as dependências

<br>

> [!WARNING]
> Pule essa etapa se você estiver usando Docker.

<br>

- Instale o `maven`.
    - Maven é necessário para instalar as dependências do DSpace.
- Instale o `tomcat`
    - Tomcat é necessário para rodar o backend.
    - Dê permissão de executável para os scripts do tomcat:
        ```sh
        chmod u+x "[TOMCAT-DIR]/bin"/*.sh
        ```
- Instale o `solr`.
    - Solr é necessário para as funcionalidades de busca e estatísticas.
- Instale o `ant`.
    - Ant é necessário para mover os arquivos corretos para a instalação do DSpace.
- Instale os pacotes `postgres` e `postgres-contrib`.
    - `postgres` é o banco de dados.
    - `postgres-contrib` é necessário para instalar a extensão `pgcrypto` no banco.
- Instale o Java.
    - Java é a linguagem de programação usada no backend do DSpace.
- Instale o Git.

<br>

## Baixando o projeto

<br>

> [!WARNING]
> Pule esse passo se você já tiver um repositório local.
> Se você tiver um repositório remoto, substitua a URL nos comandos a seguir pela URL do seu repositório.

<br>

Clone o repositório:
```sh
git clone 'https://github.com/DSpace/DSpace'
```
<details>
<summary><b>Usando Docker</b></summary>

```sh
docker run --rm -v $(pwd):/git -w /git alpine/git && git clone 'https://github.com/DSpace/DSpace'
```
</details>

<br>

Para clonar uma branch específica (p.e. dspace-8_x):
```sh
git clone 'https://github.com/DSpace/DSpace' -b dspace-8_x
```
<details>
<summary><b>Usando Docker</b></summary>

```sh
docker run --rm -v $(pwd):/git -w /git alpine/git && git clone 'https://github.com/DSpace/DSpace' -b dspace-8_x
```
</details>

<br>

> [!WARNING]
> O diretório do DSpace será mencionado como `[DSPACE-DIR]` no resto do tutorial.

<br>

## Preparando o banco de dados

<br>

> [!WARNING]
> Pule essa etapa se você estiver usando Docker.

<br>

> [!WARNING]
> Altere o nome de usuário e a senha nos próximos comandos.

<br>

- Crie o usuário e a senha para o banco do DSpace:
    ```sh
    sudo -iu postgres psql -c "create role 'nome-de-usuario' with login password 'senha';"
    ```

- Crie o banco de dados:
    ```sh
    sudo -iu postgres createdb --owner='nome-de-usuario' --encoding=UNICODE 'senha'
    ```

<br>

## Configurando o DSpace

<br>

Para configurar o DSpace, edite o arquivo `[DSPACE-DIR]/dspace/config/local.cfg`.
> [!NOTE]
> O arquivo `[DSPACE-DIR]/dspace/config/local.cfg.EXAMPLE` é a configuração de exemplo, você pode copiá-lo para `[DSPACE-DIR]/dspace/config/local.cfg`. Essa configuração possui os valores padrões usados pelo DSpace, mas note que você ainda precisa adaptar o que for necessário.

<br>

O mínimo que você deve configurar são as opções `dspace.dir`, `db.url`, `db.username` e `db.password`:
> [!WARNING]
> Substitua `[DSPACE-INSTALL-DIR]` pelo diretório onde deseja instalar do DSpace.
> Você deve garantir que o diretório `[DSPACE-INSTALL-DIR]` existe e que seu usuário tem permissão de escrita nele.
> Substitua `[DSPACE-DB-NAME]` e `[DSPACE-DB-PASSWORD]` pelo nome e a senha do banco de dados que você criou para o DSpace.

```properties
  dspace.dir = [DSPACE-INSTALL-DIR]
  db.url = jdbc:postgresql://localhost:5432/[DSPACE-DB-NAME]
  db.username = [DSPACE-DB-NAME]
  db.password = [DSPACE-DB-PASSWORD]
```

<details>
<summary><b>Usando Docker</b></summary>

- Copie os arquivos dentro de `./docker` (neste repositório) para o `[DSPACE-DIR]`.
- Edite o arquivo docker-compose.yml colocando a senha que você deseja usar no postgres, alterando essa linha:
    ```
    - POSTGRES_PASSWORD=xxxxx
    ```
- Edite o script `prepara-postgres.sh` colocando a mesma senha postgres nessa linha (substituindo a senha `cDVf9UnCskeirG4K`):
    ```
    psql --username=postgres -c "CREATE USER dspace WITH PASSWORD 'cDVf9UnCskeirG4K';"
    ```
</details>

<br>

## Instalando as dependências internas, compilando o projeto e fazendo a instalação

<br>

Para instalar as dependências internas e compilar o DSpace, rode os seguintes comandos, substituindo `[DSPACE-DIR]` pelo diretório do código do DSpace e `[MAVEN-DIR]` pelo diretório do Maven (ou use apenas `mvn` se estiver instalado globalmente):

```sh
cd [DSPACE-DIR]
[MAVEN-DIR]/bin/mvn clean package
```

<details>
<summary><b>Usando Docker</b></summary>

Obs.: substitua `[DSPACE-DIR]` no comando a seguir.
```sh
docker run -v ~/.m2:/var/maven/.m2 -v "[DSPACE-DIR]":/tmp/dspacebuild -w /tmp/dspacebuild -ti --rm -e MAVen_CONFIG=/var/maven/.m2 maven:3.8.6-openjdk-11 mvn -q --no-transfer-progress -Duser.home=/var/maven clean package
```
</details>

<br>

Após fazer isso, será gerado o diretório `[DSPACE-DIR]/dspace/target/dspace-installer`. Agora rode esses comandos (assumindo que o `ant` esteja instalado globalmente, se não tiver, use o caminho completo para o executável):

```sh
cd [DSPACE-DIR]/dspace/target/dspace-installer
ant fresh_install
```
<details>
<summary><b>Usando Docker</b></summary>

Obs.: substitua `[DSPACE-DIR]` e `[DSPACE-INSTALL-DIR]` nos comandos a seguir.
```sh
docker run -v ~/.m2:/var/maven/.m2 -v [DSPACE-INSTALL-DIR]:/dspace -v [DSPACE-DIR]:/tmp/dspacebuild -w /tmp/dspacebuild -ti --rm -e MAVen_CONFIG=/var/maven/.m2 maven:3.8.6-openjdk-11 /bin/bash -c "wget https://archive.apache.org/dist/ant/binaries/apache-ant-1.10.12-bin.tar.gz && tar -xvzf apache-ant-1.10.12-bin.tar.gz && cd dspace/target/dspace-installer && ../../../apache-ant-1.10.12/bin/ant init_installation update_configs update_code update_webapps && cd ../../../ && rm -rf apache-ant-*"

cp -r [DSPACE-INSTALL-DIR]/config [DSPACE-INSTALL-DIR]
```
</details>

<br>

## Copiando cores do Solr

<br>

> [!WARNING]
> Pule essa etapa se você estiver usando Docker (isso é feito automaticamente em `docker-compose.yml`).

<br>

Rode o seguinte comando para copiar os cores para o diretório do solr:

```sh
cp -R "[DSPACE-INSTALL-DIR]/solr"/* "[SOLR-DIR]/server/solr/configsets"
```

<br>

## Adicionando webapps ao tomcat

<br>

> [!WARNING]
> Pule essa etapa se você estiver usando Docker.

<br>

Rode o seguinte comando para adicionar os webapps do DSpace ao Tomcat:

```sh
cp -r [DSPACE-INSTALL-DIR]/webapps/* [TOMCAT-DIR]/webapps/
```

<br>

## Criando um usuário administrador

<br>

Rode o seguinte comando para criar um usuário administrador de forma interativa:

```sh
[DSPACE-INSTALL-DIR]/bin/dspace create-administrator
```
<details>
<summary><b>Usando Docker</b></summary>

Obs.: usando Docker, você precisa primeiro iniciar o sistema antes de rodar esse comando.
```sh
docker exec -it dspace7 /dspace/bin/dspace create-administrator
```
</details>

<br>

## Iniciando o sistema

<br>

> [!WARNING]
> Se você estiver usando Docker, use esse comando e pule os passos "Iniciando o Solr" e "Iniciando o Tomcat":
```sh
docker compose -f [DSPACE-DIR]/docker-compose.yml up --build -d
```

<br>

### Iniciando o Solr

<br>

Execute o Solr:

```sh
[SOLR-DIR]/bin/solr start
```

Obs.: o Solr deve iniciar com a mesma URL configurada na opção `solr.server`
(valor padrão: `http://localhost:8983/solr`) nas configurações do DSpace.

Para iniciar o Solr em uma porta diferente da padrão (8983), execute o comando assim (substituindo [PORT] pelo número da porta desejada):

```sh
[SOLR-DIR]/bin/solr start -p [PORT]
```

Se o Solr não iniciou na porta correta, faça os seguintes passos:

- Interrompa a execução do Solr que iniciou na porta errada:
    ```sh
    [SOLR-DIR]/bin/solr stop -p [PORT]
    ```

- Garanta que nada está ocupando a porta que você pretende usar.
- Inicie o Solr na porta certa:
    ```sh
    [SOLR-DIR]/bin/solr start -p [PORT]
    ```

Se tudo ocorrer bem, você verá uma mensagem como essa:

```
Started Solr server on port 8983 (pid=2296782). Happy searching!
```

<br>

### Iniciando o Tomcat

<br>

Para iniciar o Tomcat, existem duas opções:

1. Inicie normalmente:
    ```sh
    [TOMCAT-DIR]/bin/catalina.sh run
    ```

    Isso usa o terminal atual e mostra os logs diretamente. Se você interromper o comando, o Tomcat para.
2. Inicie no background:
    ```sh
    [TOMCAT-DIR]/bin/catalina.sh start
    ```

    Isso inicia o Tomcat no plano de fundo, sem usar o terminal atual. Dessa forma, os logs não são mostrados diretamente, mas ainda são salvos nos arquivos em `[TOMCAT-DIR]/logs`.

<br>

## Interrompendo a execução do sistema

<br>

> [!WARNING]
> Se você estiver usando Docker, use esse comando e pule os passos "Interrompendo o Solr" e "Interrompendo o Tomcat":
```sh
docker compose -f [DSPACE-DIR]/docker-compose.yml down
```

<br>

### Interrompendo o Solr

<br>

Para interromper a execução do Solr, use o seguinte comando, substituindo [PORT] pela porta que está sendo usada:

```sh
[SOLR-DIR]/bin/solr stop -p [PORT]
```

Se o Solr foi iniciado apenas uma vez, você pode executar o comando sem o parâmetro `-p`:

```sh
[SOLR-DIR]/bin/solr stop
```

Se você deseja interromper as execuções do Solr em todas as portas (se você iniciou mais de uma vez), você pode rodar o seguinte comando:

```sh
[SOLR-DIR]/bin/solr stop --all
```

<br>

### Interrompendo o Tomcat

<br>

Para interromper a execução do Tomcat, use o seguinte comando:

```sh
[TOMCAT-DIR]/bin/shutdown.sh
```
