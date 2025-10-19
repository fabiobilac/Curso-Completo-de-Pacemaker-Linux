# Módulo 01: Introdução à Alta Disponibilidade e Conceitos Fundamentais

## 1.1 O que é Alta Disponibilidade (HA) e por que é importante?

Alta Disponibilidade (HA) refere-se à capacidade de um sistema ou serviço permanecer operacional e acessível por um longo período, minimizando o tempo de inatividade (downtime). Em ambientes de tecnologia da informação, a interrupção de serviços pode resultar em perdas financeiras significativas, danos à reputação e insatisfação do cliente. Por isso, a implementação de soluções de HA é crucial para garantir a continuidade dos negócios.

Um sistema de alta disponibilidade é projetado para eliminar pontos únicos de falha (Single Points of Failure - SPOF), utilizando redundância e mecanismos de failover automático. Quando um componente falha, outro assume sua função sem intervenção manual e com o mínimo de interrupção para os usuários. [1]

### Benefícios da Alta Disponibilidade:

*   **Continuidade de Negócios:** Garante que serviços críticos estejam sempre disponíveis, mesmo diante de falhas de hardware, software ou rede.
*   **Redução de Perdas:** Minimiza as perdas financeiras associadas ao tempo de inatividade, como perda de vendas, produtividade e dados.
*   **Melhora da Reputação:** Mantém a confiança dos clientes e parceiros ao oferecer serviços confiáveis e ininterruptos.
*   **Conformidade:** Ajuda a atender a requisitos regulatórios e acordos de nível de serviço (SLAs).

## 1.2 Componentes de um Cluster HA

Um cluster de alta disponibilidade é um conjunto de servidores (nós) que trabalham em conjunto para fornecer serviços de forma contínua. Os principais componentes de um cluster HA baseado em Pacemaker incluem:

### Nós (Nodes)

Os nós são os servidores físicos ou virtuais que compõem o cluster. Cada nó executa o sistema operacional e os softwares necessários para o cluster. Em um cluster HA típico, há pelo menos dois nós para garantir redundância. Se um nó falhar, os serviços podem ser migrados para outro nó saudável.

### Recursos (Resources)

Recursos são os serviços ou aplicações que o cluster gerencia e torna altamente disponíveis. Exemplos comuns de recursos incluem:

*   **Endereços IP Virtuais:** Um IP que pode ser movido entre os nós, garantindo que o serviço seja sempre acessível pelo mesmo endereço.
*   **Serviços de Aplicação:** Servidores web (Apache, Nginx), bancos de dados (PostgreSQL, MySQL), servidores de arquivos (NFS), etc.
*   **Sistemas de Arquivos:** Montagens de sistemas de arquivos (ex: NFS, GFS2) que precisam ser acessíveis pelos serviços.
*   **Máquinas Virtuais:** O próprio Pacemaker pode gerenciar o ciclo de vida de máquinas virtuais, garantindo que elas sejam reiniciadas ou migradas em caso de falha do host.

### Pacemaker: O Gerenciador de Recursos do Cluster

Pacemaker é o coração do cluster de alta disponibilidade. Ele é responsável por:

*   **Monitorar o estado dos recursos e dos nós:** Verifica constantemente se os serviços estão em execução e se os nós estão saudáveis.
*   **Iniciar e parar recursos:** Ativa ou desativa serviços nos nós do cluster conforme necessário.
*   **Gerenciar a movimentação de recursos:** Em caso de falha de um nó, o Pacemaker move os recursos afetados para um nó saudável.
*   **Aplicar políticas:** Garante que os recursos sejam executados nos nós corretos e na ordem desejada, utilizando regras e restrições.

> "Pacemaker is a high-availability cluster resource manager – software that runs on a set of hosts (a cluster of nodes) in order to preserve integrity and ..." [2]

### Corosync: A Camada de Comunicação e Consenso

Corosync é um motor de comunicação de cluster que fornece as funcionalidades de mensagens e gerenciamento de adesão ao cluster. Ele garante que todos os nós tenham uma visão consistente do estado do cluster, evitando o problema de *split-brain*. O *split-brain* ocorre quando os nós de um cluster perdem a comunicação entre si e cada um acredita ser o único nó ativo, o que pode levar à corrupção de dados. O Corosync utiliza um mecanismo de quórum para evitar essa situação.

### STONITH (Shoot The Other Node In The Head) / Fencing

STONITH, ou Fencing, é um mecanismo crítico para a integridade de um cluster HA. Seu objetivo é garantir que um nó que falhou ou está se comportando de forma errática seja completamente isolado do cluster e de seus recursos compartilhados, impedindo-o de causar danos (como corrupção de dados). Isso é feito, por exemplo, desligando fisicamente o nó problemático ou revogando seu acesso ao armazenamento compartilhado.

> "STONITH (Shoot The Other Node In The Head) is a critical component of any high-availability cluster. It ensures that a failed node is truly dead and not just unresponsive, preventing data corruption." [3]

## 1.3 Terminologia Básica

Para entender o Pacemaker, é importante familiarizar-se com alguns termos:

*   **Quórum:** O número mínimo de nós que devem estar ativos e se comunicando para que o cluster funcione corretamente. Se o número de nós ativos cair abaixo do quórum, o cluster pode parar de operar para evitar *split-brain*.
*   **Split-Brain:** Uma situação indesejável em que os nós de um cluster perdem a comunicação e cada parte acredita ser a única ativa, tentando assumir o controle dos mesmos recursos, o que pode levar à corrupção de dados.
*   **Recurso:** Qualquer serviço ou aplicação que o Pacemaker gerencia. (Já detalhado acima)
*   **Grupo de Recursos:** Uma coleção de recursos que devem ser iniciados, parados e movidos juntos, em uma ordem específica.
*   **Restrições:** Regras que definem como os recursos devem se comportar, incluindo onde eles podem ou não ser executados (restrições de localização), a ordem em que devem ser iniciados (restrições de ordem) e quais recursos devem ser executados no mesmo nó (restrições de colocation).

## Referências

[1] Red Hat. (2021). *How to set up a Pacemaker cluster for high availability Linux*. Disponível em: [https://www.redhat.com/en/blog/rhel-pacemaker-cluster](https://www.redhat.com/en/blog/rhel-pacemaker-cluster)

[2] ClusterLabs. *Pacemaker Explained*. Disponível em: [https://clusterlabs.org/projects/pacemaker/doc/2.1/Pacemaker_Explained/singlehtml/](https://clusterlabs.org/projects/pacemaker/doc/2.1/Pacemaker_Explained/singlehtml/)

[3] Linbit. *Using the Libvirt Daemon For Fencing Pacemaker Clusters*. Disponível em: [https://kb.linbit.com/pacemaker-stack/stonith/libvirtd-fencing-for-pacemaker-clusters/](https://kb.linbit.com/pacemaker-stack/stonith/libvirtd-fencing-for-pacemaker-clusters/)

