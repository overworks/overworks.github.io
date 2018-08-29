---
layout: post
title: "그때는 맞고 지금은 틀리다 - StringBuilder보다 문자열 연결이 나을때"
categories: unity
comments: true
---
- string
  - 리터럴 문자열 결합 시에는 미리 결합된 상태로 처리됨
  - 결합 연산자는 기본적으로 Concat()으로 변경됨
  - 문자열 4개까지는 한번에 Concat() 전부 결합되고, 그보다 많으면 루프를 돌면서 결합되게 됨
- string.Format()
  - 내부적으로 StringBuilder를 생성해서 사용
- StringBuilder
  - 내부에 string이 있고, 미리 할당된 크기 이상을 추가하는 경우 내부 string보다 큰 사이즈(보통 기존 string의 2배)의 영역을 할당 후 새로운 string을 생성한 후 복사
  - 루프에서 계속해서 Append 시 재할당이 일어나고 가비지 행
  - 재할당이 일어나지 않도록 하려면 처음 생성시에 완성될 문자열의 크기를 어느 정도 예상하여 미리 할당해둬야 함. 그러지 않으면 그냥 string 결합하는 것과 차이가 없음(O(N)과 O(LogN)의 차이는 있겠지만)
  - StringBuilder.AppendFormat()은 내부적으로 string.Format()을 사용 후 붙이는 형태. 즉 StringBuilder 내에서 새로운 StringBuilder 객체가 새로 생성되게 됨
- 결론
  - 리터럴 문자열 뿐이라면 그냥 결합 연산자 사용.
  - 4개까지라면 결합 연산자 혹은 Concat() 사용
  - 그 이상은 StringBuilder 사용. 단, 미리 할당크기를 정해둘 것.
