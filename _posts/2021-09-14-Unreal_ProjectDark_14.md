---
layout: post
title:  "ProjectDark - 14 - EnemyWrath"
date:   2021-09-14
excerpt: "ProjectDark - 14 - EnemyWrath"
tag:
- Unreal
comments: false
---

# EnemyWrath
각 공격에 대한 정보를 저장할 구조체와, 현재 공격 타입을 결정할 enum을 만들어준다.

<details>
<summary style="color:green">Wrath enum class, Struct</summary>
<div markdown="1">

```
UENUM(BlueprintType)
enum class EEnemyWeaponType : uint8
{
	EWT_Sword UMETA(DisplayName = "Sword"),
	EWT_Fist UMETA(DisplayName = "Fist")
};

USTRUCT(Atomic, BlueprintType)
struct FAttackInfo
{
	GENERATED_USTRUCT_BODY()

public:
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Attack")
	class UAnimMontage* AttackMontage;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Attack")
	TArray<int> Damages;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Attack")
	float CoolTime;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Attack")
	float CurrentCoolTime;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Attack")
	float RestTime;
};
```

</div>
</details>

그리고 EnemyCharacterBase를 상속받아서 필요한 부분을 override하고,  
각 부분의 변수를 만들어주자.

<details>
<summary style="color:green">EnemyWrath.h</summary>
<div markdown="1">

```
UCLASS()
class ENEMYCODES_API AEnemyWrath : public AEnemyCharacterBase
{
	GENERATED_BODY()
	
public:
	AEnemyWrath();

protected:
	void BeginPlay() override;

public:
	void Tick(float DeltaTime) override;

#pragma region Idle
private:
	virtual void IdleEnter() override;
	virtual void Idle() override;
	virtual void IdleExit() override;
#pragma endregion

#pragma region Targeting
private:
	virtual void TargetingEnter() override;
	virtual void Targeting() override;
	virtual void TargetingExit() override;

private:
	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, Category = "Targeting", meta = (AllowPrivateAccess = true))
	class USphereComponent* TargetingCollider;

	FName TargetTag;

private:
	UFUNCTION()
	void BeginTargetingCollider(class UPrimitiveComponent* HitComp, class AActor* OtherActor, class UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);
	UFUNCTION()
	void EndTargetingCollider(class UPrimitiveComponent* HitComp, class AActor* OtherActor, class UPrimitiveComponent* OtherComp, int32 OtherBodyIndex);

public:
	FORCEINLINE void SetTargetActor(AActor* Target){ TargetActor = Target; }
#pragma endregion

#pragma region Attack
private:
	virtual void AttackEnter() override;
	virtual void Attack() override;
	virtual void AttackExit() override;

private:
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Attack", meta = (AllowPrivateAccess = true))
	TArray<FAttackInfo> SwordCloseRangeAttackInfos;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Attack", meta = (AllowPrivateAccess = true))
	TArray<FAttackInfo> SwordFarRangeAttackInfos;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Attack", meta = (AllowPrivateAccess = true))
	TArray<FAttackInfo> FistAttackInfos;

	FAttackInfo* CurrentAttack;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Attack", meta = (AllowPrivateAccess = true))
	float CloseRangeAttackDistance;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Attack", meta = (AllowPrivateAccess = true))
	float FarRangeAttackDistance;

private:
	void CalAtkCooltimes(const float& DeltaTime);

	void SetAttackInfo();

	void SetAttackMontage(TArray<FAttackInfo>& Infos);


public:
#pragma endregion

#pragma region Dead
public:
	virtual void DeadEnter() override;
	virtual void Dead() override;
	virtual void DeadExit() override;
#pragma endregion

#pragma region Weapon
private:
	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, Category = "Weapon", meta = (AllowPrivateAccess = true))
	EEnemyWeaponType CurrentWeaponType;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Weapon", meta = (AllowPrivateAccess = true))
	class AEnemyWeaponBase* Weapon;

	UPROPERTY(EditAnywhere, blueprintReadWrite, Category = "Weapon", meta = (AllowPrivateAccess = true))
	class UAnimMontage* SummonWeaponMontage;

private:
	void AttachWeapon();

public:
	FORCEINLINE AEnemyWeaponBase* GetWeapon() const { return Weapon; }
#pragma endregion

#pragma region Combat
public:
	void BeginCombat(AActor* Target) override;
#pragma endregion

#pragma region CombatHitBox
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "HitBox", meta = (AllowPrivateAccess = true))
	class UBoxComponent* LeftFistHitBox;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "HitBox", meta = (AllowPrivateAccess = true))
	class UBoxComponent* RightFistHitBox;


	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "HitBox", meta = (AllowPrivateAccess = true))
	class UBoxComponent* LeftKickHitBox;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "HitBox", meta = (AllowPrivateAccess = true))
	class UBoxComponent* RightKickHitBox;
#pragma endregion

#pragma region Rest
private:
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Rest", meta = (AllowPrivateAccess = true))
	float RestTime;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Rest", meta = (AllowPrivateAccess = true))
	float CurrentRestTime;

private:
	void Rest(const float& DeltaTime);
	void SetRestTime(const float& Time);
#pragma endregion
};
```

</div>
</details>

FAttackInfo 포인터 변수인 CurrentAttack을 계속해서 바꿔주면서 현재 상태에 따라 공격을 정하게 만들어준다.

공격은 쿨타임이 다 돌아 있는 것들 중에 랜덤으로 선택되며,  
공격이 끝나면 RestTime동안 공격을 하지 않는다.

알단, 공격을 정하는 것들만 만들어두자.

<details>
<summary style="color:green">EnemyWrath.cpp</summary>
<div markdown="1">

```
// Fill out your copyright notice in the Description page of Project Settings.


#include "EnemyWrath.h"
#include "AIController.h"
#include "Components/SphereComponent.h"
#include "Components/BoxComponent.h"

#include "EnemyWeaponBase.h"

#include "MainPlayerCodes/Public/MainPlayer.h"

AEnemyWrath::AEnemyWrath()
{
	TargetingCollider = CreateDefaultSubobject<USphereComponent>(TEXT("TargetingCollider"));
	TargetingCollider->SetupAttachment(RootComponent);

	LeftFistHitBox = CreateDefaultSubobject<UBoxComponent>(TEXT("LeftFistHitBox"));
	LeftFistHitBox->AttachToComponent(GetMesh(), FAttachmentTransformRules::SnapToTargetIncludingScale, "LeftWeaponSocket");

	RightFistHitBox = CreateDefaultSubobject<UBoxComponent>(TEXT("RightFistHitBox"));
	RightFistHitBox->AttachToComponent(GetMesh(), FAttachmentTransformRules::SnapToTargetIncludingScale, "RightWeaponSocket");

	LeftKickHitBox = CreateDefaultSubobject<UBoxComponent>(TEXT("LeftKickHitBox"));
	LeftKickHitBox->AttachToComponent(GetMesh(), FAttachmentTransformRules::SnapToTargetIncludingScale, "LeftKickSocket");

	RightKickHitBox = CreateDefaultSubobject<UBoxComponent>(TEXT("RightKickHitBox"));
	RightKickHitBox->AttachToComponent(GetMesh(), FAttachmentTransformRules::SnapToTargetIncludingScale, "RightKickSocket");

	CurrentAttack = NULL;
}

void AEnemyWrath::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	CalAtkCooltimes(DeltaTime);
	Rest(DeltaTime);
}

void AEnemyWrath::AttackEnter()
{
	Super::AttackEnter();
	UE_LOG(LogTemp, Log, TEXT("AttackEnter"));
	SetAttackInfo();
}

void AEnemyWrath::Attack()
{
}

void AEnemyWrath::AttackExit()
{
	SetRestTime(CurrentAttack->RestTime);
}

void AEnemyWrath::CalAtkCooltimes(const float& DeltaTime)
{
	for (int i = 0; i < SwordCloseRangeAttackInfos.Num(); i++)
	{
		SwordCloseRangeAttackInfos[i].CurrentCoolTime += DeltaTime;
		if (SwordCloseRangeAttackInfos[i].CurrentCoolTime >= SwordCloseRangeAttackInfos[i].CoolTime)
		{
			SwordCloseRangeAttackInfos[i].CurrentCoolTime = SwordCloseRangeAttackInfos[i].CoolTime;
		}
	}

	for (int i = 0; i < SwordFarRangeAttackInfos.Num(); i++)
	{
		SwordFarRangeAttackInfos[i].CurrentCoolTime += DeltaTime;
		if (SwordFarRangeAttackInfos[i].CurrentCoolTime >= SwordFarRangeAttackInfos[i].CoolTime)
		{
			SwordFarRangeAttackInfos[i].CurrentCoolTime = SwordFarRangeAttackInfos[i].CoolTime;
		}
	}

	for (int i = 0; i < FistAttackInfos.Num(); i++)
	{
		FistAttackInfos[i].CurrentCoolTime += DeltaTime;
		if (FistAttackInfos[i].CurrentCoolTime >= FistAttackInfos[i].CoolTime)
		{
			FistAttackInfos[i].CurrentCoolTime = FistAttackInfos[i].CoolTime;
		}
	}
}

void AEnemyWrath::SetAttackInfo()
{
	switch (CurrentWeaponType)
	{
		case EEnemyWeaponType::EWT_Sword:
			if (TargetActor)
			{
				float Distance = FVector::Distance(TargetActor->GetActorLocation(), GetActorLocation());

				if (Distance < CloseRangeAttackDistance)
				{
					SetAttackMontage(SwordCloseRangeAttackInfos);
				}
				else if (Distance >= CloseRangeAttackDistance && Distance < FarRangeAttackDistance)
				{;
					SetAttackMontage(SwordFarRangeAttackInfos);
				}
				else
				{
					SetCurrentState(EEnemyState::ES_Targeting);
				}
			}
		break;
		case EEnemyWeaponType::EWT_Fist:;
			SetAttackMontage(FistAttackInfos);
		break;
	}
}

void AEnemyWrath::SetAttackMontage(TArray<FAttackInfo>& Infos)
{
	TArray<FAttackInfo*> SelectableAttacks;
	for (int i = 0; i < Infos.Num(); i++)
	{
		if (Infos[i].CurrentCoolTime >= Infos[i].CoolTime)
		{
			SelectableAttacks.Add(&Infos[i]);
		}
	}
	int RandomValue = FMath::RandRange(0, SelectableAttacks.Num() - 1);
	CurrentAttack = SelectableAttacks[RandomValue];
}

void AEnemyWrath::AttachWeapon()
{
	if (Weapon)
	{
		Weapon->AttachToCharacter(this);
	}
}

void AEnemyWrath::BeginCombat(AActor* Target)
{
	Super::BeginCombat(Target);

	if (SummonWeaponMontage)
	{
		UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();
		if (AnimInstance)
		{
			AnimInstance->Montage_Play(SummonWeaponMontage);
		}
	}

	CurrentWeaponType = EEnemyWeaponType::EWT_Sword;
}

void AEnemyWrath::SetRestTime(const float& Time)
{
	CurrentRestTime = 0.0f;
	RestTime = Time;
}

void AEnemyWrath::Rest(const float& DeltaTime)
{
	CurrentRestTime += DeltaTime;
	if (CurrentRestTime >= RestTime)
	{
		CurrentRestTime = RestTime;
	}
}

```

</div>
</details>

일단 함수 안에 아무것도 적지 않는 것들은 이 게시물에서 지웠다(너무 길다 ㅠㅠ)  
Tick에서는 항상 쿨타임을 계산해주며, RestTime도 계산해준다.

다음에는 플레이어를 따라다니고 가까히 오면 공격하는 AI를 만들자.