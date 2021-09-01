---
layout: post
title:  "ProjectDark - 04"
date:   2021-08-29
excerpt: "ProjectDark - 04"
tag:
- C++
- Unreal
comments: false
---

# AnimNotifyState

이제 각 무기가 공격할때 실행할 변수와, 콤보 그리고 공격이 끝났을때 처리해야할 AnimNotify, AnimNotifyState를 만들어 준다.


(Header파일은 전부 Begin, Tick, End Override한것들 밖에 없어서 따로 적진 않음)
<details>
<summary style="color:green">MainPlayerAtkNotifyState.cpp</summary>
<div markdown="1">

```
// Fill out your copyright notice in the Description page of Project Settings.


#include "MainPlayerAtkNotifyState.h"

#include "MainPlayer.h"

void UMainPlayerAtkNotifyState::NotifyBegin(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation, float TotalDuration)
{
	Super::NotifyBegin(MeshComp, Animation, TotalDuration);

	AActor* Player = MeshComp->GetOwner();
	if (Player)
	{
		AMainPlayer* MainPlayer = Cast<AMainPlayer>(Player);
		if (MainPlayer)
		{
			MainPlayer->AttackStart();
		}
	}
}

void UMainPlayerAtkNotifyState::NotifyTick(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation, float FrameDeltaTime)
{
	Super::NotifyTick(MeshComp, Animation, FrameDeltaTime);
}

void UMainPlayerAtkNotifyState::NotifyEnd(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation)
{
	Super::NotifyEnd(MeshComp, Animation);

	AActor* Player = MeshComp->GetOwner();
	if (Player)
	{
		AMainPlayer* MainPlayer = Cast<AMainPlayer>(Player);
		if (MainPlayer)
		{
			MainPlayer->AttackEnd();
		}
	}
}
```

</div>
</details>

공격할 때, 각 무기의 HitBox를 켜주는 역할을 해준다.

<details>
<summary style="color:green">DualWeaponRightAtkNotifyState.cpp</summary>
<div markdown="1">

```
// Fill out your copyright notice in the Description page of Project Settings.


#include "DualWeaponRightAtkNotifyState.h"

#include "MainPlayer.h"

void UDualWeaponRightAtkNotifyState::NotifyBegin(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation, float TotalDuration)
{
	Super::NotifyBegin(MeshComp, Animation, TotalDuration);

	AActor* PlayerActor = MeshComp->GetOwner();

	if (PlayerActor)
	{
		AMainPlayer* MainPlayer = Cast<AMainPlayer>(PlayerActor);
		if (MainPlayer)
		{
			MainPlayer->AttackStartRight();
		}
	}
}

void UDualWeaponRightAtkNotifyState::NotifyTick(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation, float FrameDeltaTime)
{
	Super::NotifyTick(MeshComp, Animation, FrameDeltaTime);
}

void UDualWeaponRightAtkNotifyState::NotifyEnd(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation)
{
	Super::NotifyEnd(MeshComp, Animation);

	AActor* PlayerActor = MeshComp->GetOwner();

	if (PlayerActor)
	{
		AMainPlayer* MainPlayer = Cast<AMainPlayer>(PlayerActor);
		if (MainPlayer)
		{
			MainPlayer->AttackEndRight();
		}
	}
}

```

</div>
</details>

왼쪽은 이름만 조금 바꾸고 했다.

왼쪽 오른쪽도 해주고,

<details>
<summary style="color:green">MainPlayerComboNotify.cpp</summary>
<div markdown="1">

```
#include "MainPlayerComboNotify.h"

#include "MainPlayer.h"

void UMainPlayerComboNotify::Notify(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation)
{
	Super::Notify(MeshComp, Animation);
	AActor* Player = MeshComp->GetOwner();

	if (Player)
	{
		AMainPlayer* MainPlayer = Cast<AMainPlayer>(Player);
		if (MainPlayer)
		{
			MainPlayer->ComboAttachNotify();
		}
	}
}
```

</div>
</details>

콤보를 이어갈 수 있는지 체크 해주는 것도 넣고

<details>
<summary style="color:green">MainPlayerAtkEndNotify.cpp</summary>
<div markdown="1">

```
#include "MainPlayerAtkEndNotify.h"

#include "MainPlayer.h"

void UMainPlayerAtkEndNotify::Notify(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation)
{
	Super::Notify(MeshComp, Animation);

	AActor* PlayerActor = MeshComp->GetOwner();

	if (PlayerActor)
	{
		AMainPlayer* MainPlayer = Cast<AMainPlayer>(PlayerActor);

		if (MainPlayer)
		{
			MainPlayer->AttackEndNotify();
		}
	}
}
```

</div>
</details>

공격이 끝났을 때 처리할 함수도 실행 할 수 있게 넣어준다.
