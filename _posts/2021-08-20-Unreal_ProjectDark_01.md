---
layout: post
title:  "ProjectDark - 01"
date:   2021-08-20
excerpt: "ProjectDark - 01"
tag:
- C++
- Unreal
comments: false
---

며칠동안 다크사이더스3에 있는 모델링 데이터를 뽑아 와서 애니메이션과 모델들을 전부 FBX 확장자로 변경했다.

(Fury는 무슨 이틀이 걸렸다.... ㅠㅠ)

이제 이 데이터들로 한번 다크소울 비슷하게 다크사이더스 비슷하게 만들어보자

# MainPlayerCharacter
일단 플레이어 캐릭터를 만들자.

캐릭터를 부모로 갖고있는 MainPlayer를 만들어두자

<details>
<summary>MainPlayer.h</summary>
<div markdown="1">

```
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once


#include "CoreMinimal.h"
#include "GameFramework/Character.h"
#include "MainPlayer.generated.h"

USTRUCT(Atomic, BlueprintType)
struct FMainPlayerStatus
{
	GENERATED_USTRUCT_BODY()

public:
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	float MaxHealth;
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	float CurrentHealth;
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	float Damage;
};

UCLASS()
class MAINPLAYERCODES_API AMainPlayer : public ACharacter
{
	GENERATED_BODY()

public:
	// Sets default values for this character's properties
	AMainPlayer();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	// Called to bind functionality to input
	virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;

private:
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Statuc", meta = (AllowPrivateAccess = "True"))
	FMainPlayerStatus Status;

private:
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Camera, meta = (AllowPrivateAccess = "True"))
	class USpringArmComponent* CameraBoom;

	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Camera, meta = (AllowPrivateAccess = "True"))
	class UCameraComponent* FollowCamera;

private:
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Camera, meta = (AllowPrivateAccess = "True"))
	float BaseTurnRate;

	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Camera, meta = (AllowPrivateAccess = "True"))
	float BaseLookUpRate;


#pragma region TargetingSystem
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Targeting", meta = (AllowPrivateAccess = "True"))
	AActor* TargetingActor;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Targeting", meta = (AllowPrivateAccess = "True"))
	float TargetingAngle = 65.0f;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Targeting", meta = (AllowPrivateAccess = "True"))
	float TargetiingInterpSpeed = 1.0f;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Targeting", meta = (AllowPrivateAccess = "True"))
	float TargetiingInterpDeltaTime = 0.01f;
#pragma endregion

private:
	void MoveForward(float Value);

	void MoveRight(float Value);

	void TurnAtRate(float Rate);

	void LookUpAtRate(float Rate);

public:
	FORCEINLINE class USpringArmComponent* GetCameraBoom() const { return CameraBoom; }

	FORCEINLINE class UCameraComponent* GetFollowCamera() const { return FollowCamera; }

	FORCEINLINE const FMainPlayerStatus& GetMainPlayerStatus() const {return Status;}
};

```

</div>
</details>

일단 간단하게 움직이는 것들과 카메라 정도만 만들어두자

<details>
<summary>MainPlyer.h</summary>
<div markdown="1">

```
// Fill out your copyright notice in the Description page of Project Settings.


#include "MainPlayer.h"
#include "HeadMountedDisplayFunctionLibrary.h"
#include "Camera/CameraComponent.h"
#include "Components/CapsuleComponent.h"
#include "Components/InputComponent.h"
#include "GameFramework/CharacterMovementComponent.h"
#include "GameFramework/Controller.h"
#include "GameFramework/SpringArmComponent.h"
#include "Components/PrimitiveComponent.h"

AMainPlayer::AMainPlayer()
{
	PrimaryActorTick.bCanEverTick = true;

	BaseTurnRate = 45.0f;
	BaseLookUpRate = 45.0f;

	bUseControllerRotationPitch = false;
	bUseControllerRotationYaw = false;
	bUseControllerRotationRoll = false;

	GetCharacterMovement()->bOrientRotationToMovement = true;
	GetCharacterMovement()->RotationRate = FRotator(0.0f, 540.0f, 0.0f);
	GetCharacterMovement()->JumpZVelocity = 350.0f;
	GetCharacterMovement()->AirControl = 0.1f;

	CameraBoom = CreateDefaultSubobject<USpringArmComponent>(TEXT("CameraBoom"));
	CameraBoom->SetupAttachment(RootComponent);
	CameraBoom->TargetArmLength = 300.0f;
	CameraBoom->bUsePawnControlRotation = true;

	FollowCamera = CreateDefaultSubobject<UCameraComponent>(TEXT("FollowCamera"));
	FollowCamera->SetupAttachment(CameraBoom, USpringArmComponent::SocketName);
	FollowCamera->bUsePawnControlRotation = false;
}

void AMainPlayer::BeginPlay()
{
	Super::BeginPlay();
	
}

void AMainPlayer::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	ComboAttackCheck();
}

// Called to bind functionality to input
void AMainPlayer::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);
	check(PlayerInputComponent);

	PlayerInputComponent->BindAxis("MoveForward", this, &AMainPlayer::MoveForward);
	PlayerInputComponent->BindAxis("MoveRight", this, &AMainPlayer::MoveRight);

	PlayerInputComponent->BindAxis("Turn", this, &APawn::AddControllerYawInput);
	PlayerInputComponent->BindAxis("TurnRate", this, &AMainPlayer::TurnAtRate);
	PlayerInputComponent->BindAxis("LookUp", this, &APawn::AddControllerPitchInput);
	PlayerInputComponent->BindAxis("LookUpRate", this, &AMainPlayer::LookUpAtRate);
}

void AMainPlayer::MoveForward(float Value)
{
	if (bIsEvade == true) { return; }
	if (bAttacking == true) { return; }

	if ((Controller != nullptr) && (Value != 0.0f))
	{
		const FRotator Rotation = Controller->GetControlRotation();
		const FRotator YawRotation(0, Rotation.Yaw, 0);

		const FVector Direction = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::X);
		AddMovementInput(Direction, Value);
	}
}

void AMainPlayer::MoveRight(float Value)
{
	if (bIsEvade == true) { return; }
	if (bAttacking == true) { return; }

	if ((Controller != nullptr) && (Value != 0.0f))
	{
		const FRotator Rotation = Controller->GetControlRotation();
		const FRotator YawRotation(0, Rotation.Yaw, 0);

		const FVector Direction = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::Y);
		AddMovementInput(Direction, Value);
	}
}

void AMainPlayer::TurnAtRate(float Rate)
{
	AddControllerYawInput(Rate * BaseTurnRate * GetWorld()->GetDeltaSeconds());
}

void AMainPlayer::LookUpAtRate(float Rate)
{
	AddControllerPitchInput(Rate * BaseLookUpRate * GetWorld()->GetDeltaSeconds());
}
```

</div>
</details>

일단 간단하게 만들어 준다.

그런 뒤, AnimInstance를 만들어준다.

<details>
<summary>MainPlyerAnimInstance.h</summary>
<div markdown="1">

```
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "Animation/AnimInstance.h"
#include "MainPlayerAnimInstance.generated.h"

/**
 * 
 */
UCLASS()
class MAINPLAYERCODES_API UMainPlayerAnimInstance : public UAnimInstance
{
	GENERATED_BODY()

public:
	virtual void NativeInitializeAnimation() override;

	virtual void NativeUpdateAnimation(float DeltaSeconds) override;

private:
	UPROPERTY(EditAnyWhere, BlueprintReadOnly, Category = Movement, meta = (AllowPrivateAccess = true))
	class APawn* Pawn;
	UPROPERTY(EditAnyWhere, BlueprintReadOnly, Category = Movement, meta = (AllowPrivateAccess = true))
	class AMainPlayer* PlayerCharacter;

	UPROPERTY(EditAnyWhere, BlueprintReadOnly, Category = Movement, meta = (AllowPrivateAccess = true))
	float MovementSpeed;

	UPROPERTY(EditAnyWhere, BlueprintReadOnly, Category = Movement, meta = (AllowPrivateAccess = true))
	float CurrentDirection;
	UPROPERTY(EditAnyWhere, BlueprintReadOnly, Category = Movement, meta = (AllowPrivateAccess = true))
	bool bIsInAir;

public:
	FORCEINLINE const float& GetMovementSpeed() { return MovementSpeed; }
	FORCEINLINE void SetMovementSpeed(const float& _MovementSpeed) { MovementSpeed = _MovementSpeed; }

	FORCEINLINE const float& GetCurrentDirrection() { return CurrentDirection; }
	FORCEINLINE void SetCurrentDirrection(const float& _CurrentDirection) { CurrentDirection = _CurrentDirection; }

	FORCEINLINE const bool& GetIsInAir() { return bIsInAir; }
	FORCEINLINE void SetIsInAir(const bool& _bIsInAir) { bIsInAir = _bIsInAir; }

	FORCEINLINE const AMainPlayer* GetPlayerCharacter() { return PlayerCharacter; }
	FORCEINLINE void SetPlayerCharacter(AMainPlayer* Character) { PlayerCharacter = Character; }
};

```

</div>
</details>

<details>
<summary>MainPlayerAnimInstance.h</summary>
<div markdown="1">

```
// Fill out your copyright notice in the Description page of Project Settings.


#include "MainPlayerAnimInstance.h"
#include "MainPlayer.h"

void UMainPlayerAnimInstance::NativeInitializeAnimation()
{
	if (Pawn == nullptr)
	{
		Pawn = TryGetPawnOwner();

		if (Pawn)
		{
			PlayerCharacter = Cast<AMainPlayer>(Pawn);
		}
	}
}

void UMainPlayerAnimInstance::NativeUpdateAnimation(float DeltaSeconds)
{
	Super::NativeUpdateAnimation(DeltaSeconds);

	if (Pawn == nullptr)
	{
		Pawn = TryGetPawnOwner();
	}

	if (Pawn)
	{
		FVector Speed = Pawn->GetVelocity();
		MovementSpeed = Speed.Size();
		CurrentDirection = CalculateDirection(Speed, Pawn->GetActorRotation());


		if (PlayerCharacter == nullptr)
		{
			PlayerCharacter = Cast<AMainPlayer>(Pawn);
		}
	}
}
```

</div>
</details>

애니메이션을 동기화 해서 만들어준다.

이렇게 한뒤, 두개 전부 블루프린트를 만들어서 적용해주자.

