---
layout: post
title: "유니티의 그래픽스 티어 구분"
categories: Unity
tags: [Unity]
comments: true
---
## 유니티의 기본 구분

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
|   3  | 자동으로 설정되지 않음  |

### 데스크탑

| 티어 |   |
|------|---|
|   1  | DirectX 9 이전 |
|   2  |  |
|   3  | OpenGL, Vulkan[^1], Metal, DirectX 11 이후 |

[^1]: 데스크탑용 Vulkan을 사용시 티어 3지만 모바일용 Vulkan은 티어 2.

## 모바일에서의 티어 3 설정

[Graphics.activeTier](https://docs.unity3d.com/ScriptReference/Graphics-activeTier.html)로 체크 및 전환가능. 모바일에서는 기본적으로 티어3으로 설정되지 않지만 해당 API를 통해 수동으로 올려줄 수 있음. 단, 셰이더에 영향이 가므로 셰이더를 로드하기 전에 처리해야함.

```C#
// 아이폰X 이후의 아이폰은 티어3로 변경
var deviceID = SystemInfo.deviceModel;
if (deviceID == "iPhone10,3" || deviceID == "iPhone10,6" ||
    deviceID.indexOf("iPhone11") >= 0 || deviceID.indexOf("iPhone12") >= 0)
{
    Graphics.activeTier = GraphicsTier.Tier3;
}
```

iOS 기기의 기기코드는 [이 곳](https://gist.github.com/adamawolf/3048717)을 참조.

## 참고

[Tiers in GraphicSettings](https://forum.unity.com/threads/tiers-in-graphicsettings.485408/)
