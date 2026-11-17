# 1초 로드무비 (웹앱)

## 무엇을 하는 앱인가
- 아이폰에서 1초짜리 영상 클립을 여행 내내(며칠에 걸쳐) 24개 모은 뒤, 한 편의 영상(약 24초)으로 합쳐 주는 웹앱.
- 음악은 넣지 않는다. 사용자가 완성 영상을 사진 앱에 저장한 뒤 Edits 앱에서 음악을 넣는다.
- 사용자: 나와 아내 2명. 각자 자기 아이폰에서 따로 쓴다.

## 실행 환경
- 기기: iPhone 12, iOS Safari. 사파리에서 "홈 화면에 추가"한 상태(standalone)로 사용한다.
- 호스팅: GitHub Pages (정적 파일만, 서버 없음).
- 맥이 없다 → Safari 웹 인스펙터를 쓸 수 없다. 디버깅은 화면 안 로그 패널(예: ?debug=1일 때 eruda 로드)로 한다.

## 반드시 지킬 원칙
- 영상·소리는 절대 외부로 전송하지 않는다. fetch/XHR/분석 도구 금지. 모든 처리는 기기 안에서.
- 클립은 IndexedDB에 저장한다(ArrayBuffer + mime type). 홈 화면 앱으로 쓰면 iOS가 지우지 않는다.
- 외부 라이브러리는 꼭 필요할 때만, 무거운 것(ffmpeg.wasm 등)은 피한다.
- 모든 UI 문구는 한국어.

## 현재 구현 (index.html 한 파일)
- getUserMedia(후면 카메라 기본, 마이크 켜기/끄기 토글) → MediaRecorder로 1.3초 녹화 → IndexedDB 저장, 썸네일 jpeg 저장.
- 합치기: 각 클립을 <video>로 재생하며 canvas에 그리고, canvas.captureStream + MediaRecorder로 다시 녹화.
  - 클립 앞 0.2초는 건너뛰고 1.0초만 사용(SKIP, SEG 상수).
  - 클립 사이 로딩 동안은 rec.pause()/resume()으로 정지 화면이 끼지 않게 함.
  - 소리는 decodeAudioData로 미리 풀어서 AudioBufferSourceNode → MediaStreamDestination으로 섞음.
- 결과는 Web Share API(navigator.share files)로 "비디오 저장".
- 출력 비율: 세로 1080x1920 / 가로 1920x1080 선택, cover 방식으로 잘라 맞춤.

## 작업 방식
- 기능 하나씩 고치고, 바꿀 때마다 아이폰에서 직접 확인할 수 있게 무엇을 테스트하면 되는지 짧게 알려 줄 것.
- 아이폰 Safari에서 안 되는 API는 쓰기 전에 지원 여부를 확인하고, 안 되면 대안을 쓸 것.
- 커밋 메시지는 한국어로.
