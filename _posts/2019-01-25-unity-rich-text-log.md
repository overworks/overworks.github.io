---
layout: post
title: "유니티 로그에서 리치 텍스트 사용"
categories: Unity
tags: [Unity, Tips]
comments: true
---
의외로 모르시는 분이 많은데, 유니티에서 Debug.Log()를 사용하여 로그를 남길때, 마크업 태그를 사용하여 색이나 크기, 형태를 지정할 수 있습니다.

{% gist 08bb322d8c076a218195c49da82bac8e %}

![리치 텍스트 로그]({{ site.url }}/assets/unity-rich-text-log.png)

이 기능은 UI(uGUI)나 레거시 GUI 시스템 등에서도 사용할 수 있습니다.

더 자세한 설명과 사용가능한 마크업 태그에 관해서는 [유니티 공식 문서](https://docs.unity3d.com/kr/current/Manual/StyledText.html)를 참조하시기 바랍니다.