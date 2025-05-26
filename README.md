# Tutorial DSpace

Este é tutorial de como instalar o DSpace. Esse procedimento funciona para as
versões 7, 8 e 9, podendo mudar apenas as versões de dependências específicas.

## Instalando as dependências

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

## Baixando o projeto

- Clone o repositório
    - Comando:
        ```sh
        git clone 'https://github.com/DSpace/DSpace'
        ```
    - Para clonar uma branch específica (p.e. dspace-8_x):
        ```sh
        git clone 'https://github.com/DSpace/DSpace' -b dspace-8_x
        ```

> [!WARNING]
> O diretório do DSpace será mencionado como `[DSPACE-DIR]` no resto do tutorial.

## Preparando o banco de dados

> [!WARNING]
> Altere o nome de usuário e a senha nos próximos comandos.

- Crie o usuário e a senha para o banco do DSpace:
    ```sh
    sudo -iu postgres psql -c "create role 'nome-de-usuario' with login password 'senha';"
    ```
- Crie o banco de dados:
    ```sh
    sudo -iu postgres createdb --owner='nome-de-usuario' --encoding=UNICODE 'senha'
    ```

## Configurando o DSpace

- Para configurar o DSpace, edite o arquivo `[DSPACE-DIR]/dspace/config/local.cfg`.

> [!NOTE]
> O arquivo `[DSPACE-DIR]/dspace/config/local.cfg.EXAMPLE` é a configuração de exemplo, você pode copiá-lo para `[DSPACE-DIR]/dspace/config/local.cfg`. Essa configuração possui os valores padrões usados pelo DSpace, mas note que você ainda precisa adaptar o que for necessário.

- O mínimo que você deve configurar são as opções `dspace.dir`, `db.url`, `db.username` e `db.password`.

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

## Instalando as dependências internas, compilando o projeto e fazendo a instalação

Para instalar as dependências internas e compilar o DSpace, rode os seguintes comandos, substituindo `[DSPACE-DIR]` pelo diretório do código do DSpace e `[MAVEN-DIR]` pelo diretório do Maven (ou use apenas `mvn` se estiver instalado globalmente):

```sh
cd [DSPACE-DIR]
[MAVEN-DIR]/bin/mvn clean package
```

Após fazer isso, será gerado o diretório `[DSPACE-DIR]/dspace/target/dspace-installer`. Agora rode esses comandos (assumindo que o `ant` esteja instalado globalmente, se não tiver, use o caminho completo para o executável):

```sh
cd [DSPACE-DIR]/dspace/target/dspace-installer
ant fresh_install
```

## Copiando cores do Solr

Rode o seguinte comando para copiar os cores para o diretório do solr:

```sh
cp -R "[DSPACE-INSTALL-DIR]/solr"/* "[SOLR-DIR]/server/solr/configsets"
```

## Adicionando webapps ao tomcat

Rode o seguinte comando para adicionar os webapps do DSpace ao Tomcat:

```sh
cp -r [DSPACE-INSTALL-DIR]/webapps/* [TOMCAT-DIR]/webapps/
```

## Criando um usuário administrador

Rode o seguinte comando para criar um usuário administrador de forma interativa:

```sh
[DSPACE-INSTALL-DIR]/bin/dspace create-administrator
```

## Iniciando o sistema

### Iniciando o solr

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

### Iniciando o Tomcat

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

## Interrompendo a execução do sistema

### Interrompendo o Solr

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

### Interrompendo o Tomcat

Para interromper a execução do Tomcat, use o seguinte comando:

```sh
[TOMCAT-DIR]/bin/shutdown.sh
```
