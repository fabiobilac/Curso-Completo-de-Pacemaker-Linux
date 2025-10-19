# Exercícios Práticos Detalhados para Pacemaker Linux

Este documento contém **3 exercícios práticos detalhados e avançados** para os módulos mais críticos do curso. Cada exercício inclui cenários realistas, resultados esperados e desafios de aprofundamento.

---

## Exercício 01: Configuração Completa de um Cluster HA com Failover Automático

**Módulo:** Módulo 03 - Instalação e Configuração Básica do Pacemaker  
**Nível:** Intermediário  
**Tempo Estimado:** 90-120 minutos  
**Objetivo:** Criar um cluster Pacemaker funcional com dois nós e testar failover automático de um serviço.

### Cenário

Você é um administrador de sistemas responsável por implementar alta disponibilidade para um serviço web crítico em uma empresa. O serviço deve estar disponível em um IP virtual (VIP) que se move automaticamente entre dois servidores em caso de falha.

### Pré-requisitos

- Duas máquinas virtuais (node1 e node2) com Ubuntu 22.04 LTS ou CentOS Stream 9
- Conectividade de rede entre as VMs (bridge Libvirt)
- Acesso root ou sudo em ambas as VMs
- Firewall desabilitado (apenas para laboratório)

### Passos Detalhados

#### Passo 1: Preparação do Ambiente (15 minutos)

**Em node1 e node2:**

```bash
# 1. Atualizar sistema
sudo apt update && sudo apt upgrade -y  # Para Ubuntu
# ou
sudo dnf update -y  # Para CentOS/RHEL

# 2. Instalar dependências
sudo apt install -y pacemaker corosync pcs resource-agents  # Ubuntu
# ou
sudo dnf install -y pacemaker corosync pcs resource-agents  # CentOS

# 3. Iniciar e habilitar serviços
sudo systemctl start pcsd
sudo systemctl enable pcsd

# 4. Definir senha para usuário hacluster (necessária para autenticação)
sudo passwd hacluster
# Digite uma senha segura (ex: Cluster@123)

# 5. Verificar status
sudo systemctl status pcsd
```

**Resultado Esperado:**
```
● pcsd.service - PCS GUI and remote configuration interface
     Loaded: loaded (/usr/lib/systemd/system/pcsd.service; enabled; vendor preset: enabled)
     Active: active (running) since ...
```

#### Passo 2: Criar o Cluster (20 minutos)

**Em node1 (coordenador do cluster):**

```bash
# 1. Autenticar os nós no cluster
sudo pcs host auth node1 node2 -u hacluster -p Cluster@123

# 2. Criar o cluster
sudo pcs cluster setup mycluster node1 node2 --start --enable

# 3. Aguardar a sincronização (30-60 segundos)
sleep 60

# 4. Verificar status do cluster
sudo pcs cluster status
```

**Resultado Esperado:**
```
Cluster Status:
  Stack: corosync
  Current DC: node1 (version 2.x.x) - partition with quorum
  Last updated: ...
  2 nodes configured
  0 resource instances configured

Node node1: online
Node node2: online
```

#### Passo 3: Configurar um Recurso de IP Virtual (VIP) (20 minutos)

```bash
# 1. Criar um recurso de IP virtual (VIP)
sudo pcs resource create ClusterIP ocf:heartbeat:IPaddr2 \
  ip=192.168.122.150 \
  cidr_netmask=24 \
  op monitor interval=30s

# 2. Verificar o recurso foi criado
sudo pcs resource status

# 3. Verificar em qual nó o IP está ativo
ip addr show  # Execute em ambos os nós
```

**Resultado Esperado:**

Em node1 (ou node2):
```
inet 192.168.122.150/24 brd 192.168.122.255 scope global secondary eth0
```

O IP virtual deve estar ativo em um dos nós.

#### Passo 4: Adicionar um Serviço Gerenciado (Apache/Nginx) (25 minutos)

```bash
# 1. Instalar Apache em ambos os nós
sudo apt install -y apache2  # Ubuntu
# ou
sudo dnf install -y httpd  # CentOS

# 2. Criar uma página HTML simples para identificar o nó
# Em node1:
echo "<h1>Serviço rodando em node1</h1>" | sudo tee /var/www/html/index.html

# Em node2:
echo "<h1>Serviço rodando em node2</h1>" | sudo tee /var/www/html/index.html

# 3. Parar o Apache em ambos os nós (Pacemaker gerenciará)
sudo systemctl stop apache2  # Ubuntu
sudo systemctl disable apache2
# ou
sudo systemctl stop httpd  # CentOS
sudo systemctl disable httpd

# 4. Criar recurso Apache no Pacemaker
sudo pcs resource create WebServer ocf:heartbeat:apache \
  configfile=/etc/apache2/apache2.conf \
  op monitor interval=30s

# Ou para CentOS/RHEL:
# sudo pcs resource create WebServer ocf:heartbeat:apache \
#   configfile=/etc/httpd/conf/httpd.conf \
#   op monitor interval=30s

# 5. Agrupar os recursos (VIP e Web Server)
sudo pcs resource group add WebGroup ClusterIP WebServer

# 6. Verificar status
sudo pcs resource status
```

**Resultado Esperado:**
```
Resource Group: WebGroup
  ClusterIP (ocf::heartbeat:IPaddr2): Started node1
  WebServer (ocf::heartbeat:apache): Started node1
```

#### Passo 5: Testar Failover Automático (20 minutos)

**Teste 1: Parar o serviço Apache**

```bash
# Em node1 (onde o serviço está rodando):
sudo systemctl stop apache2

# Aguardar 30-60 segundos

# Verificar status do cluster
sudo pcs resource status

# Acessar o VIP de outro computador ou VM:
curl http://192.168.122.150
```

**Resultado Esperado:**
- O Apache deve ser reiniciado automaticamente em node1 OU migrado para node2
- A página HTML deve estar acessível via VIP
- O status do Pacemaker deve mostrar o recurso como "Started" em um dos nós

**Teste 2: Simular Falha de Nó**

```bash
# Em node1, simular uma falha desligando o nó
sudo shutdown -h now

# De node2, verificar status
sudo pcs cluster status

# Acessar o VIP:
curl http://192.168.122.150
```

**Resultado Esperado:**
- node1 deve aparecer como "offline"
- Os recursos (ClusterIP e WebServer) devem estar rodando em node2
- A página HTML deve estar acessível via VIP (agora servida por node2)

**Teste 3: Recuperação do Nó**

```bash
# Ligar node1 novamente
# Verificar status do cluster
sudo pcs cluster status

# Os recursos devem retornar ao seu estado original (ou conforme as políticas definidas)
```

### Desafios de Aprofundamento

1. **Adicione Restrições de Localização:** Configure o Pacemaker para preferir que o VIP rode em node1, mas permita failover para node2 se necessário.

2. **Implemente Monitoramento Customizado:** Crie um script de monitoramento que verifica se o Apache está respondendo em um endpoint específico (ex: `/health`).

3. **Configure Múltiplos Grupos de Recursos:** Adicione um segundo grupo de recursos (ex: banco de dados) com diferentes preferências de localização.

### Checklist de Conclusão

- [ ] Cluster criado e ambos os nós online
- [ ] VIP criado e ativo em um nó
- [ ] Apache instalado e gerenciado pelo Pacemaker
- [ ] Failover automático testado e funcionando
- [ ] Recuperação de nó testada
- [ ] Todos os testes passaram com sucesso

---

## Exercício 02: Implementação e Teste de STONITH/Fencing

**Módulo:** Módulo 05 - STONITH/Fencing  
**Nível:** Avançado  
**Tempo Estimado:** 90-120 minutos  
**Objetivo:** Configurar STONITH com fence_virsh e testar proteção contra split-brain.

### Cenário

Você descobriu que seu cluster Pacemaker está vulnerável a split-brain (partição de rede) porque não possui STONITH configurado. Sua tarefa é implementar fencing usando fence_virsh (apropriado para ambientes virtualizados com Libvirt) e testar cenários de falha.

### Pré-requisitos

- Cluster Pacemaker funcional (do Exercício 01)
- Acesso ao host Libvirt (máquina que executa as VMs)
- Chave SSH configurada entre o host Libvirt e os nós do cluster
- Permissões para executar comandos virsh

### Passos Detalhados

#### Passo 1: Preparar Acesso SSH do Host Libvirt (15 minutos)

**No host Libvirt:**

```bash
# 1. Criar chave SSH para o usuário que executa Libvirt
ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa

# 2. Copiar a chave pública para os nós do cluster
ssh-copy-id -i ~/.ssh/id_rsa.pub hacluster@node1
ssh-copy-id -i ~/.ssh/id_rsa.pub hacluster@node2

# 3. Testar acesso sem senha
ssh -i ~/.ssh/id_rsa hacluster@node1 "echo 'Acesso OK'"
```

**Resultado Esperado:**
```
Acesso OK
```

#### Passo 2: Instalar e Configurar fence_virsh (20 minutos)

**Em node1 e node2:**

```bash
# 1. Instalar fence-agents (contém fence_virsh)
sudo apt install -y fence-agents  # Ubuntu
# ou
sudo dnf install -y fence-agents-virsh  # CentOS

# 2. Testar fence_virsh manualmente (sem executar, apenas verificar)
sudo fence_virsh -a <host_libvirt_ip> -l hacluster -o list
```

**Resultado Esperado:**
```
node1
node2
```

#### Passo 3: Criar Dispositivo STONITH no Pacemaker (25 minutos)

**Em node1:**

```bash
# 1. Criar dispositivo STONITH para node1
sudo pcs stonith create fence-node1 fence_virsh \
  ipaddr=<host_libvirt_ip> \
  login=hacluster \
  identity_file=/home/hacluster/.ssh/id_rsa \
  port=node1 \
  op monitor interval=60s

# 2. Criar dispositivo STONITH para node2
sudo pcs stonith create fence-node2 fence_virsh \
  ipaddr=<host_libvirt_ip> \
  login=hacluster \
  identity_file=/home/hacluster/.ssh/id_rsa \
  port=node2 \
  op monitor interval=60s

# 3. Verificar status
sudo pcs stonith status
sudo pcs stonith config
```

**Resultado Esperado:**
```
fence-node1 (stonith:fence_virsh): Started node1
fence-node2 (stonith:fence_virsh): Started node1
```

#### Passo 4: Habilitar STONITH no Cluster (10 minutos)

```bash
# 1. Habilitar STONITH
sudo pcs property set stonith-enabled=true

# 2. Verificar configuração
sudo pcs property list | grep stonith
```

**Resultado Esperado:**
```
stonith-enabled: true
```

#### Passo 5: Testar STONITH (40 minutos)

**Teste 1: Teste Manual de Fencing**

```bash
# Em node1, testar fencing de node2 (sem realmente desligá-lo)
sudo fence_virsh -a <host_libvirt_ip> -l hacluster \
  -o reboot -n node2 --verbose

# Verificar que node2 foi "fenceado" (desligado)
sudo pcs cluster status
```

**Resultado Esperado:**
- node2 deve aparecer como "offline"
- Os recursos devem ser movidos para node1

**Teste 2: Simular Partição de Rede (Split-Brain)**

```bash
# Em node1, simular uma partição de rede desligando a interface de rede
sudo ip link set eth0 down

# Aguardar 30-60 segundos

# Em node2, verificar status do cluster
sudo pcs cluster status

# Verificar se node1 foi "fenceado" automaticamente
sudo virsh list  # No host Libvirt
```

**Resultado Esperado:**
- node1 deve ser automaticamente desligado pelo STONITH
- Os recursos devem estar rodando em node2
- Não deve haver conflito de acesso aos dados compartilhados

**Teste 3: Recuperação Após Fencing**

```bash
# Ligar node1 novamente
# No host Libvirt:
sudo virsh start node1

# Em node1, verificar status
sudo pcs cluster status

# Os recursos devem ser rebalanceados conforme as políticas
```

### Desafios de Aprofundamento

1. **Implemente Fencing Redundante:** Configure múltiplos dispositivos STONITH para cada nó (ex: fence_virsh + fence_ipmi).

2. **Teste Fencing com Timeout:** Configure timeouts de fencing e teste o comportamento quando o fencing falha.

3. **Monitore Eventos de Fencing:** Configure logging para rastrear todos os eventos de fencing e analise os logs.

### Checklist de Conclusão

- [ ] fence_virsh instalado e testado
- [ ] Dispositivos STONITH criados para ambos os nós
- [ ] STONITH habilitado no cluster
- [ ] Teste manual de fencing bem-sucedido
- [ ] Partição de rede simulada e recuperada corretamente
- [ ] Nenhum split-brain ou conflito de dados

---

## Exercício 03: Troubleshooting Avançado de Cluster

**Módulo:** Módulo 06 - Monitoramento e Troubleshooting  
**Nível:** Avançado  
**Tempo Estimado:** 120-150 minutos  
**Objetivo:** Diagnosticar e resolver problemas complexos em um cluster Pacemaker.

### Cenário

Você recebe um relatório de que o cluster Pacemaker está tendo comportamento anômalo: recursos não estão migrando corretamente, alguns nós estão sendo fenceados desnecessariamente, e há suspeita de problemas de quórum. Sua tarefa é diagnosticar e resolver esses problemas.

### Pré-requisitos

- Cluster Pacemaker com STONITH configurado
- Conhecimento básico de logs do Linux
- Ferramentas de análise (grep, awk, tail)
- Acesso root ou sudo

### Passos Detalhados

#### Passo 1: Coleta de Informações de Diagnóstico (20 minutos)

```bash
# 1. Verificar status geral do cluster
sudo pcs cluster status
sudo pcs status

# 2. Verificar status dos nós
sudo pcs node status

# 3. Verificar status dos recursos
sudo pcs resource status

# 4. Verificar configuração do cluster
sudo pcs config

# 5. Verificar status do quórum
sudo corosync-quorumtool -s

# 6. Coletar logs recentes
sudo journalctl -u pacemaker -n 100 --no-pager
sudo journalctl -u corosync -n 100 --no-pager

# 7. Verificar arquivo de configuração CIB
sudo pcs cluster cib

# 8. Exportar CIB para análise
sudo pcs cluster cib > cluster_backup.xml
```

**Resultado Esperado:**
- Informações consolidadas sobre o estado do cluster
- Logs recentes para análise
- Backup da configuração

#### Passo 2: Analisar Problemas Comuns (30 minutos)

**Problema 1: Nó Não Sincroniza com o Cluster**

```bash
# Sintomas:
# - Nó aparece como "offline" ou "standby"
# - Recursos não são migrados para este nó

# Diagnóstico:
sudo pcs cluster status  # Verificar status do nó
sudo systemctl status corosync  # Verificar Corosync
sudo systemctl status pacemaker  # Verificar Pacemaker

# Solução:
sudo systemctl restart corosync
sudo systemctl restart pacemaker
sudo pcs cluster start  # Se necessário

# Verificação:
sudo pcs cluster status
```

**Problema 2: Recursos Não Migram Corretamente**

```bash
# Sintomas:
# - Recurso fica em estado "failed"
# - Recurso não responde a comandos de migração

# Diagnóstico:
sudo pcs resource status
sudo pcs resource show <resource_name>
sudo journalctl -u pacemaker -f  # Monitorar logs em tempo real

# Solução:
# Limpar estado do recurso
sudo pcs resource cleanup <resource_name>

# Ou forçar migração
sudo pcs resource move <resource_name> <node_name>

# Verificação:
sudo pcs resource status
```

**Problema 3: Quórum Perdido (Cluster Parado)**

```bash
# Sintomas:
# - Cluster não responde a comandos
# - Mensagens de "no quorum" nos logs

# Diagnóstico:
sudo corosync-quorumtool -s
sudo journalctl -u corosync | grep -i quorum

# Solução (para cluster de 2 nós):
# Editar configuração de quórum
sudo pcs property set no-quorum-policy=ignore  # Apenas para laboratório!

# Ou forçar quórum
sudo corosync-quorumtool -e 1

# Verificação:
sudo corosync-quorumtool -s
```

**Problema 4: Split-Brain (Sem STONITH Configurado)**

```bash
# Sintomas:
# - Múltiplos nós tentam gerenciar os mesmos recursos
# - Conflitos de acesso aos dados

# Diagnóstico:
sudo pcs cluster status
sudo journalctl -u pacemaker | grep -i "split\|partition"

# Solução:
# Configurar STONITH (veja Exercício 02)
sudo pcs stonith create ...

# Ou desabilitar recursos em um nó
sudo pcs node standby <node_name>

# Verificação:
sudo pcs cluster status
```

#### Passo 3: Análise Profunda de Logs (30 minutos)

```bash
# 1. Filtrar logs de Pacemaker por severidade
sudo journalctl -u pacemaker -p err -n 50

# 2. Procurar por padrões de erro específicos
sudo journalctl -u pacemaker | grep -E "ERROR|WARN|CRIT"

# 3. Rastrear eventos de migração de recursos
sudo journalctl -u pacemaker | grep -i "migrate\|move"

# 4. Analisar eventos de fencing
sudo journalctl -u pacemaker | grep -i "fence\|stonith"

# 5. Exportar logs para arquivo
sudo journalctl -u pacemaker --since "1 hour ago" > pacemaker.log
sudo journalctl -u corosync --since "1 hour ago" > corosync.log

# 6. Usar ferramentas de análise
sudo pcs cluster verify -V  # Verificação detalhada
```

#### Passo 4: Testes de Recuperação (40 minutos)

**Teste 1: Recuperação de Falha de Nó**

```bash
# 1. Simular falha de nó
sudo shutdown -h now  # Em um nó

# 2. Monitorar recuperação
sudo pcs cluster status  # Em outro nó
sudo journalctl -u pacemaker -f

# 3. Verificar se recursos foram migrados
sudo pcs resource status

# 4. Ligar nó novamente
# (Iniciar VM no host Libvirt)

# 5. Verificar rebalanceamento
sudo pcs cluster status
```

**Teste 2: Teste de Failover Controlado**

```bash
# 1. Migrar recurso para outro nó
sudo pcs resource move <resource_name> <node_name>

# 2. Monitorar migração
sudo journalctl -u pacemaker -f

# 3. Verificar status
sudo pcs resource status

# 4. Remover restrição de migração
sudo pcs resource clear <resource_name>
```

**Teste 3: Teste de Carga**

```bash
# 1. Criar múltiplos recursos
for i in {1..5}; do
  sudo pcs resource create Resource$i ocf:heartbeat:IPaddr2 \
    ip=192.168.122.15$i cidr_netmask=24
done

# 2. Monitorar balanceamento
sudo pcs resource status

# 3. Simular falha e observar rebalanceamento
sudo shutdown -h now  # Em um nó

# 4. Verificar se todos os recursos foram migrados
sudo pcs resource status
```

### Desafios de Aprofundamento

1. **Implemente Monitoramento Contínuo:** Configure alertas para eventos críticos do cluster.

2. **Crie Scripts de Automação:** Desenvolva scripts que detectem e resolvam problemas comuns automaticamente.

3. **Teste Cenários Complexos:** Simule múltiplas falhas simultâneas e observe o comportamento do cluster.

### Checklist de Conclusão

- [ ] Informações de diagnóstico coletadas e analisadas
- [ ] Problemas comuns identificados e resolvidos
- [ ] Logs analisados com sucesso
- [ ] Testes de recuperação bem-sucedidos
- [ ] Cluster estável e respondendo corretamente
- [ ] Documentação de problemas e soluções criada

---

## Resumo e Próximos Passos

Estes três exercícios cobrem os aspectos mais críticos de um cluster Pacemaker:

1. **Exercício 01** - Fundações: Configuração básica e failover automático
2. **Exercício 02** - Segurança: Proteção contra split-brain com STONITH
3. **Exercício 03** - Operação: Diagnóstico e resolução de problemas

### Recomendações para Aprofundamento

- Repita os exercícios várias vezes até dominar completamente
- Experimente variações (ex: 3 nós em vez de 2)
- Integre serviços reais (bancos de dados, aplicações web)
- Documente seus aprendizados e crie runbooks para operação

### Recursos Adicionais

- Documentação oficial: https://clusterlabs.org/pacemaker/doc/
- Red Hat HA Guide: https://access.redhat.com/documentation/
- Comunidades: ClusterLabs mailing lists, Red Hat forums



