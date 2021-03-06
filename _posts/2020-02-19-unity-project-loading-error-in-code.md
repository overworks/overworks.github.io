---
layout: post
title: "비주얼 스튜디오 코드에서 유니티 2019.3의 프로젝트가 제대로 로딩이 되지 않는 현상"
categories: Unity
tags: [Unity]
comments: true
---
예전에는 유니티 작업을 하면서 비주얼 스튜디오를 사용했었습니다만, 맥에서 작업을 하게 되면서 비주얼 스튜디오 코드를 코드 에디터를 사용하게 되고, 어느 정도 익숙해진 이후에는 작업 환경 통일을 위해 윈도에서도 코드를 메인 에디터로 사용하게 되었습니다.

그런데 유니티의 버전을 2019.3으로 올린 이후로 프로젝트를 제대로 로딩하지 못하는 문제가 발생하였습니다. Unity UI를 포함한 수많은 모듈을 로딩하지 못하여 각종 코드에 에러 표시가 뜨고, 인텔리센스도 작동하지 않아 작업에 엄청난 지장이 발생했습니다.

이것은 비주얼 스튜디오 코드용 프로젝트 파일을 작성하는 Visual Studio Code Editor 패키지가 원인으로, 이 글을 쓰고 있는 2019년 2월 19일 현재 2019.3 verified로 되어있는 1.1.4에서 프로젝트 파일을 작성할때 프로젝트 내부가 아닌 유니티가 설치된 경로에서 어셈블리를 참조하도록 만드는 버그가 있습니다. 이것이 해결되기 전까지는 다른 에디터를 쓰시거나 1.1.3으로 낮춰서 쓰시기 바랍니다.

![packages/manifest.json 파일 수정]({{ site.url }}/assets/vscode-editor-setting-in-package-manager.png)
![packages/manifest.json 파일 수정]({{ site.url }}/assets/vscode-editor-setting-in-text-editor.png)
