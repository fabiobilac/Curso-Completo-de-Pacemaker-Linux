# Módulo 02: Preparando o Ambiente de Laboratório com Libvirt

Para realizar os exercícios práticos deste curso, é fundamental configurar um ambiente de laboratório que simule um cluster de alta disponibilidade. Utilizaremos o **Libvirt** e o **KVM** (Kernel-based Virtual Machine) para criar máquinas virtuais (VMs) que atuarão como os nós do nosso cluster Pacemaker. O Libvirt é uma API e um conjunto de ferramentas para gerenciar plataformas de virtualização, enquanto o KVM é a tecnologia de virtualização nativa do Linux.

## 2.1 Instalação e Configuração do Libvirt/KVM

### 2.1.1 Verificando a Virtualização de Hardware

Antes de instalar o KVM, verifique se seu processador suporta virtualização de hardware (Intel VT-x ou AMD-V). Você pode fazer isso executando o seguinte comando no seu sistema host:

```bash
egrep -c '(vmx|svm)' /proc/cpuinfo
```

Se o resultado for `0`, seu CPU não suporta virtualização de hardware ou ela está desabilitada na BIOS/UEFI. Se for `1` ou mais, a virtualização é suportada.

### 2.1.2 Instalação dos Pacotes Essenciais

Os pacotes necessários para o Libvirt/KVM variam ligeiramente entre as distribuições Linux. Assegure-se de ter privilégios de `sudo`.

#### Para sistemas baseados em RHEL/CentOS (ex: CentOS Stream 8/9):

```bash
sudo dnf install @virtualization -y
sudo systemctl enable --now libvirtd
sudo systemctl status libvirtd
```

#### Para sistemas baseados em Debian/Ubuntu (ex: Ubuntu Server 20.04/22.04):

```bash
sudo apt update
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager -y
sudo systemctl enable --now libvirtd
sudo systemctl status libvirtd
```

Após a instalação, adicione seu usuário ao grupo `libvirt` para gerenciar VMs sem `sudo` (reinicie a sessão ou faça logout/login para que as mudanças tenham efeito):

```bash
sudo usermod -aG libvirt $(whoami)
```

## 2.2 Criação de Máquinas Virtuais para o Cluster

Criaremos duas máquinas virtuais que servirão como `node1` e `node2` do nosso cluster. Recomenda-se usar uma distribuição Linux minimalista para os nós do cluster, como Ubuntu Server ou CentOS Stream, para economizar recursos.

### 2.2.1 Download da Imagem ISO

Baixe a imagem ISO da sua distribuição Linux preferida. Por exemplo, para Ubuntu Server 22.04:

```bash
wget https://releases.ubuntu.com/22.04/ubuntu-22.04.3-live-server-amd64.iso -P /var/lib/libvirt/images/
```

### 2.2.2 Criação das VMs com `virt-install`

`virt-install` é uma ferramenta de linha de comando para criar VMs. Criaremos duas VMs com 2 vCPUs, 2GB de RAM e um disco de 20GB. Certifique-se de ajustar os caminhos da ISO e os nomes dos nós conforme sua preferência.

#### Criar `node1`:

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

#### Criar `node2`:

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

*   `--name`: Nome da máquina virtual.
*   `--ram`, `--vcpus`, `--disk`: Especificações de hardware da VM.
*   `--os-variant`: Ajuda o `virt-install` a otimizar a configuração para o SO especificado.
*   `--network bridge=virbr0`: Conecta a VM à bridge de rede padrão do Libvirt (`virbr0`), que geralmente fornece NAT e DHCP. Para um ambiente de cluster, precisaremos de IPs estáticos.
*   `--graphics none`: Desabilita a saída gráfica, permitindo a instalação via console serial (útil para servidores).
*   `--location`: Caminho para a imagem ISO de instalação.
*   `--extra-args "console=ttyS0,115200n8 serial"`: Configura a saída do console serial para a instalação, permitindo interagir com a VM via `virsh console`.
*   `--autostart`: Faz com que a VM inicie automaticamente com o host.

### 2.2.3 Acessando as VMs para Instalação

Após a criação, as VMs iniciarão o processo de instalação. Você pode acessá-las via console serial:

```bash
sudo virsh console node1
# Para sair do console, pressione Ctrl + ]
```

Siga as instruções de instalação do sistema operacional para cada VM. Certifique-se de configurar:

*   **Nome do Host:** `node1` e `node2` respectivamente.
*   **Usuário e Senha:** Crie um usuário com permissões `sudo`.
*   **Configuração de Rede:** Configure IPs estáticos para as VMs. Por exemplo:
    *   `node1`: `192.168.122.101/24`
    *   `node2`: `192.168.122.102/24`
    *   Gateway: `192.168.122.1` (geralmente o IP da bridge `virbr0` no host).
    *   DNS: Configure para resolver nomes internos e externos.

Após a instalação, reinicie as VMs e verifique se elas estão acessíveis via SSH a partir do seu host.

## 2.3 Configuração de Rede para o Cluster

Para que o cluster funcione corretamente, a comunicação entre os nós é vital. Além dos IPs estáticos, a resolução de nomes e a configuração do firewall são cruciais.

### 2.3.1 Configuração de `/etc/hosts`

Em **ambos os nós** (`node1` e `node2`), edite o arquivo `/etc/hosts` para incluir os nomes e IPs dos nós do cluster:

```bash
# Em node1 e node2:
sudo nano /etc/hosts

# Adicione as seguintes linhas:
192.168.122.101 node1.example.com node1
192.168.122.102 node2.example.com node2
```

Teste a conectividade e resolução de nomes entre os nós:

```bash
ping node2
ssh node2
```

### 2.3.2 Configuração do Firewall

O Corosync e o Pacemaker utilizam portas específicas para comunicação. É essencial que essas portas estejam abertas no firewall de **todos os nós** do cluster. Em um ambiente de laboratório, para simplificar, você pode desabilitar o firewall temporariamente, mas em produção, configure regras específicas.

#### Para sistemas baseados em RHEL/CentOS (usando `firewalld`):

```bash
sudo systemctl stop firewalld
sudo systemctl disable firewalld
```

#### Para sistemas baseados em Debian/Ubuntu (usando `ufw`):

```bash
sudo ufw disable
```

**Em um ambiente de produção, as portas a serem abertas para Corosync e Pacemaker são:**

*   **Corosync:** UDP 5405 (multicast ou unicast, dependendo da configuração)
*   **Pacemaker:** TCP 2224 (para `pcsd`), TCP 3121 (para `crmd`)

## Exercícios Práticos

1.  Verifique o suporte à virtualização de hardware no seu sistema host.
2.  Instale o Libvirt/KVM e as ferramentas associadas na sua máquina host.
3.  Baixe uma imagem ISO de uma distribuição Linux de sua escolha (Ubuntu Server ou CentOS Stream).
4.  Crie duas máquinas virtuais (`node1` e `node2`) usando `virt-install` com as especificações recomendadas.
5.  Acesse as VMs via `virsh console` e realize a instalação completa do sistema operacional, configurando nomes de host e IPs estáticos.
6.  Configure o arquivo `/etc/hosts` em ambas as VMs para que possam resolver os nomes uma da outra.
7.  Desabilite o firewall em ambas as VMs para permitir a comunicação irrestrita entre os nós do cluster (apenas para fins de laboratório).
8.  Teste a conectividade entre `node1` e `node2` usando `ping` e `ssh`.

## Referências

[1] Red Hat. (n.d.). *Configuring and managing high availability clusters*. Disponível em: [https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html-single/configuring_and_managing_high_availability_clusters/index](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html-single/configuring_and_managing_high_availability_clusters/index)

[2] KVM. *Installation*. Disponível em: [https://www.linux-kvm.org/page/Installation](https://www.linux-kvm.org/page/Installation)

