---
layout: post
title: "로컬에 Jekyll 설치 후 GitHub Pages 플러그인 설정 방법"
categories: Jekyll
comments: true
---
1. Gemfile을 열고 `gem "jekyll"`(뒤에 버전이 지정되어 있을수도 있음) 행을 주석화.
2. Gemfile 주석 중에 `gem "github-pages", group: :jekyll_plugins`이라고 써진 부분에서 주석화 제거. 없으면 새로 추가.
3. _config.yml에 플러그인 추가
4. bundle exec jekyll serve로 실행
