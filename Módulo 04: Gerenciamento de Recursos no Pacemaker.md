# Módulo 04: Gerenciamento de Recursos no Pacemaker

Após configurar e iniciar o cluster Pacemaker/Corosync, o próximo passo crucial é definir e gerenciar os recursos que o cluster tornará altamente disponíveis. Este módulo abordará os diferentes tipos de recursos, como criá-los, organizá-los em grupos e aplicar restrições para controlar seu comportamento.

## 4.1 Tipos de Recursos no Pacemaker

O Pacemaker pode gerenciar uma vasta gama de recursos, que são essencialmente scripts ou agentes que sabem como iniciar, parar e monitorar um serviço ou aplicação. Existem três categorias principais de agentes de recursos:

*   **LSB (Linux Standard Base) / `systemd`:** Para serviços gerenciados pelo sistema operacional (ex: Apache, Nginx, PostgreSQL). O Pacemaker pode usar os scripts `init.d` ou as unidades `systemd` existentes.
*   **OCF (Open Cluster Framework):** Agentes de recursos mais sofisticados, escritos especificamente para clusters HA. Eles fornecem funcionalidades avançadas como monitoramento detalhado, parâmetros de configuração e ações de recuperação. Exemplos incluem `IPaddr2` (para endereços IP flutuantes), `Filesystem` (para montar sistemas de arquivos) e `LVM` (para gerenciar volumes lógicos).
*   **Systemd:** O Pacemaker tem suporte nativo para gerenciar unidades `systemd` diretamente, o que simplifica a configuração de muitos serviços modernos.

## 4.2 Criação e Configuração de Recursos

Recursos são criados e gerenciados usando o comando `pcs resource`. A sintaxe básica para adicionar um recurso é `pcs resource create <nome_recurso> <tipo_agente> <parâmetros> [meta parâmetros]`.

### Exemplo 1: Recurso de Endereço IP Virtual (`IPaddr2`)

Um endereço IP virtual é um dos recursos mais comuns em clusters HA, permitindo que os clientes acessem um serviço independentemente do nó em que ele esteja sendo executado.

```bash
sudo pcs resource create virtual_ip ocf:heartbeat:IPaddr2 ip=192.168.122.200 cidr_netmask=24 op monitor interval=10s
```

*   `virtual_ip`: Nome descritivo para o recurso IP.
*   `ocf:heartbeat:IPaddr2`: Indica que estamos usando um agente OCF do namespace `heartbeat` chamado `IPaddr2`.
*   `ip=192.168.122.200`: O endereço IP virtual a ser gerenciado.
*   `cidr_netmask=24`: A máscara de rede para o IP virtual.
*   `op monitor interval=10s`: Define uma operação de monitoramento que será executada a cada 10 segundos para verificar a saúde do recurso.

Verifique o status do cluster após a criação:

```bash
sudo pcs status
```

Você deverá ver o `virtual_ip` em execução em um dos nós.

### Exemplo 2: Recurso de Serviço `systemd` (Apache HTTP Server)

Vamos supor que você tenha o Apache instalado e configurado em ambos os nós. O Pacemaker pode gerenciar o serviço `httpd` (ou `apache2` no Debian/Ubuntu).

Primeiro, certifique-se de que o Apache esteja instalado e desabilitado para iniciar no boot em ambos os nós, pois o Pacemaker será responsável por iniciá-lo:

```bash
# Em ambos os nós:
sudo dnf install httpd -y # RHEL/CentOS
# ou
sudo apt install apache2 -y # Debian/Ubuntu
sudo systemctl disable httpd # ou apache2
sudo systemctl stop httpd # ou apache2
```

Agora, crie o recurso no Pacemaker:

```bash
sudo pcs resource create web_server systemd:httpd op monitor interval=20s
# ou para Ubuntu/Debian:
sudo pcs resource create web_server systemd:apache2 op monitor interval=20s
```

*   `web_server`: Nome descritivo para o recurso Apache.
*   `systemd:httpd`: Indica que estamos gerenciando a unidade `systemd` `httpd`.
*   `op monitor interval=20s`: Monitora o serviço a cada 20 segundos.

Verifique o status novamente:

```bash
sudo pcs status
```

## 4.3 Grupos de Recursos

Em muitos cenários, vários recursos dependem uns dos outros e precisam ser iniciados e parados em uma ordem específica, e sempre no mesmo nó. Para isso, utilizamos **grupos de recursos**. Um grupo de recursos garante que todos os recursos dentro dele sejam executados no mesmo nó e que a ordem de início/parada seja respeitada.

Vamos criar um grupo com o IP virtual e o servidor web para que eles sempre funcionem juntos.

Primeiro, remova os recursos criados individualmente:

```bash
sudo pcs resource delete virtual_ip
sudo pcs resource delete web_server
```

Agora, crie o grupo de recursos:

```bash
sudo pcs resource group add web_service_group virtual_ip web_server
```

*   `web_service_group`: Nome do grupo de recursos.
*   `virtual_ip`: O recurso IP virtual (será recriado como parte do grupo).
*   `web_server`: O recurso do servidor web (será recriado como parte do grupo).

Para adicionar os recursos com seus parâmetros específicos, você pode usar a seguinte sintaxe, que é mais explícita e permite definir os parâmetros diretamente na criação do grupo:

```bash
sudo pcs resource create virtual_ip ocf:heartbeat:IPaddr2 ip=192.168.122.200 cidr_netmask=24 \
  op monitor interval=10s --group web_service_group
sudo pcs resource create web_server systemd:httpd \
  op monitor interval=20s --group web_service_group
# (ou systemd:apache2 para Debian/Ubuntu)
```

Verifique o status do cluster. Você verá o grupo `web_service_group` com seus recursos aninhados, todos em execução no mesmo nó.

```bash
sudo pcs status
```

## 4.4 Restrições de Recursos

As restrições permitem controlar onde e como os recursos são executados no cluster. Existem três tipos principais:

### 4.4.1 Restrições de Localização (Location Constraints)

Definem onde um recurso pode ou não ser executado, ou preferencialmente onde ele deve ser executado.

*   **Preferência:** Fazer com que um recurso prefira um nó específico (mas possa ser movido se o nó falhar).
    ```bash
    sudo pcs constraint location web_service_group prefers node1=50
    ```
    (O grupo `web_service_group` terá uma preferência de 50 para ser executado em `node1`.)

*   **Obrigatoriedade:** Forçar um recurso a ser executado em um nó específico (não recomendado para HA, pois cria um SPOF).
    ```bash
    sudo pcs constraint location web_service_group mandates node1
    ```

*   **Proibição:** Impedir que um recurso seja executado em um nó específico.
    ```bash
    sudo pcs constraint location web_service_group avoids node2
    ```

### 4.4.2 Restrições de Ordem (Order Constraints)

Definem a sequência em que os recursos devem ser iniciados ou parados. Se você usa grupos de recursos, a ordem já é implícita dentro do grupo, mas restrições de ordem são úteis para recursos fora de grupos ou para dependências mais complexas.

```bash
sudo pcs constraint order start resource_A then resource_B
```

Isso garante que `resource_A` seja iniciado antes de `resource_B`.

### 4.4.3 Restrições de Colocação (Colocation Constraints)

Definem se dois ou mais recursos devem ser executados no mesmo nó ou em nós diferentes.

*   **Colocação Positiva:** Recursos devem ser executados no mesmo nó.
    ```bash
    sudo pcs constraint colocation add resource_X with resource_Y INFINITY
    ```
    (O `INFINITY` garante que eles sempre estarão juntos. Um valor positivo menor indica preferência.)

*   **Colocação Negativa:** Recursos devem ser executados em nós diferentes.
    ```bash
    sudo pcs constraint colocation add resource_X with resource_Y -INFINITY
    ```
    (O `-INFINITY` garante que eles nunca estarão juntos. Um valor negativo menor indica preferência por nós diferentes.)

## Exercícios Práticos

1.  Crie um recurso de endereço IP virtual (`ocf:heartbeat:IPaddr2`) com o IP `192.168.122.201` e monitore-o a cada 5 segundos.
2.  Instale e configure o Nginx em ambos os nós do seu ambiente de laboratório. Desabilite-o para iniciar no boot.
3.  Crie um recurso `systemd:nginx` e adicione-o ao cluster, monitorando-o a cada 15 segundos.
4.  Crie um grupo de recursos chamado `web_app_group` que inclua o IP virtual (`192.168.122.201`) e o serviço Nginx.
5.  Configure uma restrição de localização para que o `web_app_group` prefira ser executado no `node2` com uma pontuação de 100.
6.  Teste o failover do grupo de recursos desligando o `node2` e observando se o grupo se move para o `node1`.
7.  Remova todas as restrições e recursos criados neste exercício.

## Referências

[1] ClusterLabs. *Pacemaker Explained*. Disponível em: [https://clusterlabs.org/projects/pacemaker/doc/2.1/Pacemaker_Explained/singlehtml/](https://clusterlabs.org/projects/pacemaker/doc/2.1/Pacemaker_Explained/singlehtml/)

[2] Red Hat. (n.d.). *Configuring and managing high availability clusters*. Disponível em: [https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html-single/configuring_and_managing_high_availability_clusters/index](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html-single/configuring_and_managing_high_availability_clusters/index)

