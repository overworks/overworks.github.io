---
layout: post
title: ".NET과 유니티에서의 가비지 관리"
categories: Unity
comments: true
---
C/C++에서는 메모리 할당은 전적으로 프로그래머 담당이었습니다. C에서는 malloc, C++에서는 new를 통해 메모리를 할당한 후에는 반드시 free와 delete로 삭제를 해주어야 했고, 그러지 않으면 메모리 누수가 발생했습니다. 또한 메모리 할당과 해제를 반복하다보면 사용할 수 있는 공간이 띄엄띄엄 배치되어 정작 필요한 공간을 확보할 수 없는 파편화(fragmentation)가 발생합니다. C#과 Java 등의 현대적인 언어가 C/C++보다 우월한 점중의 하나는 쓰레기 수집기가 있어 메모리 관리에 신경쓸 필요성이 **상대적으로** 적다는 것일 것입니다.[^1]

하지만 그렇다고 완전히 메모리 관리에서 손을 놓아도 된다는 의미가 아니며, 주의해야 할 부분이 존재합니다. 개인적인 공부를 겸하여 그런 부분들을 정리해보았습니다.

[^1]: C++에서도 std::shared_ptr 등의 스마트 포인터를 통한 레퍼런스 카운팅 기반의 쓰레기 수집을 지원하고, 일부 C++ 구루들은 "아직도 스마트 포인터를 사용하지 않고 원시 포인터 쓰다가 메모리 누수를 발생시키는 건 얼간이나 할 짓이다"라고 이야기하기도 합니다.

## .NET에서의 가비지 관리

### 쓰레기 수집 작업 순서

닷넷에서는 다음과 같은 순서로 쓰레기를 수집합니다.

1. 시스템의 메모리가 부족하거나, 관리되는 힙에 할당되는 메모리가 임계치를 넘어서거나, [System.Gc.Collect()](https://msdn.microsoft.com/ko-kr/library/system.gc.collect(v=vs.110).aspx)를 호출
2. 현재 수행중인 모든 쓰레드가 정지되고, 평소에는 잠들어있는 GC 쓰레드가 활성화됨
3. GC 쓰레드가 힙 상에서 사용중인 객체를 조사하여 표시(mark)하고, 그래프를 생성
4. 사용되지 않는 객체를 제거(sweep)하고 사용중인 객체를 옮기고 참조 위치를 업데이트(compact)
5. 현 세대에서의 수집만으로 충분한 메모리를 확보할 수 없는 경우 다음 세대의 힙에서 수집 처리

### 세대기반

.NET에서는 관리되는 힙을 0~2까지 3개의 세대로 나누어서 관리합니다. 처음 객체가 힙에 할당되면 0세대로 분류됩니다. 대부분의 객체는 수명이 길지 않으므로, 모든 객체를 한번에 검사하기보다는 만들어진지 얼마 안된 것들만 모아서 한번 거르는 것이 효과적입니다.

그렇게 거르고 난 후 남은 객체들은 1세대 힙으로 옮겨집니다. 그렇게 점점 1세대 힙의 크기가 커지고, 0세대 힙의 쓰레기 수집(GC 0)만으로는 메모리를 확보할 수 없는 경우 1세대 힙이 GC의 대상이 됩니다(GC 1). 그리고 여기서도 살아남은 1세대 객체는 2세대로 넘어갑니다.

GC 1 이후에도 메모리가 부족할 경우에는 2세대 힙의 GC가 발생합니다(GC 2). 3세대 이상의 힙은 존재하지 않으므로, GC 2는 모든 관리되는 메모리를 검사하는 것이며, 그만큼 부하도 큽니다.

### 컴팩트화

사용되지 않는 객체의 공간을 회수하고 난 후에는 힙에 남은 객체들을 앞쪽으로 옮겨서 사용가능한 메모리를 확보합니다. 동시에 그 객체들의 참조도 업데이트해 줍니다. 이 과정을 컴팩트화(compaction)라고 합니다.[^2] 따라서 .NET에서는 기본적으로 메모리 파편화가 발생하지 않아야 합니다...만 아쉽게도 그렇지만은 않습니다.

[^2]: MSDN의 한국어 번역에서는 '압축'이라는 단어를 사용하고 있습니다.

### 대형객체 힙

이상은 일반적인 객체가 저장되는 SOH(Small Object Heap)에서의 설명이고, 85000바이트 이상의 대형객체는 LOH(Large Object Heap: 대형객체 힙)에 저장됩니다. 이곳에 들어가는 대형객체는 처음부터 2세대로 간주되며, 일반적으로 성능 이슈로 인해 컴팩트화하지 않습니다.[^3] 따라서 C/C++에서의 동적 메모리 할당과 마찬가지로 파편화가 발생할 수 있고, 2세대로 간주되는 만큼 더 오랫동안 남아있게 되며, 수집할 때 0,1세대 힙도 같이 처리되게 됩니다.

[^3]: .NET 4.5.1 이후로는 GCSettings.LargeObjectHeapCompactionMode으로 컴팩트화 설정을 할 수 있습니다.

## 유니티에서는?

유니티의 관리 힙은 .NET과는 달리 세대구분이 없고 컴팩트화도 하지 않습니다. 따라서 필수적으로 파편화가 발생하게 되며, 쓰레기 수집 시 힙 전체를 검사합니다. 게다가 관리 힙이 한번 커지면 좀처럼 줄어들지 않습니다. 따라서 쓰레기 수집이 한번 발생하면 매우 많은 시간을 소모하게 되며, 이것은 유저 경험에 악영향을 끼칩니다.

예를 들어보겠습니다. 초당 60프레임으로 움직이는 게임의 한 Update() 함수에서 매번 1KB의 임시 객체를 생성한다고 합시다. 이러면 매초 60KB, 1분이면 3.5MB의 메모리가 필요하게 됩니다. .NET이었다면 자주 해제되고 부하도 크지 않은 0세대 힙에 소속될테니 좀 낫겠지만, 유니티에서는 그런거 없으니 유저들은 주기적으로 끊기는 화면을 보게 될 수 있습니다.

그러므로 프로그래머는 최대한 가비지 생성을 억제하고, 객체 풀 등을 통해 적극적으로 자원을 재사용해야 합니다.

### 프로파일러를 이용한 가비지 체크

유니티에 내장되어 있는 프로파일러를 통해 가비지가 얼마나 쌓이고 있는지 쉽게 체크할 수 있습니다. 유니티 메뉴에서 Window/Profiler를 누르거나 단축키 Ctrl+7를 누르면 프로파일러 창이 뜨고, 상단의 Deep Profile 버튼을 눌러 활성화시킵니다. 그 후 CPU Usage에 포커스를 두면 아래에 호출스택과 힙에 할당된 메모리가 표시됩니다.

## 주요 사례

### 박싱/언박싱

C#의 객체는 값(value) 타입과 참조(reference) 타입으로 나뉘고, 값 타입을 참조 타입으로 변환할때 박싱, 참조 타입을 값 타입으로 바꿀때 언박싱이 일어나며, 이 과정은 느리니 가급적 지양하는 것이 좋다...는 것은 상식이니 많은 분들이 알고 계실 것이고, 또한 피하고 있으리라 생각합니다. 캐스팅 자체의 속도도 있고, 힙에 할당을 하면서 가비지도 발생시킵니다. 문제는 프로그래머는 박싱을 피한다고 했는데, 자신도 모르는 사이에 내부적으로 박싱/언박싱을 하는 경우가 있다는 것입니다.

#### Dictionary에서 enum이나 struct를 key로 사용하는 경우

Dictionary에서는 key를 검증하기 위해 Object.GetHashCode()를 호출하는데, key 타입은 값 타입이므로 이 때 박싱이 일어납니다. 이것은 Add(), TryGetValue(), ContainsKey() 등 key 값을 사용하는 모든 메서드에서 벌어지고, 그 때마다 가비지가 생성되게 됩니다. 이 것을 막기 위해서는 System.Collections.Generic 네임스페이스 안의 IEqualityComparer\<T\> 인터페이스를 상속받는 클래스를 만들고, 그 인스턴스를 Dictionary를 생성할 때 넣어주어야 합니다.[^4] ([참고링크](https://ejonghyuck.github.io/blog/2016-12-12/dictionary-struct-enum-key/))

[^4]: 아니면 key 타입으로 int를 사용해도 되지만 커스텀 비교자를 사용하는 것보다는 느리다고 합니다.

#### foreach

"유니티에서 가급적이면 foreach 루프를 쓰지 말라"는 것도 꽤나 잘 알려져있는 팁입니다. foreach를 사용하면 루프를 종료하고 Enumerator를 해제하면서 박싱이 일어납니다.[^5] 속도도 수동으로 for 문을 돌리는 것보다 훨씬 느리지만, 그래도 전혀 쓰지 않을 수는 없는지라 쓰면서도 무언가 찝찝함을 느끼게 만들곤 했습니다.

사실, 이 문제는 유니티 5.5 이후로는 어느 정도 해결되어, 예전보다는 훨씬 적은 가비지를 생성합니다. Dictionary, HashSet, LinkedList, Queue, Stack은 처음 루프를 돌릴 때에만 가비지가 생성되고, 특히 가장 자주쓰이는 List에서 가비지가 아예 사라졌습니다! ([참고링크](https://jacksondunstan.com/articles/3805)) 여전히 속도는 느리지만 업데이트 루프와 같은 곳이 아니라면 써주어도 큰 문제가 되지는 않을 것이라 생각합니다.

[^5]: 원인을 설명하는 영상: 유니티에서 foreach를 쓰면 안된다는게 정말인가요? [전편](https://www.youtube.com/watch?v=41syxzusX0w) [후편](https://www.youtube.com/watch?v=WgEz6DutNkM)

#### string.Format() 등의 포맷 문자열 메서드

string.Format()이나 Debug.LogFormat() 등의 포맷 문자열 메서드는 객체의 값을 문자열로 바꾼 후 끼워넣어서 출력합니다. C의 printf()와 비슷한 느낌으로 많이 사용하는 메서드입니다만, 이 메서드는 object를 인수로 받기 때문에 값 타입을 바로 집어넣으면 박싱이 발생합니다. 어차피 내부에서 ToString()을 호출하므로, 값 타입을 넣을 것이라면 미리 ToString()을 해서 넣는 것이 좋습니다.

```C#
int i = 5;
string str1 = string.Format("bad usage: {0}", i);
string str2 = string.Format("good usage: {0}", i.ToString());
```

### 메서드 참조와 클로저

작업을 하다보면 콜백을 위해서 대리자를 통한 메서드의 참조값을 넘기는 경우가 많고, System.Action 등 미리 정의된 값을 쓰기도 합니다. C#에서 메서드 참조는 모두 레퍼런스 타입이며, 힙 할당이 일어납니다. 이것은 실제 델리게이트를 사용한 코드의 예시를 보면 쉽게 이해할 수 있습니다.

```C#
delegate int Del(string s);

static int DelFunc(string s)
{
    return int.Parse(s);
}

static void UseDel(Del del, string s)
{
    int res = del(s);
    Console.WriteLine(res);
}

static void Main(string[] args)
{
    Del del = new Del(DelFunc);  // 여기서 힙 할당이 일어남
    UseDel(del, "24");

    // 위 코드의 약식. new 키워드가 없어서 잊어버리기 쉽지만 내부적으로는 힙 할당을 하고 있음
    UseDel(DelFunc, "24");
}
```

이것은 무명 메서드와 람다에도 모두 적용되는 사항입니다. 따라서 메서드를 인수로 넘길 때에는 반드시 힙 할당이 일어나며, 사용이 끝난 후에는 가비지가 된다는 것을 기억해야 합니다.

또한 클로저를 사용하면 외부의 값을 캡처하기 위해 익명 클래스를 만들고, 더 많은 메모리를 힙에 할당합니다. 따라서 가비지를 줄이기 위해서는 메서드 참조, 특히 클로저의 사용을 피하는 것이 좋습니다.

```C#
public class MyClass
{
    public int id;
    ....
}

List<MyClass> myList;

public MyClass FindClass1(int id)
{
    // 클로저. 대량의 가비지가 생성된다.
    return myList.Find(c => c != null && c.id == id);
}

public MyClass FindClass2(int id)
{
    // 직접 풀어서 쓴 루프. 가비지가 생성되지 않는다.
    for (int i = 0; i < myList.Count; ++i)
    {
        MyClass c = myList[i];
        if (c != null && c.id == id)
        {
            return c;
        }
    }
    return null;
}
```

물론 성능만큼이나 가독성은 중요한 요소이고, 모든 유틸리티 메서드를 무시하고 죄다 풀어서 쓰는 것도 바람직하지 않습니다. 하지만 적어도 Update() 등 매 프레임마다 호출되는 곳에서는 클로저를 사용하지 말아야 합니다.

### 문자열 결합

여러개의 문자열을 결합하는 경우 일반적으로 StringBuilder 클래스의 사용을 권장합니다. string 타입은 불변(immutable)이므로, 문자열에 변경이 있는 경우 힙에 새로운 문자열을 생성하고 기존의 값은 가비지가 됩니다. 따라서 미리 버퍼를 확보하고 변경할 수 있는 StringBuilder 클래스를 사용하면 가비지의 생성을 줄일 수 있다…고 설명하는데, 언제나 그런 것은 아닙니다. 상황에 따라서는 + 연산자를 사용한 문자열 결합이 훨씬 더 효율적일 수 있습니다.

자세한 것은 [이 글](http://www.simpleisbest.net/post/2013/04/24/Review-StringBuilder.aspx)에서 설명하고 있는데, 요약하면 다음과 같습니다.

1. 문자열 상수(리터럴)를 결합하는 경우 그냥 +로 연결하면 컴파일러가 미리 연결된 문자열을 생성해준다.
2. 4개 이하의 문자열을 생성하는 경우에도 StringBuilder를 사용하기보다는 string.Concat이나 + 연산자를 사용하는 것이 더 효과적이다.
3. StringBuilder를 사용할 경우에는 미리 크기를 예상하여 생성할때 초기 버퍼 크기를 넣어줄 것.

추가로 테스트한 결과, string.Format()과 StringBuilder.AppendFormat()은 가능하면 사용하지 않는 것이 좋다는 결론을 내렸습니다. string.Format()은 내부적으로 StringBuilder 인스턴스를 생성하여 처리하고 있으며, StringBuilder.AppendFormat()은 string.Format()을 사용하고 있습니다. StringBuilder를 사용할 경우에는 Append()만을 이용하는 것이 가장 효율이 좋습니다. 이에 관해서는 차후 다른 글에서 이야기할 예정입니다.

## 참고링크

- [가비지 컬렉션 기본 사항](https://docs.microsoft.com/ko-kr/dotnet/standard/garbage-collection/fundamentals)
- [닷넷 가비지 컬렉션 다시 보기](http://www.simpleisbest.net/post/2011/04/01/Review-NET-Garbage-Collection.aspx)
- [관리되는 힙에 대한 이해(Understanding the managed heap)](https://docs.unity3d.com/kr/current/Manual/BestPracticeUnderstandingPerformanceInUnity4-1.html)