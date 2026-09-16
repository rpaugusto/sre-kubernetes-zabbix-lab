# Laboratório SRE: Monitoramento Avançado de Kubernetes com Zabbix

Este repositório contém a infraestrutura como código (IaC) automatizada para implantar um ambiente completo de observabilidade, focado na avaliação arquitetural de coletas nativas do Kubernetes (K8S) utilizando o Zabbix Server.

---

## 🔄 Notas de Atualização & Melhorias Arquiteturais
Para aproximar o laboratório das necessidades de extrema leveza (alvo: notebooks com 8 GB de RAM), simplicidade e portabilidade, as seguintes mudanças estruturais foram consolidadas:
1. **Correção do Provisionador Vagrant:** Migração para **`ansible_local`**. O Ansible agora é instalado e executado automaticamente de dentro da VM Rocky Linux 9. Isso elimina a dependência de binários do Ansible no sistema hospedeiro e resolve o erro clássico de execução em ambientes Windows/macOS.
2. **Abordagem de Arquivos Estáticos (Sem Jinja2):** Removidos todos os arquivos de template `.j2`. Toda a configuração da infraestrutura foi reescrita utilizando arquivos declarativos puros dentro da pasta `files/` de cada respectiva role, sendo transferidos de forma determinística utilizando o módulo `copy` do Ansible.
3. **Mitigação de Sobrecarga (SRE View):** O ambiente foi otimizado para simular o comportamento de threads de pré-processamento (*preprocessing*) do Zabbix Server ao lidar com grandes volumes de dados advindos de LLDs no cluster Kubernetes.

---

## 🎯 Objetivo do Projeto
O propósito deste laboratório é validar os limites de performance e o impacto de processamento no Zabbix Server ao monitorar clusters dinâmicos (Kubernetes) diretamente via Helm Chart (Coleta Direta), servindo de base para estudos comparativos com arquiteturas intermediárias utilizando Prometheus ou OpenTelemetry (OTel Collector).

---

## 🏗️ Tecnologias Utilizadas
* **Vagrant:** Provisionamento automatizado da máquina virtual de testes.
* **Rocky Linux 9:** Sistema operacional base do laboratório (Padrão Enterprise baseado em RHEL).
* **Ansible:** Automação idempotente aplicada localmente na máquina alvo.
* **Docker & Docker Compose Plugin:** Orquestração da stack de monitoramento baseada em imagens Alpine altamente enxutas.
* **k3s (Kubernetes enxuto):** Instanciação do cluster Kubernetes local de forma efêmera e leve (customizado sem plugins pesados adicionais para preservar RAM).

---

## 📊 Arquitetura do Laboratório
* Uma VM isolada rodando Rocky Linux 9 (IP privado: `192.168.56.10`).
* Stack de Monitoramento rodando em containers Docker isolados via rede de bridge.
* Um cluster Kubernetes conectado de forma híbrida à rede para viabilizar as coletas locais pelo agente e as requisições sintéticas em HTTP.

---

## 📂 Estrutura do Projeto
A arquitetura de arquivos adota o padrão de desenvolvimento profissional com Ansible Roles, segmentando as tarefas de infraestrutura por responsabilidades isoladas e utilizando arquivos de configuração estáticos:

```text
sre-kubernetes-zabbix-lab/
├── .gitignore                      # Proteção de credenciais e arquivos do Vagrant
├── LICENSE                         # Licença pública do projeto (GPL-3.0)
├── README.md                       # Documentação técnica do laboratório
├── Vagrantfile                     # Provisionamento e limites da VM (Rocky Linux 9 + ansible_local)
└── ansible/
    ├── ansible.cfg                 # Otimizações de execução do Ansible
    ├── inventory.ini               # Inventário apontando para o ambiente local
    ├── site.yml                    # Playbook orquestrador principal (Chama as Roles)
    └── roles/
        ├── common/                 # Setup de repositórios base (EPEL, DNF)
        ├── docker/                 # Instalação e ativação da engine do Docker
        ├── zabbix_docker/          # Gerenciamento da Stack Zabbix + MySQL via Compose
        │   ├── tasks/
        │   │   └── main.yml
        │   └── files/              # Configurações estáticas do monitoramento
        │       └── docker-compose.yml
        ├── zabbix_agent/           # Agente nativo instalado e configurado no Host
        ├── k3s/                    # Cluster Kubernetes leve sem componentes extras
        ├── sample_app_docker/      # Aplicação isolada de teste no Docker
        └── sample_app_k8s/         # Aplicação com limites rígidos de recursos no k3s
            ├── tasks/
            │   └── main.yml
            └── files/              # Manifesto estático de implantação Kubernetes
                └── k8s-app-manifest.yml
```

---

## 🚀 Como Executar o Laboratório

### Pré-requisitos
* VirtualBox instalado.
* Vagrant instalado.

### Passo a Passo
1. Clone este repositório para a sua máquina local:
   ```bash
   git clone https://github.com
   cd sre-kubernetes-zabbix-lab
   ```
2. Suba toda a infraestrutura com um único comando:
   ```bash
   vagrant up
   ```
   *O Vagrant irá criar a VM Rocky Linux 9, instalar o Ansible localmente dentro dela e disparar a execução de todas as roles modulares sequencialmente de forma automatizada.*

---

## 🔍 Comandos de Validação Mínima (Troubleshooting)
Após o término do provisionamento, acesse a VM para auditar os status de saúde do ambiente:
```bash
vagrant ssh lab01
```

Execute as validações operacionais de SRE:
```bash
# Verificar se os containers do Zabbix Server, Frontend e Banco de dados estão de pé
docker compose -f /opt/zabbix-docker/docker-compose.yml ps

# Validar se o nó do Kubernetes (k3s) está ativo e pronto
kubectl get nodes -o wide

# Verificar o estado de saúde dos pods internos do cluster
kubectl get pods -A

# Testar as aplicações de teste expostas em HTTP
curl -I http://localhost:8081   # Aplicação Docker isolada
curl -I http://localhost:30080  # Aplicação Kubernetes exposta via NodePort
```

---

## 📈 Monitoração Ativa no Zabbix Web
Acesse a interface de monitoramento no seu navegador pelo endereço: `http://192.168.56.10` (Credenciais: `Admin` / `zabbix`).

* **Host Linux (`lab01`):** Monitorado via template nativo `Linux by Zabbix agent`.
* **Aplicação Docker:** Validação de processos ativos utilizando a chave `proc.num[,,,nginx]` na porta `8081`.
* **Serviço do k3s:** Monitoramento de saúde do daemon principal através da chave do agente `proc.num[,,,k3s]`.
* **Aplicação no Kubernetes:** Monitorada via **Cenário Web** (*Web Scenario*) apontando para `http://192.168.56.10:30080` esperando retorno HTTP Status `200`.
