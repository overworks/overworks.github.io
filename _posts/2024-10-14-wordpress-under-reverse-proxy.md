---
layout: post
title: "리버스 프록시 안쪽에서 워드프레스 http로 연결하기"
categories: Web
tags: [Web, WordPress, PHP]
comments: true
---
https 연결은 http로 연결하는 것보다 느리기 때문에, 리버스 프록시 서버 안쪽은 http로 연결하는 경우가 많습니다. 하지만 워드프레스의 경우에는 설정에 따라 다시 https로 리다이렉트를 시키면서 무한히 리다이렉트되는 경우가 있습니다.

이것을 수정하기 위해서는 워드프레스가 설치된 디렉토리의 wp-config.php 파일에 다음 내용을 추가합니다.

```
if (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] === 'https') {
	$_SERVER['HTTPS'] = 'on';
}
```
