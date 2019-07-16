---
layout: post
title: "Unity - CharacterController에 대하여"
categories: Unity
tags: [Unity]
comments: true
---

[CharacterController 개요](https://docs.unity3d.com/Manual/CharacterControllers.html)
[CharacterController 매뉴얼 레퍼런스](https://docs.unity3d.com/Manual/class-CharacterController.html)
[CharacterController 스크립트 레퍼런스](https://docs.unity3d.com/ScriptReference/CharacterController.html)

CharacterController는 물리법칙보다 과장된 캐릭터의 움직임을 위해 만들어진 특수한 클래스입니다. 이 것은 1인칭 혹은 3인칭 캐릭터의 이동을 위해 사용되며, 경사나 계단을 오르내리는 등의 행동을 간단하게 처리할 수 있습니다.

CharacterController를 캡슐 콜라이더와 같은 모습을 지니나, 일반적인 콜라이더와는 사용하는 방법이 다릅니다. CharacterController 컴포넌트가 붙어있는 게임오브젝트는 OnControllerColliderHit 메시지를 통해 충돌한 오브젝트에 힘을 가할 수 있으나, 다른 오브젝트로부터 힘을 받았을 때에는 직접 계산을 하거나, CharacterController를 사용하지 않고 Rigidbody를 사용하여야합니다.

CharacterController는 게임오브젝트의 이동시에 Move() 혹은 SimpleMove()를 사용합니다. Transform.position을 사용하면 다른 충돌체를 통과해버리거나 할 수 있으므로 일반적인 이동시에는 사용하지 말아야합니다. Move()와 SimpleMove()를 사용하면 자동으로 그 앞에서 멈추게 됩니다.

SimpleMove()와 Move()의 차이는 Y축 값의 처리여부입니다. SimpleMove()는 인수로 넣은 Vector3의 Y값이 무시되며, 자동으로 중력이 계산되어 처리됩니다. 반면 Move()를 호출하는 경우에는 직접 중력 등을 계산하여 Y축 위치를 결정해야합니다. 만약 점프 등 Y축과 관련된 움직임이 필요하다면 Move()를 사용하고, 2차원인 움직임만 필요하다면 SimpleMove()를 사용하는 것이 간단합니다.