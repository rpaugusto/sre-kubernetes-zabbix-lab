# Laboratório SRE: Monitoramento Avançado de Kubernetes com Zabbix

Este repositório contém a infraestrutura como código (IaC) automatizada para implantar um ambiente completo de observabilidade, focado na avaliação arquitetural de coletas nativas do Kubernetes (K8S) utilizando o Zabbix Server.

## 🎯 Objetivo do Projeto
O propósito deste laboratório é validar os limites de performance e o impacto de processamento no Zabbix Server ao monitorar clusters dinâmicos (Kubernetes) diretamente via Helm Chart (Coleta Direta), servindo de base para estudos comparativos com arquiteturas intermediárias utilizando Prometheus ou OpenTelemetry (OTel Collector).

## 🏗️ Tecnologias Utilizadas
* **Vagrant:** Provisionamento automatizado da máquina virtual de testes.
* **Rocky Linux 9:** Sistema operacional base do laboratório (Padrão Enterprise).
* **Ansible:** Automação da instalação do Docker Engine e ferramentas nativas de Cloud (Kubectl, Kind, Helm).
* **Docker & Docker Compose:** Orquestração da stack de monitoramento (Zabbix Server, Web Nginx, MySQL, Grafana e Zabbix Agent 2).
* **Kind (Kubernetes in Docker):** Instanciação do cluster Kubernetes local de forma efêmera e leve.

## 📊 Arquitetura do Laboratório
1. Uma VM isolada rodando Rocky Linux 9 (IP privado: `192.168.56.10`).
2. Stack de Monitoramento rodando em containers Docker isolados via rede de bridge.
3. Um cluster Kubernetes (Kind) conectado de forma híbrida à rede do Docker Compose para viabilizar a coleta do Zabbix Proxy interno.

## 📂 Estrutura do Projeto

A arquitetura de arquivos do repositório adota o padrão de desenvolvimento profissional com **Ansible Roles**, segmentando as tarefas de infraestrutura de forma modular:

```text
sre-kubernetes-zabbix-lab/
├── .gitignore                      # Proteção de credenciais e arquivos do Vagrant
├── LICENSE                         # Licença pública do projeto (GPL-3.0)
├── README.md                       # Documentação técnica do laboratório
├── Vagrantfile                     # Provisionamento e limites da VM (Rocky Linux 9)
└── ansible/
    ├── inventory.ini               # Inventário local para o provisionamento
    ├── site.yml                    # Playbook orquestrador principal (Chama as Roles)
    └── roles/
        ├── docker_observability/   # Role dedicada à Stack de Monitoramento Base
        │   ├── tasks/
        │   │   └── main.yml        # Instalação do Docker e setup dos containers base
        │   └── files/              # Arquivos estáticos de configuração (Padrão Ansible)
        │       ├── docker-compose.yml
        │       ├── env/
        │       │   └── zabbix.env   # Variáveis de ambiente da stack
        │       ├── zabbix/          # Diretório de persistência do Zabbix Server
        │       └── grafana/         # Provisionamento automático do Grafana
        │           └── provisioning/
        │               ├── dashboards/
        │               │   └── dashboard.yaml
        │               ├── datasources/
        │               │   └── zabbix.yaml
        │               └── plugins/
        │                   └── plugins.yaml
        │
        └── kubernetes_lab/         # Role dedicada ao Cluster Kubernetes
            └── tasks/
                └── main.yml        # Instalação do Kubectl, Kind, Helm e criação do cluster
```

## 🚀 Como Executar o Laboratório

### Pré-requisitos
* VirtualBox instalado.
* Vagrant instalado.

### Passo a Passo
1. Clone este repositório:
   ```bash
   git clone https://github.com
   cd sre-kubernetes-zabbix-lab
   ```
2. Prepare as variáveis de ambiente baseando-se no modelo:
   ```bash
   cp env/zabbix.env.example env/zabbix.env
   # Edite o arquivo env/zabbix.env com suas credenciais de teste
   ```
3. Suba toda a infraestrutura com um único comando:
   ```bash
   vagrant up
   ```
   *O Vagrant irá criar a VM, rodar o Ansible, instalar o Docker, subir a stack do Zabbix/Grafana e preparar o ambiente para a injeção do cluster Kubernetes.*