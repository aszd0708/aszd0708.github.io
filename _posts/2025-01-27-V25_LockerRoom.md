---
layout: post
title:  "락커룸"
excerpt: "락커룸"
tag:
-
comments: false
---

락커룸에선 유니폼, 락커룸 효과만 담당

## 유니폼
<img src = "../assets/img/project/fortpolio/LockerRoom
/uniform_main.jpg" width="40%">

원하는 유니폼을 선택 후 제작하기를 누르면,

<img src = "../assets/img/project/fortpolio/LockerRoom
/uniform_preview.jpg" width="40%">

해당 유니폼을 만들기 위한 재료들을 보여줍니다.

재료들에 대한건 이 문서를 참고 [참고](https://aszd0708.github.io/V25_CardMaterial/)

만들게 되면,

<img src = "../assets/img/project/fortpolio/LockerRoom
/uniform_made.jpg" width="40%">

착용하기 버튼이 활성화 되며, 착용을 하게 되면

<img src = "../assets/img/project/fortpolio/LockerRoom
/main.jpg" width="40%">

게임의 메인 화면 및 인게임에서 적용이 됩니다.

메인화면에서 현재와 지금의 선택된 유니폼 값이 다르다면, 리소스를 로딩하는 과정이 있습니다.

### 알림

<img src = "../assets/img/project/fortpolio/LockerRoom
/uniform_material_alarm.jpg" width="40%">

원하는 알림을 하게 되면, 그 조건에 맞춰 락커룸에 레드닷을 띄워줍니다.

조건은 카드중에
- 종류
- 등급
- 이름관련 ID값
- 강화 등급
- 훈련 등급
- 연도
- 구단

중에 기획데이터에서 체크한 값들만 갖고와서 비교를 합니다.

따라서 조건이 많고 체크해야 하는 카드들도 많기 때문에, 게임 로그인 및 알림에 변경사항이 생기면, 조건을 먼저 캐싱을 합니다. 그리고 현재 인벤토리에 있는 선수들을 체크 합니다.

그리고 인벤토리의 변경사항이 생기면, 그 변경된 선수들만 조건을 체크 합니다.

## 락커룸 효과

<img src = "../assets/img/project/fortpolio/LockerRoom
/effect_main.jpg" width="40%">

이모티콘 및 유니폼을 만들게 되면, 포인트를 받으며 포인트를 소모하여 스탯을 올릴수 있습니다. 해당 스탯은 적용하는 즉시 반영이 되며, 되돌리기 버튼을 클릭 하면 가장 최근에 선택한 스탯부터 하나씩 선택이 해제됩니다.