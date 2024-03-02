### 서비스 디스커버리

MSA에서는 서비스들 마다 다들 각자 다른 IP, Port를 가지고 있다.

이러한 서로 다른 서비스들의 IP와 Port의 정보들에 대해서 저장하고 관리할 필요가 있는데 이러한 메커니즘을 서비스 디스커버리라고 한다.

클라우드 기반 마이크로서비스 환경에서 견고한 서비스 디스커버리 메커니즘을 사용한다면 어떠한 이점이 있을까

### 고가용성

서비스 디스커버리는 클러스터 노드 간 서비스 검색 정보가 공유되는 핫 클러스터링 환경을 지원할 수 있어야한다.

한 노드가 가용하지 않으면 다른 노드가 그 역할을 대신 수행할 수 있다.

> 클러스터는 서버와 인스턴스들의 그룹으로 정의할 수 있다.

이 경우 모든 인스턴스들은 고가용성, 안정성, 확장성을 제공하고자 동일한 구성을 가지고 협업한다.

로드 밸런서와 통합된 클러스터 서비스 중단을 방지하는 페일오버와 세션 데이터를 저장하는 세션 복제 기능을 제공할 수 있다.


### P2P (Peer to Peer)

서비스 디스커버리는 모든 노드들 간에 자신의 서비스 인스턴스 상태를 공유한다.

### 부하분산

서비스 디스커버리는 요청을 동적으로 분산시켜 관리하는 모든 인스턴스에 분배한다.

초창기 더 고정적이며 수동으로 관리되는 로드밸런서를 대체한다.

### 회복성

서비스 디스커버리 클라이언트는 서비스 정보를 로컬에 캐싱한다.

로컬 캐싱은 서비스 디스커버리 기능이 점진적으로 저하되는 것을 고려했기 때문에 생겨났다.

서비스 디스커버리가 서비스를 가용하지 않아도 애플리케이션은 여전히 작동할 수 있는 것이다.

### 결함내성

서비스 디스컵리가 비정상 서비스 인스턴스를 탐지하면 클라이언트 요청을 처리하는 가용 서비스 목록에서 해당 인스턴스를 제거해야한다.

서비스를 이용하여 이러한 결함을 탐지하고 사람의 개입 없이 조치되어야한다.