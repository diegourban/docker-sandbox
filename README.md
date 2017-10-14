# docker-sandbox

docker ps - exibe todos os containers em execução no momento.

docker ps -a - exibe todos os containers, independente de estarem em execução ou não.

docker run -it NOME_DA_IMAGEM - conecta o terminal que estamos utilizando com o do container.

docker start ID_CONTAINER - inicia o container com id em questão.

docker stop ID_CONTAINER - interrompe o container com id em questão.

docker start -a -i ID_CONTAINER - inicia o container com id em questão e integra os terminais, além de permitir interação entre ambos.

docker rm ID_CONTAINER - remove o container com id em questão.

docker container prune - remove todos os containers que estão parados.

docker rmi NOME_DA_IMAGEM - remove a imagem passada como parâmetro.

docker run -d -P --name NOME dockersamples/static-site - ao executar, dá um nome ao container.

docker run -d -p 12345:80 dockersamples/static-site - define uma porta específica para ser atribuída à porta 80 do container, neste caso 12345.

docker inspect ID_DO_CONTAINER - retorna informações do container

docker run -v CAMINHO_HOST:CAMINHO_CONTAINER - configura o volume(dados persistentes) na maquina e container

docker run -p 8080:3000 -v "C:\Users\Usuario\Desktop\volume-exemplo:/var/www" -w "/var/www" node npm start - roda um container apontando para o volume e inicia a aplicação do volume
