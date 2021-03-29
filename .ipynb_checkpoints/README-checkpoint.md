# <center>Implementação do MySQL Communit Server no Linux com Docker</center>


### Resumo


Esta página foi elaborada com informações básicas sobre como você pode instalar e configurar o MySQL Communit Server em um ambiente de desenvolvimento local.


O [Docker MySQL Server](https://hub.docker.com/r/mysql/mysql-server) é uma Imagem otimizada do MySQL Comunit Server. Criado, mantido e apoiado pela equipe MySQL da Oracle.


### Aviso

Esta é uma sugestão de uma configuração inicial para uso em máquinas de testes ou desenvolvimento. Não implemente em um servidor MySQL na produção sem considerar a implementação de regras de segurança e performance.



### Pré-requisitos:    

    - Máquina com sistema operacional Linux.
    - Você precisa ter o Docker instalado em sua máquina local.



### Vamos iniciar a instalação


Baixe a imagem otimizada do MySQL Server Docker

<!-- #region -->
```python
$ docker pull mysql/mysql-server
```
<!-- #endregion -->

![](img/docker-pull-mysql-server.png)


A imagem foi baixada com sucesso, para visualizar a imagem digite:

<!-- #region -->
```python
$ docker image ls
```
<!-- #endregion -->

![](img/docker-image-ls.png)


Vamos iniciar um container novo.


![](img/docker-container-run.png)


Até o momento da publicação deste documento a última versão estável do MySQL Server é 8.0 (tag: 8.0, latest).


Antes de inicializamos o container MySQL, vamos criar os volumes "datasets" e "db_mysql". Criaremos estes volumes por dois motivos:

1. Porque é uma das formas de compartilhar dados entre os containers e a máquina local;
2. Para manter o volume /var/lib/mysql persistente; 

Os dados gravados no container Docker são voláteis, ou seja, quando um container é encerrado os dados ou configurações adicionais serão perdidos. Então, criar um volume e mapeá-lo em um diretório na máquina local permite mantê-los persistentes. 

Esta é uma sugestão, você não precisa a criar esses volumes. No meu caso, para o ambiente de desenvolvimento que estou contruindo será necessário.



Para criamos os volumes persistentes na máquina local digite:

<!-- #region -->
```python
$ docker volume create datasets

```

<!-- #endregion -->

![](img/docker-volume-create.png)

<!-- #region -->
```python
$ docker volume create db_mysql

```
<!-- #endregion -->

![](img/docker-volume-create2.png)


Vamos listar os volumes criados:

<!-- #region -->
```python
$ docker volume ls

```
<!-- #endregion -->

![](img/docker-volumes.png)


Caso você queira saber onde os volumes foram criados na máquina local, digite:

<!-- #region -->
```python
$ docker inspect datasets

ou 

$ docker inspect db_mysql

```
<!-- #endregion -->

![](img/docker-inspect.png)


**Os volumes foram criados com sucesso!**


Vamos iniciar o container MySQL.

<!-- #region -->
```python
$ docker container run --name=MySQL \
                     -e MYSQL_ROOT_HOST=% \
                     -d --mount type=volume,source=datasets,destination=/opt/datasets \
                     --mount type=volume,source=db_mysql,destination=/var/lib/mysql  \
                     mysql/mysql-server:8.0
        
```
<!-- #endregion -->

**Aguarde a inicialização do serviço!**


A inicialização do container pode levar algum tempo. Quando o servidor está pronto para uso, o STATUS do container na saída do comando docker ps muda de (health: starting) para (healthy).


![](img/docker-container-run-ps.png)


O container está sendo executado em segundo plano para monitorar a saída do container, use o seguinte comando:

<!-- #region -->
```python
$ docker logs MySQL

```
<!-- #endregion -->

![](img/docker-log-mysql.png)


![](img/docker-logs-mysql.png)


Para copiar a senha do usário root do servidor MySQL, você pode selecionar e copiar do console ou usar o comando abaixo, que exibe somente a linha com a senha.

<!-- #region -->
```python
$ docker logs MySQL 2>&1 | grep GENERATED
```
<!-- #endregion -->

![](img/docker-logs-senha.png)


### Configuração


Agora vamos conectar no container do servidor MySQL que foi criado e instanciado.


Use o comando ``` docker exec -it ``` para iniciar um cliente mysql dentro do container Docker que você iniciou.


![](img/docker-exec-it.png)

<!-- #region -->
```python
$ docker exec -it MySQL mysql -u root -p
```
<!-- #endregion -->

Informe a senha aleatória que foi criada na inicialização do servidor. Lembrando, para recuperar a senha utilize o comando ``` docker logs ```

<!-- #region -->
```python
$ docker logs MySQL 2>&1 | grep GENERATED
[Entrypoint] GENERATED ROOT PASSWORD: @D@GUDybcap13H-URDoHraLgaDJ 
    ```
<!-- #endregion -->

![](img/docker-exec-it-root.png)


Após digitar a senha @D@GUDybcap13H-URDoHraLgaDJ no console, temos o acesso ao SHELL do MySQL.


![](img/mysql.png)


Vamos mudar a senha do root e permitir o acesso remoto via console da máquina local. Para  redefinir a senha do root do servidor execute esta instrução:


```mysql 
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'senha@12345678';
``` 


![](img/alter-user1.png)


### Vamos testar o acesso via ``` docker exec -it ``` com a nova senha

<!-- #region -->
```python 
$ docker exec -it MySQL mysql -u root -psenha@12345678
``` 
<!-- #endregion -->

![](img/docker-exe-mysql-uroot.png)


### Agora via console do Linux local


Para acessar o servidor MySQL que foi instanciado no Docker, você precisa descobrir o IP da máquia. Use o comando ``` docker inspect ``` para examinar detalhes da configuração do Docker.

<!-- #region -->
```python 
$ docker inspect MySQL
```
<!-- #endregion -->

![](img/ip-addr.png)


### Tentativa de acesso 01 

<!-- #region -->
```bash 
$ mysql -u root -p -h 172.17.0.3 -u root
``` 
<!-- #endregion -->

![](img/erro1.png)


**Foi reportando um Erro!**. Acontece que o usuário root não tem permissão para acesso remoto, então para resolver esse problema vamos criar um usuário para acesso remoto.


### A sequência de instruções para essa operação são:


1. Acessar o console SHELL do MySQL;

<!-- #region -->
```python 
$ docker exec -it MySQL mysql -u root -psenha@12345678
``` 
<!-- #endregion -->

2. No SHELL digite os seguintes comandos: (**Lembrando** que "remoto@123" é uma senha de demonstração)

<!-- #region -->
```bash 
mysql> CREATE USER 'santana'@'%' IDENTIFIED WITH mysql_native_password BY 'remoto@123';
Query OK, 0 rows affected (0.39 sec)

mysql> GRANT ALL PRIVILEGES ON *.* TO 'santana'@'%';
Query OK, 0 rows affected (0.47 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.27 sec)
```
<!-- #endregion -->

### Tentativa de acesso 02

<!-- #region -->
```bash 
$ mysql -u santana -p -h 172.17.0.3 -u root
``` 
<!-- #endregion -->

![](img/mysql-h.png)


### Concluído!

Agora temos um servidor MySQL rodando em um Docker em nossa máquina local de desenvolvimento ou testes, com volume de dados persistente e com acesso a uma pasta local compartilhada. E, o acesso ao Banco de Dados poderá ocorrer via instrução ```docker exec -it ``` ou via qualquer cliente MySQL remoto.



**Instruções adicionais:**

<!-- #region -->
```bash 
$ docker ps (Lista todos os containes que estão sendo executados)
$ docker stop MySQL (Para o serviço)
$ docker start MySQL (Inicia o serviço)
$ docker restart MySQL (Reinicia o serviço)

``` 
<!-- #endregion -->

Espero ter contribuido com o seu desenvolvimento de alguma forma.


[Carlos Eugênio](https://carlosemsantana.github.io/) 


### Referências


Documentação detalhada: Consulte, [Implementando MySQL no Linux com Docker](https://dev.mysql.com/doc/refman/8.0/en/linux-installation-docker.html) ou [Manual de Referência](https://dev.mysql.com/doc/refman/8.0/en/)

```python

```
