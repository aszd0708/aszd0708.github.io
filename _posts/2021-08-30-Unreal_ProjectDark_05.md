---
layout: post
title:  "ProjectDark - 05 - Weapon Anim Montage"
date:   2021-08-30
excerpt: "ProjectDark - 05 - Weapon Anim Montage"
tag:
- C++
- Unreal
comments: false
---

# AnimationMontage
이제 이 전에 만들었던 AnimNotify들을 활용해서 실행시켜주자

<img src = "../assets/img/project/unreal_project_dark/05/Hammer_Mesh.png" width="50%">

이 무기를

<img src = "../assets/img/project/unreal_project_dark/05/Hammer_Atk.gif" width="50%">

이런 동작에 잘 넣어보자

<img src = "../assets/img/project/unreal_project_dark/05/Hammer_AnimMontage_Combo1.png" width="80%">

(마지막은 AtkEndNotify다)
이런 식으로 넣어서 각 무기의 공격마다 넣게 되면 애니메이션이 저 프레임을 지날 때 마다 저 노티파이들을 싱행시켜준다.

일단 장착하는 기능은 안만들었지만, 억지로 장착 시킨다음 실행시켜보면

<img src = "../assets/img/project/unreal_project_dark/05/Hammer_Atk_Combo.gif" width="50%">

오함마

<img src = "../assets/img/project/unreal_project_dark/05/DualSword_Atk_Combo.gif" width="50%">

쌍검

<img src = "../assets/img/project/unreal_project_dark/05/Mix_Combo.gif" width="50%">

물론 각 무기의 콤보를 서로 다른 무기로 할 수도 있다.

