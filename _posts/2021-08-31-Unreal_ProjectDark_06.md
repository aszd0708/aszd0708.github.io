---
layout: post
title:  "ProjectDark - 06 - InteractionActor"
date:   2021-08-31
excerpt: "ProjectDark - 06 - InteractionActor"
tag:
- Unreal
comments: false
---

# Interaction Actor
이 전에 무기를 만들었지만, 장착할 수 있는 방법이나 액터가 없었다.

그래서 플레이어가 직접 상호작용 할 수 있는 액터 클래스를 만든뒤, 장착할 수 있게 만들자.

<details>
<summary style="color:green">InteractionActorBase.h</summary>
<div markdown="1">

```
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "InteractionActorBase.generated.h"

UCLASS()
class PROJECTDARK_API AInteractionActorBase : public AActor
{
	GENERATED_BODY()
	
public:	
	// Sets default values for this actor's properties
	AInteractionActorBase();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

private:

public:
	virtual void PlayInteraction();

	virtual void PlayInteraction(const class AMainPlayer* MainPlayer);
};
```

</div>
</details>

<details>
<summary style="color:green">InteractionActorBase.cpp</summary>
<div markdown="1">

```
#include "InteractionActorBase.h"

#include "Components/BoxComponent.h"

#include "MainPlayerCodes/MainPlayer.h"

AInteractionActorBase::AInteractionActorBase()
{
	PrimaryActorTick.bCanEverTick = true;
}

void AInteractionActorBase::BeginPlay()
{
	Super::BeginPlay();
	
}

void AInteractionActorBase::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

}

void AInteractionActorBase::PlayInteraction()
{

}

void AInteractionActorBase::PlayInteraction(const AMainPlayer* MainPlayer)
{
	UE_LOG(LogTemp, Log, TEXT("Iteraction!!!"));
}
```

</div>
</details>

(마지막에 있는 로그는 무시하자... 체크 하기 위해 넣었다.)

PlayerInteraction을 사용해서 플레이어와 상호작용 할 수 있게 플레이어에게 다른 명령어를 넣어주자.

<details>
<summary style="color:green">MainPlayer.h</summary>
<div markdown="1">

```
#pragma region Interaction

private:
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Interaction", meta = (AllowPrivateAccess = "True"))
	class USphereComponent* InteractionSphere;

	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, Category = "Interaction", meta = (AllowPrivateAccess = "True"))
	TArray<class AInteractionActorBase*> InteractionActors;

public:
	UFUNCTION()
	void BeginInteractionSphereOverlap(class UPrimitiveComponent* HitComp, class AActor* OtherActor, class UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);

	UFUNCTION()
	void EndInteractionSphereOverlap(class UPrimitiveComponent* HitComp, class AActor* OtherActor, class UPrimitiveComponent* OtherComp, int32 OtherBodyIndex);

	void InteractionMonobehavior();

#pragma endregion
```

</div>
</details>

따로 콜라이더를 만들어 그 안으로 들어올 때, 밖으로 나갈때, 이 둘을 체크해서 현재 인터렉션 액터들을 모은 뒤, 그 액터중에 가장 가까운 InteractionActor에게 명령어를 실행시키게 만든다.

<details>
<summary style="color:green">MainPlyer.cpp</summary>
<div markdown="1">

```
void AMainPlayer::BeginPlay()
{
	Super::BeginPlay();

	InteractionSphere->OnComponentBeginOverlap.AddDynamic(this, &AMainPlayer::BeginInteractionSphereOverlap);
	InteractionSphere->OnComponentEndOverlap.AddDynamic(this, &AMainPlayer::EndInteractionSphereOverlap);
}

void AMainPlayer::BeginInteractionSphereOverlap(UPrimitiveComponent* HitComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	UE_LOG(LogTemp, Log, TEXT("Actor In"));
	if (OtherActor)
	{
		AInteractionActorBase* InteractionActor = Cast<AInteractionActorBase>(OtherActor);
		if (InteractionActor)
		{
			InteractionActors.Add(InteractionActor);
		}
	}
}

void AMainPlayer::EndInteractionSphereOverlap(UPrimitiveComponent* HitComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex)
{
	UE_LOG(LogTemp, Log, TEXT("Actor Out"));
	if (OtherActor)
	{
		AInteractionActorBase* InteractionActor = Cast<AInteractionActorBase>(OtherActor);
		if (InteractionActor)
		{
			InteractionActors.Remove(InteractionActor);
		}
	}
}

void AMainPlayer::InteractionMonobehavior()
{
	if (InteractionActors.Num() > 0)
	{
		AInteractionActorBase* NearActor = NULL;
		float Distance = 999999;
		for (int i = 0; i < InteractionActors.Num(); i++)
		{
			if (InteractionActors[i]->IsHidden() == true)
			{
				continue;
			}
			float CurrentDiatance = FVector::Distance(GetActorLocation(), InteractionActors[i]->GetActorLocation());
			if (CurrentDiatance < Distance)
			{
				NearActor = InteractionActors[i];
			}
		}

		if (NearActor)
		{
			NearActor->PlayInteraction(this);
		}
	}
}
```

</div>
</details>

델리게이트를 사용해서 SphereComponent에 들어온 오브젝트르 검사할 수 있게 만든다.

