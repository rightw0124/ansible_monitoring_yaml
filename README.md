# Ansible Monitoring Playbook

Rocky Linux 9 환경에서 **Prometheus + Grafana** 중앙 모니터링 서버와  
여러 **Node Exporter** 에이전트를 자동 설치/구성하기 위한 Ansible 프로젝트입니다.

---

## 📋 목차

1. [요구사항](#-요구사항)  
2. [디렉터리 구조](#-디렉터리-구조)  
3. [설치 전 준비](#-설치-전-준비)  
4. [SSH 키 부트스트랩](#-ssh-키-부트스트랩)  
5. [플레이북 실행](#-플레이북-실행)  
6. [변수 커스터마이징](#-변수-커스터마이징)  
7. [추가 자료](#-추가-자료)  

---

## 🔧 요구사항

- 제어 머신: Ansible 2.9 이상  
- 대상 노드: Rocky Linux 9  
- Python 3 (대상 노드)  
- SSH 키 기반 접근 설정(초기 연결 시 비밀번호 또는 기존 키 허용)

---

## 📂 디렉터리 구조

ansible-monitoring/
├── ansible.cfg
├── inventory/
│ └── hosts.ini
├── playbooks/
│ ├── monitoring.yml
│ └── bootstrap-ssh.yml # (선택) SSH 키 자동 배포
├── roles/
│ ├── prometheus/
│ │ ├── tasks/main.yml
│ │ └── templates/prometheus.yml.j2
│ ├── grafana/
│ │ ├── tasks/main.yml
│ │ └── templates/grafana.ini.j2
│ └── node_exporter/
│ ├── tasks/main.yml
│ └── files/node_exporter.service
└── README.md


---

## ⚙️ 설치 전 준비

1. 리포지토리 클론  
   ```bash
   git clone git@your.git.server:username/ansible-monitoring.git
   cd ansible-monitoring
