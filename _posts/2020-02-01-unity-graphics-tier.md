---
layout: post
title: "유니티의 그래픽스 티어"
categories: Unity
tags: [Unity]
comments: true
---
## 티어 구분 및 설정

유니티에서는 구동하는 하드웨어에 따라 자동적으로 티어를 3단계로 구분하여 내장 셰이더의 성능을 조절합니다. 각 티어의 셰이더 품질은 Project Settings의 Graphics 페이지에서 확인하고 수정할 수 있으며, Edit - Graphics Tier 메뉴를 통해 에디터에서 시뮬레이션할 수 있습니다.

![셰이더 품질 설정 페이지]({{ site.url }}/assets/unity-graphics-tier-settings.png)

![에디터에서의 하드웨어 티어 설정]({{ site.url }}/assets/unity-hardwear-tier.png)

유니티의 티어 자동 구분 조건은 아래와 같습니다.

### 안드로이드

| 티어 |                                    |
|------|------------------------------------|
|   1  | OpenGL ES 2.0만 지원하는 모든 기기 |
|   2  | OpenGL ES 3.0을 지원하는 모든 기기 |
|   3  | 자동으로 설정되지 않음             |

### iOS

| 티어 |   |
|------|---|
|   1  | 5 이전의 모든 아이폰 (5C 포함), 4세대 이전의 아이패드, 아이패드 미니 1세대, 5세대 이전의 아이팟 |
|   2  | 5S 이후의 모든 아이폰, 아이패드 에어 이후, 2세대 이후의 아이패드 미니, 6세대 이후의 아이팟, 애플TV |
|   3  | 자동으로 설정되지 않음 |

### 데스크탑

| 티어 |                |
|------|----------------|
|   1  | DirectX 9 이전 |
|   2  |                |
|   3  | OpenGL, Vulkan[^1], Metal, DirectX 11 이후 |

## 모바일에서의 티어 3 설정

[Graphics.activeTier](https://docs.unity3d.com/ScriptReference/Graphics-activeTier.html)로 현재 티어 상태 체크 및 전환이 가능합니다. 모바일에서는 기본적으로 티어3으로 설정되지 않지만 해당 API를 통해 수동으로 올려줄 수 있습니다. 단, 티어 전환은 셰이더에 영향이 가므로 셰이더를 로드하기 전에 미리 처리해야 합니다.

```C#
using UnityEngine;
using UnityEngine.Rendering;
#if !UNITY_EDITOR && UNITY_IOS
using UnityEngine.iOS;
#endif

...

public static void TryChangeToGraphicsTier3()
{
#if !UNITY_EDITOR && UNITY_IOS
    // 아이폰X 이후의 iOS 기기는 티어3로 변경
    var generation = Device.generation;
    if (DeviceGeneration.iPhoneX <= generation && generation < DeviceGeneration.iPhoneUnknown)
    {
        Graphics.activeTier = GraphicsTier.Tier3;
    }
#endif
}
```

## 참고

[Tiers in GraphicSettings](https://forum.unity.com/threads/tiers-in-graphicsettings.485408/)

[Apple_mobile_device_types.txt](https://gist.github.com/adamawolf/3048717)

[^1]: 데스크탑용 Vulkan을 사용시 티어 3지만 모바일용 Vulkan은 티어 2.
