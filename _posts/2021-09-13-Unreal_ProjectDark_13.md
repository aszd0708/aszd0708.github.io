---
layout: post
title:  "ProjectDark - 13 - EnemyWeapon"
date:   2021-09-13
excerpt: "ProjectDark - 13 - EnemyWeapon"
tag:
- Unreal
comments: false
---

적 패턴 디자인 구상하고 코딩테스트 시험이랑 곂쳐서 요 몇일간 개발을 좀 못했다 데헷~(맞다 변명이다.)

적 패턴은 각각 공격마다 쿨타임을 적용시킬거다.  
그 다음에 무기가 있을때, 그리고 없을때 enum으로 나눠서 패턴을 세분화 하고,  
만약 무기가 있을때는 중,장거리 그리고 근거리 2개로 나눠서 세분화 하고,  
무기가 없을때는 중거리, 근거리 2개로 나눠서 만들 것이다. (아마 하다가 애니메이션이 없으면 근거리 하나로만 작업할 것이다.)

# EnemyWeaponBase

<details>
<summary style="color:green">EnemyWeaponBase.h</summary>
<div markdown="1">

```
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "EnemyWeaponBase.generated.h"

UCLASS()
class ENEMYCODES_API AEnemyWeaponBase : public AActor
{
	GENERATED_BODY()
	
public:	
	// Sets default values for this actor's properties
	AEnemyWeaponBase();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;


private:
	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, Category = "Mesh", meta = (AllowPrivateAccess = "True"))
	class USkeletalMeshComponent* WeaponMesh;

public:
	void AttachToCharacter(const class ACharacter* Character);
	void SetActiveActor(bool bIsActive);

	virtual void AttackStart();
	virtual void Attack();
	virtual void AttackEnd();

protected:
	UFUNCTION()
	virtual void BeginWeaponHitBoxOverlap(class UPrimitiveComponent* HitComp, class AActor* OtherActor, class UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);

public:
	FORCEINLINE USkeletalMeshComponent* GetWeaponMesh() const {return WeaponMesh;}
};

```

</div>
</details>

<details>
<summary style="color:green">EnemyWeaponBase.cpp</summary>
<div markdown="1">

```
// Fill out your copyright notice in the Description page of Project Settings.


#include "EnemyWeaponBase.h"

#include "GameFramework/Character.h"
#include "Components/SkeletalMeshComponent.h"
#include "Components/SkinnedMeshComponent.h"

// Sets default values
AEnemyWeaponBase::AEnemyWeaponBase()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

	WeaponMesh = CreateDefaultSubobject<USkeletalMeshComponent>(TEXT("WeaponMesh"));
	SetRootComponent(WeaponMesh);
}

// Called when the game starts or when spawned
void AEnemyWeaponBase::BeginPlay()
{
	Super::BeginPlay();
	
}

// Called every frame
void AEnemyWeaponBase::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

}

void AEnemyWeaponBase::AttachToCharacter(const ACharacter* Character)
{
	if (WeaponMesh)
	{
		WeaponMesh->USkinnedMeshComponent::SetMasterPoseComponent(Character->GetMesh(), true);
		AttachToComponent(Character->GetMesh(), FAttachmentTransformRules::SnapToTargetIncludingScale);
	}
}

void AEnemyWeaponBase::SetActiveActor(bool bIsActive)
{
	SetActorHiddenInGame(!bIsActive);

	SetActorEnableCollision(bIsActive);

	SetActorTickEnabled(bIsActive);
}

void AEnemyWeaponBase::AttackStart()
{
}

void AEnemyWeaponBase::Attack()
{
}

void AEnemyWeaponBase::AttackEnd()
{
}

void AEnemyWeaponBase::BeginWeaponHitBoxOverlap(UPrimitiveComponent* HitComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
}
```

</div>
</details>

무기파트이다 무기를 없을때, 있을때 구분해야 하기 때문에 Active를 따로 만들어서 사용했고, Attach할때 애니메이션을 맞춰준다.

## WrathWeapon

<details>
<summary style="color:green">EnemyWrathWeapon.h</summary>
<div markdown="1">

```
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "EnemyWeaponBase.h"
#include "EnemyWrathWeapon.generated.h"

/**
 * 
 */
UCLASS()
class ENEMYCODES_API AEnemyWrathWeapon : public AEnemyWeaponBase
{
	GENERATED_BODY()
	
public:
	AEnemyWrathWeapon();

protected:
	virtual void BeginPlay() override;

public:
	virtual void Tick(float DeltaTime) override;

private:
	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, Category = "Weapon", meta = (AllowPrivateAccess = true))
	class UBoxComponent* LeftHitBox;
	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, Category = "Weapon", meta = (AllowPrivateAccess = true))
	class UBoxComponent* RightHitBox;

public:
	void SetMaterialAlpha(const float& AlphaValue);

	void AttackStart() override;
	void Attack() override;
	void AttackEnd() override;

private:
	void BeginWeaponHitBoxOverlap(class UPrimitiveComponent* HitComp, class AActor* OtherActor, class UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult) override;
};

```

</div>
</details>

<details>
<summary style="color:green">EnemyWrathWeapon.cpp</summary>
<div markdown="1">

```
// Fill out your copyright notice in the Description page of Project Settings.


#include "EnemyWrathWeapon.h"

#include "Components/BoxComponent.h"

AEnemyWrathWeapon::AEnemyWrathWeapon()
	:AEnemyWeaponBase()
{
	LeftHitBox = CreateDefaultSubobject<UBoxComponent>(TEXT("LeftHitBox"));
	LeftHitBox->AttachToComponent(GetWeaponMesh(), FAttachmentTransformRules::SnapToTargetIncludingScale, "LeftWeaponSocket");

	RightHitBox = CreateDefaultSubobject<UBoxComponent>(TEXT("RightHitBox"));
	RightHitBox->AttachToComponent(GetWeaponMesh(),FAttachmentTransformRules::SnapToTargetIncludingScale, "RightWeaponSocket");
}

void AEnemyWrathWeapon::BeginPlay()
{
	Super::BeginPlay();

	LeftHitBox->OnComponentBeginOverlap.AddDynamic(this, &AEnemyWrathWeapon::BeginWeaponHitBoxOverlap);
	RightHitBox->OnComponentBeginOverlap.AddDynamic(this, &AEnemyWrathWeapon::BeginWeaponHitBoxOverlap);
}

void AEnemyWrathWeapon::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);
}

void AEnemyWrathWeapon::SetMaterialAlpha(const float& AlphaValue)
{
	GetWeaponMesh()->SetScalarParameterValueOnMaterials(TEXT("MaterialAlpha"), AlphaValue);
}

void AEnemyWrathWeapon::AttackStart()
{
}

void AEnemyWrathWeapon::Attack()
{
}

void AEnemyWrathWeapon::AttackEnd()
{
}

void AEnemyWrathWeapon::BeginWeaponHitBoxOverlap(UPrimitiveComponent* HitComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	Super::BeginWeaponHitBoxOverlap(HitComp, OtherActor, OtherComp, OtherBodyIndex, bFromSweep, SweepResult);
}
```

</div>
</details>

일단 필요한것들만 만들었다.