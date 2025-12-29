# home-lab-kubernetes

On-premise home lab Kubernetes project using Proxmox and Raspberry Pi (k3s)

## Overview
본 프로젝트는 애플리케이션 개발이 아닌,  
**온프레미스 환경에서 Kubernetes 클러스터를 직접 구축하고 운영 관점에서 검증하는 것**을 목표로 한 홈랩 프로젝트이다.

클라우드 매니지드 서비스 없이 네트워크, 스토리지, 상태 저장 워크로드를 직접 구성함으로써  
실제 운영 환경과 유사한 조건에서 Kubernetes 동작을 확인한다.

---

## Architecture Overview
전체 환경은 단일 온프레미스 네트워크(`192.168.0.0/24`) 상에 구성되어 있으며,  
Proxmox 서버와 Raspberry Pi 노드들이 동일한 LAN에 연결되어 있다.

Proxmox는 가상화 계층으로 사용되며, 인프라 관리 및 공통 서비스를 담당하는 **Utility VM**을 운영한다.  
Utility VM은 다음 역할을 수행한다.

- Ansible 기반 Infrastructure as Code(IaC)를 통한 노드 초기 설정 및 k3s 클러스터 부트스트랩
- Kubernetes 워크로드를 위한 NFS 기반 중앙 스토리지 제공

Kubernetes 플랫폼은 Raspberry Pi 기반 **k3s 클러스터**로 구성되어 있다.

- Control Plane 노드 1대 (4GB RAM)
- Worker 노드 2대 (각 8GB RAM)

클러스터 외부 트래픽 처리를 위해 **MetalLB(L2 모드)** 를 사용하여  
온프레미스 환경에서 `ServiceType: LoadBalancer`를 구현하였다.  
HTTP 기반 서비스 진입을 위해 **Ingress**를 구성하였다.

상태 저장 워크로드 검증을 위해 **StatefulSet과 PersistentVolumeClaim(PVC)** 를 사용하였으며,  
스토리지는 Utility VM에서 제공하는 **NFS Server**와 연동된 NFS StorageClass를 사용한다.

이를 통해 파드 재시작, 파드 재스케줄, 노드 재부팅 상황에서도  
데이터 영속성이 유지되는지 여부를 검증한다.

---

## Architecture Diagram
![architecture](./docs/architecture.png)

---

## Key Components
- Proxmox VE
- Utility VM (Ansible, NFS)
- k3s Cluster
- MetalLB (L2)
- Ingress
- StatefulSet + PVC (NFS)

---

## Validation Scenarios
- Pod 재시작 시 데이터 유지 여부 확인
- Pod 재스케줄 시 PVC 유지 여부 확인
- Worker 노드 재부팅 후 데이터 무결성 확인
- NFS 기반 StorageClass 동작 검증

---

## Notes
본 프로젝트는 학습 목적의 실습 환경이 아닌,  
**온프레미스 Kubernetes 운영 관점에서의 설계와 검증 기록**을 남기는 것을 목표로 한다.
