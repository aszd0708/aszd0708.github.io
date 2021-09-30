---
layout: post
title:  "ProjectDark - 17 - WeaponThrow"
date:   2021-09-29
excerpt: "ProjectDark - 17 - WeaponThrow"
tag:
- Unreal
comments: false
---

공격 몽타주 중에 무기를 던지는 공격이 있다. 그거 제작하는데 좀 걸렸다.
<img src = "../assets/img/project/unreal_project_dark/17/throw_attack.gif" width="80%">  
이 공격이다.

<details>
<summary style="color:green">EnemyWrathThrowWeapon.h</summary>
<div markdown="1">

```
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "EnemyWrathThrowWeapon.generated.h"

UCLASS()
class ENEMYCODES_API AEnemyWrathThrowWeapon : public AActor
{
	GENERATED_BODY()
	
public:	
	// Sets default values for this actor's properties
	AEnemyWrathThrowWeapon();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

private:
	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, Category = "Mesh", meta = (AllowPrivateAccess = true))
	class USkeletalMeshComponent* WeaponMesh;

	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, Category = "HitBox", meta = (AllowPrivateAccess = true))
	class UBoxComponent* HitBoxCollider;

	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, Category = "EffectParticle", meta = (AllowPrivateAccess = true))
	class UParticleSystem* EffectParticle;

private: 
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Attack", meta = (AllowprivateAccess = true))
	FVector Destination;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Attack", meta = (AllowPrivateAccess = true))
	float RotationSpeed;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Attack", meta = (AllowPrivateAccess = true))
	float ThrowSpeed;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Attack", meta = (AllowPrivateAccess = true))
	float MaxDistance;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Attack", meta = (AllowPrivateAccess = true))
	float GroundHitRotation;

	bool bIsRotate;

private:
	void RotateToGround(const float& DeltaTime);

	void HitGround();

public:
	void SetActiveActor(bool bIsActive);
	void SetTargetAndThrow(const FVector& ActorLocation);
};

```

</div>
</details>

<details>
<summary style="color:green">EnemyWrathThrowWeapon.cpp</summary>
<div markdown="1">

```
// Fill out your copyright notice in the Description page of Project Settings.


#include "EnemyWrathThrowWeapon.h"

#include "EnemyWeaponBase.h"
#include "Components/SkeletalMeshComponent.h"
#include "Components/BoxComponent.h"

#include "Kismet/KismetMathLibrary.h"

// Sets default values
AEnemyWrathThrowWeapon::AEnemyWrathThrowWeapon()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;
	
	HitBoxCollider = CreateDefaultSubobject<UBoxComponent>(TEXT("HitBox"));
	SetRootComponent(HitBoxCollider);

	WeaponMesh = CreateDefaultSubobject<USkeletalMeshComponent>(TEXT("WeaponMesh"));
	//HitBoxCollider->AttachToComponent(WeaponMesh, FAttachmentTransformRules::SnapToTargetIncludingScale, "LeftWeaponSocket");
	WeaponMesh->AttachToComponent(GetRootComponent(), FAttachmentTransformRules::SnapToTargetIncludingScale);
	bIsRotate = false;
	MaxDistance = 10.0f;
}

// Called when the game starts or when spawned
void AEnemyWrathThrowWeapon::BeginPlay()
{
	Super::BeginPlay();

}

// Called every frame
void AEnemyWrathThrowWeapon::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	RotateToGround(DeltaTime);
}

void AEnemyWrathThrowWeapon::SetActiveActor(bool bIsActive)
{
	SetActorHiddenInGame(!bIsActive);

	SetActorEnableCollision(bIsActive);

	SetActorTickEnabled(bIsActive);
}

void AEnemyWrathThrowWeapon::RotateToGround(const float& DeltaTime)
{
	if(bIsRotate == false)
	{
		return;
	}

	const float Distance = FVector::Distance(Destination, GetActorLocation());
	if (Distance <= MaxDistance)
	{
		bIsRotate = false;
		HitGround();
		return;
	}

	FVector Direction = Destination - GetActorLocation();
	Direction = Direction.GetSafeNormal();	
	SetActorLocation(GetActorLocation() + (Direction * ThrowSpeed));

	const FRotator CurRotation = FRotator(DeltaTime * -RotationSpeed,0, 0);
	const FQuat QuatRotation = FQuat(CurRotation);
	AddActorLocalRotation(QuatRotation);
}

void AEnemyWrathThrowWeapon::HitGround()
{
	const FRotator CurRotator = FRotator(GroundHitRotation, GetActorRotation().Yaw, GetActorRotation().Roll);
	const FQuat QuatRotation = FQuat(CurRotator);
	const FRotator LookAtRotator = UKismetMathLibrary::FindLookAtRotation(GetActorLocation(), Destination);
	SetActorLocationAndRotation(Destination, LookAtRotator);
	SetActorRotation(QuatRotation);
}

void AEnemyWrathThrowWeapon::SetTargetAndThrow(const FVector& ActorLocation)
{
	Destination = ActorLocation;
	bIsRotate = true;
	const FRotator LookAtRotator = UKismetMathLibrary::FindLookAtRotation(GetActorLocation(), ActorLocation);
	SetActiveActor(true);

	SetActorRotation(LookAtRotator);
}

```

</div>
</details>

던지면 그 방향으로 회전하면서 날아가는 동작이다.

<details>
<summary style="color:green">EnenyWrath.cpp</summary>
<div markdown="1">

```
void AEnemyWrath::ThrowWeapon()
{
	Weapon->SetActiveActor(false);
	SetWeaponType(EEnemyWeaponType::EWT_Fist);

	const ACharacter* Character = Cast<ACharacter>(TargetActor);
	const FVector TargetLocation = TargetActor->GetActorLocation();
	//const FVector TargetLocation = Character->GetMesh()->GetRelativeLocation();
	const FVector ThrowableWeaponLocation = GetMesh()->GetSocketLocation(FName("LeftWeaponSocket"));
	ThrowableWeapon->SetActiveActor(true);
	ThrowableWeapon->SetActorLocation(ThrowableWeaponLocation);
	ThrowableWeapon->SetTargetAndThrow(TargetLocation);
}
```

</div>
</details>

던졌을때 현재 공격 타입을 바꾸고 현재 무기 부분에서 던져준다.  
(테스트 할때는 공격 타입을 바꾸진 않았다.)

