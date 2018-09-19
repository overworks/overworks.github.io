---
layout: post
title: "그때는 맞고 지금은 틀리다 - 문자열 연결 시에 가장 효율적인 방법은 StringBuilder가 아닐 수도 있다"
categories: Unity
comments: true
---
**StringBuilder.AppendFormat()에 대한 설명에 잘못된 점이 있어 수정했습니다. (2018.9.1)**

많은 분들이 아시듯이 C#의 string은 불변객체입니다. 한번 문자열이 지정되고 난 이후에는 변경이 불가능합니다. Replace()이나 \+= 연산자등은 문자열 내부의 값을 바꾸는 것이 아니라, 새로운 문자열을 생성한 후 반환하는 것입니다. 그래서 C#의 최적화에 관련된 글들을 보면, 문자열 결합이 필요한 경우 결합 연산자 사용을 지양하고, string.Format() 메서드로 포매팅한 문자열을 사용하거나 내부에 버퍼를 가진 StringBuilder 클래스를 사용하는 것을 권장하고 있습니다.

"그러니 문자열을 자주 연결할 일이 있으면 무조건 StringBuilder!"로 모두 해결된다면 참 편하겠습니다만... 때로는 오히려 StringBuilder를 사용하는 것이 더 비효율적일 수 있습니다. 제가 알고 있는 범위 내에서, 문자열 결합에 가장 효율적인 방법을 적어보겠습니다.

## 리터럴 문자열을 결합하는 경우에는 그냥 + 연산자로 연결

리터럴 문자열만을 사용해서 문자열을 결합하는 경우에는 미리 최적화되어 결합된 상태로 반환됩니다. 아무런 코스트 없이 처리되므로 그냥 결합해서 쓰면 됩니다. "처음부터 연결된 문자열을 쓰지 리터럴끼리 결합할 일이 있나?"라고 생각하시는 분이 계실지도 모르겠는데(사실 저도 그랬습니다), 문자열이 너무 길어지거나 가독성을 위해서 적당한 부분에서 잘라서 쓰고 싶은 경우에 쓰면 되겠지요.

```C#
string literal1 = "리터럴 문자열끼리 결합하면 컴파일러가 자동으로 연결해줍니다." +
    " 문장이 너무 길거나 적당히 나눠서 가독성을 높이고 싶을때" +
    " 이런식으로 나누어서 쓰면 한층 보기 좋아집니다.";
string literel2 = "SELECT count(*)" +
    " FROM user" +
    " WHERE level > 5";
```

## 문자열 4개 이하를 연결하는 경우에는 + 연산자나 string.Concat() 사용

string 클래스의 \+ 연산자를 사용한다는 것은 string.Concat() 메서드를 사용하는 것과 같습니다. 유니티에서 프로파일러로 보면 컴파일러가 string.Concat()으로 바꾸어서 처리하고 있다는 것을 알 수 있습니다.

또한, string.Concat() 메서드의 오버로딩 목록을 보면 하나부터 4개의 문자열을 인수로 갖는 것이 있고, 그 이후로는 가변 개수로 받을 수 있게 되어있습니다. 이 string.Concat() 메서드는 미리 만들어진 인수 4개까지는 비관리 코드를 사용하여 빠르게 처리할 수 있도록 구현되어 있고, 그 이후로는 루프를 돌리면서 처리하도록 되어있습니다.

따라서, 리터럴이 아니더라도 문자열 4개까지는 그냥 \+ 연산자 혹은 string.Concat() 메서드를 사용하면 됩니다. 사실 그 이후라도 결합해야할 문자열의 수가 적고 크기가 크지 않다면 \+ 연산자를 사용하는 것이 더 나을 수 있습니다.

단, 이것은 어디까지나 \+ 연산자의 경우로, \+= 연산자를 사용하는 경우에는 그런 도움을 받을 수 없으니 유의하시기 바랍니다.

## StringBuilder

StringBuilder가 일반적인 string 결합보다 추천되는 이유는, 내부에 버퍼를 갖고 있고, 문자열에 변경이 있을때마다 새로운 문자열을 생성하지 않고 버퍼의 내용을 변경하기 때문입니다.

다만 이 설명은 절반만 맞고 절반은 맞지 않습니다. StringBuilder 역시 내부 버퍼의 크기를 넘는 문자열이 추가되면 더 큰 크기의 버퍼를 할당하고,[^1] 기존 버퍼의 내용을 모두 복사하는 식으로 처리를 합니다.[^2] 즉, 경우에 따라서는 string 연결과 마찬가지로 문자열이 변경될때마다 재할당이 일어나고, 기존의 버퍼는 버려져 가비지가 됩니다.

```C#
string appendString1 = "기본 용량 16자가 넘는 문자열을 추가합니다.";
string appendString2 = "한번 더 문자열을 추가합니다.";
string appendString3 = "다시 한번 문자열을 추가합니다.";

// ToString()을 해서 넣는 이유는 나중에 설명합니다.
Debug.LogFormat("string size: {0}, {1}, {2}", appendString1.Length.ToString(), appendString2.Length.ToString(), appendString3.Length.ToString());

StringBuilder sb = new StringBuilder();
int capacity0 = sb.Capacity; // 여기서는 16이라고 나오는데 실제 Capacity는 0입니다. Mono 구현이 그렇게 되어있음.
sb.Append(appendString1);    // 여기에서 처음으로 문자열 크기 만큼의 버퍼 할당이 됩니다.
int capacity1 = sb.Capacity;
sb.Append(appendString2);    // 용량을 초과했으므로 재할당이 일어납니다.
int capacity2 = sb.Capacity;
sb.Append(appendString3);    // 다시 두배로 늘어납니다.
int capacity3 = sb.Capacity;

Debug.LogFormat("capacity0: {0}, capacity1: {1}, capacity2: {2}, capacity3: {3}", capacity0.ToString(), capacity1.ToString(), capacity2.ToString(), capacity3.ToString());
```

![StringBuilder 기본 생성자 사용시 결과]({{ site.url }}/assets/stringbuilder-capacity1.png)

이것을 방지하기 위해서는 생성할 때 미리 충분한 크기의 영역을 확보해두면 됩니다. 예상되는 최종 출력 문자열의 크기를 미리 계산하여, 생성자에 넣어주거나 Capacity 속성을 통해 영역을 확보해 주면 재할당이 일어나지 않습니다.

```C#
string appendString1 = "기본 용량 16자가 넘는 문자열을 추가합니다.";
string appendString2 = "한번 더 문자열을 추가합니다.";
string appendString3 = "다시 한번 문자열을 추가합니다.";

Debug.LogFormat("string size: {0}, {1}, {2}", appendString1.Length.ToString(), appendString2.Length.ToString(), appendString3.Length.ToString());

StringBuilder sb = new StringBuilder(128); // 미리 버퍼를 할당해둡니다.
int capacity0 = sb.Capacity;
sb.Append(appendString1);    // 문자열을 추가해도 재할당이 일어나지 않습니다.
int capacity1 = sb.Capacity;
sb.Append(appendString2);
int capacity2 = sb.Capacity;
sb.Append(appendString3);
int capacity3 = sb.Capacity;

Debug.LogFormat("capacity0: {0}, capacity1: {1}, capacity2: {2}, capacity3: {3}", capacity0.ToString(), capacity1.ToString(), capacity2.ToString(), capacity3.ToString());
```

![StringBuilder 할당 생성자 사용시 결과]({{ site.url }}/assets/stringbuilder-capacity2.png)

[^1]: Mono에서는 기존버퍼의 크기 2배와 필요한 버퍼크기 중에서 더 큰 쪽으로 잡습니다.
[^2]: .NET 4.0 이후부터는 방식이 바뀌어서 새로운 StringBuilder를 만들고 연결 리스트로 연결하는 식으로 처리를 합니다. 따라서 기존의 버퍼가 버려지지 않는 대신, ToString() 메서드로 출력할때의 오버헤드는 더 커졌습니다.

## StringBuilder.AppendFormat()과 string.Format()

string.Format은 내부적으로 StringBuilder 객체를 생성하여 처리하도록 되어있고, StringBuilder.AppendFormat()은 반대로 string.Format(정확히는 string.Format()이 호출하는 내부 메서드)을 호출하는 식으로 구현되어 있습니다. ~~즉, StringBuilder.AppendFormat()은 StringBuilder 객체를 하나 더 생성하게 되고, 직접 하나씩 Append()로 넣는 것보다 훨씬 큰 오버헤드가 발생합니다.~~[^3] [^4]

또, 이러한 포매팅 문자열을 사용하는 메서드들은 인수타입이 object로 되어 있습니다. 즉, 값 타입을 사용할 경우 자연스럽게 박싱이 발생하게 됩니다.[^5] 이런 오버헤드를 줄이기 위해서는 넣기 전에 미리 ToString() 메서드를 호출하여 참조 타입으로 변경하는 것이 좋습니다.

```C#
int gold = 500;
int ruby = 20;

string targetString1 = string.Format("gold: {0:D2}", gold); // 박싱이 발생하는 좋지 않은 사용법
string targetString2 = string.Format("gold: {0}", gold.ToString("D2")); // ToString하면서 같이 서식도 지정해주는 식으로

string targetString3 = $"ruby: {ruby}"; // C# 6에서 추가된 문자열 보간 사용. 여기서도 박싱이 발생.
string targetString4 = $"ruby: {ruby.ToString()}"; // 이곳에서도 마찬가지로 미리 변환해줌
```

[^3]: 기존 문서에 오류가 있었습니다. string.Format()과 StringBuilder.AppendFormat() 사용시 string.FormatHelper()라는 내부 메서드를 통해 StringBuilder.Append() 처리를 하는데, string.Format() 사용시에는 StringBuilder 객체를 생성하나 StringBuilder.AppendFormat()은 자기자신을 넘겨주기 때문에 새로운 객체가 생성되지는 않습니다.
[^4]: 위 주석의 설명은 유니티에서 사용하는 모노의 구현방식으로, 닷넷에서는 StringBuilder.AppendFormat()에서 string.FormatHelper()으로 넘겨주는 것이 아니라 StringBuilder.AppendFormat()에서 바로 처리하고, string.Format()도 StringBuilder.AppendFormat()로 넘겨줍니다. 최신버전의 모노에서도 닷넷과 같은 방식으로 바뀌었습니다.
[^5]: StringBuilder.Append()는 대부분의 내장타입이 오버로드되어있어 이런 문제가 발생할 가능성이 적습니다. 물론 enum이나 struct를 사용하면 같은 문제가 발생하게 됩니다.

## 벤치마크

그럼 간단하게 벤치마크를 해보겠습니다. 테스트 환경은 유니티 2017.1.3p3, Windows 10이고, 코드는 다음과 같습니다.

```C#
public class StringTest : MonoBehaviour
{
  const string string0 = "문자열0";
  const string string1 = "문자열1";
  const string string2 = "문자열2";
  const string string3 = "문자열3";
  const string string4 = "문자열4";
  const string string5 = "문자열5";
  const string string6 = "문자열6";
  const string string7 = "문자열7";
  const string string8 = "문자열8";
  const string string9 = "문자열9";

  private void Update()
  {
    ConcatTest1();
    ConcatTest2();
    StringBuilderTest1();
    StringBuilderTest2();
    StringFormatTest();
  }

  void ConcatTest1()
  {
    // 리터럴 연결
    string target = string0 + string1 + string2 + string3 + string4 + string5 + string6 + string7 + string8 + string9;
  }

  void ConcatTest2()
  {
    // 리터럴 연결이 되지 않도록 미리 변수에 대입해둠
    string str0 = string0;
    string str1 = string1;
    string str2 = string2;
    string str3 = string3;
    string str4 = string4;
    string str5 = string5;
    string str6 = string6;
    string str7 = string7;
    string str8 = string8;
    string str9 = string9;
    string target = str0 + str1 + str2 + str3 + str4 + str5 + str6 + str7 + str8 + str9;
  }

  void StringBuilderTest1()
  {
    // 미리 할당하지 않은 경우
    StringBuilder sb = new StringBuilder();
    sb.Append(string0);
    sb.Append(string1);
    sb.Append(string2);
    sb.Append(string3);
    sb.Append(string4);
    sb.Append(string5);
    sb.Append(string6);
    sb.Append(string7);
    sb.Append(string8);
    sb.Append(string9);
    string target = sb.ToString();
  }

  void StringBuilderTest2()
  {
    // 사용전에 미리 할당한 경우
    StringBuilder sb = new StringBuilder(64);
    sb.Append(string0);
    sb.Append(string1);
    sb.Append(string2);
    sb.Append(string3);
    sb.Append(string4);
    sb.Append(string5);
    sb.Append(string6);
    sb.Append(string7);
    sb.Append(string8);
    sb.Append(string9);
    string target = sb.ToString();
  }

  void StringFormatTest()
  {
    string target = string.Format("{0}{1}{2}{3}{4}{5}{6}{7}{8}{9}", string0, string1, string2, string3, string4, string5, string6, string7, string8, string9);
  }
}
```

결과는 다음과 같습니다.

![문자열 연결 벤치마크 결과]({{ site.url }}/assets/string-concat-profiling.png)

리터럴 결합은 가비지나 오버헤드가 전혀 발생하고 있지 않습니다. 그 이외에는 StringBuilder에 미리 적당한 크기로 할당해서 사용한 경우가 가장 가비지가 적었습니다. \+ 연산자(string.Concat() 메서드)은 10개를 결합했는데 StringBuilder를 사용한 경우와 속도는 비슷하고, 가비지는 좀 더 많았습니다. 그 다음이 미리 할당하지 않았을때의 StringBuilder이고, string.Format은 속도든 가비지 생성이든 가장 좋지 못했습니다.

## 요약

 1. 리터럴 문자열끼리 연결할 때에는 오버헤드가 없다.
 2. 4개 이하의 문자열을 연결할때는 \+ 연산자를 사용하자. 그보다 조금 많아도 괜찮다.
 3. StringBuilder를 사용할때에는 미리 사용할 크기를 지정해두자.
 4. 문자열 포매팅은 가장 비효율적인 방법이다. 굳이 사용할 때에는 미리 ToString()을 해서 박싱이 발생하지 않도록 하자.