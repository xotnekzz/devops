🏗️ 아키텍처 요약 (Architecture Overview)

Host (Control Node): 내 맥북 (Ansible 실행 주체, 터미널 접속)

Hypervisor: VirtualBox + Vagrant

Guests (Managed Nodes):

OS: Rocky Linux 9.x

수량: 5대 (server-1 ~ server-5)

Spec (대당): 2 vCPU / 2GB RAM (총 10GB RAM 사용)

Network: 192.168.56.x (Host-Only Network)

Provisioning: VM 생성 직후 Ansible이 자동으로 OS 필수 설정을 완료함.

🛠️ 1단계: 준비물 설치 (Prerequisites)

터미널에서 아래 명령어로 필수 도구를 설치합니다. (이미 설치했다면 생략)

Bash



# 1. VirtualBox (가상화 엔진)

brew install --cask virtualbox# 2. Vagrant (VM 관리자)

brew install --cask vagrant# 3. Ansible (설정 자동화 도구)

brew install ansible

📂 2단계: 프로젝트 디렉토리 및 코드 작성

원하는 위치에 폴더를 만들고(mkdir my-lab), 아래 3개 파일을 생성하세요.

📄 파일 1: Vagrantfile (인프라 정의서)

Ruby



# Vagrantfile

Vagrant.configure("2") do |config|

  config.vm.box = "rockylinux/9"

  

  # 서버 5대 정의 (IP: 11~15)

  servers = [

    { name: "server-1", ip: "192.168.56.11" },

    { name: "server-2", ip: "192.168.56.12" },

    { name: "server-3", ip: "192.168.56.13" },

    { name: "server-4", ip: "192.168.56.14" },

    { name: "server-5", ip: "192.168.56.15" }

  ]



  servers.each do |server|

    config.vm.define server[:name] do |node|

      node.vm.hostname = server[:name]

      node.vm.network "private_network", ip: server[:ip]

      

      node.vm.provider "virtualbox" do |vb|

        vb.name = server[:name]

        vb.memory = "2048"   # 2GB RAM

        vb.cpus = 2          # 2 vCPU

        vb.linked_clone = true

      end

      

      # Ansible 자동 실행 (VM이 생성될 때마다 실행됨)

      node.vm.provision "ansible" do |ansible|

        ansible.playbook = "playbook.yml"

        ansible.inventory_path = "inventory.ini"

        ansible.limit = "all"

        ansible.verbose = false

        ansible.compatibility_mode = "2.0"

      end

    end

  endend

📄 파일 2: inventory.ini (서버 접속 정보)

Ansible이 서버에 접속하기 위한 정보입니다. Vagrant가 생성하는 키 경로를 미리 지정합니다.

Ini, TOML



[all_servers]

server-1 ansible_host=192.168.56.11 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/server-1/virtualbox/private_key

server-2 ansible_host=192.168.56.12 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/server-2/virtualbox/private_key

server-3 ansible_host=192.168.56.13 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/server-3/virtualbox/private_key

server-4 ansible_host=192.168.56.14 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/server-4/virtualbox/private_key

server-5 ansible_host=192.168.56.15 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/server-5/virtualbox/private_key[all_servers:vars]# SSH 접속 시 지문 확인 무시ansible_ssh_common_args='-o StrictHostKeyChecking=no'

📄 파일 3: playbook.yml (기초 공사 시나리오)

리눅스 서버로서 갖춰야 할 기본 소양(패키지, 타임존, 방화벽, 호스트 파일)을 세팅합니다.

YAML



---- name: DevOps Base Environment Setup

  hosts: all

  become: yes

  tasks:

    - name: 1. 필수 패키지 설치 (EPEL, Network Tools 등)

      dnf:

        name:

          - epel-release

          - vim

          - git

          - wget

          - curl

          - net-tools   # ifconfig 등

          - bind-utils  # nslookup 등

          - tar

        state: present



    - name: 2. 타임존 설정 (Asia/Seoul)

      timezone:

        name: Asia/Seoul



    - name: 3. 방화벽 비활성화 (실습 편의성)

      service:

        name: firewalld

        state: stopped

        enabled: no



    - name: 4. Swap 메모리 비활성화 (K8s 필수 조건)

      shell: |

        swapoff -a

        sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

    - name: 5. /etc/hosts 파일 구성 (서버 이름으로 통신 가능하게)

      blockinfile:

        path: /etc/hosts

        block: |

          192.168.56.11 server-1

          192.168.56.12 server-2

          192.168.56.13 server-3

          192.168.56.14 server-4

          192.168.56.15 server-5


