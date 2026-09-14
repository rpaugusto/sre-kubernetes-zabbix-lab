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

```
sre-kubernetes-zabbix-lab/
├── .gitignore
├── README.md
├── Vagrantfile
├── env/
│   └── zabbix.env.example        <-- APENAS o modelo, sem senhas reais
└── ansible/
    ├── site.yml                  <-- Seu main.yml atualizado
    └── files/                    <-- Onde vão os arquivos de configuração
        ├── docker-compose.yml
        └── grafana/
            └── provisioning/
                ├── datasources/
                │   └── zabbix.yaml
                └── plugins/
                    └── plugins.yaml
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
