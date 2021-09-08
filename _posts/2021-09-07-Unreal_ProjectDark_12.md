---
layout: post
title:  "ProjectDark - 12 - LockOn"
date:   2021-09-07
excerpt: "ProjectDark - 12 - LockOn"
tag:
- Unreal
comments: false
---

적을 만드는 기본 클래스를 만들었으니 간단하게 Lock을 제작해보자

<details>
<summary style="color:green">MainPlayer.h</summary>
<div markdown="1">

```
#pragma region TargetingSystem
private:
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Targeting", meta = (AllowPrivateAccess = "True"))
	class USphereComponent* TargetingCollider;
	
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Targeting", meta = (AllowPrivateAccess = "True"))
	class AEnemyCharacterBase* TargetingActor;

	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Targeting", meta = (AllowPrivateAccess = "True"))
	TArray<class AEnemyCharacterBase*> TargetingActors;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Targeting", meta = (AllowPrivateAccess = "True"))
	float TargetingAngle = 65.0f;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Targeting", meta = (AllowPrivateAccess = "True"))
	float TargetiingInterpSpeed = 1.0f;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Targeting", meta = (AllowPrivateAccess = "True"))
	float TargetiingInterpDeltaTime = 0.01f;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Targeting", meta = (AllowPrivateAccess = "True"))
	float TargetingQuitDistance = 1500.0f;

	bool bIsLockon;

private:
	void LockOn();

	FRotator GetLockOnRotator();

	void MovementLockOn();

	void SetTargetActor();
	UFUNCTION()
	void BeginTargetingCollider(class UPrimitiveComponent* HitComp, class AActor* OtherActor, class UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult); 

	UFUNCTION()
	void EndTargetingCollider(class UPrimitiveComponent* HitComp, class AActor* OtherActor, class UPrimitiveComponent* OtherComp, int32 OtherBodyIndex);
#pragma endregion
```

</div>
</details>

<details>
<summary style="color:green">MainPlayer.cpp</summary>
<div markdown="1">

```
void AMainPlayer::LockOn()
{
	if (TargetingActor)
	{
		Controller->SetControlRotation(GetLockOnRotator());
		MovementLockOn();

		float Distance = FVector::Distance(TargetingActor->GetActorLocation(), GetActorLocation());
		if (Distance > TargetingQuitDistance)
		{
			SetTargetActor();
		}
	}
}

FRotator AMainPlayer::GetLockOnRotator()
{
	FRotator Rotation = GetControlRotation();

	FVector FollowCameraLocation = FollowCamera->GetComponentLocation();
	FVector ActorLocation = TargetingActor->GetActorLocation();
	ActorLocation = FVector(ActorLocation.X, ActorLocation.Y, ActorLocation.Z - TargetingAngle);

	FRotator LookAtRotation = UKismetMathLibrary::FindLookAtRotation(FollowCameraLocation, ActorLocation);
	FRotator InterpRotation = FMath::RInterpTo(Rotation, LookAtRotation, TargetiingInterpDeltaTime, TargetiingInterpSpeed);

	FRotator ResultRotation = FRotator(InterpRotation.Pitch, InterpRotation.Yaw, Rotation.Roll);

	return ResultRotation;
}

void AMainPlayer::MovementLockOn()
{
	FVector ActorLocation = GetActorLocation();
	FVector TargetLocation = TargetingActor->GetActorLocation();
	FRotator LookAtRotation = UKismetMathLibrary::FindLookAtRotation(ActorLocation, TargetLocation);
	LookAtRotation = FRotator(0, LookAtRotation.Yaw, 0);
	FRotator InterpRotation = FMath::RInterpTo(GetActorRotation(), LookAtRotation, TargetiingInterpDeltaTime, TargetiingInterpSpeed);
	SetActorRotation(InterpRotation);
}

void AMainPlayer::SetTargetActor()
{
	if (TargetingActor)
	{
		TargetingActor = NULL;
		bIsLockon = false;
	}
	else
	{
		if (TargetingActors.Num() > 0)
		{
			TargetingActor = TargetingActors[0];
			bIsLockon = true;
		}
	}
}
```

</div>
</details>

새로운 콜라이더를 만들어서 그 안에 들어오게 되면 Enemy인지 검사한 뒤, 락온을 할 수 있는 배열에 넣어준다.  
그런 뒤, 락온을 하게 되면, 카메라와 캐릭터의 시점이 그 Enemy에 고정이 된다.  
배열은 일단 첫번째 걸로 만들었다(짜피 큰 보스 한마리만 맵에 있을거라 상관 없다)

그리고 회피 부분을 조금 수정해주자.

<details>
<summary style="color:green">MainPlayer.cpp(회피부분)</summary>
<div markdown="1">

```
void AMainPlayer::Evade()
{	
	if(bIsEvade == true){return;}
	if(bAttacking == true){return;}
	bIsEvade = true;

	FName SectionName;
	switch (bIsLockon)
	{
		case true:
		{
			if (CurrentInputY > 0.0f)
			{
				if (CurrentInputX > EvadeCheckX)
				{
					SectionName = FName(TEXT("Evade_R"));
				}
				else if(CurrentInputX < -EvadeCheckX)
				{
					SectionName = FName(TEXT("Evade_L"));
				}
				else
				{
					SectionName = FName(TEXT("Evade_F"));
				}
			}
			else
			{
				if (CurrentInputX > EvadeCheckX)
				{
					SectionName = FName(TEXT("Evade_R"));
				}
				else if (CurrentInputX < -EvadeCheckX)
				{
					SectionName = FName(TEXT("Evade_L"));
				}
				else
				{
					SectionName = FName(TEXT("Evade_B"));
				}
			}
		}
		break;

		case false:
		{
			SectionName = FName(TEXT("Evade_F"));
		}
		break;
	}

	UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();
	if (AnimInstance)
	{
		if (EvadeMontage)
		{
			AnimInstance->Montage_Play(EvadeMontage, 1.0f);
			AnimInstance->Montage_JumpToSection(SectionName);
		}
	}

	CurrentComboCount = 0;
}
```

</div>
</details>
현재 입력받은 값을 토대로 왼쪽 오른쪽 앞 뒤 중에 어떤 회피 동작을 할지 정해준다.

<img src = "../assets/img/project/unreal_project_dark/12/lockon.gif" width="30%">

잘 작동한다.
