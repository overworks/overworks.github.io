---
layout: post
title: "유니티 스크립트 템플릿 수정하기"
categories: [Unity]
tags: [Unity, Tips]
comments: true
---
유니티 에디터에서 새로운 C# 스크립트를 생성하면 다음과 같은 형태의 코드가 생성됩니다.

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class <스크립트명> : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {

    }

    // Update is called once per frame
    void Update()
    {

    }
}
```

네임스페이스를 변경하거나 메서드를 추가하는 등[^1] 템플릿을 변경하고 싶다면 <유니티 설치폴더\>/Editor/Data/Resources/ScriptTemplate/81-C# Script-NewBehaviourScript.cs.txt 파일을 열고 수정하면 됩니다. 단 관리자 계정으로 열어야 수정 후 저장할 수 있으며, 리눅스 방식의 줄바꿈(LF만 사용)을 하고 있으므로 기본 메모장이 아닌 코드 수정용의 텍스트 에디터를 사용하여야 합니다.

그 외에도 해당 폴더에는 테스트나 셰이더 등 여러가지 템플릿이 들어있으니 상황에 맞춰서 변경하여 사용하면 됩니다.

[^1]: 저 같은 경우 클래스를 저만의 네임스페이스로 감싸고, 기본 주석을 지우고, Awake() 메서드를 추가해서 사용하고 있습니다.