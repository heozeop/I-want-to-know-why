---
title: "[회고] next step ATDD with Spring 2주차"
date: 2024-02-12T19:07:19+09:00
draft: false

categories:
- experience
tags:
- next step ATDD with Spring
keywords:
- JAVA
- Spring
- ATDD
---

# TL;DR
- 인수 테스트와 함께 TDD를 함께 해보는 작업 진행
- 순서를 가진 데이터의 구현 팁
- 외부 의존성을 도메인 객체에서 활용하는 방법

# 배운 것
## 나는 평소에 outside in 테스트를 하고 있었다.

### inside out/outside in
인수 테스트를 이용해 테스트를 작성하는 방식으로 inside out, outside in 방법이 있습니다.
inside out 방법은 도메인 객체부터 차근 차근 테스트를 쌓아 올리는 방식이고, outside in은 그 반대입니다.
outside in 테스트의 과정은 웹프로젝트에서 아래와 같이 진행되었습니다.

1. 컨트롤러를 작성한다.
1. 서비스를 작성한다.
1. 서비스에 대한 유닛 테스트를 작성하며 작업한다. 이때, dependency는 Mock 객체로 대신한다.
1. 코드 작성을 마무리한다.

이러한 과정을 가진 outside in 방식의 단점은 아래와 같았습니다.
- 장점
    1. fake 객체를 이용해 항상 성공하는 테스트를 만들 수 있다.
    1. 현재의 지식을 기반으로 빠르게 테스트를 작성할 수 있다.
- 단점
    1. 데이터 구조의 변경등 작은 변경들에 영향을 크게 받는다.
    1. 테스트에 통과하더라도 실제 상황에서 오류가 안나리라는 보장이 없다.
    1. 자칫 서비스가 두꺼워질 수 있다.

이에 반해 inside out의 테스트는 도메인을 먼저 설계한다는 측면에서 차이점이 있었습니다.
inside out 방식의 테스트 과정은 아래와 같습니다.

1. 도메인을 설계한다.
1. 테스트를 쌓아 올리며 도메인을 구현한다.
1. 서비스와 컨트롤러를 차례로 쌓아 올린다.

그 결과, 아래와 같은 장단점을 지닌다고 생각했습니다.
- 장점
    1. 견고한 도메인을 구현할 수 있다.
    1. 의존의 방향성이 도메인을 향하도록 서비스를 구현할 수 있다.
- 단점
    1. 도메인 이해도가 적다면 초기 설계가 어렵다.

### inside out 방식을 연습하자.
이번 과정을 통해 제가 평소 outside in방식으로 테스트를 짜왔다는 것을 알게 되었습니다.
두 방식을 비교하였을때, inside out 방식이 상황에 따라 더 좋은 경우가 많을 것 같다고 생각하였습니다. 
또한 숙련도의 측면에서 inside out 방식을 연습하는게 이번 과정에서 얻어갈 수 있는 핵심이라는 생각이 들었습니다.

결과적으로, 저는 앞으로 남은 과정에서 inside out 방식의 설계를 해보려고 합니다.


## 순서가 지켜진다고 가정하자.
### 지하철 노선 미션
이 내용의 설명을 위해서는 미션을 잠시 소개할 필요가 있습니다.
'노선'에는 여러개의 '구간'이 존재합니다. 이 '구간'은 상행역과 하행역 정보를 가지고 있습니다.
미션은 노선의 처음/끝/가운데 구간에 역을 추가/삭제하는 것입니다.

저는 미션을 진행하며 구간의 순서를 저장할 때 지켜주지 않고, 추후 조회할때 정렬하는 방법을 생각했습니다.
해당 접근은 결과적으로 구간의 삭제시 코드의 복잡도를 높이는 결과를 초래했습니다.
아래는 제가 짰던 코드의 일부를 발췌한 내용입니다.

```JAVA
public void removeStation(Station station) {
    final Optional<Section> sectionWithUpStation = sections.stream().filter(section -> section.getUpStation() == station).findAny();
    final Optional<Section> sectionWithDownStation = sections.stream().filter(section -> section.getDownStation() == station).findAny();

    if (sectionWithUpStation.isEmpty() && sectionWithDownStation.isEmpty()) {
        throw new EntityNotFoundException();
    }

    if (sectionWithUpStation.isPresent()) {
        filterSection(sectionWithUpStation.get());
    }

    if (sectionWithDownStation.isPresent()) {
        filterSection(sectionWithDownStation.get());
    }

    if(sectionWithUpStation.isPresent() && sectionWithDownStation.isPresent()) {
        addSection(
            sectionWithDownStation.get().getUpStation(),
            sectionWithUpStation.get().getDownStation(),
            sectionWithUpStation.get().getDistance() + sectionWithDownStation.get().getDistance()
        );
    }
}
```

제가 생각한 이 코드의 문제점은 여기서 어떤 케이스를 왜 처리하는지 알 수 없다는 점입니다.
if 구문들을 보았을 때 어떤 경우를 처리하는지 알기 위해서는 기획을 알아야 할 필요가 있습니다.
저는 이 부분이 마음에 안들었고, 다른 풀이를 찾아보기 시작했습니다. 그렇게 다른 분의 풀이를 찾게 되었습니다.
기존 제 코드 보다 맥락을 훨씬 더 잘 알 수 있다는 느낌이 들었습니다. 참고하여 리펙토링한 결과는 아래와 같습니다.

```java
public void removeStation(Station station) {
    if (isFirstStation(station)) {
        remove(getFirstSection());
        return;
    }

    if(isLastStation(station)) {
        remove(getLastSection());
        return;
    }

    removeStationInTheMiddle(station);
}
```

### 핵심은 순서가 유지된다는 가정

이 설계의 핵심은 노선내 구간의 순서가 항상 유지 된다는 가정을 추가한 것이었습니다.
첫번째 역을 찾을때 0번 구간을 보면 되고, 마지막 역을 찾을때 n-1번 구간을 찾으면 됩니다. 이렇듯 가정 하나를 추가하여 직관적인 코드를 생산해 낸 것입니다. 

저도 앞으로 순서가 유지되어야 하는 경우, 순서가 항상 유지된다는 가정도 우선 고려해 봐야겠습니다.

## 외부 의존성을 도메인 객체에서 활용하는 법
### 빠른 경로 찾기 미션
이번 미션은 빠른 경로찾기 미션이었습니다. 역의 아이디 2개가 주어졌을 때, 2개 역 간의 가장 빠른 경로를 찾아내는 것입니다.
이 로직을 구현하기 위해서는 다잌스트라를 활용할 수 있습니다. 이걸 직접 구현할 수도 있겠지만, 강의의 목적에 따라 라이브러리를 사용하기로 하였습니다.
그렇다면 라이브러리로 사용하였을 때, 어떻게 도메인에서 이용하게 만들어야 할까요?

### 의존성 역전하기
저는 그 방법으로 interface와 abstract class를 고려했습니다.
외부 라이브러리에 대한 의존성을 역전시켜 도메인이 외부 라이브러리 변경에 취약하지 않도록 만들고 싶어 해당 방법을 택했습니다.
이중 abstract class를 이용했는데, 요구 조건에 validation 로직을 다른 구현체에서도 사용해야 할 것이라는 생각 때문이었습니다.

하지만 이 방법 역시 서비스 단에서는 구현체를 생성해 할당한다는 점에서 완벽하게 OCP나 DIP를 지킨 것은 아니었습니다.
배움이 짧아 아쉽지만 다음에 좀 더 학습해서 좋은 구조를 만들어 봐야겠다고 생각하였습니다.

# 느낀 것
## 객체 지향적으로 설계할 수 있겠다는 기대감
### 기존에는
저는 서비스에 모든 validation과 로직을 때려박는 사람이었습니다.
그렇게 하면 빠르게 구현은 가능했기 때문에, 단점을 외면하였습니다.
결과적으로 도메인 레이어가 서비스에 강하게 결합된 코드를 만들었고, 저조차도 수정이 꺼려지는 상황이 되었습니다.

### 앞으로는
좀 더 객체 지향적으로 구현해볼 수 있겠다는 생각이 많이 들었습니다.
물론 레거시 코드를 유지보수하는 단계에서는 적용이 어렵겠으나, 새로운 프로젝트를 진행하면서는 객체지향 설계가 가능할 것이라는 기대감이 듭니다.

## ATDD 기반 테스트를 짤 수 있겠다는 자신감
### 기존에는
기존에는 해봐야 mock 객체를 이용한 unit 테스트만 잔뜩 만들었습니다.
결과적으로 테스트는 유기 되었고, 테스트에 대한 회의감 마져 들었던 상황이었습니다.

### 앞으로는
몇 번의 연습을 통해 ATDD를 이용해 요구사항을 정리하고 테스트를 구현해 볼 수 있겠다는 자신감이 스멀스멀 올라옵니다.
기대만큼 잘되었으면 좋겠네요!

# 총평
## ATDD 만족도
### 2주차
- 만족


