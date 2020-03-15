---
layout: post
title: "모바일에서의 유니티 셰이더 for 루프 사용시 주의사항"
categories: Unity
tags: [Unity]
comments: true
---
회사에서 디자이너들이 작업하면서 Vetasoft의 [Camera Filter Pack](https://assetstore.unity.com/packages/vfx/shaders/fullscreen-camera-effects/camera-filter-pack-18433)을 사용하였습니다만, 어째서인지 단말에 올리니 제대로 화면이 나오지 않는 문제가 발생하였습니다. 유니티 버전은 2019.3.0f6이었습니다.

![에디터에서의 카메라 필터 화면]({{ site.url }}/assets/for-loop-in-unity-shader-1.png)

결론부터 말하자면 셰이더 코드에서 for 루프를 사용할때 시작 인덱스를 -1로 했을때 문제가 발생했습니다. 후처리 효과를 위해 [보로노이 다이어그램](https://ko.wikipedia.org/wiki/%EB%B3%B4%EB%A1%9C%EB%85%B8%EC%9D%B4_%EB%8B%A4%EC%9D%B4%EC%96%B4%EA%B7%B8%EB%9E%A8)을 그리는데, 아래와 같은 코드가 들어가있습니다.

```C#
for(int j = -1; j <= 1; j ++)
{
    for(int i = -1; i <= 1; i ++)
    {
        float2 b = float2(i, j);

        ...
    }
}
```

위 코드를

```C#
for(int j = 0; j <= 2; j ++)
{
    for(int i = 0; i <= 2; i ++)
    {
        float2 b = float2(i - 1, j - 1);

        ...
    }
}
```

이렇게 바꾼 후 다시 빌드해보니 문제가 해결되었습니다.

![수정 전]({{ site.url }}/assets/for-loop-in-unity-shader-2.jpg)
![수정 후]({{ site.url }}/assets/for-loop-in-unity-shader-3.jpg)
