# Estrutura de containers para monitoramento de containers Docker
O objetivo dessa estrutura é facilitar o monitoramento dos serviços que são executados através de containers Docker.

# Docker Swarm
Antes de usar a estrutura para monitoramento de containers Docker, precisamos preparar a infraestrutura que irá executar esse monitoramento. Para isso,
utilizaremos o Docker Swarm.
O Docker Swarm é um gerenciador e orquestrador de cluster Docker. Ele também permite balanceamento de carga, descoberta automática de novos serviços, escala o ambiente em momento de alta demanda e resolve automaticamente em caso de falhas em serviços em contêineres.
Quando usamos uma configuração de Cluster através do Docker Swarm, a execução dos containers utilizará a abordagem denominada serviços.
O Docker Swarm utiliza dois elementos para definir os servicos : Manager e Worker.
Um nó é uma instância  Docker Engine em execução em um host que esteja participando do cluster Swarm. Um cluster Docker deve ter pelo menos um nó Manager.

## Diferença entre Manager e Worker
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

# Docker Stack Prometheus
Depois de configurado a infraestrutura com Docker Swarm, chegou a hora de fazer o deploy dos serviços. Os servicos que serão iniciados são: Grafana (para gerar os gráficos de monitoramento a partir das métricas coletadas), InfluxDB (banco de dados que irá amazenar as métricas), cAdvisor (Monitoramento que gera metricas de todos os recursos de hardware que serviços que usam containers costumam usar).
Para iniciar esses serviços, usaremos o Docker Stack. No terminal de uma máquina que seja nó Manager, execute:

`docker stack deploy -c docker-stack.yml monitor`


Para verificar se os servicos foram criados, execute o comando:

`docker stack services monitor`

Que irá mostrar todos os serviços criados e a quantidade de replicas (de nós workers) que possuem os mesmos serviços.

Para saber se todos os containers da stack estão executando corretamente, use o comando:
`docker stack ps monitor`

# Docker Stack Prometheus + cAdvisor

**Obs**: Para usar o cAdvisor junto com o Prometheus, descomente as linhas que estão comentadas no arquivo docker-stack.yml

Para usar as métricas já disponibilizadas no cAdvisor, junto com o Prometheus, basta, seguir os passos:

Crie a stack:

`docker stack deploy -c docker-stack.yml monitor`

Depois de criar a stack, execute o seguinte comando para criar o banco de dados InfluxDB:

    `docker exec `docker ps | grep -i influx | awk '{print $1}'` influx -execute 'CREATE DATABASE cadvisor'`

Depois de configurar o bd do influx, acesse o Grafana (ipmaquinamanager:8012) para configura-lo.

A primeira configuração será adicionar o bd do Influx ao mesmo. Para isso na página inicial do Grafana, clique em "Create your first data source". Na paǵina que irá se abrir, preencha conforme a imagem abaixo e salve.

![](https://github.com/tainahemmanuele/monitoramento_tcc/blob/master/img/conf_grafana.png)

# Usando dashboards no grafana #

Todos as dashboards usadas nessa solução, são de uso oficial e estão disponibilizados no site do [Grafana Labs](https://grafana.com/dashboards)

Se estiver usando apenas o Prometheus, use a dashboard:

- [179](https://grafana.com/dashboards/179)

Se estiver usando Prometheus + cAdvisor, use a dashboard:

- [8321](https://grafana.com/dashboards/8321)

Para instalar a dashboard, na página inicial do Grafana, selecione "+" e clique em "import"

![](https://github.com/tainahemmanuele/monitoramento_tcc/blob/master/img/conf_dash_1.png)

Depois, coloque o número da dashboard a ser usada no campo "Grafana.com dashboard" e clique em "load"

![](https://github.com/tainahemmanuele/monitoramento_tcc/blob/master/img/conf_dash_2.png)

Por último, selecione "Prometheus" em options e clique em "import". A dashboard já estará configurada e pronta para uso

![](https://github.com/tainahemmanuele/monitoramento_tcc/blob/master/img/conf_dash_3.png)
