### Considerações iniciais

O Elastic Stack é um solução bem completa que atende diversas áreas e necessidade deste universo de T.I. Sabemos que as plataformas e aplicações estão cada vez mais complexas, com inserção de arquitetura de micro serviços, orquestradores de contêineres, cloud e alta disponibilidade. Com isso, o Elastic Stack oferece desde solução de monitoramento de aplicação e infraestrutura até análise de logs e segurança com o SIEM. Permitindo construções de dashboards personalizadas e alertas das anomalias/problemas identificados no monitoramento através de uma única console. A stack dispõe de um módulo de machine learning que agrega valor ao serviço, oferecendo comodidade nas tarefas diárias, relacionadas a performance da aplicação dentre outras.

A instalação e configuração da elastic estack foi realizada com docker utilizando as imagens na versão 7.8.0

São 5 tecnologias a serem implementadas (elasticSearch, kibana, apm-server, heartbeat, metricbeat)

Foram criados 2 volumes(elastic-stack_elasticsearch e elastic-stack_kibana), a configuração do elasticsearch e kibana estão configuradas nesses volumes.

O metricbeats deverá ser configurado em cada servidor que se pretende monitorar, ele será instalado como um serviço dentro desses servidores

### Instalação

1. configurando docker
```
 docker-compose up
```
### Configuração inicial

1. Configurando heartbeat:
Acesse o arquivo: `/home/ubuntu/elastic-stack/heartbeat/heartbeat.yml`
Adiciona todos os monitores que deseja monitorar conforme exemplo da documentação: `https://www.elastic.co/pt/blog/uptime-monitoring-with-heartbeat-and-the-elastic-stack` 

2. Habilitando pack security da elastic stack:

executar comando:
```
docker exec -it elasticsearch bash
```
executar comando:
```
bin/elasticsearch-setup-passwords interactive
```
Inserir senhas de acordo com sua escolha.

acessar arquivo `/usr/share/elasticsearch/config/elasticsearch.yml`
adicionar linha
```
pack.security.enabled: true
```
sair do container
executar comando:
```
docker exec -it kibana bash
```
acessar arquivo `/usr/share/kibana/config/kibana.yml`
adicionar linhas:
```
elasticsearch.hosts: [ "http://elasticsearch:9200" ]
monitoring.ui.container.elasticsearch.enabled: true
elasticsearch.username: "elastic"
elasticsearch.password: "ElastiCPassword"
xpack.security.sessionTimeout: 7200000
xpack.security.encryptionKey: "something_at_least_32_characters"
```
sair do container
no caminho `/home/ubuntu/elastic-stack/ executar comando "docker-compose restart && docker-compose up -d --b"`

3. Configurando metricbeat:

Acesse: http://localhost:5601
Insira login `elastic` e senha conforme senha configurada seguindo exemplo do arquivo `senhas`
Após realizar login no kibana, no card Observability, pressione a opção `add metric data`
Escolha as opções que deseja configurar o metrics e realize passo a passo conforme documentação
Sempre que realizar a configuração do metrics em algum lugar, sempre atualize o arquivo `metricbeat.yml` geralmente no caminho `/etc/metricbeat` e insira as credencias do elasticsearch


### Configurações adicionais

1. Configurando usuários:

Essa configuração de segurança as vezes demora para aparecer após habilitar modo de segurança, então tenha calma se ela não aparecer de primeira
Acesse: http://localhost:5601
Abra o menu `management->stack management->security->users`
Pressione o botão create user
Crie os usuários que desejar

2. Configurando índices:

Acesse: http://localhost:5601
Abra o menu: `management->stack management->-kibana>index patterns`
Pressione o botão create index pattern
Digite o nome do serviço até que só uma opção apareça
É importante que sempre digite o nome completo por exemplo `heartbeat`, `metricbeat`. Dessa forma facilita consultas elastic futuras
Selecione a opção de filtro como `timestamp`
Precione `create index pattern`
Após criar os index você pode gerenciar politicas de tamanho e vida do index na opção `management->stack management->-elasticsearch>index lifecycle policies`

### Visualizando dashboards

Abra o menu `kibana->dashboard`
Pesquise o dashboard de acordo com o que você configurou do metricbeats

### Configurações do apm-server nas apis

As variáveis de configuração do apm-server serão lidas do arquivo .env
No arquivo main.ts terás uma linha como essa `const apm = require('elastic-apm-node').start();`

Essa linha inicia a lib do apm.

É importante iniciar o as configurações do `.env` antes de executar essa linha geralmente com esse comando `dotenv.config();`

Seguindo todas essas recomendações, você terá uma elastic stack configurada com já várias ferramentas para brincar! :)
