---
layout: post
title: "ddclient를 통한 Cloudflare DNS 설정 업데이트"
categories: Web
tags: [Web, Cloudflare, DNS, ddclient]
comments: true
---
[ddclient](https://ddclient.net/)를 사용하면 지원하는 DNS 서버 정보를 자동으로 업데이트할 수 있습니다. 저는 [Cloudflare](https://www.cloudflare.com/)를 사용중이므로 이에 맞춰서 설정을 해보겠습니다.

직접 빌드를 해서 사용을 해도 되겠지만 왠만한 리눅스 배포판에서는 패키지 제공을 하므로 그것을 가져오겠습니다. [이 곳](https://github.com/ddclient/ddclient?tab=readme-ov-file#installation)을 참조하여 자신이 사용중인 배포판에서 제공하는 버전을 확인합니다.

ddclient 버전이 3.10 이상인 경우, Global API Key 대신 API Token을 사용할 수 있습니다. Cloudflare 대시보드에서 프로필 - API 토큰 페이지로 이동하여 Zone.DNS 권한을 가진 API 토큰을 생성합니다. 생성 후에 안내해주기도 합니다만, 한번 표시된 이후에는 다시는 볼 수 없으니 잘 적어둡니다. 만약 ddclient 버전이 Global API Key를 사용해야 합니다. Global API Key 보기 버튼을 클릭하여 키를 확인할 수 있습니다.

리눅스로 돌아와서 ddclient를 설치합니다.
```
sudo apt install ddclient
```

그럼 초기설정 UI가 뭔가 이것저것 물어볼텐데, 어차피 나중에 설정파일을 바꿀 것이니 cloudflare만 설정하고 대충 넘깁니다. 설정이 끝난 후 /etc/ddclient.conf 파일을 수정합니다.

```
protocol=cloudflare, \
use=web, web=ipify-ipv4, \
zone=your.domain, \
login=token, \
password=api.token \
your.domain,subdomain.your.domain
```
- protocol : cloudflare
- zone : 갱신할 도메인
- login : API Token을 사용한다면 token, API Key를 사용한다면 Cloudflare 관리자 메일 주소
- password : API Token 혹은 API Key

위 설정과 함께 마지막에 comma로 구분된 도메인을 입력합니다.

설정을 마쳤다면 ddclient를 실행하여 제대로 작동하는지 확인합니다.
```
sudo ddclient
```
