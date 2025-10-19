# Módulo 07: Tópicos Avançados e Casos de Uso Reais

Com os fundamentos do Pacemaker estabelecidos, este módulo explora funcionalidades mais avançadas e cenários de uso específicos. Abordaremos como o Pacemaker pode gerenciar máquinas virtuais diretamente, a funcionalidade Pacemaker Remote para estender o cluster e a integração com serviços de aplicação mais complexos.

## 7.1 Gerenciamento de Máquinas Virtuais com Pacemaker

O Pacemaker pode ir além de apenas gerenciar serviços dentro de VMs; ele pode gerenciar as próprias máquinas virtuais como recursos. Isso significa que, em caso de falha do host (nó físico), o Pacemaker pode migrar automaticamente a VM para outro host saudável. Isso é particularmente útil em ambientes de virtualização.

### 7.1.1 Agente de Recurso `ocf:heartbeat:VirtualDomain`

O agente `VirtualDomain` permite que o Pacemaker controle VMs gerenciadas por hypervisors como KVM/QEMU (via Libvirt), Xen, ou VMware. Ele pode iniciar, parar, migrar e monitorar o estado de uma VM.

**Pré-requisitos:**

*   O Pacemaker e o Corosync devem estar instalados e configurados nos **hosts** que executarão as VMs.
*   O Libvirt deve estar instalado e configurado nos hosts.
*   A VM deve estar registrada no Libvirt e ter um domínio XML definido.

**Exemplo de Criação de Recurso de VM:**

Suponha que você tenha uma VM chamada `my_application_vm` registrada no Libvirt. Você pode criar um recurso Pacemaker para ela:

```bash
sudo pcs resource create vm_resource ocf:heartbeat:VirtualDomain domain=my_application_vm hypervisor_uri="qemu:///system" migration_transport=ssh op monitor interval=30s
```

*   `vm_resource`: Nome do recurso Pacemaker para a VM.
*   `ocf:heartbeat:VirtualDomain`: O agente de recurso para VMs.
*   `domain=my_application_vm`: O nome do domínio da VM no Libvirt.
*   `hypervisor_uri="qemu:///system"`: O URI do hypervisor (Libvirt). Para KVM local, `qemu:///system` é comum.
*   `migration_transport=ssh`: Especifica que a migração da VM deve usar SSH. Certifique-se de que o acesso SSH sem senha esteja configurado entre os hosts.
*   `op monitor interval=30s`: Monitora o estado da VM a cada 30 segundos.

Com este recurso, o Pacemaker pode:

*   Iniciar a VM em um host disponível.
*   Parar a VM.
*   Migrar a VM para outro host em caso de falha do host atual ou para balanceamento de carga.

## 7.2 Pacemaker Remote

**Pacemaker Remote** é uma funcionalidade que permite estender um cluster Pacemaker para incluir nós que não executam Corosync. Isso é útil para gerenciar recursos em máquinas virtuais, contêineres ou servidores físicos que não precisam da sobrecarga completa de um cluster Corosync, mas ainda se beneficiam do gerenciamento de recursos do Pacemaker. [1]

O Pacemaker Remote consiste em um daemon (`pacemaker_remote`) que é executado no nó remoto e se conecta a um nó do cluster principal via TCP. Ele atua como um proxy, permitindo que o Pacemaker no cluster principal gerencie recursos no nó remoto.

### 7.2.1 Cenários de Uso:

*   **VMs e Contêineres:** Gerenciar serviços dentro de VMs ou contêineres sem a necessidade de instalar um cluster Corosync completo dentro de cada um.
*   **Clusters Estendidos:** Incluir servidores em locais geograficamente distantes ou em redes com alta latência onde o Corosync não seria ideal.

### 7.2.2 Configuração Básica (Exemplo Simplificado):

1.  **No nó remoto (VM ou servidor):**
    *   Instale o pacote `pacemaker-remote` (o nome pode variar, ex: `pacemaker-remote` no RHEL/CentOS, `pacemaker-remote` no Debian/Ubuntu).
    *   Habilite e inicie o serviço `pacemaker_remote`.

    ```bash
    # Exemplo para RHEL/CentOS:
    sudo dnf install pacemaker-remote -y
    sudo systemctl enable --now pacemaker_remote
    ```

2.  **No cluster principal (em um dos nós):**
    *   Crie um recurso `remote-node` que represente o nó remoto.

    ```bash
    sudo pcs resource create my_remote_node ocf:pacemaker:remote_node hostname=remote.example.com reconnect_delay=10 op monitor interval=60s
    ```

    *   `my_remote_node`: Nome do recurso que representa o nó remoto.
    *   `ocf:pacemaker:remote_node`: Agente de recurso para nós remotos.
    *   `hostname=remote.example.com`: Nome do host ou IP do nó remoto.
    *   `reconnect_delay=10`: Tempo em segundos para tentar reconectar se a conexão cair.

Após configurar o nó remoto como um recurso, você pode criar outros recursos e configurá-los para serem executados neste nó remoto, assim como faria com um nó regular do cluster.

## 7.3 Integração com Serviços Específicos e Casos de Uso Reais

O Pacemaker pode ser integrado com praticamente qualquer serviço ou aplicação que possa ser iniciado, parado e monitorado por scripts. Abaixo, alguns exemplos e considerações:

### 7.3.1 Servidores Web (Apache/Nginx)

Já vimos como gerenciar o Apache/Nginx como um recurso `systemd`. Em cenários mais avançados, você pode precisar de monitoramento mais granular (ex: verificar se uma página específica está respondendo) ou ações de recuperação mais complexas. Para isso, você pode criar seus próprios agentes OCF personalizados ou usar agentes existentes que suportem mais parâmetros.

### 7.3.2 Bancos de Dados (PostgreSQL, MySQL/MariaDB)

Gerenciar bancos de dados com Pacemaker é um caso de uso comum para HA. Isso geralmente envolve:

*   **Recurso de IP Virtual:** Para que a aplicação se conecte sempre ao mesmo IP.
*   **Recurso de Sistema de Arquivos:** Para montar o diretório de dados do banco de dados (que deve estar em armazenamento compartilhado ou replicado).
*   **Recurso do Serviço de Banco de Dados:** Para iniciar/parar o serviço do banco de dados.
*   **Monitoramento:** Verificações de saúde que vão além de apenas saber se o processo está rodando, como verificar a capacidade de conexão ao banco de dados ou a replicação.

**Exemplo (PostgreSQL com replicação primário/secundário):**

Em um cenário de replicação, o Pacemaker pode gerenciar qual nó é o primário e qual é o secundário, promovendo o secundário a primário em caso de falha do primário. Isso geralmente envolve agentes de recursos específicos para o banco de dados (ex: `ocf:heartbeat:pgsql` para PostgreSQL) que entendem o estado de replicação.

### 7.3.3 Balanceadores de Carga (HAProxy, Keepalived)

Embora o Pacemaker possa gerenciar IPs virtuais, para balanceamento de carga real de aplicações web, ele pode ser usado em conjunto com balanceadores de carga como HAProxy ou Keepalived. O Pacemaker garantiria a alta disponibilidade do próprio serviço HAProxy, enquanto o HAProxy distribuiria o tráfego para os servidores de aplicação.

### 7.3.4 Aplicações Personalizadas

Para aplicações desenvolvidas internamente, você pode criar seus próprios agentes de recurso OCF. Estes são scripts shell que implementam as ações `start`, `stop`, `monitor` e `meta-data` que o Pacemaker espera. Isso oferece total flexibilidade para integrar qualquer aplicação ao cluster HA.

## Exercícios Práticos

1.  **Gerenciamento de VM:**
    *   Certifique-se de que o SSH sem senha esteja configurado entre seus hosts Libvirt (se você tiver mais de um) ou entre o nó do cluster e o host Libvirt.
    *   Crie uma VM simples no seu host Libvirt (se ainda não tiver uma para testes).
    *   Crie um recurso Pacemaker do tipo `ocf:heartbeat:VirtualDomain` para gerenciar esta VM.
    *   Teste a migração da VM manualmente (`sudo pcs resource move vm_resource <outro_host>`) e observe o comportamento.
    *   Simule a falha do host onde a VM está rodando e observe se o Pacemaker migra a VM para outro host.

2.  **Pacemaker Remote (Opcional, para quem deseja explorar):**
    *   Crie uma terceira VM ou use um contêiner como nó remoto.
    *   Instale e configure o `pacemaker_remote` nesta VM/contêiner.
    *   No seu cluster principal, crie um recurso `ocf:pacemaker:remote_node` para o nó remoto.
    *   Tente criar um recurso simples (ex: um IP virtual ou um serviço `systemd`) e configure-o para ser executado no nó remoto.

3.  **Integração de Banco de Dados (Exemplo simplificado com PostgreSQL):**
    *   Em ambos os nós do seu cluster, instale o PostgreSQL.
    *   Configure um IP virtual e um recurso `systemd:postgresql`.
    *   Crie um grupo de recursos para o IP e o PostgreSQL.
    *   (Desafio) Pesquise sobre `ocf:heartbeat:pgsql` e tente configurar um recurso PostgreSQL mais avançado que entenda o conceito de primário/secundário.

## Referências

[1] ClusterLabs. *Pacemaker Remote: Guest Node Walk-through*. Disponível em: [https://clusterlabs.org/projects/pacemaker/doc/2.1/Pacemaker_Remote/html/kvm-tutorial.html](https://clusterlabs.org/projects/pacemaker/doc/2.1/Pacemaker_Remote/html/kvm-tutorial.html)

[2] ClusterLabs. *Pacemaker Explained*. Disponível em: [https://clusterlabs.org/projects/pacemaker/doc/2.1/Pacemaker_Explained/singlehtml/](https://clusterlabs.org/projects/pacemaker/doc/2.1/Pacemaker_Explained/singlehtml/)

