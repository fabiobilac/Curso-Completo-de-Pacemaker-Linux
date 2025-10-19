# Módulo 06: Monitoramento e Troubleshooting

Um cluster de alta disponibilidade é um sistema complexo, e o monitoramento contínuo é essencial para garantir sua saúde e identificar problemas rapidamente. Este módulo abordará as ferramentas e técnicas para monitorar o status do Pacemaker e como realizar o troubleshooting de problemas comuns.

## 6.1 Comandos `pcs` para Monitoramento

A ferramenta `pcs` é a interface principal para interagir com o cluster Pacemaker. Ela oferece uma série de comandos para verificar o status, a configuração e o comportamento dos recursos e nós.

### 6.1.1 `pcs status`

Este é o comando mais fundamental para obter uma visão geral do cluster. Ele exibe informações sobre:

*   **Nome do Cluster:** Identificação do seu cluster.
*   **Current DC (Designated Coordinator):** O nó que atualmente atua como coordenador do cluster. Este nó é responsável por tomar decisões sobre onde os recursos devem ser executados.
*   **Nós Online:** Lista dos nós que estão ativos e participando do cluster.
*   **Recursos Configurados:** Lista de todos os recursos, seus estados (iniciado/parado) e em qual nó estão sendo executados.
*   **Status dos Daemons:** Corosync, Pacemaker e Pcsd.

```bash
sudo pcs status
```

### 6.1.2 `pcs status resources`

Exibe uma visão mais detalhada dos recursos, incluindo seus ID, o agente de recurso utilizado, o nó onde estão ativos e o status de monitoramento.

```bash
sudo pcs status resources
```

### 6.1.3 `pcs status nodes`

Mostra o status de cada nó do cluster, indicando se está online, em standby, ou com algum problema.

```bash
sudo pcs status nodes
```

### 6.1.4 `pcs property`

Permite visualizar e modificar as propriedades globais do cluster. Por exemplo, para ver todas as propriedades:

```bash
sudo pcs property list
```

Para verificar uma propriedade específica, como `stonith-enabled`:

```bash
sudo pcs property show stonith-enabled
```

### 6.1.5 `pcs resource show <nome_recurso>`

Exibe a configuração detalhada de um recurso específico, incluindo todos os seus parâmetros e meta-parâmetros.

```bash
sudo pcs resource show virtual_ip
```

### 6.1.6 `pcs constraint`

Lista todas as restrições configuradas no cluster (localização, ordem, colocation).

```bash
sudo pcs constraint
```

## 6.2 Análise de Logs

Os logs são uma fonte inestimável de informações para diagnosticar problemas no cluster. Os principais logs a serem consultados são:

*   **Journald:** Em sistemas modernos com `systemd`, o `journalctl` é a ferramenta principal para acessar os logs do sistema, incluindo os do Pacemaker e Corosync.

    *   **Logs do Pacemaker:**
        ```bash
        sudo journalctl -u pacemaker -f
        ```
        (O `-f` mostra o log em tempo real, útil durante testes de failover).

    *   **Logs do Corosync:**
        ```bash
        sudo journalctl -u corosync -f
        ```

*   **`/var/log/pacemaker.log` (ou similar):** Em algumas configurações ou versões mais antigas, o Pacemaker pode registrar seus eventos neste arquivo. Verifique a configuração do seu sistema.

Ao analisar os logs, procure por mensagens de erro, avisos, eventos de failover, decisões do Pacemaker e qualquer indicação de problemas de comunicação entre os nós.

## 6.3 Cenários Comuns de Troubleshooting

### 6.3.1 Cluster Não Forma Quórum

**Sintoma:** `pcs status` mostra que o cluster não tem quórum ou que os nós estão em estado `pending` ou `offline`.

**Possíveis Causas:**

*   **Problemas de Rede:** Firewall bloqueando portas do Corosync (UDP 5405), IPs incorretos, problemas de roteamento.
*   **Configuração do Corosync:** Arquivo `corosync.conf` incorreto ou inconsistente entre os nós.
*   **Nós Insuficientes:** Número de nós ativos abaixo do mínimo necessário para o quórum (geralmente `(N/2) + 1`, onde N é o número total de nós).
*   **Serviços Parados:** `corosync` ou `pacemaker` não estão em execução em um ou mais nós.

**Solução:**

1.  Verifique a conectividade de rede entre todos os nós (ping, ssh).
2.  Confira as regras do firewall para as portas do Corosync.
3.  Inspecione os logs do Corosync (`journalctl -u corosync`) em todos os nós para mensagens de erro.
4.  Certifique-se de que `corosync` e `pacemaker` estão iniciados e habilitados em todos os nós (`sudo systemctl status corosync pacemaker`).

### 6.3.2 Recurso Não Inicia ou Não Migra

**Sintoma:** Um recurso permanece `stopped` ou não se move para outro nó após uma falha.

**Possíveis Causas:**

*   **Agente de Recurso com Problemas:** O script do agente de recurso (OCF, systemd) contém erros ou não consegue iniciar/parar o serviço subjacente.
*   **Restrições Conflitantes:** Restrições de localização, ordem ou colocation impedindo o recurso de iniciar no nó desejado.
*   **Dependências Faltantes:** O recurso depende de algo que não está disponível no nó (ex: sistema de arquivos não montado).
*   **STONITH Desabilitado ou Falho:** Se o STONITH estiver desabilitado ou não funcionando, o Pacemaker pode hesitar em mover recursos para evitar split-brain.

**Solução:**

1.  Verifique os logs do Pacemaker (`journalctl -u pacemaker`) para ver as decisões tomadas sobre o recurso.
2.  Teste o agente de recurso manualmente no nó onde ele deveria estar (ex: `sudo /usr/lib/ocf/resource.d/heartbeat/IPaddr2 start`).
3.  Liste e revise todas as restrições (`sudo pcs constraint`) para identificar conflitos.
4.  Verifique as dependências do recurso (ex: se um recurso de sistema de arquivos precisa ser montado antes de um serviço web).
5.  Confirme se o STONITH está habilitado e funcionando corretamente (`sudo pcs property show stonith-enabled`, `sudo pcs stonith status`).

### 6.3.3 Split-Brain

**Sintoma:** Ambos os nós do cluster tentam assumir o controle dos mesmos recursos, ou o cluster se divide em duas partes independentes.

**Possíveis Causas:**

*   **STONITH Ausente ou Falho:** A causa mais comum. Sem STONITH, o Pacemaker não tem como garantir que um nó falho esteja realmente "morto" antes de permitir que outro nó assuma seus recursos.
*   **Problemas de Comunicação:** Perda de comunicação entre os nós que não é detectada corretamente pelo Corosync.

**Solução:**

1.  **NUNCA desabilite o STONITH em produção.** Em ambiente de laboratório, se você desabilitou, reabilite e configure-o corretamente.
2.  Verifique a configuração e o funcionamento do seu dispositivo STONITH.
3.  Garanta que a rede entre os nós seja robusta e que o Corosync esteja configurado para detectar falhas de comunicação rapidamente.

## Exercícios Práticos

1.  Com seu cluster em funcionamento e com o `web_service_group` (do Módulo 4) ativo, execute `sudo pcs status` e analise a saída.
2.  Execute `sudo pcs status resources` e `sudo pcs status nodes` para obter informações mais específicas.
3.  Use `sudo pcs resource show virtual_ip` para ver a configuração detalhada do seu IP virtual.
4.  Abra dois terminais, um para `node1` e outro para `node2`. Em cada um, execute `sudo journalctl -u pacemaker -f`.
5.  No nó onde o `web_service_group` está ativo, pare o serviço `httpd` (ou `apache2`) manualmente (`sudo systemctl stop httpd`). Observe os logs e o `pcs status` para ver como o Pacemaker reage e tenta reiniciar o recurso.
6.  No nó onde o `web_service_group` está ativo, simule uma falha de rede (ex: `sudo ifdown eth0` ou desative a interface de rede da VM). Observe o failover e as mensagens nos logs.
7.  Corrija a falha de rede e observe o cluster se recuperar. Se o STONITH estiver configurado, simule uma falha de energia em um nó (`sudo reboot -f`) e observe o fencing e o failover.

## Referências

[1] ClusterLabs. *Pacemaker Explained*. Disponível em: [https://clusterlabs.org/projects/pacemaker/doc/2.1/Pacemaker_Explained/singlehtml/](https://clusterlabs.org/projects/pacemaker/doc/2.1/Pacemaker_Explained/singlehtml/)

[2] Red Hat. (n.d.). *Configuring and managing high availability clusters*. Disponível em: [https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html-single/configuring_and_managing_high_availability_clusters/index](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html-single/configuring_and_managing_high_availability_clusters/index)

