# Estrutura de containers para monitoramento de containers Docker
O objetivo dessa estrutura é facilitar o monitoramento dos serviços que são executados através de containers Docker.

# Docker Swarm
Antes de usar a estrutura para monitoramento de containers Docker, precisamos preparar a infraestrutura que irá executar esse monitoramento. Para isso,
utilizaremos o Docker Swarm.
O Docker Swarm é um gerenciador e orquestrador de cluster Docker. Ele também permite balanceamento de carga, descoberta automática de novos serviços, escala o ambiente em momento de alta demanda e resolve automaticamente em caso de falhas em serviços em contêineres.
Quando usamos uma configuração de Cluster através do Docker Swarm, a execução dos containers utilizará a abordagem denominada serviços.
O Docker Swarm utiliza dois elementos para definir os servicos : Manager e Worker.
Um nó é uma instância  Docker Engine em execução em um host que esteja participando do cluster Swarm. Um cluster Docker deve ter pelo menos um nó Manager.

## Diferenca entre Manager e Worker
Um nó Manager possui a responsabilidade de disparar unidades de trabalho denominadas tasks (tarefas). Um cluster Docker pode ter mais de um nó Manager.
Um nó Worker apenas replica as configurações definidas no Manager, ou seja, ele não pode disparar novas tarefas no Cluster e nem sempre terá os mesmos serviços que são executados no Manager.
OBS: Um nó Manager também é um nó worker.
OBS2: O primeiro nó criado no Cluster, sempre será um Manager.

## Definindo um nó Manager
Para definir um nó manager,e definir o uso do Docker Swarm, execute o comando:
`docker swarm init --advertise-addr ipdamaquinamanager:2377`

OBS: Após executa esse comando, aparecerá o terminal o comando que deve ser executado nos nós workers. Esse comando pode ser acessado de outra maneira, como será visto adiante.

Depois disso, se quiser adicionar um outro nó manager, execute o comando:
`docker swarm join-token  manager`

E execute o comando gerado com o token de acesso na máquina (ou máquinas) que também será  manager.

## Definindo um nó Worker
Depois de definir um nó (ou nós) manager, será preciso definir os nós workers. Para isso, na máquina que foi definido como Manager execute o comando:
`docker swarm join-token  worker`

E execute o comando gerado com o token de acesso na máquina (ou máquinas) que também será  worker.

# Docker Stack
Depois de configurado a infraestrutura com Docker Swarm, chegou a hora de fazer o deploy dos serviços. Os servicos que serão iniciados são: Grafana (para gerar os gráficos de monitoramento a partir das métricas coletadas), InfluxDB (banco de dados que irá amazenar as métricas), cAdvisor (Monitoramento que gera metricas de todos os recursos de hardware que serviços que usam containers costumam usar).
Para iniciar esses serviços, usaremos o Docker Stack. No terminal de uma máquina que seja nó Manager, execute:
`docker stack deploy -c docker-stack.yml monitor`

Depois, execute o seguinte comando para criar o banco de dados InfluxDB:
`docker exec `docker ps | grep -i influx | awk '{print $1}'` influx -execute 'CREATE DATABASE cadvisor'`

Para verificar se os servicos foram criados, execute o comando:
`docker stack services monitor`
Que irá mostrar todos os serviços criados e a quantidade de replicas (de nós workers) que possuem os mesmos serviços.

# Configurando o Grafana
Depois de instalar o Grafana , acesse o mesmo (ipmaquinamanager:8002) para configura-lo.
A primeira configuração será adicionar o bd do Influx ao mesmo. Para isso na página inicial do Grafana, clique em "Create your first data source". Na paǵina que irá se abrir, preencha conforme a imagem abaixo e salve.
![Image of Yaktocat](https://git.epol.splab.ufcg.edu.br/epol/epol-containers/raw/master/servicos/monitoramento-docker/grafana_conf.png)

 Depois, volte a página inicial do Grafana e selecione a opção "Create Your first Dashboard". Selecione o botão "+" no lado esquerdo da tela, e a opção "Import".
 Na tela que irá se abrir, selecione a opção "Upload .json file" e selecione o arquivo "dashboard_docker.json"
 Depois clique em "import" e o grafana estará pronto para uso.
