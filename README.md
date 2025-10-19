# Curso Completo de Pacemaker Linux

Bem-vindo ao curso completo de Pacemaker Linux, projetado para iniciantes que desejam dominar os fundamentos e a aplicação prática de clusters de Alta Disponibilidade (HA) no ambiente Linux. Este curso abrange desde os conceitos básicos de HA até a configuração avançada de recursos e o troubleshooting, com um forte foco em exercícios práticos utilizando o Libvirt para simular um ambiente de cluster real.

## Sobre o Curso

Este material foi desenvolvido para proporcionar uma compreensão sólida do Pacemaker, Corosync e dos mecanismos de fencing (STONITH), essenciais para garantir a continuidade dos serviços em sistemas críticos. Cada módulo inclui explicações detalhadas, exemplos de comandos e exercícios práticos para reforçar o aprendizado.

## Estrutura do Curso

O curso está dividido nos seguintes módulos, separados para facilitar a navegação e o estudo:

*   **Módulo 01: Introdução à Alta Disponibilidade e Conceitos Fundamentais**
    *   Explora o que é HA, sua importância e os componentes essenciais de um cluster.
*   **Módulo 02: Preparando o Ambiente de Laboratório com Libvirt**
    *   Guia passo a passo para configurar um ambiente de virtualização com Libvirt/KVM e criar as máquinas virtuais para o cluster.
*   **Módulo 03: Instalação e Configuração Básica do Pacemaker/Corosync**
    *   Detalha a instalação dos pacotes, configuração inicial do `pcs` e a criação do cluster.
*   **Módulo 04: Gerenciamento de Recursos no Pacemaker**
    *   Aborda a criação, configuração e gerenciamento de diferentes tipos de recursos, grupos de recursos e restrições.
*   **Módulo 05: STONITH/Fencing**
    *   Explica a importância do STONITH para a integridade do cluster e como configurá-lo, com foco em `fence_virsh`.
*   **Módulo 06: Monitoramento e Troubleshooting**
    *   Ensina a usar comandos `pcs` e logs para monitorar o cluster e diagnosticar problemas comuns.
*   **Módulo 07: Tópicos Avançados e Casos de Uso Reais**
    *   Aprofunda em gerenciamento de VMs com Pacemaker, Pacemaker Remote e integração com serviços específicos como bancos de dados e servidores web.
*   **Módulo 08: Índice e Materiais Complementares**
    *   Um índice completo do curso e links complementares.

Espero que este material seja **muito útil** para você e para quem deseja aprender sobre Pacemaker Linux!

## Guia Rápido de Instalação do Ambiente de Laboratório

Para iniciar os exercícios práticos, você precisará de um ambiente de virtualização. Este curso utiliza o Libvirt/KVM. Siga os passos abaixo para configurar seu ambiente:

### 1. No seu sistema Host (Máquina Física ou VM onde o Libvirt/KVM será executado):

*   **Verifique a Virtualização de Hardware:**
    ```bash
    egrep -c \'(vmx|svm)\' /proc/cpuinfo
    ```
*   **Instale os Pacotes Libvirt/KVM:**
    *   **RHEL/CentOS:** `sudo dnf install @virtualization -y && sudo systemctl enable --now libvirtd`
    *   **Debian/Ubuntu:** `sudo apt update && sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager -y && sudo systemctl enable --now libvirtd`
*   **Adicione seu usuário ao grupo `libvirt`:**
    ```bash
    sudo usermod -aG libvirt $(whoami)
    # Reinicie a sessão ou faça logout/login.
    ```
*   **Baixe a Imagem ISO (ex: Ubuntu Server 22.04):**
    ```bash
    sudo wget https://releases.ubuntu.com/22.04/ubuntu-22.04.3-live-server-amd64.iso -P /var/lib/libvirt/images/
    ```
*   **Crie as VMs `node1` e `node2`:**
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

### 2. Em cada VM (`node1` e `node2`):

*   **Acesse via `virsh console` e instale o SO:**
    ```bash
    sudo virsh console node1 # Para sair: Ctrl + ]
    ```
    *   Configure o nome do host (`node1` ou `node2`).
    *   Configure IPs estáticos (ex: `node1`: `192.168.122.101/24`, `node2`: `192.168.122.102/24`).
*   **Configure `/etc/hosts`:**
    ```bash
    sudo nano /etc/hosts
    # Adicione:
    192.168.122.101 node1.example.com node1
    192.168.122.102 node2.example.com node2
    ```
*   **Desabilite o Firewall (APENAS PARA LABORATÓRIO):**
    *   **RHEL/CentOS:** `sudo systemctl stop firewalld && sudo systemctl disable firewalld`
    *   **Debian/Ubuntu:** `sudo ufw disable`
*   **Teste a conectividade:** `ping node2`, `ssh node2`.

## Contribuições

Contribuições são bem-vindas! Sinta-se à vontade para abrir issues, sugerir melhorias ou enviar pull requests.

## Licença

Este projeto está licenciado sob a licença MIT. Veja o arquivo `LICENSE` para mais detalhes.
