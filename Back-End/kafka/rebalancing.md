# Kafka 리밸런싱

카프카 컨슈머는 토픽의 각 파티션에서 메시지를 처리하는 역할을 한다.

그런데, 특정 컨슈머가 문제를 겪게 되면 그 컨슈머가 처리하던 파티션의 소유관은 다른 컨슈머로 넘어가게 된다.

이런 과정을 **리밸런싱**이라고 부르며 주로 아래와 같은 상황에서 발생한다.

1. 컨슈머 그룹에 새로운 컨슈머가 추가 될 때 (+)
2. 기존 컨슈머가 그룹에서 나가게 될 때 (-)
3. 구독하는 토픽에 새로운 파티션이 생길 때
4. 컨슈머가 구독하는 토픽이 변경될 때

리밸런싱이 가장 많이 일어나는 일반적인 상황은 애플리케이션 배포 상황이다.

기존 애플리케이션이 종료 후에 애플리케이션이 실행되면서, 기존 컨슈머가 삭제되고 새로운 컨슈머가 생성되기 때문이다. 이 과정에서 리밸런싱은 최소 두번 이상 발생한다.

리밸런싱은 아래와 같은 문제점을 동반한다.
- **downtime 발생**: 리밸런싱이 진행되는 동안 메시지를 처리하지 않는다. 이로 인해 애플리케이션 지연 시간이 발생한다.
- **메시지 중복/누락 문제**: 리밸런싱 과정에서 파티션의 어느 위치부터 메시지를 읽어야 할지 결정하는데, 이 과정에서 메시지가 중복되거나 누락될 수 있다.

<br>

## Partition Assignment Strategy

파티션 할당 전략(Partition Assignment Strategy)는 카프카 컨슈머가 토픽의 어떤 파티션을 소비할 것인지 결정하는 방식을 의미한다.

이 전략에 따라서 컨슈머와 파티션 간의 관계가 결정되며, 데이터 처리 효율성과 성능에 영향을 미치게 된다.

카프카의 파티션 할당은 두 가지로 나뉘게 되는데 **적극적 리밸런싱**, **협력적 리밸런싱**이 있다.

그리고 이에 따라서 4가지의 파티션 할당 전략이 존재한다.

1. 범위(Range) 파티션 할당 전략
2. 라운드 로빈(RoundRobin) 파티션 할당 전략
3. 스티키(Sticky) 파티션 할당 전략
4. 협력적 스티키(CooperativeSticky) 파티션 할당 전략


### 적극적 리밸런싱(Eager Rebalance)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbjMAGZ%2FbtsC4KsCUrd%2FO2ydf2K2gYhRWvfsuK7GMk%2Fimg.png)

범위, 라운드 로빈, 스티키 할당 전략이 적극적 리밸런싱 할당 전략에 해당하는 방식이다.

리밸런싱이 일어나게 된다면 모든 컨슈머는 아파치 카프카로부터 데이터 수신을 중단하고, 자신들이 가지고 있던 파티션의 그룹 구성을 포기한다.

이 과정에서 모든 컨슈머가 동시에 작업을 멈추는 STW가 발생하게 된다. 이로 인해서 전체 컨슈머 그룹의 데이터 처리가 일시 중단되게 된다.

이 시간동안 컨슈머 그룹은 유휴상태가 되어 메시지를 소비하거나 오프셋 커밋을 허용하지 않지만, 프로듀서의 경우에는 리밸런싱과 무관하게 파티션에 계속해서 쓰기작업을 처리하기 때문에 대기 시간 동안에는 **LAG**가 급격하게 증가하게 되는 문제가 발생한다.

이로 인해서 컨슈머 그룹의 데이터 처리 성능에 일시적인 영향을 미칠 수 있다.

리밸런싱 이후에는 컨슈머들이 그룹에 다시 참여한다. 그리고 새로운 파티션을 할당받게 되며, 이때 주의할 점은 컨슈머들이 **이전에 가졌던 파티션을 반드시 다시 받는다는 보장이 없다는 것이다.**

즉, 리밸런싱 이후에는 컨슈머들이 새로운 파티션을 할당받을 가능성이 크다. 이로 인해서 적극적 리밸런싱 그룹은 그룹의 안정성이나 데이터 처리 성능에 영향을 미칠 수 있으므로 신중하게 사용해야한다.


### 협력적 리밸런싱(Cooperative Rebalance, Incremental Rebalance)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbfNaii%2FbtsC4G4UlwS%2FCgiceJq9OOhzm8rYQttcyK%2Fimg.png)

협력적 스티키 파티션 할당 전략이 해당하는 방식이다. 이 방식은 `Kafka v2.4` 부터 도입되어 꽤나 최근에 나온 진보된 리밸런싱 방식이다.

이 방식은 파티션의 **일부만이 한 컨슈머에서 다른 컨슈머로 이동하는 특징**을 가진다.

이는, 리밸런싱과 직접적인 관련이 없는 다른 카프카 컨슈머들이 데이터 처리를 계속할 수 있도록 하며, 전체 그룹의 성능에 미치는 영향을 최소화한다.

안정적인 파티션 할당 상태를 점진적으로 찾아가는 과정으로, 여러 차례의 리밸런싱을 거친다.

1. 첫 번째 단계에서는 리더가 모든 컨슈머에게 일부 파티션의 소유권을 잃게 될 것임을 알린다.
2. 두 번째 단계에서는 컨슈머는 이러한 파티션의 소유권을 포기하게 된다.
3. 그 후 단계에서는 조정자는 이 고아가된 파티션을 새로운 컨슈머에게 할당한다.

이런 방식은 안정적인 파티션 할당 상태가 이루어질 때까지 몇 번의 반복이 필요하다.

그렇지만, 적극적 리밸런싱에서 발생하는 서비스 중단 상황을 피할 수 있어, 리밸런싱이 많은 시간이 소요되는 큰 규모의 컨슈머 그룹에서는 특히나 중요하게 사용되는 방법이다.

따라서, 협력적 리밸런싱은 전체 그룹의 성능을 유지하면서 파티션 할당을 유연하게 조절할 수 있는 장점이 있다.

하지만, 이 방식을 활용할 때에도 신중하게 접근해야 하며, 그룹의 상황에 따라 적절한 전략을 선택해야한다.
