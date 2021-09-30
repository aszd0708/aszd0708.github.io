---
layout: post
title:  "ProjectDark - 18 - WeaponThrow"
date:   2021-09-29
excerpt: "ProjectDark - 18- WeaponThrow"
tag:
- Unreal
comments: false
---

던질 무기를 블루프린트로 만들어주고, 몽타주에 AnimNotify를 만들어서 붙여주자

```
// Fill out your copyright notice in the Description page of Project Settings.


#include "WrathThrowWeaponAnimNotify.h"

#include "EnemyWrath.h"

void UWrathThrowWeaponAnimNotify::Notify(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation)
{
	Super::Notify(MeshComp, Animation);

	AActor* Actor = MeshComp->GetOwner();
	if(Actor)
	{
		AEnemyWrath* Wrath = Cast<AEnemyWrath>(Actor);
		if(Wrath)
		{
			Wrath->ThrowWeapon();
		}
	}
}

```
라고 하기엔 너무 짧다....

이제 이 Notify를 붙여서 만들면

<img src = "../assets/img/project/unreal_project_dark/17/throw_attack.gif" width="80%">  