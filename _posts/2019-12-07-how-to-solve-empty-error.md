---
layout: post
title: "유니티에서 아무것도 표시되지 않는 에러가 발생하는 현상 수정"
categories: Unity
comments: true
---
언제부터인가 유니티를 켰는데 다음과 같이 아무것도 표시되지 않는 에러가 발생하면서 플레이가 되지 않는 현상이 발생하였습니다. 기존의 프로젝트만이 아니라 새로 만든 프로젝트에도 같은 현상이 발생하였습니다.

![설명이 적혀있지 않은 컴파일 에러]({{ site.url }}/assets/unity-empty-compile-error.png)

이 문제를 해결하는 방법은 다음과 같습니다. (윈도우 기준)

1. 텍스트 에디터로 C:\Program Files\Unity\Hub\Editor\2019.1.8f1\Editor\Data\Tools\RoslynScripts\unity_csc.bat 파일 열기[^1]
2. 2번째 줄 `"%APPLICATION_CONTENTS%\Tools\Roslyn\csc" /shared %*`에서 `csc`를 `csc.exe`로 수정 후 저장

이제 해당 유니티 프로젝트를 다시 열면 정상적으로 컴파일 될 것입니다.

[^1]: Program Files 안이므로 관리자 권한으로 열어야 합니다.
