# docker-sandbox

### Introdução

```docker version``` - versão do docker.

```docker run NOME_DA_IMAGEM``` - cria um container com a respectiva imagem passada como parâmetro.


### Tabalhando com imagens

- Imagens do Docker possuem um sistema de arquivos em camadas (Layered File System) e os benefícios dessa abordagem principalmente para o download de novas imagens;
- Imagens são Read-Only sempre (apenas para leitura);
- Containers representam uma instância de uma imagem;
- Como imagens são Read-Only os containers criam nova camada (layer) para guardar as alterações;
- O comando Docker run e as possibilidades de execução de um container.

#### Comandos:

```docker ps``` - exibe todos os containers em execução no momento.

```docker ps -a``` - exibe todos os containers, independente de estarem em execução ou não.

```docker run -it NOME_DA_IMAGEM``` - conecta o terminal que estamos utilizando com o do container.

```docker start ID_CONTAINER``` - inicia o container com id em questão.

```docker stop ID_CONTAINER``` - interrompe o container com id em questão.

```docker start -a -i ID_CONTAINER``` - inicia o container com id em questão e integra os terminais, além de permitir interação entre ambos.

```docker rm ID_CONTAINER``` - remove o container com id em questão.

```docker container prune``` - remove todos os containers que estão parados.

```docker rmi NOME_DA_IMAGEM``` - remove a imagem passada como parâmetro.

```docker run -d -P --name NOME dockersamples/static-site``` - ao executar, dá um nome ao container.

```docker run -d -p 12345:80 dockersamples/static-site``` - define uma porta específica para ser atribuída à porta 80 do container, neste caso 12345.


### Usando volumes

- Container são voláteis, isso é, ao remover um removemos os dados juntos;
- Para deixar os dados persistente devemos usar Volumes;
- Volumes salvos não ficam no container e sim no Docker Host;

```docker inspect ID_DO_CONTAINER``` - retorna informações do container.

```docker run -v CAMINHO_HOST:CAMINHO_CONTAINER``` - configura o volume(dados persistentes) na maquina e container.

```docker run -p 8080:3000 -v "C:\Users\Usuario\Desktop\volume-exemplo:/var/www" -w "/var/www" node npm start``` - roda um container apontando para o volume e inicia a aplicação do volume.

### Construindo nossas próprias imagens

- O Dockerfile define os comandos para executar instalações complexas e com características específicas;
- Principais comandos como FROM, MAINTAINER, COPY, RUN, EXPOSE e ENTRYPOINT;
- É possível subir uma imagem criada através de um Dockerfile para o Docker Hub e disponibilizar para os desenvolvedores;

#### Comandos:

```docker build -f Dockerfile``` - cria uma imagem a partir de um Dockerfile.

```docker build -f Dockerfile -t NOME_USUARIO/NOME_IMAGEM``` - constrói e nomeia uma imagem não-oficial.

```docker build -f Dockerfile -t NOME_USUARIO/NOME_IMAGEM CAMINHO_DOCKERFILE``` - constrói e nomeia uma imagem não-oficial informando o caminho para o Dockerfile.

```docker build -f Dockerfile -t diegourban/node .``` - faz o build da imagem de acordo com a descrição do Dockerfile.

```docker run -d -p 8080:3000 diegourban/node``` - roda a imagem criada.

```docker login``` - solicita login e senha para se autenticar no docker hub.

```docker push diegourban/node``` - envia a imagem desejada para o dockerhub.

```docker pull diegourban/node``` - obtem a imagem desejada do dockerhub.

### Comunicação entre containers

- Imagens criadas pelo Docker acessam a mesma rede, porém apenas através de IP.
- É possível criar suas próprias redes e realizar a conexão entre os containers.
- Durante a criação de uma rede precisamos indicar qual driver utilizaremos, geralmente, o driver bridge

Comandos:

```hostname -i``` - mostra o ip atribuído ao container pelo docker (funciona apenas dentro do container).

```docker network create --driver bridge minha-rede``` - cria uma rede chamada "minha-rede" usando o driver de bridge.

```docker network ls``` - lista as redes.

```docker run -it --name app-01 --network minha-rede ubuntu``` - roda um ubuntu com nome de "app-01" na rede "minha-rede".

```apt-get update && apt-get install iptuils-ping``` - se o ping não estiver instalado na imagem do containers.

```ping app-02``` - rodar no container do app-01.

#### Exemplo:

Baixando as imagens:
```
docker pull douglasq/alura-books:cap05
docker pull mongo
```

Subindo os containers:
```
docker run -d --name meu-mongo --network minha-rede mongo
docker run -d --name meu-app --network minha-rede -p 8080:3000 douglasq/alura-books:cap05
```

http://localhost:8080/ - para acessar o app
http://localhost:8080/seed/ - para popular os dados no bd


### Docker Compose

Como subir múltiplos containers?
