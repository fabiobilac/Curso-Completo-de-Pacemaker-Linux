# Módulo 03: Instalação e Configuração Básica do Pacemaker/Corosync

Neste módulo, abordaremos os passos essenciais para instalar e configurar um cluster Pacemaker/Corosync básico. A ferramenta `pcs` (Pacemaker/Corosync Configuration System) será a principal interface de linha de comando utilizada para gerenciar o cluster.

## 3.1 Pré-requisitos de Rede e Sistema Operacional

Antes de iniciar a instalação, é fundamental que seu ambiente de laboratório (configurado no Módulo 2) esteja com os seguintes pré-requisitos atendidos em todos os nós do cluster:

*   **Sistema Operacional:** Uma distribuição Linux compatível (ex: CentOS Stream 8/9, Ubuntu Server 20.04/22.04, RHEL 8/9). Para este curso, assumiremos um ambiente baseado em RHEL/CentOS ou Ubuntu, com comandos adaptados quando necessário.
*   **Conectividade de Rede:** Todos os nós devem ser capazes de se comunicar entre si via rede. Isso inclui:
    *   **Endereços IP Estáticos:** Configurar IPs estáticos para todas as interfaces de rede dos nós.
    *   **Resolução de Nomes:** Configurar `/etc/hosts` ou um servidor DNS para que os nós possam resolver os nomes uns dos outros.
    *   **Firewall:** Desabilitar ou configurar o firewall (ex: `firewalld` ou `ufw`) para permitir a comunicação entre os nós nas portas necessárias para Corosync e Pacemaker (portas 5404/udp, 2224/tcp, 3121/tcp, 5560/tcp).

### Exemplo de configuração de `/etc/hosts` (em todos os nós):

```bash
192.168.122.101 node1.example.com node1
192.168.122.102 node2.example.com node2
```

## 3.2 Instalação dos Pacotes Essenciais

A instalação dos componentes do Pacemaker e Corosync geralmente envolve a instalação de um conjunto de pacotes. Os principais são `pacemaker`, `corosync` e `pcs`.

### Para sistemas baseados em RHEL/CentOS (usando `dnf` ou `yum`):

```bash
sudo dnf install pacemaker corosync pcs -y
```

### Para sistemas baseados em Debian/Ubuntu (usando `apt`):

```bash
sudo apt update
sudo apt install pacemaker corosync pcs -y
```

Após a instalação, é recomendável habilitar e iniciar os serviços `pcsd` e `corosync` (embora `corosync` seja geralmente iniciado pelo `pcsd`): [1]

```bash
sudo systemctl enable pcsd corosync
sudo systemctl start pcsd corosync
```

## 3.3 Configuração Inicial do `pcs` e Autenticação

O `pcsd` é um daemon que permite que a ferramenta `pcs` se comunique com os nós do cluster. Para que `pcs` possa gerenciar o cluster, é necessário definir uma senha para o usuário `hacluster` (criado automaticamente durante a instalação) em todos os nós e autenticá-los.

### 3.3.1 Definir senha para o usuário `hacluster` (em todos os nós):

```bash
sudo passwd hacluster
# Digite uma senha forte e anote-a, pois será usada para autenticação.
```

### 3.3.2 Autenticar os nós do cluster (executar em **apenas um** dos nós, ex: `node1`):

```bash
sudo pcs host auth node1 node2
# Será solicitada a senha do usuário hacluster para cada nó.
```

Se a autenticação for bem-sucedida, você verá uma mensagem similar a:

```
node1: Authorized
node2: Authorized
```

## 3.4 Criação do Cluster

Com os pacotes instalados e os nós autenticados, o próximo passo é criar o cluster. Este comando também deve ser executado em **apenas um** dos nós (ex: `node1`).

```bash
sudo pcs cluster setup my_first_cluster node1 node2
```

*   `my_first_cluster`: É o nome que você dará ao seu cluster. Escolha um nome descritivo.
*   `node1 node2`: São os nomes dos nós que farão parte do cluster. Certifique-se de que correspondem aos nomes de host ou entradas em `/etc/hosts`.

Após a criação, inicie e habilite o cluster para que ele seja iniciado automaticamente no boot:

```bash
sudo pcs cluster start --all
sudo pcs cluster enable --all
```

## 3.5 Verificação do Status do Cluster

Para verificar se o cluster foi configurado corretamente e está em execução, utilize o comando `pcs status`.

```bash
sudo pcs status
```

A saída esperada mostrará os nós online, o quórum estabelecido e, inicialmente, nenhum recurso configurado.

### Exemplo de saída esperada:

```
Cluster name: my_first_cluster
Cluster Summary:
  * Stack: corosync
  * Current DC: node1 (version 2.1.6-8.el9_2.1 - pacemaker-2.1.6-8.el9_2.1)
  * Last updated: Mon Oct 28 10:00:00 2025
  * Last change: Mon Oct 28 10:00:00 2025 by root via crmd on node1
  * 2 nodes configured
  * 0 resources configured

Online: [ node1 node2 ]

Full list of resources:

No resources configured

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled
```

## 3.6 Desabilitando STONITH (Temporariamente para Laboratório)

Para fins de laboratório e aprendizado inicial, onde a configuração de um dispositivo STONITH pode ser complexa, é comum desabilitar temporariamente o STONITH. **No entanto, em ambientes de produção, o STONITH é absolutamente essencial e nunca deve ser desabilitado.**

Para desabilitar o STONITH:

```bash
sudo pcs property set stonith-enabled=false
```

Para reabilitar (o que será feito em um módulo futuro):

```bash
sudo pcs property set stonith-enabled=true
```

## Referências

[1] Red Hat. (n.d.). *Configuring and managing high availability clusters*. Disponível em: [https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html-single/configuring_and_managing_high_availability_clusters/index](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html-single/configuring_and_managing_high_availability_clusters/index)

