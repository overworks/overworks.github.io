---
layout: post
title: "유니티 오브젝트의 null 비교 시 유의사항"
categories: [Unity]
tags: [Unity]
comments: true
---

**Update** : null 병합 연산자와 null 조건부 연산자에 대한 서술이 잘못된 점이 있어 수정했습니다. (2019-07-17)

먼저 아래 코드를 봅시다. 임의의 컴포넌트를 Destroy한 후에 null인지 체크하는 간단한 코드입니다. 유니티 버전은 2019.1.6f1, OS는 윈도10 입니다.

{% gist 49b87f57f532e76b630b8383a3e18aed %}

결과는 다음과 같습니다.

![유니티 오브젝트와 닷넷 오브젝트]({{ site.url }}/assets/unity-object-null-is-not-system-object-null.png)

삭제하기는 했으나 해당 변수에 null을 대입하지는 않았기에 닷넷 오브젝트로는 null이 아닌 것으로 표시되고 있는 한편, 유니티 오브젝트로서 null과 비교를 하면 true를 돌려주고 있습니다. 여기서 브레이크포인트를 걸어보면 아래와 같습니다.

![유니티 오브젝트의 fake null]({{ site.url }}/assets/unity-object-fake-null.png)

유니티 오브젝트는 C++로 작성된 네이티브 객체의 래퍼입니다. 이 네이티브 객체는 씬이 변경되거나 Object.Destroy()를 사용하면 제거됩니다만, 그 객체를 C#으로 래핑한 유니티 오브젝트는 가비지 컬렉터가 수집을 완료할 때까지 남아있게 됩니다. 유니티에서는 이 상태를 "fake null"이라고 하고 있습니다. 이런 이유로 UnityEngine.Object 클래스에서는 [같음 연산자(==, !=)](https://docs.microsoft.com/ko-kr/dotnet/csharp/language-reference/operators/equality-operators)를 오버로딩하여 네이티브 객체의 존재 여부까지 판단해서 비교한 후 결과를 돌려주고 있습니다만, 닷넷의 기본 오브젝트로 보았을 때와 결과가 일치하지 않는 문제가 발생합니다.

또다른 문제는 그렇게 네이티브 리소스가 아직 남아있는지 체크하는 과정에 비용이 소모된다는 것입니다. 예를 들어 GetComponent()나 transform 속성 호출이 비싼 작업이라는 것은 잘 알려져있고, 그래서 다음과 같은 코드를 많이 사용하는데, 실제로는 null 체크 비용 때문에 효과가 없었다는 이야기가 있습니다.

```C#
private Transform _cachedTransform;

public Transform cachedTransform
{
    get
    {
        if (_cachedTransform == null)
        {
            _cachedTransform = transform;
        }
        return _cachedTransform;
    }
}
```

실제로 테스트를 해보겠습니다. 다음 두 코드를 서로 다른 게임오브젝트에 붙이고 결과를 보았습니다.

{% gist 62790f972fefac1f4c3322b47e50deb4 %}
{% gist 66c37689ef899b2575e4d6b1db45357b %}

결과는 다음과 같습니다.

![transform 캐싱 속도 비교]({{ site.url }}/assets/transform-caching-comparision.png)

놀랍게도 직접 transform으로 가져오는 것이 30% 정도 더 빠릅니다. ~~이 문제는 이런 직접적인 조건문만이 아니라 [null 결합 연산자(::)](https://docs.microsoft.com/ko-kr/dotnet/csharp/language-reference/operators/null-coalescing-operator)나 [null 조건 연산자(?. 및 ?[])](https://docs.microsoft.com/ko-kr/dotnet/csharp/language-reference/operators/member-access-operators#null-conditional-operators--and-)를 사용하는 경우에도 해당됩니다.~~[^1]

그렇다면 어떻게 하는 것이 좋을까요? 위의 코드와 같은 경우 이미 대입이 되어있는지의 여부를 판단하기 위한 것으로, 딱히 네이티브 리소스의 실재 여부까지 볼 필요는 없습니다. 그러므로 유니티 오브젝트의 오버로딩된 연산자를 피하여 원시 오브젝트로 비교합니다. 다음 코드를 추가하여 비교해보겠습니다.

{% gist c152bf2924c78558ff16edd2553eda2b %}

![transform 캐싱 속도 비교 2]({{ site.url }}/assets/transform-caching-comparision2.png)

거의 3배 가까이 빨라졌습니다. 위와 같은 단순한 캐싱이나, 생성된 이후에는 파괴되지 않을 싱글턴의 구현같은 경우에는 이렇게 유니티 오브젝트가 아닌 닷넷 오브젝트로서 null 비교를 하는 것이 좋습니다. 그렇지 않고 직접 파괴처리를 한다면 반드시 유니티 오브젝트로서 null 체크를 하고, 더 나아가서는 C++에서 그랬듯이 Destroy를 한 후에 해당 변수에 null을 명시적으로 넣어주는 것을 추천합니다.

```C#
if (bullets[i]) // 암시적 null 체크. bullets[i] != null 과 같음
{
    ...

    Destroy(bullets[i]);
    bullets[i] = null;
}
```

### 참고 자료

- [Custom == operator, should we keep it?](https://blogs.unity3d.com/2014/05/16/custom-operator-should-we-keep-it/)
- [Don't Use == null On Unity Objects](https://jacx.net/2015/11/20/dont-use-equals-null-on-unity-objects.html)
- [Possible unintended bypass of lifetime check of underlying Unity engine object](https://github.com/JetBrains/resharper-unity/wiki/Possible-unintended-bypass-of-lifetime-check-of-underlying-Unity-engine-object)

[^1]: [null 결합 연산자(::)](https://docs.microsoft.com/ko-kr/dotnet/csharp/language-reference/operators/null-coalescing-operator)와 [null 조건 연산자(?. 및 ?[])](https://docs.microsoft.com/ko-kr/dotnet/csharp/language-reference/operators/member-access-operators#null-conditional-operators--and-)는 오버로딩된 UnityEngine.Object의 연산자를 사용하지 않기 때문에 네이티브 리소스 확인 비용이 들지 않습니다. 오히려 object.ReferenceEquals()로 비교 후 호출하는 것보다 더 빠릅니다. 대신 destroyed된 유니티 오브젝트에 사용되면 null이 아니라고 판단하게 되고, fake null 상태인 유니티 오브젝트는 MissingReferenceException을 발생시킵니다.
