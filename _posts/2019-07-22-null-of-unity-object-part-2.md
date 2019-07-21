---
layout: post
title: "유니티 오브젝트의 null 비교 시 유의사항 2"
categories: [Unity]
tags: [Unity]
comments: true
---

이전글: [유니티 오브젝트의 null 비교 시 유의사항](https://overworks.github.io/unity/2019/07/16/null-of-unity-object.html)

[지난번 글](https://overworks.github.io/unity/2019/07/16/null-of-unity-object.html)에서는 유니티 오브젝트의 fake null과, ==과 != 연산자가 오버로딩되어 있으며 그 때문에 null 비교할 때 일반적인 닷넷 오브젝트보다 많은 비용이 들어간다는 것, 그리고 그것을 피하기 위해서는 object.ReferenceEquals()를 사용하면 된다는 것을 이야기했습니다.

이번에는 잠깐 그 오버로딩된 연산자가 어떻게 구현되어 있는지 보겠습니다. 닷넷 디컴파일러 [ILSpy](https://github.com/icsharpcode/ILSpy)로 유니티 스크립트 어셈블리를 열어[^1] UnityEngine.Object를 찾아보면 다음과 같이 CompareBaseObjects()를 호출하고 있다는 것을 알 수 있습니다.

![UnityEngine.Object 연산자 오버로딩]({{ site.url }}/assets/unity-object-operator-overloading.png)

그 내용은 다음과 같습니다.

![UnityEngine.Object.CompareBaseObjects()]({{ site.url }}/assets/unity-object-comparebaseobjects.png)

내용은 그리 복잡하지 않으니 이해하시는데 크게 어렵지 않을 것이라 생각합니다. 먼저 양변이 모두 null이면 그대로 돌려주고, 둘 다 null이 아니면 인스턴스 ID로 비교, 어느 한쪽만 null일 때 네이티브 객체 체크에 들어갑니다. 거기서도 포인터와 MonoBehaviour 혹은 ScriptableObject가 아닌지 한 번 더 체크한 후 네이티브 함수를 호출합니다.

즉, 가장 비용이 비싼 경우는 한쪽만 null일때이며, 그렇지 않은 경우에는 함수 점프 두번과 캐스팅 비용 정도가 들어갑니다. 이미 해당 변수에 인스턴스가 대입이 된 상태라고 해도, 많이들 쓰는 싱글턴 패턴 같은 경우에는 MonoBehaviour이니 네이티브 호출까지는 가지 않을 것입니다. 저도 Transform과 null 비교가 transform을 직접 호출하는 것보다 비싸다는 사실에 놀라 호들갑을 떤 느낌이 있습니다만, 이걸 보면 그렇게까지 비싸다고 할 건 아닌가하는 생각이 들기도 합니다. 물론 이것도 object.ReferenceEquals()만 호출하는 것보다는 확실히 비싼 작업이긴 합니다.

사실 위의 내용은 그냥 분량채우기고(...) 하고 싶은 이야기는 이제부터입니다. 지난번에 주석으로 가볍게 짚고 넘어간 내용입니다만, null 병합 연산자 ??와 C# 6.0 문법에 추가된 null 조건 연산자(?., ?[])는 유니티 오브젝트의 오버로딩된 연산자를 통하지 않기에 fake null 체크를 하지 않습니다. 사실, 속도로만 따지자면 object.ReferenceEquals()로 null 비교 후 메서드를 호출하는 것보다 미세하게 더 빠릅니다. 그럼 비용 걱정없이 마구 사용해도 되겠구나 생각하실지도 모르겠는데, 이 상황에서 빠지기 쉬운 함정이 하나 더 있습니다.

바로 **public이나 SerializeField로 선언되어 인스펙터에서 노출된 후, 대입되지 않은 유니티 오브젝트는 fake null 상태로 들어온다**는 것입니다.

다음 코드를 게임오브젝트에 붙인 후, public 변수 cam에 아무것도 넣지 않고 디버깅을 해보겠습니다.

{% gist fdb5a29ed991ffff4b77543560644841 %}

![인스펙터 상태]({{ site.url }}/assets/null-test-gameobject-inspector.png)

![디버깅 결과]({{ site.url }}/assets/null-test-inspector-debugging.png)

보시다시피 object.ReferenceEquals()는 빠져나가고, 유니티 오브젝트의 null 체크에서 걸리는 fake null입니다. 멤버 선언할때 초기값으로 null을 주었지만 전혀 영향을 주지 못합니다.

이 사실을 알지 못한채로 null 병합 연산자나 null 조건 연산자를 사용하면 재앙의 근원이 될 수 있습니다. 예를 들어보겠습니다. 내부에서 사용할 카메라를 외부에 공개하고, 대입되지 않으면 메인 카메라를 사용하는 코드가 있다고 해봅시다. 만약에 다음과 같은 코드라면 어떻게 될까요?

{% gist 450704d5587430725f273f16d8f0c4bd %}

이 코드에서는 절대로 ?? 연산자의 우변을 가져가지 않습니다. 인스펙터에서 사용할 카메라를 넣해두었다면 괜찮겠지만, 그렇지 않다면 fake null을 참조하게 되어 사용할 때 예외가 발생하게 됩니다. 원래 의도대로 사용하고 싶다면 결국 유니티의 연산자나 암시적 bool 변환을 사용하여야 합니다.

{% gist e2d49d4e5e7da55323bfc06a22150205 %}

이 글을 쓰면서 제가 하고 싶은 말은, "속도를 위해 object.ReferenceEquals()를 써라" 혹은 "조금 느리지만 안정성을 위해 유니티 오브젝트의 비교 혹은 암시적 bool 변환을 이용하라"가 아니라, fake null이라는 개념과 그것이 왜 생겼으며 그것이 미치는 영향을 이해하여야 한다는 것입니다. 그러지 못하면 처음 설명했듯이 한쪽은 null, 한쪽은 null이 아닌 상황에 혼동을 겪을 수 있고, 위와 같은 사이드 이펙트가 발생할 수도 있습니다. 결국 모든 것에 통용되는 은탄환은 없으며, "그때는 맞고 지금은 틀린 상황"이 얼마든지 있다는 것을 유념하시기 바랍니다.

[^1]: 해당 어셈블리의 경로는 유니티 에디터/Data/Managed/UnityEngine.dll 입니다.

### 덧글

사실 이렇게 헷갈리게 만든 유니티가 제일 욕을 먹어야한다고 생각하지만 그쪽은 또 그 나름대로의 고충이 있을테고, 어떻게든 사용자측에서 좀 더 이해하기 쉽게 할 필요가 있다 생각합니다. 제 생각으로는 확장 메서드를 사용하여 별도의 명칭을 만들어 null 체크를 하는 것이 바람직하지 않나 싶습니다.

{% gist d2fe2f16c3e8c0b325619f6c92a68ed0 %}
