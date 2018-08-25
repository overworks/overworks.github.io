---
layout: post
title: "그때는 맞고 지금은 틀리다 - enumerator 대신 foreach를 사용하자"
categories: unity
comments: true
---
# 더이상 유니티에서는 foreach 시에 가비지를 생성시키지 않는다

인터넷의 유니티 관련 최적화 팁을 검색하다 보면 루프를 돌리는 경우 foreach를 피하라는 글이 자주 보입니다. [이 글](http://geekcoders.tistory.com/56)을 보면 for와 foreach, enumerator를 사용한 루프의 프로파일링 결과를 통해 가급적이면 for를 사용하고, for를 사용하지 못하는 경우에는 enumerator를 사용하여 루프를 돌릴 것을 권장하고 있습니다.

하지만 이 내용은 이제 더 이상 유효하지 않습니다. 현재 제가 사용하고 있는 유니티 2017.3에서 같은 소스로 프로파일링을 해보겠습니다.

```C#
public class ForUpdate : MonoBehaviour
{
    List<string> list = new List<string>(50);

    private void Start()
    {
        for (int i = 0; i < 50; ++i)
        {
            list.Add(i.ToString());
        }
    }

    private void Update()
    {
        for (int i = 0; i < 50; ++i)
        {
            string str = list[i];
        }
    }
}
```

```C#
public class ForEachUpdate : MonoBehaviour
{
    List<string> list = new List<string>(50);

    private void Start()
    {
        for (int i = 0; i < 50; ++i)
        {
            list.Add(i.ToString());
        }
    }

    private void Update()
    {
        foreach (string s in list)
        {
            string str = s;
        }
    }
}
```

```C#
public class EnumeratorUpdate : MonoBehaviour
{
    List<string> list = new List<string>(50);

    private void Start()
    {
        for (int i = 0; i < 50; ++i)
        {
            list.Add(i.ToString());
        }
    }

    private void Update()
    {
        var enumerator = list.GetEnumerator();
        while (enumerator.MoveNext())
        {
            string str = enumerator.Current;
        }
    }
}
```

그리고 프로파일링을 해본 결과는 다음과 같습니다.

![foreach_vs_enumerator_1]({{ site.url }}/assets/foreach_vs_enumerator_1.png)

보시다시피 더이상은 리스트에 foreach를 사용해도 가비지가 생성되지 않습니다. 그리고 여전히 아주 적은 차이이지만 foreach가 enumerator보다 조금 느립니다.

# foreach 대신 enumerator를 권장했던 이유

사실 foreach는 enumerator를 사용한 루프의 syntax sugar에 불과합니다. 제대로 IEnumerator 인터페이스를 구현했다면, enumerator를 사용하나 foreach를 사용하나 같은 결과가 나와야합니다. 그렇다면 왜 이런 차이가 발생했을까요? 내부를 한번 더 까보면 그 차이를 알 수 있습니다.

![foreach_vs_enumerator_2]({{ site.url }}/assets/foreach_vs_enumerator_2.png)

보시다시피 foreach 내부에서 enumerator를 Dispose() 해주고 있습니다. foreach는 enumerator가 IDisposable을 구현했을때 Dispose() 처리를 해주도록 되어있습니다. 따라서 올바른 foreach의 구현은 다음과 같이 되어야합니다.

```C#
var enumerator = strs.GetEnumerator();
try
{
    while (enumerator.MoveNext())
    {
        string str = enumerator.Current;
    }
}
finally
{
    if (enumerator is System.IDisposable)
    {
        enumerator.Dispose();
    }
}
```

이에 맞춰서 EnumeratorUpdate.Update()를 바꾸고 다시 한번 프로파일링해본 결과[^1], ForEachUpdate와 EnumeratorUpdate가 엎치락뒤치락하는 것을 알 수 있었습니다.

![foreach_vs_enumerator_3]({{ site.url }}/assets/foreach_vs_enumerator_3.png)

[^1]: 이때만 경과시간이 길어졌는데, 50개만 넣으니 차이를 알기 어려워서 이 스크린샷을 찍을 때에 5000으로 늘렸습니다.

모바일(갤럭시 노트2)에서도 테스트한 결과도 같았습니다.

![foreach_vs_enumerator_mobile]({{ site.url }}/assets/foreach_vs_enumerator_mobile.png)

사실, 과거의 유니티에서 foreach를 돌릴때마다 가비지가 생성되는 원인도 바로 여기에 있었습니다. Dispose()를 호출할때 값 타입인 List\<T\>.Enumerator()을 참조 타입인 박싱하면서 가비지도 발생하고 느려지는 것입니다.[^2] 이 문제는 유니티 5.5(5.6일 수도 있습니다)에서 수정되어 최신버전에서는 더 이상 발생하지 않습니다.

[^2]: [이 영상](https://www.youtube.com/watch?v=WgEz6DutNkM)을 참조하시기 바랍니다.

# 결론

enumerator를 사용하면서 Dispose() 과정을 제외시킨다면 여전히 foreach를 사용하는 것보다 더 빠를 것입니다. 하지만 예외안정성을 생각하면 enumerator 대신 foreach 루프를 사용하는 것을 추천하고 싶습니다.