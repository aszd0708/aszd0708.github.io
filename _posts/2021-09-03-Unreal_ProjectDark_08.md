---
layout: post
title:  "ProjectDark - 08 - Evade"
date:   2021-09-02
excerpt: "ProjectDark - 08 - Evade"
tag:
- Unreal
comments: false
---

후.... 적 모듈 추가하다가 멘탈 뻥뻥 터졌다.. .후.... 해결 했는데 추후 포스트 올려야징

# Evade
회피다

만약 지금 록온 중이면 왼쪽 오른쪽 뒤로 가는 Evade를 실행

아닐 경우 앞으로 가는 Evade를 실행함.

(하지만, 현재 LockonCharacter 없기 때문에! 왼쪽 오른쪽은 안함)
(그리고 피할 공격이 없기 때문에! 다른 방법으로 테스트 실행)

다크 사이더스3를 플레이 해보면 회피할때 특정한 구간에서 회피가 완벽하게 성공했을 경우 시간이 느려지고 카운터 어택을 한다.

나도 구현할거시다

<details>
<summary style="color:green">MainPlyer.h</summary>
<div markdown="1">

```
#pragma region Evade
private:
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Combat", meta = (AllowPrivateAccess = "True"))
	class UAnimMontage* EvadeMontage;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Evade", meta = (AllowPrivateAccess = "True"))
	float EvadeSize = 10.0f;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Evade", meta = (AllowPrivateAccess = "True"))
	bool bIsEvade = false;

	bool bIsPerfectEvade = false;

	bool bCanEvadeAttack = false;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Evade", meta = (AllowPrivateAccess = "True"))
	class UCurveFloat* SlowMotionCurve;

	bool bIsSlowDown = false;

	float CurrentDeltaTime = 0;
	float EvadeTotlaDeltaTime = 0;;

public:
	UFUNCTION(BlueprintCallable, Category = "Evade")
	void Evade();

	UFUNCTION(BlueprintCallable, Category = "Evade")
	void EndEvade();

	UFUNCTION(BlueprintCallable, Category = "Evade")
	void StartEvade();

	void PerfectEvadeCheckStart();

	void PerfectEvadeCheckEnd();

	void EvadeAttack();

	void EvadeAttackEnd();

	void PerfectEvade();

	void SlowDown();
#pragma endregion
```

</div>
</details>

<details>
<summary style="color:green">MainPlayer.cpp</summary>
<div markdown="1">

```
void AMainPlayer::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	CurrentDeltaTime = DeltaTime;

	ComboAttackCheck(); 
	SlowDown();
}

void AMainPlayer::StartEvade()
{
	GetCharacterMovement()->BrakingFrictionFactor = 0.0f;
	FVector EvadeVector = FVector(GetActorForwardVector().X, GetActorForwardVector().Y, 0);
	LaunchCharacter(EvadeVector.GetSafeNormal() * EvadeSize, true, true);
}

void AMainPlayer::EndEvade()
{
	bIsEvade = false;
	bCanEvadeAttack = false;
}

void AMainPlayer::PerfectEvadeCheckStart()
{
	bIsPerfectEvade = true;
}

void AMainPlayer::PerfectEvadeCheckEnd()
{
	bIsPerfectEvade = false;
}

void AMainPlayer::EvadeAttack()
{
	if (CurrentWeapon == NULL) { return; }
	if (bCanEvadeAttack == true)
	{
		bAttacking = true;
		bCanEvadeAttack = false;
		bIsEvade = true;
		UE_LOG(LogTemp, Log, TEXT("EvadeAttack"));
		// 회피 성공시 공격 몽타주 재생

		UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();
		if (AnimInstance)
		{
			UAnimMontage* Montage = CurrentWeapon->GetWeaponInfo().EvadeAtkMontage;
			if (Montage)
			{
				AnimInstance->Montage_Play(Montage, Status.AttackSpeed);
			}
		}

		CurrentWeapon->SetEvadeAtkDamage(Status.Damage);
	}
}

void AMainPlayer::EvadeAttackEnd()
{
	bAttacking = false;
	bIsEvade = false;
	bCanEvadeAttack = false;
	CurrentComboCount = 0;
}

void AMainPlayer::PerfectEvade()
{
	bIsPerfectEvade = false;
	bCanEvadeAttack = true;

	// 시간 천천히
	bIsSlowDown = true;
}

void AMainPlayer::SlowDown()
{
	if (bIsSlowDown == true)
	{
		EvadeTotlaDeltaTime += CurrentDeltaTime;
		if (SlowMotionCurve)
		{
			float CurveValue = SlowMotionCurve->GetFloatValue(EvadeTotlaDeltaTime);
			UGameplayStatics::SetGlobalTimeDilation(GetWorld(), CurveValue);
			UE_LOG(LogTemp, Log, TEXT("CurveValue : %f"), CurveValue);
		}

		if (EvadeTotlaDeltaTime >= 0.5f)
		{
			EvadeTotlaDeltaTime = 0;
			bIsSlowDown = false;
			UGameplayStatics::SetGlobalTimeDilation(GetWorld(), 1.0f);
		}

		UE_LOG(LogTemp, Log, TEXT("Time : %f"), EvadeTotlaDeltaTime);
	}
}

void AMainPlayer::Attack()
{
	if (bIsEvade == true)
	{
		EvadeAttack();
		return;
	}
	if (CurrentWeapon == NULL) { return; }
	if (bAttacking == false)
	{
		if (bLeftWeapon)
		{
			EqipWeapon(0);
		}
		else
		{
			EqipWeapon(1);
		}

		UAnimInstance* AnimInatance = GetMesh()->GetAnimInstance();
		if (AnimInatance)
		{
			UAnimMontage* AttackMontage = CurrentWeapon->GetWeaponInfo().AtkMontage;
			if (AttackMontage)
			{
				AnimInatance->Montage_Play(AttackMontage, Status.AttackSpeed);
				AnimInatance->Montage_JumpToSection("Combo1");
			}
			CurrentComboCount = 1;
			CurrentWeapon->SetAtkDamage(CurrentComboCount, Status.Damage);
		}
		bAttacking = true;
	}

	if (bComboAttachCheck == true)
	{
		bComboAttack = true;
	}
}
```

</div>
</details>

내가 현재 만든 방법은  
회피하는 시간과 완벽한 회피 시간을 때로 체크해준다.  
만약 완벽한 회피 시간에 공격을 맞았을 경우  
피해를 받지 않고 시간을 느려지게 해준 뒤에  
다음 공격은 카운터 어택이 나가게 만들어 준다  
만약 공격을 안하고 회피가 끝났을 경우  
카운터 어택은 발동하지 않고 그냥 공격한다.

그리고 현재 무기에 카운터 어택 몽타주를 만들어 주고 공격 데미지를 만들어 준다  
(무기 클래스에 있는 공격을 데미지 계한 함수만 따로 파서 만들었다.)
(아 그리고 이펙트도 추가해줬다.)


<details>
<summary style="color:green">MainPlayerEvadeNotifyState.cpp</summary>
<div markdown="1">

```
// Fill out your copyright notice in the Description page of Project Settings.


#include "MainPlayerEvadeNotifyState.h"

#include "MainPlayer.h"

void UMainPlayerEvadeNotifyState::NotifyBegin(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation, float TotalDuration)
{
	Super::NotifyBegin(MeshComp, Animation, TotalDuration);

	AActor* PlayerActor = MeshComp->GetOwner();
	if (PlayerActor)
	{
		AMainPlayer* MainPalyer = Cast<AMainPlayer>(PlayerActor);
		if (MainPalyer)
		{
			MainPalyer->StartEvade();
		}
	}
}

void UMainPlayerEvadeNotifyState::NotifyTick(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation, float FrameDeltaTime)
{
	Super::NotifyTick(MeshComp, Animation, FrameDeltaTime);
}

void UMainPlayerEvadeNotifyState::NotifyEnd(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation)
{
	Super::NotifyEnd(MeshComp, Animation);

	AActor* PlayerActor = MeshComp->GetOwner();
	if (PlayerActor)
	{
		AMainPlayer* MainPalyer = Cast<AMainPlayer>(PlayerActor);
		if (MainPalyer)
		{
			MainPalyer->EndEvade();
		}
	}
}

```

</div>
</details>

<details>
<summary style="color:green">PerfectEvadeNotifyState.cpp</summary>
<div markdown="1">

```
// Fill out your copyright notice in the Description page of Project Settings.


#include "PerfectEvadeNotifyState.h"

#include "MainPlayer.h"

void UPerfectEvadeNotifyState::NotifyBegin(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation, float TotalDuration)
{
	Super::NotifyBegin(MeshComp, Animation, TotalDuration);

	AActor* PlayerActor = MeshComp->GetOwner();
	if (PlayerActor)
	{
		AMainPlayer* MainPalyer = Cast<AMainPlayer>(PlayerActor);
		if (MainPalyer)
		{
			MainPalyer->PerfectEvadeCheckStart();
		}
	}
}

void UPerfectEvadeNotifyState::NotifyTick(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation, float FrameDeltaTime)
{
	Super::NotifyTick(MeshComp, Animation, FrameDeltaTime);
}

void UPerfectEvadeNotifyState::NotifyEnd(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation)
{
	Super::NotifyEnd(MeshComp, Animation);

	AActor* PlayerActor = MeshComp->GetOwner();
	if (PlayerActor)
	{
		AMainPlayer* MainPalyer = Cast<AMainPlayer>(PlayerActor);
		if (MainPalyer)
		{
			MainPalyer->PerfectEvadeCheckEnd();
		}
	}
}

```

</div>
</details>

<details>
<summary style="color:green">EvadeAttackEndNotify.cpp</summary>
<div markdown="1">

```
// Fill out your copyright notice in the Description page of Project Settings.


#include "EvadeAttackEndNotify.h"

#include "MainPlayer.h"

void UEvadeAttackEndNotify::Notify(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation)
{
	AActor* PlayerActor = MeshComp->GetOwner();

	if (PlayerActor)
	{
		AMainPlayer* MainPlyer = Cast<AMainPlayer>(PlayerActor);

		if (MainPlyer)
		{
			MainPlyer->EvadeAttackEnd();
		}
	}
}
```

</div>
</details>

그리고 각각 Notify를 만들어 준 뒤...

To Be Continue...