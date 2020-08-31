# 안드로이드 개발자, 디자이너와 협업하기 - 아이콘 Export 😱



​	먼 글이냐

안드로이드 개발자 입장에서 디자이너분들이 만든 벡터 아이콘을 AS로 가져오는 깔끔한 방법이 있을까? 라는 고민에서 출발.



​	왜 Android Studio Vector Asset 툴 안씀?

1. 한 번에 하나밖에 못가져온다.
2. 필요 이상으로 정확하려고 함. (소수점 4자리+)
3. GUI - 다른 툴과 통합 안 됨.



​	왜 [svg2android](https://inloop.github.io/svg2android/) 안씀?

1. 웹 기반 - 다른 툴과 통합 어려움.
2. 자기 말로 deprecated 됬다고 함.



​	뭐 어쩌자는거임?

내가 원하는 것

1. 이름을 원하는 형식으로 정의할 수 있어야 함.
2. 원하는 폴더로 자동으로 이동
3. svg 최적화
4. CLI 기반



이 전체를 묶어서 따로 만드는 건 너무 불필요한 작업인 것 같아서, 이미 존재하는 여러 기능들을 섞어서 스크립트를 한 번 만들어보기로 했음.



0. 사전 준비

   우선 스크립트는 쉘 기반으로 만들건데 Node.js 모듈들을 좀 쓸거라서 Node 의존성이 필요함. 아래 두 가지가 필요하다.

   - https://www.npmjs.com/package/svg2vectordrawable
   - https://www.npmjs.com/package/avocado



1. 이름 변경

   