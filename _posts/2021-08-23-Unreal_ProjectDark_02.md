---
layout: post
title:  "ProjectDark - 02 - Create Weapon Base"
date:   2021-08-23
excerpt: "ProjectDark - 02 - Create Weapon Base"
tag:
- C++
- Unreal
comments: false
---

# Weapon

역시 게임에는 무기가 필요하지

현재 다크사이더스3에서 갖고온 무기 모델들은 쌍검, 쌍절곤, 창, 오함마
이다.(Nunchaku가 쌍절곤인걸 오늘 알았다....)

이 무기들 중 2개를 갖고 보스를 만나러 가는 것으로 만들어 줄것이다.

그럼 장착하는 것과 공격 이 필요하다.

이 무기들의 큰 틀을 갖고 베이스 클래스를 만들어 주자.

## WeaponBase Class
Actor를 상속받아 WeaponBase를 만들어준다.

<details>
<summary style="color:green">WeaponBase.h</summary>
<div markdown="1">

```
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "WeaponBase.generated.h"

UENUM(BlueprintType)
enum class EWeaponType : uint8
{
	WT_DualSword UMETA(DisplayName = "DualSword"),
	WT_Hammer UMETA(DisplayName = "Hammer"),
	WT_Nunchaku UMETA(DisplayName = "Nunchaku"),
	WT_Spear UMETA(DisplayName = "Spear"),
	WT_Sword UMETA(DisplayName = "Sword")
};

enum class EDualWeaponDirection
{
	Left, Right
};

USTRUCT(Atomic, BlueprintType)
struct FWeaponInfo
{
	GENERATED_USTRUCT_BODY()

public:
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
		EWeaponType WeaponType;
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
		UAnimMontage* AtkMontage;
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
		int MaxComboCount;
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
		TArray<int> ComboDamage;
};

UCLASS()
class MAINPLAYERCODES_API AWeaponBase : public AActor
{
	GENERATED_BODY()

public:
	// Sets default values for this actor's properties
	AWeaponBase();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	void SetActiveActor(bool IsActive);

protected:
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Mesh")
	class USkeletalMeshComponent* WeaponMesh;

private:
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Mesh", meta = (AllowPrivateAccess = "True"))
	FWeaponInfo WeaponInfo;

	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Mesh", meta = (AllowPrivateAccess = "True"))
	float CurrentDamage;

public:
	virtual void AttachToPlayer();

	virtual void Deattach();

	virtual void Equip();

	virtual void TakeOff();

	virtual void AttackStart(const int32& ComboIndex, const float& AddDamage);

	virtual void AttackEnd();

	virtual void DualAttackStart(const int32& ComboIndex, const float& AddDamage, const EDualWeaponDirection& WeaponDirection);

	virtual void DualAttackEnd(const EDualWeaponDirection& WeaponDirection);

protected:
	UFUNCTION()
		virtual void WeaponBeginOverlap(class UPrimitiveComponent* HitComp, class AActor* OtherActor, class UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);

public:
	FORCEINLINE const FWeaponInfo& GetWeaponInfo() const { return WeaponInfo; }

	FORCEINLINE const float& GetCurrentDamage() const { return CurrentDamage; }
	FORCEINLINE void SetCurrentDamage(const float& Damage) { CurrentDamage = Damage; }
};
```
</div>
</details>

<details>
<summary style="color:green">WeaponBase.cpp</summary>
<div markdown="1">

```
// Fill out your copyright notice in the Description page of Project Settings.


#include "WeaponBase.h"

#include "Kismet/GameplayStatics.h"
#include "GameFramework/Character.h"
#include "Components/SkinnedMeshComponent.h"

#include "MainPlayer.h"

// Sets default values
AWeaponBase::AWeaponBase()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;
	
	WeaponMesh = CreateDefaultSubobject<USkeletalMeshComponent>(TEXT("WeaponMesh"));
	RootComponent = WeaponMesh;
	//WeaponMesh->SetupAttachment(RootComponent);
}

// Called when the game starts or when spawned
void AWeaponBase::BeginPlay()
{
	Super::BeginPlay();
	
}

// Called every frame
void AWeaponBase::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

}

void AWeaponBase::SetActiveActor(bool IsActive)
{
	SetActorHiddenInGame(!IsActive);

	SetActorEnableCollision(!IsActive);

	SetActorTickEnabled(!IsActive);
}

void AWeaponBase::AttachToPlayer()
{
	ACharacter* PlayerCharacter = UGameplayStatics::GetPlayerCharacter(GetWorld(),0);
	
	WeaponMesh->USkinnedMeshComponent::SetMasterPoseComponent(PlayerCharacter->GetMesh(), true);
	AttachToComponent(PlayerCharacter->GetMesh(), FAttachmentTransformRules::SnapToTargetIncludingScale);
	AMainPlayer* MainPlayer = Cast<AMainPlayer>(PlayerCharacter);
	if (MainPlayer)
	{
		MainPlayer->AttachWeapon(this);
	}
}

void AWeaponBase::Deattach()
{
	
}

void AWeaponBase::Equip()
{
	SetActiveActor(true);
}

void AWeaponBase::TakeOff()
{
	SetActiveActor(false);
}

void AWeaponBase::AttackStart(const int32& ComboIndex, const float& AddDamage)
{
	CurrentDamage = GetWeaponInfo().ComboDamage[ComboIndex - 1] + AddDamage;
}

void AWeaponBase::AttackEnd()
{
}

void AWeaponBase::DualAttackStart(const int32& ComboIndex, const float& AddDamage, const EDualWeaponDirection& WeaponDirection)
{
	CurrentDamage = GetWeaponInfo().ComboDamage[ComboIndex - 1] + AddDamage;
}

void AWeaponBase::DualAttackEnd(const EDualWeaponDirection& WeaponDirection)
{
}

void AWeaponBase::WeaponBeginOverlap(UPrimitiveComponent* HitComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	// 공격
}
```

</div>
</details>

쌍검과 쌍절곤 때문에 왼쪽 오른쪽을 따로 분리해 놓은 것과 오함마와 창은 하나의 히트박스를 갖고 있기 때문에 attackStart에서 처리해 줄 것이다.

그리고 무기에 어떤 애니메이션 몽타주를 갖고있는지 각 콤보마다 공격력을 어떻게 해줄것인지, 그리고 콤보가 몇개까지 있는지 구조체로 정의해준다.

그리고 SkeletalMesh로 왜 무기를 만든 이유는 추후 설명하겠다.

데미지는 플레이어의 데미지도 같이 합산해서 한다(레벨업, 스텟조정 때문에)

이 클래스를 부모로 해서 각각 만들어준다.
