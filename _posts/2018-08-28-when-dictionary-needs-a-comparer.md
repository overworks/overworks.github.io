---
layout: post
title: "그때는 맞고 지금은 틀리다 - 제네릭 컬렉션도 박싱이 발생할 수 있다"
categories: unity
comments: true
---
## 제네릭 컬렉션에서 박싱이 발생할 때

일반적으로 제네릭 컬렉션의 장점으로 값 타입을 사용해도 박싱이 발생하지 않는다는 것을 꼽습니다. 제네릭이 아닌 컬렉션은 object 타입만을 받고, 값 타입의 컬렉션을 사용하면 박싱-언박싱이 일어나서 퍼포먼스에 영향을 주므로, 가급적이면 제네릭 컬렉션을 사용하는 것을 추천합니다.

하지만 제네릭 컬렉션을 사용하기만 하면 그런 문제를 모두 회피할 수 있는 것은 아닙니다. Dictionary나 HashSet 등, 해시함수를 사용하는 컬렉션은 사용법에 따라 같은 문제가 발생할 수 있습니다. 그럼 어떤 경우에 그렇게 되는지 알아보겠습니다.

테스트 코드는 다음과 같습니다.

```C#
using System.Collections.Generic;
using UnityEngine;

public class IntDictionary : MonoBehaviour
{
    private Dictionary<int, string> dict = new Dictionary<int, string>();

    private void Start()
    {
        for (int i = 0; i < 5; ++i)
        {
            dict.Add(i, i.ToString());
        }
    }

    private void Update()
    {
        string str;
        for (int i = 0; i < 5; ++i)
        {
            dict.TryGetValue(i, out str));
        }
    }
}
```

```C#
using System.Collections.Generic;
using UnityEngine;

public enum Enums
{
    Enums0,
    Enums1,
    Enums2,
    Enums3,
    Enums4,

    Max
}

public class EnumDictionary : MonoBehaviour
{
    Dictionary<Enums, string> dict = new Dictionary<Enums, string>();

    private void Start()
    {
        for (int i = 0; i < 5; ++i)
        {
            dict.Add((Enums)i, i.ToString());
        }
    }

    private void Update()
    {
        string str;
        for (int i = 0; i < 5; ++i)
        {
            dict.TryGetValue((Enums)i, out str);
        }
    }
}
```

```C#
using System.Collections.Generic;
using UnityEngine;

public struct Struct
{
    public int value;

    public Struct(int value)
    {
        this.value = value;
    }
}

public class StructDictionary : MonoBehaviour
{
    private Dictionary<Struct, string> dict = new Dictionary<Struct, string>();

    private void Start()
    {
        for (int i = 0; i < 5; ++i)
        {
            dict.Add(new Struct(i), i.ToString());
        }
    }

    private void Update()
    {
        string str;
        for (int i = 0; i < 5; ++i)
        {
            dict.TryGetValue(new Struct(i), out str);
        }
    }
}
```

![기본 Dictionary 생성자 사용시 프로파일링 결과]({{ site.url }}/assets/dictionary-garbage-default.png)

IntDictionary에서는 문제가 없으나, EnumDictionary와 StructDictionary에서는 매프레임 300바이트, TryGetValue() 호출 한번당 60바이트의 가비지가 생성되고 있습니다. 속도도 두배 정도 느립니다. 박싱이 발생하고 있다는 것을 예측할 수 있습니다.

![기본 Dictionary 생성자 사용시 비교자]({{ site.url }}/assets/dictionary-garbage-inner.png)

내부를 보면 int를 key로 받았을때에는 GenericEquilityComparer를 사용하고 있고, 그 외에는 DefaultComparer를 사용하고 있습니다. 그럼 비교자가 다른 것이 원인일까요?

[ILSpy](https://github.com/icsharpcode/ILSpy)로 유니티의 Mono 구현 내부를 까보겠습니다. ILSpy로 유니티 에디터 설치 폴더/Editor/Data/Mono/lib/mono/unity/mscorlib.dll 파일을 엽니다. GenericEquilityComparer는 System.Collection.Generic 네임스페이스에 있고, DefaultComparer는 EquiltyComparer 클래스에 내부 클래스로 구현되어 있습니다.

![DefaultComparer 구현]({{ site.url }}/assets/default-comparer.png)

![GenericEqualityComparer 구현]({{ site.url }}/assets/generic-equality-comparer.png)

코드는 별 차이가 없습니다. 비교자 자체가 원인은 아닌것 같군요.

문제는 비교자에서 사용하는 두 메서드, object.Equals()와 object.GetHashCode()입니다. int가 System.Int32에 대응이 되듯, struct는 System.ValueType 클래스에, enum은 System.Enum 클래스에 대응되는데, 두 클래스 모두 Equals()와 GetHashCode()의 인수로 object를 받습니다.[^1] 즉, 인수를 넘기는 순간 박싱이 일어난다는 이야기입니다.

[^1]: GetHashCode() 자체에는 인수가 없으나, 내부의 비공개 메서드로 넘길때 object 타입을 사용합니다.

## 해결방법

이제 원인을 알았으니 문제를 해결해봅시다.

### 방안 1. key로 내장타입만을 사용한다

내장타입은 모두 object만이 아니라 자기자신을 인수타입으로 하는 메서드가 구현되어 있습니다. key로 enum을 쓰고, 중복되는 값이 없을 경우 처음부터 int를 key로 설정하고 사용할때마다 int로 캐스팅해서 사용하면 박싱이 발생하지 않습니다. 하지만 enum 타입이 아니거나 int로 캐스팅하기 어려운 struct의 경우에는 사용하기 어렵습니다.

### 방안 2. 자기자신을 인수타입으로 하는 비교 및 해시함수를 구현한다

struct 타입 선언시에 System.IEquatable\<T\> 인터페이스를 받고 Equals()를 구현하고, GetHashCode()를 오버라이딩합니다. 당연히 박싱을 일으키지 않는 형태로 구현되어야 합니다. 위의 StructDictionary 스크립트에서 사용된 Struct 타입을 다시 구현해보겠습니다.

```C#
using System;
using System.Collections.Generic;

public struct Struct : IEquatable<Struct>
{
    public int value;

    public Struct(int value)
    {
        this.value = value;
    }

    public bool Equals(Struct other)
    {
        return value == other.value;
    }

    public override int GetHashCode()
    {
        return value.GetHashCode();
    }
}
```

이렇게 하면 더이상 박싱이 발생하지 않습니다. 하지만 이 역시 enum 타입에서는 쓸 수 없다는 단점이 있습니다.

### 방안 3. IEqualityComparer\<TKey\>를 구현하고 Dictionary를 생성할때 같이 넘긴다

위에서 EqualityComparer.DefaultComparer나 GenericEquilityComparer가 사용된 것은 Dictionary가 생성될 때 비교자를 넘기지 않았기 때문입니다. IEqualityComparer\<T\> 인터페이스를 구현하는 커스텀 비교자를 만들어 생성자 인수로 넘기면 기본 비교자 대신 이것을 사용하게 됩니다. 이 방법을 통해 EnumDictionary와 StructDictionary 스크립트를 다시 작성해보겠습니다.

```C#
public class EnumDictionary : MonoBehaviour
{
    private class EqualityComparer : IEqualityComparer<Enums>
    {
        public bool Equals(Enums x, Enums y)
        {
            return x == y;
        }

        public int GetHashCode(Enums obj)
        {
            return ((int)obj).GetHashCode();
        }

        public static EqualityComparer Default = new EqualityComparer();
    }

    Dictionary<Enums, string> dict = new Dictionary<Enums, string>(EqualityComparer.Default);

    private void Start()
    {
        for (int i = 0; i < 5; ++i)
        {
            dict.Add((Enums)i, i.ToString());
        }
    }

    private void Update()
    {
        string str;
        for (int i = 0; i < 5; ++i)
        {
            dict.TryGetValue((Enums)i, out str);
        }
    }
}
```

```C#
public class StructDictionary : MonoBehaviour
{
    private class EqualityComparer : IEqualityComparer<Struct>
    {
        public bool Equals(Struct x, Struct y)
        {
            return x.value == y.value;
        }

        public int GetHashCode(Struct obj)
        {
            return obj.value.GetHashCode();
        }

        public static EqualityComparer Default = new EqualityComparer();
    }

    private Dictionary<Struct, string> dict = new Dictionary<Struct, string>(EqualityComparer.Default);

    private void Start()
    {
        for (int i = 0; i < 5; ++i)
        {
            dict.Add(new Struct(i), i.ToString());
        }
    }

    private void Update()
    {
        string str;
        for (int i = 0; i < 5; ++i)
        {
            dict.TryGetValue(new Struct(i), out str);
        }
    }
}
```

![커스텀 비교자 사용시 프로파일링 결과]({{ site.url }}/assets/dictionary-garbage-custom.png)

이제는 더이상 박싱이 발생하지 않고, 가비지도 사라졌습니다.

## 세줄요약

1. 값 타입을 key로 하는 해시 컬렉션 사용시 박싱이 일어날 수 있다
2. 원인은 Equals()와 GetHashCode() 메서드 호출시 인수 타입이 object라서
3. 이것을 막기 위해서는 IEqualityComparer\<T\> 인터페이스를 구현한 커스텀 비교자를 만들어서 생성시 넘겨줄 것
