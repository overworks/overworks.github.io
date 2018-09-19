---
layout: post
title: "비주얼 스튜디오의 디버그 모드에서 파일입출력이 제대로 되지 않을때"
categories: "Visual Studio"
comments: true
---
가끔씩 비주얼 스튜디오의 디버그 모드에서 파일 입출력을 하려고 할때 파일을 찾지 못하는 경우가 있습니다. 정작 빌드를 하고 나서 실행시키면 정상적으로 잘 읽어오는데 말이죠.

이것은 출력 디렉터리와 디버그 모드 시의 경로가 다르기 때문입니다. 프로젝트 속성 페이지 창을 열어서 디버깅 탭의 작업 디렉토리를 보면 기본적으로 작업 디렉터리 항목이 비어있거나 프로젝트 디렉터리로 설정되어 있습니다. 이것을 $(TargetDir)로 바꿔주면 빌드 후 실행할 때와 같은 경로로 설정되어 파일을 찾지 못하는 문제가 사라집니다.

[!디버그 모드 작업 디렉터리 설정]({{ site.url }}/assets/debugging-working-dir.png)

참고 - [C++ 디버그 구성에 대한 프로젝트 설정](https://msdn.microsoft.com/ko-kr/library/kcw4dzyf.aspx)