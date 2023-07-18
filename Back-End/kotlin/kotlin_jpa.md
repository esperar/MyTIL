# Kotlin, JPA가 서로 지향하는 방향의 차이

코틀린과 JPA가 서로 다른 방향을 지향하는 것을 알고 있는가 
  
어째서 코틀린과 JPA는 맞지 않는 방향을 추구하고 있는지에 대해서 알아보겠다.

## 코틀린이 추구하는 방향

코틀린은 **불변성**을 지향하며, 변경 가능한 상태를 최소화하여 코드의 안정성을 높이려 한다.
  
이를 위해 `val 키워드`를 사용하여 변수를 선언하고, 데이터 클래스를 사용하여 불변 데이터 모델을 정의할 수 있다.

## JPA가 추구하는 방향

반면에 JPA는 객체와 데이터베이스 간의 매핑을 중심으로 설계되었으며, **데이터베이스의 상태를 변경**하는 것이 주요 목적이다.
  
따라서 JPA는 엔티티 클래스를 사용하여 변경 가능한 데이터 모델을 정의하고, `EntityManager`를 사용하여 데이터베이스와 상호작용한다.

## 결론

코틀린과 JPA는 객체의 변화 상태에 관해서 지향하는 방향이 다르다.
  
각각의 장단점을 고려해서 사용해야 하는 것이 좋겠다. -> 도메인과 엔티티의 분리