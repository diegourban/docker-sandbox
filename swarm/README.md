# Docker Swarm

## Cluster com Docker Swarm

### Ambiente

Necessário instalar Docker Swarm, Docker Machine e VirtualBox.

### O que é Docker Swarm

Com o Docker Engine é possível subir N container na mesma máquina, entretanto teremos limitações de memória é processador.
Um desses containers pode cair, e a Docker Engine não consegue lidar com isso, é necessário uma intervenção manual.

O Docker Swarm entra para tentar resolver esses dois casos.
Supondo que temos uma rede de N computadores.
Podemos criar um cluster com essas N máquinas, instalar o docker em todas essas máquinas e dividir a carga entre todas elas.
O Docker Swarm serve como um orquestrador num cluster de máquinas, que gerencia em qual máquina deve ser executado cada container e o que fazer em caso um dos containers falhe. Ele faz isso através de um Dispatcher.

### Usando a Docker Machine

Para simular o Docker Swarm em apenas uma máquina física, podemos utilizar a Docker Machine.
Dentro de uma máquina física ela consegue criar N máquinas virtuais leves e já com o Docker instalado.
Essas máquinas virtuais será nosso cluster do Swarm.

Versão da docker-machine: docker-machine version

Listar máquinas virtuais: docker-machine ls

Criar uma máquina de nome "vm1" usando o driver do virtualbox:
docker-machine create -d virtualbox vm1

Iniciar uma máquina:
docker-machine start vm1

Acessar a máquina:
docker-machine ssh vm1

### Criando o cluster

docker-machine ssh vm1

Para incializar o swarm: docker swarm init --advertise-addr 192.168.99.100
O advertise-addr fixa o endereço da máquina para que todas as outras máquinas que entrem no Swarm tenham uma comunicação estável com quem criou o Swarm.
O Swarm foi inicializado e o nó corrente se torna um Manager, o líder do Swarm.

Para saber se a máquina está dentro de um Swarm:
docker info

Agora é necessário Workers...

## Responsabilidade dos Workers

Os Workers são responsáveis por executar os containers enquanto que o Manager irá gerenciar Swarm.

### Criando Workers

Criando duas novas máquinas virtuais:
docker-machine create -d virtualbox vm2
docker-machine create -d virtualbox vm3

Para adicionar um Worker no Swarm:
docker swarm join --token SWMTKN-1-4vks86f5ywpt8k1igho70qbn6xcb9288zy06lew0zym5rp64zz-b1opgyadlo718h7fgnekcndpg 192.168.99.100:2377

É necessário executar esse comando dentro de cada um dos Workers, vm2 e vm3.
O token define que o que está sendo adicionado é um Worker e o Worker se comunica com o Manager através do IP informando no último parâmetro

Para gerar esse comando que adiciona uma máquina no Swarm, necessário acessar a manager e executar o comando: docker swarm join-token worker

### Comandos básicos

Para listar o ID e o hostname, status dos nós que fazem parte do Swarm:
docker node ls

Coluna ManagerStatus define se o nó é Manager ou Worker.
Esse comando somente pode executar no Manager.
Comandos de leitura ou alteração do estado do Swarm somente podem ser executado em Managers.

Como remover um nó do Swarm
No Worker executar: docker swarm leave
Irá mudar o estado do nó de Ready para Down.

No Manager executar: docker node rm {id do nó worker}
Irá efetivamente remover o nó do Swarm.

### Subindo um serviço

Para subir um container, é possível acessar um Worker e executar:
docker container run -p 8080:3000 -d aluracursos/barbearia

Esse container está sendo executado dentro do Worker.
Para acessar a aplicação é necessário saber o endereço desse Worker.
Através do comando "docker node inspect vm2" é possível saber qual o endereço da vm2.
Esse comando é necessário executar no Manager.

O problema é que criando um container diretamente no worker, os outros nós não saberão da existência desse container. Esse container é criado com escopo local.

Para criar um container no escopo do Swarm será necessário adicioná-lo como serviço.
docker service create -p 8080:3000 aluracursos/barbearia
Esse comando deve ser executado no Manager.

### Tarefas e Routing Mesh

Para listar os serviços sendo executados:
docker service ls

Uma tarefa é uma instância de um serviço sendo executado.

Para saber em qual máquina a tarefa está sendo executada:
docker service ps {id serviço}

Criando um serviço no contexto do Swarm, por mais que ele esteja sendo executado em uma vm específica, é possível acessar a aplicação independente do IP utilizado na URL.
Através da porta 8080(porta de ingresso), o Routing Mesh busca o nó que está sendo executado nessa porta e redireciona pra esse container.
Portanto subir outro serviço na porta 8080 não é possível.

Se removermos esse container no Workeratravés do comando docker container rm {id container}, o Manager irá automaticamente criar uma nova task e realocar ela em uma outra máquina, inclusive podendo ser alocado no Manager, dependendo da política de redistribuição.

## Gerenciando o cluster com managers

### O papel do manager

Para simular uma queda do manager:
docker-machine ssh vm1
docker swarm leave --force

Sem um nó manager, os Workers estão "perdidos"
Quando o único manager cair, não é possível mais fazer nenhuma operação de leitura de alteração de estado do Swarm.
Entretanto, os serviços continuam funcionando através dos IPs dos Workers.

### Como fazer backup do Swarm

Para evitar que o Swarm será perdido, é possível fazer o backup copiando o conteúdo de logs e configuração do Swarm para outra pasta no Manager.

sudo su
cp -r /var/lib/docker/swarm backup

Simulando um desaster no Swarm:
docker node rm vm2 --force
docker node rm vm3 --force
docker swarm leave --force

A pasta de configurações do Swarm foi removido.
Para criar um novo Swarm com as mesmas configurações, devemos copiar o back para a pasta do swarm.
cp -r backup/* /var/lib/docker/swarm/

Recriando o Swarm forçando a utilização das configurações do backup
docker swarm init --force-new-cluster --advertise-addr 192.168.99.100

### Criando mais manager

Para deixar o Swarm menos sucestível a erros, é possível criar mais Manangers.

Para gerar o comando de adicionar um nó Manager:
docker swarm join-token manager

Criando um Cluster com 5 nós, 3 manager e 2 workers:
docker-machine create -d virtualbox vm4
docker-machine create -d virtualbox vm5

na vm1: docker swarm init --advertise-addr 192.168.99.100

nad vm2 e 3: docker swarm join --token SWMTKN-1-4enlntz5icesew9kkuovqopmxeedacsjdtscgeizs31upxhklq-abembgyls698yx8behrw3sttu 192.168.99.100:2377

nas vm 4 e 5 executar: docker swarm join --token SWMTKN-1-4enlntz5icesew9kkuovqopmxeedacsjdtscgeizs31upxhklq-e4i0s469lthqbuefbyq2yce45 192.168.99.100:2377

Mananer vm1 é Leader.
Manager vm2 e vm3 agora estão com o status Reachable.
Workers vm4 e vm5 são os Workers.

Se a vm1 parar de funcionar:
docker swarm leave --force

A vm1 se torna Unreachable mas é elegido um novo Leader entre os Managers.

### Algoritmo de consenso RAFT

Quando um Leader cai, é feito uma eleição e definido o novo Leader entre os Managers.
Essa eleição é feito através de um algoritmo de consenso chamado RAFT.

Regras do RAFT:

N = número de Managers
Suporta: (N-1) / 2 Falhas
Deve ter no mínimo: (N/2) + 1 de quórum para eleição.

Exemplo:
3 Mananger suportam 1 falha e deve ter um quórum de 2.
5 Mananger suportam 2 falha e deve ter um quórum de 3.

Quanto mais Manager, maior é a taxa de escrita e leitura, o desempenho cai no longo prazo.
Docker recomenda, 3, 5 ou 7 Managers.

## Separando as responsabilidades

Como separar a responsabilidade de leitura/escrita nos manager e as tarefas nos Workers.

### Remover e readicionar um manager

Antes de remover um Manager do RAFT Cluster, é necessário rebaixar o nó.
Para tornar um manager em worker: docker node demote mv1
Para remover o nó: Para docker node rm vm1

Em seguida adicionar a vm1 com o comando join-token de manager.
A vm1 deve se tornar Reachable novamente.

### Restringindo nós

Como evitar que um Manager execute serviços?

Removendo todos os serviços: docker service rm ${docker service ls -q}

É possível restringir um nó para que não execute serviços através do Availability.

Para definir que o nó não execute serviço podemos atualizar o status dele com o comando:
docker node update --availability drain vm2

docker node ls, deve mostrar a vm2 com Availability = Drain e os serviços sendo executados na vm2 deve ser realocados para outros nós com Availability = Active.

O problema dessa abordagem é que para cada novo nó manager devemos lembrar de atualizar a Availability.
Além disso, e esse nó com Drain não poderá executar nenhum outro serviço.

### Restringindo serviços

Outra possibilidade menos invasiva é restringir os serviços.

Como evitar que um serviço seja executado nos Managers?

Adicionar uma restrição num serviço para que ele seja executado somente em Workers:
docker service update --constraint-add node.role==worker {id_serviço}

Ao atualizar o estado desse serviço, se ele estiver sendo executado num Manager, ele será realocado para um Worker.

Se adicionar uma restrição inválida, por exemplo um papel inexistente, o Swarm vai berrar e o serviço vai cair.

## Serviços globais e replicados

Quando e quais tipos de serviços a gente vai executar num Manager?
No Manager é ideal que sejam executado serviços críticos para a aplicação, como serviços de segurança e de monitoramento.

### Serviços replicados

Como fazer um serviço rodar me todos os nós?

Através do comando docker service ls, é possível ver que o serviço está no modo replicated.

Esse modo replicado é o padrão do serviço do Swarm, onde é criado apenas uma réplica.
Nesse modo é possível ter N replicas com o comando de update do serviço.
docker service update --replicas 4 {id_serviço}

Outra forma de escalar um serviço é através do comando:
docker service scale {id_serviço}=4

### Serviços globais

Serviços globais são serviços que tem uma instância em cada nó.

docker service create -p 8080:3000 --mode global aluracursos/barbearia

Irá criar um serviço em cada nó do Swarm.

## Driver Overlay

O responsável por toda essa comunicação entre os nós do Swarm.

### A rede ingress

Comando para listar as redes que o nó está conectado:
docker network ls

O driver bridge, permite a comunicação entre diversos container no mesmo host.
O driver host, serve para fazer a comunicação entre container e máquina.
O driver overlay da rede ingress é o que faz a mágica acontecer no Swarm.

Todos os nós dentro do Swarm estão dentro na rede ingress.
Essa rede é criada ao iniciar o Swarm.

Toda a comunicação entre os nós é criptografada na rede.

### Service Discovery

docker service create --name servico --replicas 2 alpine sleep 1d
Irá alocar duas task do serviço e chamar o comando sleep de 1 dia para não derrubar o container.

docker service ls

docker service ps {id_serviço}
Em cada nó, chamar para entrar dentro do container:
docker exec -it {id} sh
ping {nome do serviço}

É possivel realizar o ping entre os dois containers mas só com ip não com o nome.

### User-Defined Overlay

Como fazer a comunicação entre os containers através do nome.

Para criar uma rede própria com driver overlay:
docker network create -d overlay my_overlay

A criação dessa rede dentro do Swarm é de maneira lazy.
Todos os Managers do Swarm saberão dessa rede, mas os Workers somente irão conhecê-la a partir do momento que um container for alocado dentro dessa rede.

Para criar o serviço usando a rede my_overlay:
docker service create --name servico --network my_overlay --replicas 2 alpine sleep 1d

Serviços que utilizam redes customizadas conseguem descobrir outros serviços diretamente pelo nome. Agora é possível chamar o ping utilizando o nome do serviço.

Por mais que o driver overlay seja responsável por comunicar múltiplos hosts em uma mesma rede, também podemos conectar containers em escopo local criados com o comando docker container run em redes criadas com esse driver.

Para isso, basta no momento da criação da rede utilizarmos a flag --attachable:
docker network create -d overlay --attachable my_overlay

Com o comando acima, conseguiremos conectar tanto serviços como containers "standalone" em nossa rede my_overlay.

## Deploy com Docker Stack

Como é criado um contexto de serviços utilizando o Swarm?

### Lembrando do Docker Compose

Parecido com o arquivo Docker Compose podemos criar o Swarm.

### Definindo o arquivo de composição

A nova versão (3) do Docker Compose tem novas tags para o Swarm.
deploy, replicas, restart_policy, contraints, etc..

Ver docker-compose-swarm.yml

### Subindo a stack

Para subir o contexto do Swarm utilizando o arquivo, será necessário compiar o conteúdo do docker compose para um Manager.

Dentro do manager fazer o deploy dessa configuração (stack):
docker stack deploy --compose-file docker-compose.yml my_stack_name

Para listar a stack:
docker stack ls

Para visualizar os serviços:
docker service ls

Para remover toda a stack:
docker stam rm my_stack_name

### Volumes e Swarm

Por padrão, tanto o Docker no modo standalone quanto o Docker Swarm, partilham apenas de um driver local para uso de volumes. Isso quer dizer que o Docker Swarm não possui, até então, solução nativa para distribuir volumes entre os nós.

Então, no exemplo do vídeo anterior, ao definirmos o volume para cada serviço, criamos um volume local dentro de cada nó que for executar a tarefa. Logo, os volumes não são compartilhados entre os diferentes nós do cluster.

Existem soluções que não são nativas do Docker Swarm para utilizar volumes distribuídos entre nós, que podem ser consultadas na Docker Store.
