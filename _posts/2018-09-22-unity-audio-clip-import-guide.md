---
layout: post
title: "유니티 오디오 클립 임포트 설정 가이드"
categories: Unity
tags: [Unity, Audio]
comments: true
---
## 옵션 설명

공통 항목은 워낙 간단하니 샘플 세팅 부분만 설명함.

### Load Type

오디오 에셋 로딩 시 사용하는 방법

#### Decompress On Load

디폴트 설정. 파일 로딩 후 바로 압축해제하여 메모리에 적재해둠. 재생 레이턴시 면에서 이점이 있지만 막대한 메모리를 사용. 사실상 PCM 전용. 레이턴시가 무엇보다도 우선되어야 하는 경우에만 사용하며, 일반적으로는 Compressed In Memory를 쓰고 레이턴시 우선의 경우에는 ADPCM를, 그렇지 않은 경우에는 Quality를 낮춘 Vorbis/MP3를 사용하는 것을 추천.

#### Compressed In Memory

압축상태로 메모리에 두었다가 재생할때 압축해제. 압축해제 시 CPU 리소스 사용 및 약간의 레이턴시가 발생할 수 있음. 재생 시의 오버헤드는 프로파일러의 DSP CPU에서 모니터링 가능. 일반적으로 이것을 사용하게 됨.

#### Streaming

디스크에서 바로 읽어들여서 재생. Compressed In Memory와의 차이점은 데이터를 읽어들이는 곳이 메모리인가 디스크인가하는 것. 빈 파일이라도 대략 200KB 정도의 오버헤드가 있다고 함. 프로파일러의 Streaming CPU에서 모니터링 가능. 이 것을 사용시에는 Preload Audio Data 옵션이 꺼짐.

### Preload Audio Data

옵션이 켜져있으면 씬 로딩 시에 미리 로드함. 꺼져있는 경우 AudioSource에서 플레이하거나 AudioSource.LoadAudioData()를 사용할때 로드.

### Compression Format

오디오 압축 방법 설정

#### PCM

무압축. 이것을 사용하는 경우에는 Load Type을 Decompress On Load으로 할 것.

#### ADPCM

약간의 압축. 총격음이나 폭발음등 노이즈가 많은 효과음에 적절.

#### Vorbis/MP3

실생활에서 흔히 쓰이는 압축포맷. 예전에는 iOS에서는 MP3, 안드로이드에서는 Vorbis로 하는 것을 권장했는데 언제부턴가 문서에서 사라짐. iOS 기기에서 Vorbis의 하드웨어 디코딩이 가능하게 된 건 아닐테고, CPU 파워가 충분해서 굳이 신경쓸 필요가 없다고 생각하는 건지...

### Quality

음질 수준을 결정. Compression Format이 PCM나 ADPCM일 경우에는 나타나지 않음.

### Sampling Rate Setting

샘플링 레이트 설정

#### Preserve Sample Rate

오디오 파일의 원래 샘플링 레이트를 사용. 기본설정이긴 한데 이걸 사용할 바에야 Optimize Sample Rate를 사용할 것.

#### Optimize Sample Rate

유니티가 분석한 최적의 샘플링 레이트를 지정. 생각하기 귀찮거나 잘 모르겠으면 이걸로.

#### Override Sample Rate

샘플링 레이트를 직접 지정. 목록에 나와있는 샘플링 레이트의 수준은 대략 다음과 같음(출처는 [위키백과](https://ko.wikipedia.org/wiki/%EC%83%98%ED%94%8C%EB%A7%81_%EB%A0%88%EC%9D%B4%ED%8A%B8#%EC%98%A4%EB%94%94%EC%98%A4)).

| 샘플링 레이트 | 음질 수준                                   |
|---------------|---------------------------------------------|
| 8000 Hz       | 전화, 암호화된 무전기                       |
| 11025 Hz      | 서브우퍼 대역                               |
| 22050 Hz      | SP 레코드(20세기 초기의 축음기 음반)        |
| 44100 Hz      | 오디오 CD                                   |
| 48000 Hz      | 디지털 영상장비의 표준 오디오 샘플링 레이트 |
| 96000 Hz      | DVD-Audio, BD 오디오 트랙, 고음질 음원      |
| 192000 Hz     | DVD-Audio, BD 오디오 트랙, 고음질 음원      |

위의 표를 보면 알겠지만 48 kHz 이상의 샘플링 레이트는 사용할 필요가 없음. BGM과 음성은 44.1 kHz, 둔탁한 효과음은 11025 Hz, 나머지는 22050 Hz로 설정하는 것을 권장.

## 설정 가이드

### 사운드의 종류와 일반적인 성향

|               | 용량 | 길이  | 음질          | 빈도 | 재생 레이턴시            | 스테레오 여부        |
|---------------|------|-------|---------------|------|--------------------------|----------------------|
| BGM           | 큼   | 김    | 중요함        | 낮음 | 중요하지 않음            | 경우에 따라 필요[^1] |
| 음성          | 큼   | 긴 편 | 중요함        | 낮음 | 보통은 중요하지 않음[^2] | 필요없음             |
| 인게임 효과음 | 작음 | 짧음  | 중요하지 않음 | 많음 | 중요함                   | 필요없음             |
| UI 효과음     | 작음 | 짧음  | 중요하지 않음 | 보통 | 중요함                   | 필요없음             |
| 징글          | 보통 | 보통  | 보통          | 보통 | 중요하지 않음            | 필요없음             |
| 환경음        | 보통 | 보통  | 보통          | 보통 | 중요하지 않음            | 필요없음             |

[^1]: 일반적으로 모바일에서는 스테레오 스피커를 사용하지 않으므로 Force To Mono 설정을 켜는 것을 추천하나, 유저가 이어폰을 사용할 수도 있고 리듬 게임처럼 음악이 중요하다면 스테레오로 유지해야 할 수도 있음.
[^2]: 입모양과 싱크를 맞춰야하는 경우에는 중요함

#### BGM

Load Type은 Streaming, Compression Format은 Vorbis/MP3로 설정. Quality는 100으로 두지말고 적당히 줄여서 쓸 것.

#### 음성

Load Type은 Compressed In Memory이나 Streaming, Compression Format은 Vorbis/MP3. BGM과 마찬가지로 Quality를 적당히 낮춰서 사용.

#### 타격음이나 발사음 등 짧고 자주 쓰이며 레이턴시가 중요한 효과음

Load Type은 Decompress On Load, Compression Format은 PCM.

#### 폭발음 등 다소 긴 편이며 둔탁한 효과음

Load Type은 Compressed In Memory, Compression Format은 ADPCM. Sample Rate는 11025 Hz 이하.

#### 징글

Load Type은 Compressed In Memory, Compression Format은 길이에 따라 ADPCM 혹은 Quality를 낮춘 Vorbis/MP3.

#### 환경음

Load Type은 Compressed In Memory, Compression Format은 길이에 따라 ADPCM 혹은 Quality를 낮춘 Vorbis/MP3. Vorbis/OGG를 사용하는 경우 Streaming도 고려가능.

## 설정 자동화

AssetPostProcessor를 이용하면 일일이 설정하는 수고를 줄일 수 있음. Editor 디렉토리 안에 다음과 같은 스크립트를 작성.

{% gist 832ca5d32bc6c1787b963a032deacba0 %}

단, 이렇게 미리 지정해둔 곳은 인스펙터에서 설정을 바꾸려고 해도 다시 돌아가버리므로 반드시 공통되는 부분만 지정할 것.