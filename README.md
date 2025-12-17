# # Projeto DevOps com Vagrant e Ansible

**Alunos:** Caua e Joao
**Disciplina:** Administração de Sistemas Abertos
**Professor:** Leonidas Lima
**Período:** 2025.2

---

## 📘 Introdução

Este projeto tem como objetivo automatizar o provisionamento e a configuração de uma infraestrutura composta por **quatro máquinas virtuais Linux (Debian)**, utilizando as ferramentas **Vagrant** e **Ansible**.

O ambiente simula um cenário real de **DevOps**, integrando e automatizando diversos serviços de rede e sistema, como:

* SSH
* NFS
* LVM
* DHCP
* Apache
* DNS
* MariaDB

Toda a infraestrutura é criada de forma reprodutível, seguindo o conceito de **Infraestrutura como Código (IaC)**.

---

## 🧰 Vagrantfile

O projeto inclui um `Vagrantfile` responsável por criar e configurar **quatro máquinas virtuais**:

### 🗄️ Servidor de Arquivos (arq)

* **Hostname:** `arq.caua.joao.devops`
* **IP estático:** `192.168.56.121` (XX = 21 da matrícula do Caua)
* **Memória RAM:** 512 MB
* **Discos adicionais:** 3 discos de 10 GB cada

---

### 🛢️ Servidor de Banco de Dados (db)

* **Hostname:** `db.caua.joao.devops`
* **IP:** via DHCP
* **MAC Address:** `08:00:27:1A:00:21`
* **Memória RAM:** 512 MB

---

### 🌐 Servidor de Aplicação (app)

* **Hostname:** `app.caua.joao.devops`
* **IP:** via DHCP
* **MAC Address:** `08:00:27:2B:21:00`
* **Memória RAM:** 512 MB

---

### 💻 Cliente (cli)

* **Hostname:** `cli.caua.joao.devops`
* **IP:** via DHCP
* **Memória RAM:** 1024 MB

---

### ⚙️ Configurações Gerais do Vagrant

* Box utilizada: `debian/bookworm64`
* `linked_clone` habilitado
* Guest Additions desabilitado
* Não gera novas chaves SSH
* Trigger para desabilitar o DHCP do VirtualBox na rede host-only

---

## 📜 Playbooks Ansible

### 🔧 common.yml

Aplica configurações comuns a todas as máquinas virtuais:

* Atualização do sistema (`apt update`)
* Instalação e configuração do **Chrony (NTP)** com `pool.ntp.br`
* Configuração do timezone para `America/Recife`
* Criação do grupo `ifpb`
* Criação dos usuários `caua` e `joao`
* Configurações de segurança do SSH:

  * Autenticação apenas por chave pública
  * Bloqueio de acesso root via SSH
  * Permissão apenas para grupos `vagrant` e `ifpb`
  * Banner de aviso legal
* Geração de chaves SSH para os usuários
* Instalação do cliente NFS
* Permissão de sudo sem senha para o grupo `ifpb`

---

### 🗄️ arq.yml — Servidor de Arquivos

Configura o servidor `arq` com múltiplos serviços:

#### DHCP (isc-dhcp-server)

* Rede: `192.168.56.0/24`
* Faixa de IPs: `192.168.56.50` a `192.168.56.100`
* Lease padrão: 180 segundos
* Lease máximo: 3600 segundos
* Gateway: `192.168.56.1`
* Domínio: `caua.joao.devops`

#### DNS (Bind9)

* Aceita consultas da rede interna
* Forwarders: `1.1.1.1` e `8.8.8.8`
* Zona direta e reversa para `caua.joao.devops`
* Registros A:

  * `arq` → `192.168.56.121`
  * `db` → `192.168.56.105`
  * `app` → `192.168.56.115`

#### LVM

* 3 discos de 10 GB
* Volume Group: `dados`
* Logical Volume: `ifpb` (15 GB)
* Sistema de arquivos: `ext4`
* Montagem automática em `/dados`

#### NFS Server

* Compartilhamento: `/dados/nfs`
* Rede permitida: `192.168.56.0/24`
* Usuário: `nfs-ifpb` (sem shell)
* Mapeamento de usuários remotos
* Escrita síncrona (`sync`)

---

### 🛢️ db.yml — Servidor de Banco de Dados

* Instalação do **MariaDB Server**
* Inicialização e habilitação do serviço MariaDB
* Instalação do **autofs**
* Montagem automática de `/dados/nfs` em `/var/nfs`
* Inicialização e habilitação do serviço autofs

---

### 🌐 app.yml — Servidor de Aplicação

* Instalação do **Apache2**
* Criação de página web personalizada com:

  * Descrição do projeto
  * Nomes e matrículas dos integrantes
  * Informações da infraestrutura
* Instalação do **autofs**
* Montagem automática de `/dados/nfs` em `/var/nfs`
* Inicialização e habilitação do Apache e autofs

---

### 💻 cli.yml — Cliente

* Instalação dos pacotes `firefox-esr` e `xauth`
* Configuração do SSH com **X11 Forwarding**
* Instalação do **autofs**
* Montagem automática de `/dados/nfs` em `/var/nfs`

---

## 📁 Arquivos de Configuração (Templates Jinja2)

### Servidor arq

* `dhcpd.conf.j2` — Configuração do DHCP
* `named.conf.options.j2` — Opções do DNS Bind9
* `named.conf.local.j2` — Zonas DNS
* `db.dominio.j2` — Zona direta
* `db.reversa.j2` — Zona reversa

### Servidor app

* `index.html.j2` — Página web do projeto

---

## ▶️ Uso do Projeto

```bash
git clone https://github.com/uperseuz/project_asa.git
cd project_asa
vagrant up
```

Após a criação das máquinas, os playbooks Ansible são executados automaticamente para configurar todo o ambiente.

---

## ✅ Considerações Finais

Este projeto demonstra a aplicação prática dos conceitos de **DevOps**, **automação**, **segurança**, **serviços de rede** e **administração de sistemas Linux**, utilizando ferramentas amplamente adotadas no mercado.

---

📌 *Projeto desenvolvido para fins acadêmicos — IFPB Campus João Pessoa*
