# Ansible을 통한 Prometheus 및 Grafana 자동화 배포

Rocky Linux 9 환경에서 Prometheus, Grafana, Node Exporter를 Ansible로 자동 설치/구성하는 프로젝트입니다. 운영 노드(hosts) 정보와, 실습에 필요한 전체 디렉터리 트리 구조를 명확하게 반영했습니다.

---

📝 **프로젝트 개요**

* **Prometheus**: 서버 및 인프라 모니터링 시계열 데이터 수집
* **Grafana**: 대시보드 시각화, 알람 설정
* **Node Exporter**: 각 노드의 시스템 메트릭 수집
* **firewall**: 서비스 포트 자동 오픈 (firewalld 설치·시작, 포트 영구 추가 및 즉시 반영)

---

📋 **운영 호스트 (/etc/hosts 예시)**

```text
192.168.10.100 control.example.com control main
192.168.10.110 db.example.com    node1  db
192.168.10.120 web.example.com   node2  web
```

위 이름들은 Ansible inventory 및 실습 환경에서 동일하게 사용합니다.

---

📂 **디렉터리 구조**

```text
ansible_monitoring_yaml/
├── README.md
└── ansible-monitoring
    ├── ansible.cfg
    ├── inventory
    │   └── hosts.ini
    ├── playbooks
    │   └── monitoring.yml
    └── roles
        ├── alertmanager
        │   ├── defaults
        │   │   └── main.yml
        │   ├── handlers
        │   │   └── main.yml
        │   ├── tasks
        │   │   └── main.yml
        │   └── templates
        │       └── alertmanager.yml.j2
        ├── firewall
        │   ├── defaults
        │   │   └── main.yml
        │   └── tasks
        │       └── main.yml
        ├── grafana
        │   ├── files
        │   │   ├── dashboards
        │   │   │   ├── cpu-usage-dashboard.json
        │   │   │   └── memory-usage-dashboard.json
        │   │   └── provisioning
        │   │       ├── dashboards
        │   │       │   └── dashboards.yml
        │   │       └── datasources
        │   │           └── prometheus.yaml
        │   ├── tasks
        │   │   └── main.yml
        │   └── templates
        │       └── grafana.ini.j2
        ├── node_exporter
        │   ├── files
        │   │   └── node_exporter.service
        │   └── tasks
        │       └── main.yml
        └── prometheus
            ├── files
            │   └── alert_rules
            │       └── alerts.yml
            ├── handlers
            │   └── main.yml
            ├── tasks
            │   └── main.yml
            └── templates
                └── prometheus.yml.j2

```

---

🔧 **요구사항 및 사전 준비**

**제어 노드(Control Node)**

* Rocky Linux 9 또는 RHEL 계열
* Ansible 2.9 이상 (ansible-core)
* EPEL 저장소 활성화
* `ansible.posix` 컬렉션 설치

```bash
sudo dnf install epel-release -y
sudo dnf install ansible-core -y
ansible-galaxy collection install ansible.posix
ansible --version
```

**관리 노드(Managed Node)**

* Python3, openssh-server 설치
* SSH 키 기반 접근

```bash
sudo dnf install openssh-server python3 -y
sudo systemctl enable --now sshd
sudo dnf install python3-cryptography -y  # 일부 모듈 필요시
systemctl is-active sshd && python3 --version
```

---

📑 **inventory/hosts.ini 샘플**

```ini
[monitoring]
control.example.com ansible_user=root

[node_exporters]
control.example.com ansible_user=root
db.example.com      ansible_user=root
web.example.com     ansible_user=root
```

*사용자 및 호스트명은 실 환경에 맞게 수정하세요.*

---

🚀 **플레이북 실행법**

```bash
ansible-playbook -i inventory/hosts.ini playbooks/monitoring.yml
```


필요에 따라 `--limit` 옵션으로 특정 노드만 지정할 수 있습니다.

ansible-playbook -i inventory/hosts.ini playbooks/monitoring.yml \
  --limit monitoring
  
-> prometheus + grafana 실행
    
ansible-playbook -i inventory/hosts.ini playbooks/monitoring.yml \
  --limit node_exporters
  
-> node_exporters 실행 


---

⚙️ **주요 역할 설명 (roles)**

* **firewall**: firewalld 설치, 서비스 활성화 및 시작

  * `firewall_ports` 변수로 전달된 포트를 영구 추가 (`permanent: yes`) 및 즉시 반영 (`immediate: yes`)
* **prometheus**: Prometheus 설치, 설정 템플릿 배포 및 서비스 관리
* **grafana**: Grafana OSS 설치, `grafana.ini` 배포, 대시보드/데이터소스 프로비저닝, 서비스 재시작
* **node\_exporter**: Node Exporter 설치, systemd 유닛 파일 배포 및 서비스 관리

---

🛠️ **커스터마이징 포인트**

* `inventory/hosts.ini`: ansible\_user, 호스트명 등
* `roles/*/defaults/main.yml`: 변수화된 값
* `roles/grafana/files/provisioning/` 하위 dashboards, datasources 파일
* `firewall_ports` 값을 Playbook에서 조정하여 추가 포트 개방

