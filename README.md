# Desafio 4Linux - Relatório técnico

#### Tempo de execução
O tempo gasto no laboratório foram cerca de 12hrs. Iniciei na quinta-feira (dia 28/04) verificando os acessos disponibilizados e realizando a cópia do ambiente Docker para uma infraestrutura local criada pelo Vagrant. Concluindo no sábado (dia 30/05) executando a *playbook* no ambiente da GCP e corrigindo alguns detalhes necessários após a mudança do ambiente local para a GCP.
#### Ambiente de desenvolvimento
Por escolha própria, iniciei todo o desenvolvimento em um ambiente local, utilizando o Vagrant e Ansible, criando duas máquinas virtuais com Debian 10, simulando o ambiente da GCP. Tenho preferência nesta abordagem, pois tenho liberdade para reiniciar e destruir todo o ambiente, sem impacto no ambiente de *"produção"*.
#### Principais dificuldades
##### Grafana
Encontrei dificuldades com o provisionamento do Grafana, a API apresentava *bugs* na criação do *datasource* do Prometheus, e a *collection* do Ansible apresentava falhas ao sobrescrever os *dashboards*. Então mesclei a chamada de API com URI do Ansible e também as *collection* da comunidade, não é a solução que mais me agrada, prefiro manter uma padronização.
```yaml
- name: SERVER | Adicionando o Prometheus 
  grafana_datasource:
    name: DS_PROMETHEUS
    grafana_url: http://localhost:3000
    grafana_user: admin
    grafana_password: "{{ GRAFANA_PASS }}"
    basic_auth_user: admin
    basic_auth_password: "{{ PROMETHEUS_PASSWORD }}"
    ds_type: prometheus
    ds_url: https://client-4linux.libre.tec.br/prometheus

- name: SERVER | Importando os dashboards
    uri:
      url: "https://server-4linux.libre.tec.br/api/dashboards/db"
      user: 'admin'
      password: "{{ GRAFANA_PASS }}"
      force_basic_auth: true
      method: POST
      body_format: json
      body: >
        {
          "dashboard": {{ lookup("file", item) }},
          "overwrite": true,
          "message": "Updated by ansible"
        }
    with_fileglob:
      - '/tmp/_grafana*.json'
```
##### Autenticação Prometheus
Como o Prometheus está exposto, adicionei autenticação, mas encontrei um problema com a *flag* **-web.config.file**, pois o Prometheus na imagem utilizada informa que não é reconhecida. Então adicionei o NGINX no *front-end* para fazer a autenticação.
##### Ansible - with_fileglob
Não foi uma dificuldade em si, porém perdi um tempo até o entendimento do problema. Como o Vagrant executa as *playbooks* localmente, o *with_fileglob* funcionou. Porém, ao executar no ambiente da GCP os arquivos não eram encontrados, pois, o *with_fileglob* busca os arquivos locais.
##### Provisionamento do Ambiente
O ambiente foi totalmente provisionado pelo Ansible, incluindo a criação dos *exporters*, *datasource*, importação dos *dashboards* e criação/alteração dos registros DNS na CloudFlare.
Para provisionar é necessário ter o Ansible instalado, com o arquivo *secrets.json* com as senhas necessárias e o arquivo de inventário *inventory.ini*. Os arquivos utilizados no provisionamento foi enviado por e-mail.  
Comando utilizado para o provisionamento com Ansible:
```shell
ansible-playbook -i inventory.ini provision/playbook.yaml --extra-vars="@secrets.json"
```
##### Implementações adicionais
Implementações adicionais se resumem na configuração de autenticação e no uso de TLS no Prometheus e Grafana com o CloudFlare. Todos na própria *playbook* do Ansible.
##### Links para acesso ao ambiente
[Grafana](https://server-4linux.libre.tec.br/login)
[APP](https://client-4linux.libre.tec.br/)
[Prometheus](https://client-4linux.libre.tec.br/prometheus)