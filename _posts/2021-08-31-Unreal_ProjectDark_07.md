---
layout: post
title:  "ProjectDark - 07 - Equipment Weapon"
date:   2021-08-31
excerpt: "ProjectDark - 07 - Equipment Weapon"
tag:
- C++
- Unreal
comments: false
---

# Equipment Weapon

이 전에 만들었던 InteractionActorBase를 상속받아 만들자

<details>
<summary style="color:green">InteractionEquipmentWeapon.h</summary>
<div markdown="1">

```
UCLASS()
class PROJECTDARK_API AInteractionEquipmentWeapon : public AInteractionActorBase
{
	GENERATED_BODY()
	
public:
	AInteractionEquipmentWeapon();

protected:
	virtual void BeginPlay() override;

public:
	virtual void Tick(float DeltaTime) override;

private:
	UPROPERTY(EditAnywhere, Category = "Weapon", meta = (AllowPrivateAccess = "True"))
	class USkeletalMeshComponent* WeaponMesh;

	UPROPERTY(EditAnywhere, Category = "Weapon", meta = (AllowPrivateAccess = "True"))
	class AWeaponBase* EqipmentWeapon;

	UPROPERTY(EditAnywhere, Category = "Weapon", meta = (AllowPrivateAccess = "True"))
	FRotator StartRotation;

	UPROPERTY(EditAnywhere, Category = "Weapon", meta = (AllowPrivateAccess = "True"))
	float RotationSpeed;

public:
	virtual void PlayInteraction() override;

	virtual void PlayInteraction(const class AMainPlayer* MainPlayer) override;

	void SetActiveActor(bool bIsActive);
};

```

</div>
</details>

<details>
<summary style="color:green">InteractionEquipmentWeapon.cpp</summary>
<div markdown="1">

```
// Fill out your copyright notice in the Description page of Project Settings.


#include "InteractionEquipmentWeapon.h"


#include "Components/SkeletalMeshComponent.h"
#include "MainPlayerCodes/WeaponBase.h"

AInteractionEquipmentWeapon::AInteractionEquipmentWeapon()
{
	WeaponMesh = CreateDefaultSubobject<USkeletalMeshComponent>(TEXT("WeaponMesh"));
	SetRootComponent(WeaponMesh);

}

void AInteractionEquipmentWeapon::BeginPlay()
{
	Super::BeginPlay();

	if (EqipmentWeapon == NULL)
	{
		Destroy();
	}

	else
	{
		EqipmentWeapon->SetInteractionWeapon(this);
		EqipmentWeapon->SetActiveActor(false);
	}

	SetActorRotation(StartRotation);
}

void AInteractionEquipmentWeapon::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	FRotator IdleRotator = FRotator(GetActorRotation().Pitch, GetActorRotation().Yaw + DeltaTime * RotationSpeed, GetActorRotation().Roll);
	SetActorRotation(IdleRotator);
}

void AInteractionEquipmentWeapon::PlayInteraction()
{
	Super::PlayInteraction();
}

void AInteractionEquipmentWeapon::PlayInteraction(const AMainPlayer* MainPlayer)
{
	Super::PlayInteraction(MainPlayer);

	EqipmentWeapon->AttachToPlayer();
	EqipmentWeapon->SetActiveActor(true);

	SetActiveActor(false);
}

void AInteractionEquipmentWeapon::SetActiveActor(bool bIsActive)
{
	SetActorHiddenInGame(!bIsActive);

	SetActorEnableCollision(bIsActive);

	SetActorTickEnabled(bIsActive);
}


```

</div>
</details>

게임이 시작되면 Weapon에게 이 액터를 넘겨주고 무기가 장착하면 꺼지고 켜지게 만들어준다.

<details>
<summary style="color:green">WeaponBase.cpp</summary>
<div markdown="1">

```
void AWeaponBase::Detach()
{
	if(InteractionWeapon)
	{
		WeaponMesh->USkinnedMeshComponent::SetMasterPoseComponent(NULL);
		DetachFromActor(FDetachmentTransformRules::KeepWorldTransform);
		InteractionWeapon->SetActiveActor(true);
		SetActiveActor(false);
	}
}
```

</div>
</details>

WeaponBase에 따로 장착 해제를 만들어줬다.

일단 Tick에서는 가만히 있으면 심심하니, 계속해서 돌도록 만들었다.

그리고 플레이어에게 장착 후 게임 내에서 보이지 않게 만들어준다.

이렇게 하면

<img src = "../assets/img/project/unreal_project_dark/07/EuipWeapon.gif" width="50%">

무기를 장착 하고 해제 하고 할 수 있다.