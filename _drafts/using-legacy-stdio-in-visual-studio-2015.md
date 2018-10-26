---
layout: post
title: "비주얼 스튜디오의 디버그 모드에서 파일입출력이 제대로 되지 않을때"
categories: "Visual Studio"
comments: true
---
비주얼 스튜디오 2008에서 작업하던 프로젝트를 2015로 마이그레이션하면서, 외부 라이브러리에서 다음과 같은 링크 에러가 발생했습니다.

```Visual Studio
error LNK2001: _vsprintf 외부 기호를 확인할 수 없습니다.
error LNK2001: __snprintf 외부 기호를 확인할 수 없습니다.
```

이 문제를 해결하려면 프로젝트 속성 페이지를 열어서 링커 - 입력 - 종속성에 legacy_stdio_definitions.lib를 추가해주면 됩니다.

참고 - [Visual C++ 2015의 주요 변경 내용](https://msdn.microsoft.com/ko-kr/library/bb531344.aspx)