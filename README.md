# Curso Completo de Pacemaker Linux

Bem-vindo ao curso completo de Pacemaker Linux, projetado para iniciantes que desejam dominar os fundamentos e a aplicação prática de clusters de Alta Disponibilidade (HA) no ambiente Linux. Este curso abrange desde os conceitos básicos de HA até a configuração avançada de recursos e o troubleshooting, com um forte foco em exercícios práticos utilizando o Libvirt para simular um ambiente de cluster real.

---

## 🎯 Por que este Curso?

Se você trabalha com infraestrutura Linux e precisa garantir que seus serviços estejam **sempre disponíveis**, este curso é para você.

### Cenários Reais

- Seu banco de dados precisa estar disponível 24/7 sem downtime
- Você precisa implementar failover automático para serviços críticos
- Sua empresa exige SLA de 99.99% de disponibilidade
- Você quer dominar a ferramenta mais usada em ambientes Linux para HA

### O que você aprenderá

- ✅ Conceitos fundamentais de Alta Disponibilidade
- ✅ Como configurar um cluster Pacemaker do zero
- ✅ Proteger seu cluster contra falhas com STONITH/Fencing
- ✅ Diagnosticar e resolver problemas em produção
- ✅ Implementar soluções de HA para serviços reais

### Informações do Curso

| Aspecto | Detalhes |
|---------|----------|
| **Nível** | Iniciante → Especialista |
| **Tempo Total** | ~40-50 horas |
| **Pré-requisitos** | Conhecimento básico de Linux e virtualização |
| **Formato** | Markdown + Exercícios Práticos |

---

## 📚 Estrutura do Curso

| Módulo | Título | Nível | Tempo | Tópicos Principais |
|--------|--------|-------|-------|-------------------|
| 01 | Introdução à Alta Disponibilidade | ⭐ Iniciante | 2h | Conceitos HA, componentes, terminologia |
| 02 | Preparando o Ambiente Libvirt | ⭐ Iniciante | 3h | VMs, rede, configuração |
| 03 | Instalação Pacemaker/Corosync | ⭐⭐ Intermediário | 4h | Instalação, cluster, quórum |
| 04 | Gerenciamento de Recursos | ⭐⭐ Intermediário | 5h | Recursos, grupos, restrições |
| 05 | STONITH/Fencing | ⭐⭐ Intermediário | 4h | Fencing, fence_virsh, testes |
| 06 | Monitoramento e Troubleshooting | ⭐⭐⭐ Avançado | 6h | Logs, diagnóstico, resolução |
| 07 | Tópicos Avançados | ⭐⭐⭐ Avançado | 5h | VMs, Pacemaker Remote, casos reais |

**Legenda:**
- ⭐ = Iniciante (sem experiência com Pacemaker)
- ⭐⭐ = Intermediário (conhecimento básico de Pacemaker)
- ⭐⭐⭐ = Avançado (experiência com clusters)

---

## 🚀 Guia Rápido de Instalação

### Pré-requisitos

- Host com Libvirt/KVM instalado
- Mínimo 8GB RAM disponível
- 30GB de espaço em disco
- Conexão com internet

### Criar VMs - Ubuntu 22.04 LTS (Recomendado)

```bash
# 1. Criar rede virtual
virsh net-create network.xml

# 2. Criar VM node1
virt-install --name node1 --memory 2048 --vcpus 2 \
  --disk size=20 --os-variant ubuntu22.04 \
  --network network=default --cdrom ubuntu-22.04-live-server-amd64.iso

# 3. Criar VM node2 (repetir com nome diferente)
virt-install --name node2 --memory 2048 --vcpus 2 \
  --disk size=20 --os-variant ubuntu22.04 \
  --network network=default --cdrom ubuntu-22.04-live-server-amd64.iso
```

### Configuração Inicial (Executar em ambas as VMs)

```bash
# 1. Atualizar sistema
sudo apt update && sudo apt upgrade -y

# 2. Configurar hostname
sudo hostnamectl set-hostname node1  # ou node2

# 3. Configurar rede estática
sudo nano /etc/netplan/00-installer-config.yaml
# Configure:
# node1: 192.168.122.101/24
# node2: 192.168.122.102/24
sudo netplan apply

# 4. Configurar /etc/hosts
echo "192.168.122.101 node1.example.com node1" | sudo tee -a /etc/hosts
echo "192.168.122.102 node2.example.com node2" | sudo tee -a /etc/hosts

# 5. Desabilitar firewall (APENAS PARA LABORATÓRIO)
sudo ufw disable

# 6. Verificar conectividade
ping node2
ssh node2
```

### Instalar Pacemaker

```bash
# 1. Instalar pacotes
sudo apt install -y pacemaker corosync pcs resource-agents

# 2. Iniciar serviços
sudo systemctl start pcsd
sudo systemctl enable pcsd

# 3. Definir senha hacluster
sudo passwd hacluster

# 4. Criar cluster (em node1)
sudo pcs host auth node1 node2 -u hacluster -p <password>
sudo pcs cluster setup mycluster node1 node2 --start --enable

# 5. Verificar status
sudo pcs cluster status
```

---

## 🔧 Exercícios Práticos Detalhados

Este curso inclui **3 exercícios práticos avançados** que cobrem cenários reais:

### Exercício 01: Configuração Completa com Failover Automático

- **Módulo:** 03 - Instalação Pacemaker
- **Nível:** ⭐⭐ Intermediário
- **Tempo:** 90-120 minutos
- **O que você fará:**
  - Criar um cluster com 2 nós
  - Configurar um IP virtual (VIP)
  - Adicionar um serviço web gerenciado
  - Testar failover automático
- **Arquivo:** [`EXERCICIOS_PRATICOS_DETALHADOS.md`](EXERCICIOS_PRATICOS_DETALHADOS.md) (Exercício 01)

### Exercício 02: Implementação de STONITH/Fencing

- **Módulo:** 05 - STONITH/Fencing
- **Nível:** ⭐⭐⭐ Avançado
- **Tempo:** 90-120 minutos
- **O que você fará:**
  - Configurar fence_virsh
  - Criar dispositivos STONITH
  - Testar proteção contra split-brain
  - Simular partições de rede
- **Arquivo:** [`EXERCICIOS_PRATICOS_DETALHADOS.md`](EXERCICIOS_PRATICOS_DETALHADOS.md) (Exercício 02)

### Exercício 03: Troubleshooting Avançado

- **Módulo:** 06 - Monitoramento e Troubleshooting
- **Nível:** ⭐⭐⭐ Avançado
- **Tempo:** 120-150 minutos
- **O que você fará:**
  - Diagnosticar problemas de cluster
  - Analisar logs do Pacemaker
  - Resolver cenários complexos
  - Implementar monitoramento
- **Arquivo:** [`EXERCICIOS_PRATICOS_DETALHADOS.md`](EXERCICIOS_PRATICOS_DETALHADOS.md) (Exercício 03)

### Como Executar os Exercícios

1. Leia o módulo correspondente
2. Abra o arquivo `EXERCICIOS_PRATICOS_DETALHADOS.md`
3. Siga os passos detalhados
4. Verifique os resultados esperados
5. Complete os desafios de aprofundamento (opcional)

**Dica:** Crie snapshots das VMs antes de cada exercício para poder reverter facilmente.

---

## 📖 Recursos Adicionais

### Documentação Oficial

- [ClusterLabs Pacemaker](https://clusterlabs.org/pacemaker/doc/) - Documentação oficial
- [Corosync](http://corosync.github.io/corosync/) - Documentação do Corosync
- [Red Hat HA Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/configuring_and_managing_high_availability_clusters/index) - Guia completo para RHEL

### Comunidades Online

- [ClusterLabs Mailing List](https://clusterlabs.org/mailman/listinfo/users) - Suporte oficial
- [Reddit r/sysadmin](https://www.reddit.com/r/sysadmin/) - Comunidade geral
- [Stack Overflow - Tag Pacemaker](https://stackoverflow.com/questions/tagged/pacemaker) - Q&A

### Blogs e Tutoriais

- [Linux HA Blog](https://blog.clusterlabs.org/) - Blog oficial
- [SUSE HA Guide](https://documentation.suse.com/sles/15-SP5/html/SLES-all/book-ha.html) - Documentação SUSE

### Ferramentas Úteis

- [Pacemaker Simulator](https://github.com/ClusterLabs/pacemaker/tree/master/tools/crm_simulate) - Simular comportamento do cluster
- [Hawk](https://github.com/ClusterLabs/hawk) - Interface web para Pacemaker
- [PCS](https://github.com/ClusterLabs/pcs) - Ferramenta CLI (já incluída no curso)

---

## ❓ Problemas Comuns e Soluções

### Problema: VMs não conseguem se comunicar

**Sintomas:** `ping node2` falha

**Solução:**
```bash
# 1. Verificar configuração de rede
ip addr show
ip route show

# 2. Verificar firewall
sudo ufw status
sudo ufw disable  # Para laboratório

# 3. Verificar bridge Libvirt
virsh net-list
virsh net-info default
```

### Problema: Cluster não inicia

**Sintomas:** `pcs cluster status` retorna erro

**Solução:**
```bash
# 1. Verificar se pcsd está rodando
sudo systemctl status pcsd

# 2. Verificar logs
sudo journalctl -u pacemaker -n 50
sudo journalctl -u corosync -n 50

# 3. Reiniciar serviços
sudo systemctl restart corosync
sudo systemctl restart pacemaker
```

### Problema: Recurso não inicia

**Sintomas:** Recurso em estado "failed"

**Solução:**
```bash
# 1. Verificar status detalhado
sudo pcs resource show <resource_name>

# 2. Limpar estado
sudo pcs resource cleanup <resource_name>

# 3. Verificar logs
sudo journalctl -u pacemaker -f
```

### Problema: Quórum perdido

**Sintomas:** Cluster não responde

**Solução:**
```bash
# 1. Verificar quórum
sudo corosync-quorumtool -s

# 2. Forçar quórum (apenas laboratório)
sudo corosync-quorumtool -e 1

# 3. Ou desabilitar requisito de quórum
sudo pcs property set no-quorum-policy=ignore
```

**Para mais problemas, consulte:** [`EXERCICIOS_PRATICOS_DETALHADOS.md`](EXERCICIOS_PRATICOS_DETALHADOS.md) (Exercício 03)

---

## 🤝 Como Contribuir

Veja o arquivo [`CONTRIBUTING.md`](CONTRIBUTING.MD) para mais detalhes.

---

## 🗺️ Roadmap do Projeto

### ✅ Concluído

- [x] 7 módulos de conteúdo
- [x] 3 exercícios práticos detalhados
- [x] Apresentação introdutória (slides)
- [x] Guia de instalação do ambiente

### 🚀 Em Desenvolvimento

- [ ] Mais exercícios práticos
- [ ] Criação do Ambiente em Vagrant

### 📋 Planejado

- [ ] Integração com Docker/Containers
- [ ] Casos de uso com bancos de dados (MySQL, PostgreSQL)

**Quer ajudar com algum item?** Abra uma [Issue](https://github.com/fabiobilac/Curso-Completo-de-Pacemaker-Linux/issues) ou [Discussion](https://github.com/fabiobilac/Curso-Completo-de-Pacemaker-Linux/discussions)!

---

## 📄 Licença

Este projeto está licenciado sob a licença MIT. Veja o arquivo [`LICENSE`](LICENSE) para mais detalhes.

---

**Última atualização:** Outubro 2025
