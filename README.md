# DevOps Data Platform with Ansible

이 프로젝트는 **Ansible**과 **Vagrant**를 사용하여 로컬 **Apple Silicon(M-Series)** 환경에서 리눅스 서버 클러스터를 구성하고, 그 위에 **StarRocks(OLAP DB)**와 **Apache Kafka(Event Streaming)**를 포함한 데이터 파이프라인 인프라를 자동으로 구축하는 IaC(Infrastructure as Code) 프로젝트입니다.

## 📚 프로젝트 시리즈 (Blog Posts)
이 프로젝트의 상세한 구축 과정과 기술적 배경은 아래 블로그 포스팅에서 확인할 수 있습니다.
*(블로그 내용은 Intel Mac 기준일 수 있으나, 본 프로젝트 코드는 Apple Silicon 환경에 맞춰 구성되었습니다.)*

1. **Infrastructure**: [맥북에 리눅스 서버 5대 구성하기 - Ansible, Vagrant](https://tedi.tistory.com/48)
2. **OLAP DB**: [StarRocks 클러스터 구축하기 with Ansible](https://tedi.tistory.com/49)
3. **Event Streaming**: [Kafka KRaft Cluster 구축하기 with Ansible](https://tedi.tistory.com/50)

---

## 🏗️ 아키텍처 (Architecture)

**VMware Fusion**을 기반으로 총 5대의 Rocky Linux 9 (ARM64) 가상 서버를 구성하여 역할을 분배하였습니다.

| Node | IP Address | Roles | Components |
|:---:|:---:|:---:|:---|
| **server-1** | `192.168.56.11` | StarRocks FE (Leader), BE | StarRocks FE, StarRocks BE |
| **server-2** | `192.168.56.12` | StarRocks FE (Follower), BE | StarRocks FE, StarRocks BE |
| **server-4** | `192.168.56.14` | StarRocks FE (Follower), BE, Kafka | StarRocks FE, StarRocks BE, Kafka Broker/Controller, **Kafka UI** |
| **server-5** | `192.168.56.15` | StarRocks BE, Kafka | StarRocks BE, Kafka Broker/Controller |
| **server-6** | `192.168.56.16` | StarRocks BE, Kafka | StarRocks BE, Kafka Broker/Controller |

### 주요 특징
* **Environment**: Apple Silicon (M1/M2/M3) MacBook
* **Hypervisor**: **VMware Fusion** (via Vagrant `vmware_desktop` provider)
* **OS**: Rocky Linux 9 (aarch64/ARM64)
* **Containerization**: 모든 애플리케이션(StarRocks, Kafka)은 **Docker** 컨테이너로 배포됩니다.
* **StarRocks**: FE(3 Node HA 구성), BE(5 Node)
* **Kafka**: KRaft Combined Mode (ZooKeeper Less), 3 Node Cluster
* **Network**: Vagrant Private Network (`192.168.56.0/24`)

---

## 📂 디렉토리 구조 (Directory Structure)

```bash
.
├── rocky9/                  # Vagrant 프로비저닝 (OS 및 하드웨어 구성)
│   ├── Vagrantfile          # 가상 서버 5대 정의 (VMware Provider 설정)
│   ├── inventory.ini        # Ansible Inventory (Base)
│   └── initial_setup.yml    # 초기 OS 설정 (패키지, Timezone, 등)
├── starrocks/               # StarRocks 클러스터 배포용 Ansible
│   ├── hosts.ini            # StarRocks 전용 Inventory
│   ├── playbook/            # 배포 시나리오 (site.yml)
│   ├── roles/               # FE, BE, Base 설정 Role
│   └── ssh_config           # SSH 접속 설정
├── kafka/                   # Kafka 클러스터 배포용 Ansible
│   ├── inventory.ini        # Kafka 전용 Inventory
│   ├── site.yml             # 배포 시나리오
│   ├── roles/               # Kafka Container 설정 Role
│   └── ssh_config           # SSH 접속 설정
└── roles/                   # 공통 Ansible Roles (Docker 설치 등)
