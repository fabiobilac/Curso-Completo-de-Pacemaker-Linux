# Módulo 05: STONITH/Fencing

No contexto de clusters de alta disponibilidade, o **STONITH** (Shoot The Other Node In The Head) ou **Fencing** é um mecanismo crucial para garantir a integridade dos dados e a consistência do cluster. Sem um mecanismo de fencing adequado, um cenário de *split-brain* pode levar à corrupção irreversível dos dados. Este módulo detalhará a importância do STONITH e como configurá-lo, com foco em ambientes virtualizados usando Libvirt.

## 5.1 A Importância do STONITH

Imagine um cenário onde a comunicação de rede entre dois nós de um cluster é interrompida. Cada nó pode erroneamente acreditar que o outro falhou e tentar assumir o controle dos mesmos recursos compartilhados (como um disco de armazenamento). Essa situação é conhecida como **split-brain**.

Se ambos os nós tentarem escrever no mesmo disco simultaneamente, a **corrupção de dados** é inevitável. O STONITH resolve esse problema garantindo que, em caso de falha ou perda de comunicação, apenas um nó possa acessar os recursos compartilhados. Ele faz isso de forma proativa, isolando o nó problemático do cluster, muitas vezes desligando-o ou removendo seu acesso aos recursos.

> "STONITH (Shoot The Other Node In The Head) is a critical component of any high-availability cluster. It ensures that a failed node is truly dead and not just unresponsive, preventing data corruption." [1]

### Princípios do STONITH:

*   **Eliminação de Split-Brain:** Impede que múltiplos nós pensem que são os únicos ativos, evitando que tentem controlar os mesmos recursos.
*   **Proteção de Dados:** Garante que os dados compartilhados não sejam corrompidos por acessos concorrentes indevidos.
*   **Recuperação Confiável:** Permite que o cluster se recupere de falhas de forma segura e automatizada.

## 5.2 Tipos de Agentes STONITH

Os agentes STONITH são programas que o Pacemaker usa para realizar o fencing. Eles interagem com hardware ou software externos para isolar um nó. Alguns tipos comuns incluem:

*   **Dispositivos de Gerenciamento de Energia (Power Fencing):** Desligam ou reiniciam um servidor remotamente (ex: IPMI, iLO, DRAC, PDU).
*   **Fencing de Armazenamento (Storage Fencing):** Revogam o acesso de um nó a um armazenamento compartilhado (ex: Fibre Channel, iSCSI).
*   **Fencing de Virtualização (Virtualization Fencing):** Interagem com o hypervisor para desligar ou reiniciar uma máquina virtual (ex: `fence_virsh` para KVM/Libvirt, `fence_vmware` para VMware).

Neste módulo, focaremos no `fence_virsh`, que é ideal para nosso ambiente de laboratório baseado em Libvirt.

## 5.3 Configuração de STONITH com `fence_virsh` e Libvirt

Para usar `fence_virsh`, o host Libvirt (onde suas VMs `node1` e `node2` estão rodando) precisa ser acessível e ter as ferramentas `virsh` instaladas. O agente `fence_virsh` permite que o Pacemaker desligue ou reinicie uma VM através do hypervisor.

### 5.3.1 Pré-requisitos

1.  **Instalação do `fence-agents`:** O pacote `fence-agents` deve ser instalado em **todos os nós do cluster**.

    #### Para sistemas baseados em RHEL/CentOS:
    ```bash
    sudo dnf install fence-agents-all -y
    ```

    #### Para sistemas baseados em Debian/Ubuntu:
    ```bash
    sudo apt install fence-agents -y
    ```

2.  **Acesso SSH ao Host Libvirt:** O Pacemaker precisa ser capaz de se conectar via SSH ao host Libvirt para executar comandos `virsh`. Crie um usuário no host Libvirt (se ainda não tiver um) e configure o acesso SSH sem senha a partir dos nós do cluster.

    *   **No host Libvirt:** Crie um usuário (ex: `libvirt_user`) e configure-o para ter permissão para executar comandos `virsh`.
    *   **Nos nós do cluster:** Gere um par de chaves SSH e copie a chave pública para o `libvirt_user` no host Libvirt.

    ```bash
    # No node1 e node2:
    ssh-keygen -t rsa -b 4096
    ssh-copy-id libvirt_user@<IP_DO_HOST_LIBVIRT>
    ```

### 5.3.2 Criando o Recurso STONITH

Agora, vamos criar o recurso STONITH no Pacemaker. Este recurso será do tipo `fence_virsh`.

```bash
sudo pcs stonith create fence_node1 fence_virsh pcmk_host_list=node1 ipaddr=<IP_DO_HOST_LIBVIRT> login=libvirt_user power_wait=2 op monitor interval=60s
sudo pcs stonith create fence_node2 fence_virsh pcmk_host_list=node2 ipaddr=<IP_DO_HOST_LIBVIRT> login=libvirt_user power_wait=2 op monitor interval=60s
```

*   `fence_node1`, `fence_node2`: Nomes dos recursos STONITH.
*   `fence_virsh`: O agente STONITH para Libvirt.
*   `pcmk_host_list=node1`: Associa este dispositivo STONITH ao nó `node1`.
*   `ipaddr=<IP_DO_HOST_LIBVIRT>`: Endereço IP do seu host Libvirt.
*   `login=libvirt_user`: Usuário SSH no host Libvirt.
*   `power_wait=2`: Tempo em segundos para esperar após uma operação de energia.
*   `op monitor interval=60s`: Monitora o dispositivo STONITH a cada 60 segundos.

Após criar os dispositivos STONITH, você deve reabilitar o STONITH no cluster (se o desabilitou temporariamente no Módulo 3):

```bash
sudo pcs property set stonith-enabled=true
```

Verifique o status do cluster:

```bash
sudo pcs status
```

Você deverá ver os dispositivos STONITH configurados e ativos.

## 5.4 Testando o STONITH

Testar o STONITH é crucial para garantir que ele funcione corretamente em um cenário de falha. **Cuidado ao testar em ambientes de produção!** Em nosso laboratório, podemos simular uma falha de nó.

### Cenário de Teste: Falha de um Nó

1.  Certifique-se de que seu cluster está em funcionamento e com recursos configurados (ex: o `web_service_group` do Módulo 4) e que o STONITH está habilitado.
2.  No `node1`, simule uma falha de energia ou kernel panic (em um ambiente real, isso aconteceria sem sua intervenção):

    ```bash
    # No node1:
    sudo reboot -f
    ```

3.  Observe o `node2` e o log do Pacemaker (`journalctl -u pacemaker -f` ou `/var/log/pacemaker.log`). O Pacemaker no `node2` detectará que `node1` está inacessível.
4.  O Pacemaker tentará fazer fencing do `node1` usando o dispositivo `fence_node1`. Você deverá ver o `node1` sendo desligado pelo `fence_virsh` no host Libvirt.
5.  Após o fencing, os recursos que estavam em `node1` (ex: `web_service_group`) deverão ser migrados para `node2` e iniciados lá.

Verifique o status do cluster no `node2`:

```bash
sudo pcs status
```

Você deverá ver `node1` como offline e os recursos em `node2`.

## Exercícios Práticos

1.  Instale o pacote `fence-agents` em ambos os nós do cluster.
2.  No seu host Libvirt, crie um usuário (`libvirt_user`) e configure o acesso SSH sem senha a partir de `node1` e `node2` para este usuário.
3.  Crie os recursos STONITH `fence_node1` e `fence_node2` usando o agente `fence_virsh`, apontando para o IP do seu host Libvirt e o usuário `libvirt_user`.
4.  Habilite o STONITH no cluster (`pcs property set stonith-enabled=true`).
5.  Verifique o status do cluster e confirme que os dispositivos STONITH estão ativos.
6.  Simule uma falha no `node1` (ex: `sudo reboot -f` ou `sudo systemctl stop pacemaker corosync`).
7.  Observe o comportamento do cluster: o `node2` deve fazer fencing do `node1`, e os recursos devem migrar para `node2`.
8.  Após o teste, ligue o `node1` novamente e observe o cluster se recuperar.

## Referências

[1] Linbit. *Using the Libvirt Daemon For Fencing Pacemaker Clusters*. Disponível em: [https://kb.linbit.com/pacemaker-stack/stonith/libvirtd-fencing-for-pacemaker-clusters/](https://kb.linbit.com/pacemaker-stack/stonith/libvirtd-fencing-for-pacemaker-clusters/)

[2] ClusterLabs. *Pacemaker Explained*. Disponível em: [https://clusterlabs.org/projects/pacemaker/doc/2.1/Pacemaker_Explained/singlehtml/](https://clusterlabs.org/projects/pacemaker/doc/2.1/Pacemaker_Explained/singlehtml/)

