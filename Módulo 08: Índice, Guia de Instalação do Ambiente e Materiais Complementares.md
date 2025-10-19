# Módulo 08: Índice, Guia de Instalação do Ambiente e Materiais Complementares

Este módulo final compila todos os recursos do curso, fornecendo um índice completo, um guia passo a passo para a configuração do ambiente de laboratório e sugestões de materiais adicionais para aprofundamento.

## 8.1 Índice do Curso

Este curso foi estruturado para guiá-lo desde os conceitos fundamentais de Alta Disponibilidade até a configuração avançada de clusters Pacemaker, com foco em exercícios práticos utilizando Libvirt. Abaixo, o índice completo dos módulos:

*   **Módulo 01: Introdução à Alta Disponibilidade e Conceitos Fundamentais**
    *   1.1 O que é Alta Disponibilidade (HA) e por que é importante?
    *   1.2 Componentes de um Cluster HA
    *   1.3 Terminologia Básica
    *   Referências

*   **Módulo 02: Preparando o Ambiente de Laboratório com Libvirt**
    *   2.1 Instalação e Configuração do Libvirt/KVM
    *   2.2 Criação de Máquinas Virtuais para o Cluster
    *   2.3 Configuração de Rede para o Cluster
    *   Exercícios Práticos
    *   Referências

*   **Módulo 03: Instalação e Configuração Básica do Pacemaker/Corosync**
    *   3.1 Pré-requisitos de Rede e Sistema Operacional
    *   3.2 Instalação dos Pacotes Essenciais
    *   3.3 Configuração Inicial do `pcs` e Autenticação
    *   3.4 Criação do Cluster
    *   3.5 Verificação do Status do Cluster
    *   3.6 Desabilitando STONITH (Temporariamente para Laboratório)
    *   Referências

*   **Módulo 04: Gerenciamento de Recursos no Pacemaker**
    *   4.1 Tipos de Recursos no Pacemaker
    *   4.2 Criação e Configuração de Recursos
    *   4.3 Grupos de Recursos
    *   4.4 Restrições de Recursos
    *   Exercícios Práticos
    *   Referências

*   **Módulo 05: STONITH/Fencing**
    *   5.1 A Importância do STONITH
    *   5.2 Tipos de Agentes STONITH
    *   5.3 Configuração de STONITH com `fence_virsh` e Libvirt
    *   5.4 Testando o STONITH
    *   Exercícios Práticos
    *   Referências

*   **Módulo 06: Monitoramento e Troubleshooting**
    *   6.1 Comandos `pcs` para Monitoramento
    *   6.2 Análise de Logs
    *   6.3 Cenários Comuns de Troubleshooting
    *   Exercícios Práticos
    *   Referências

*   **Módulo 07: Tópicos Avançados e Casos de Uso Reais**
    *   7.1 Gerenciamento de Máquinas Virtuais com Pacemaker
    *   7.2 Pacemaker Remote
    *   7.3 Integração com Serviços Específicos e Casos de Uso Reais
    *   Exercícios Práticos
    *   Referências

## 8.2 Guia Rápido de Instalação do Ambiente de Laboratório

Para facilitar a configuração do seu ambiente de laboratório, siga este guia resumido, que consolida os passos do Módulo 02.

### 8.2.1 No seu sistema Host (Máquina Física ou VM onde o Libvirt/KVM será executado):

1.  **Verifique a Virtualização de Hardware:**
    ```bash
    egrep -c '(vmx|svm)' /proc/cpuinfo
    ```
2.  **Instale os Pacotes Libvirt/KVM:**
    *   **RHEL/CentOS:** `sudo dnf install @virtualization -y && sudo systemctl enable --now libvirtd`
    *   **Debian/Ubuntu:** `sudo apt update && sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager -y && sudo systemctl enable --now libvirtd`
3.  **Adicione seu usuário ao grupo `libvirt`:**
    ```bash
    sudo usermod -aG libvirt $(whoami)
    # Reinicie a sessão ou faça logout/login.
    ```
4.  **Baixe a Imagem ISO (ex: Ubuntu Server 22.04):**
    ```bash
    sudo wget https://releases.ubuntu.com/22.04/ubuntu-22.04.3-live-server-amd64.iso -P /var/lib/libvirt/images/
    ```
5.  **Crie as VMs `node1` e `node2`:**
    *   **`node1`:**
        ```bash
        sudo virt-install \
          --name node1 \
          --ram 2048 \
          --vcpus 2 \
          --disk path=/var/lib/libvirt/images/node1.qcow2,size=20 \
          --os-variant ubuntu22.04 \
          --network bridge=virbr0,model=virtio \
          --graphics none \
          --location /var/lib/libvirt/images/ubuntu-22.04.3-live-server-amd64.iso \
          --extra-args "console=ttyS0,115200n8 serial" \
          --autostart
        ```
    *   **`node2`:**
        ```bash
        sudo virt-install \
          --name node2 \
          --ram 2048 \
          --vcpus 2 \
          --disk path=/var/lib/libvirt/images/node2.qcow2,size=20 \
          --os-variant ubuntu22.04 \
          --network bridge=virbr0,model=virtio \
          --graphics none \
          --location /var/lib/libvirt/images/ubuntu-22.04.3-live-server-amd64.iso \
          --extra-args "console=ttyS0,115200n8 serial" \
          --autostart
        ```

### 8.2.2 Em cada VM (`node1` e `node2`):

1.  **Acesse via `virsh console` e instale o SO:**
    ```bash
    sudo virsh console node1 # Para sair: Ctrl + ]
    ```
    *   Configure o nome do host (`node1` ou `node2`).
    *   Configure IPs estáticos (ex: `node1`: `192.168.122.101/24`, `node2`: `192.168.122.102/24`).
2.  **Configure `/etc/hosts`:**
    ```bash
    sudo nano /etc/hosts
    # Adicione:
    192.168.122.101 node1.example.com node1
    192.168.122.102 node2.example.com node2
    ```
3.  **Desabilite o Firewall (APENAS PARA LABORATÓRIO):**
    *   **RHEL/CentOS:** `sudo systemctl stop firewalld && sudo systemctl disable firewalld`
    *   **Debian/Ubuntu:** `sudo ufw disable`
4.  **Teste a conectividade:** `ping node2`, `ssh node2`.

## 8.3 Materiais Complementares e Leitura Adicional

Para aprofundar seus conhecimentos em Pacemaker e Alta Disponibilidade, recomendamos os seguintes recursos:

*   **Documentação Oficial do ClusterLabs:** O site [ClusterLabs.org](https://clusterlabs.org/) é a fonte primária de documentação para Pacemaker e Corosync. O guia *Pacemaker Explained* é um recurso excelente para entender os detalhes internos.
*   **Documentação da Red Hat:** A Red Hat possui extensa documentação sobre configuração e gerenciamento de clusters de alta disponibilidade, especialmente para RHEL. Muitos conceitos são universais e aplicáveis a outras distribuições Linux. Procure por *"Configuring and managing high availability clusters"*.
*   **Livros e Cursos Online:** Plataformas como O'Reilly, Udemy ou Coursera podem oferecer cursos mais aprofundados ou livros sobre Linux HA e Pacemaker.
*   **Comunidades Online:** Fóruns como o Reddit (`r/sysadmin`, `r/linuxadmin`) e comunidades de código aberto são ótimos lugares para fazer perguntas e aprender com a experiência de outros profissionais.
*   **Blogs Técnicos:** Muitos blogs de engenheiros de sistemas e administradores de rede publicam tutoriais e dicas sobre Pacemaker e HA. Uma busca por "Pacemaker tutorial" ou "Linux HA guide" pode revelar muitos recursos úteis.

Esperamos que este curso tenha fornecido uma base sólida para você começar a trabalhar com Pacemaker Linux e construir seus próprios clusters de alta disponibilidade. Continue praticando e explorando os recursos avançados para se tornar um especialista!
