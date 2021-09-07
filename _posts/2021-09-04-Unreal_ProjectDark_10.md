---
layout: post
title:  "ProjectDark - 10 - SlowMotionComponent"
date:   2021-09-04
excerpt: "ProjectDark - 10 - SlowMotionComponent"
tag:
- Unreal
comments: false
---

# Delegate
이 전에 유니티에서 UnityEvent를 사용해서 원하는 타이밍에 함수를 실행시키는 것을 많이 사용했었다.  
그것도 델리게이트를 사용해서 진행했었는데 C++에선 없지만, 언리얼에 있어서 사용해본다.(사용하는 방법이 처음이라 엄청 애먹었었다..... 허허;;)

이 전에 SlowMotion을 만들 때, Tick에서 항상 반복문 돌려서 유지보수나 보기 불편했다. 이젠 ActorComponent를 사용해서 새로 만들어보자

<details>
<summary style="color:green">MainPlayerSlowMotionComponent.h</summary>
<div markdown="1">

```
#pragma once

#include "CoreMinimal.h"
#include "Components/ActorComponent.h"
#include "MainPlayerSlowMotionComponent.generated.h"

DECLARE_DELEGATE(FSlowMotionStartEvent);
DECLARE_DELEGATE(FSlowMotionEndEvent);

UCLASS( ClassGroup=(Custom), meta=(BlueprintSpawnableComponent) )
class MAINPLAYERCODES_API UMainPlayerSlowMotionComponent : public UActorComponent
{
	GENERATED_BODY()

public:	
	// Sets default values for this component's properties
	UMainPlayerSlowMotionComponent();

protected:
	// Called when the game starts
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction) override;

#pragma region SlowMotion
private:
	bool bStartSlowMotion;
	bool bIsStart;


	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "SlowMotion", meta = (AllowPrivateAccess = "True"))
	class UCurveFloat* SlowMotionCurve;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "SlowMotion", meta = (AllowPrivateAccess = "True"))
	float MaxSlowMotionTime;
	float CurrentTime = 0;

private:
	void SlowMotion(const float& DeltaTime);

public:
	void StartSlowMotion();

#pragma endregion

#pragma region Delegate
public:
	FSlowMotionStartEvent SlowMotionStartEvent;
	FSlowMotionEndEvent SlowMotionEndEvent;

public:
	FORCEINLINE FSlowMotionEndEvent& GetSlowMotionEndEvent() {return SlowMotionEndEvent;}
#pragma endregion
};
```

</div>
</details>

<details>
<summary style="color:green">MainPlayerSlowMotionComponent.cpp</summary>
<div markdown="1">

```
// Fill out your copyright notice in the Description page of Project Settings.


#include "MainPlayerSlowMotionComponent.h"
#include "Kismet/GameplayStatics.h"

#include "MainPlayer.h"

UMainPlayerSlowMotionComponent::UMainPlayerSlowMotionComponent()
{
	PrimaryComponentTick.bCanEverTick = true;
	CurrentTime = 0.0f;
	MaxSlowMotionTime = 0.5f;

	bStartSlowMotion = false;
	bIsStart = true;
}

void UMainPlayerSlowMotionComponent::BeginPlay()
{
	Super::BeginPlay();
}

void UMainPlayerSlowMotionComponent::TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction)
{
	Super::TickComponent(DeltaTime, TickType, ThisTickFunction);
	SlowMotion(DeltaTime);
}

void UMainPlayerSlowMotionComponent::SlowMotion(const float& DeltaTime)
{
	if(bStartSlowMotion == true)
	{
		if (bIsStart == true)
		{
			if (SlowMotionStartEvent.IsBound())
			{
				SlowMotionStartEvent.Execute();
				ClearSlowMotionStartEvent();
			}
			bIsStart = false;
		}
		CurrentTime += DeltaTime;
		if (CurrentTime >= MaxSlowMotionTime)
		{
			CurrentTime = 0;
			bStartSlowMotion = false;
			UGameplayStatics::SetGlobalTimeDilation(GetWorld(), 1.0f);
			bIsStart = true;
			if (SlowMotionEndEvent.IsBound())
			{
				SlowMotionEndEvent.Execute();
				ClearSlowMotionEndEvent();
				return;
			}
		}
		float Value;
		if (SlowMotionCurve)
		{
			Value = SlowMotionCurve->GetFloatValue(CurrentTime);
		}

		else
		{
			Value = CurrentTime;
		}

		UGameplayStatics::SetGlobalTimeDilation(GetWorld(), Value);
	}
}

void UMainPlayerSlowMotionComponent::StartSlowMotion()
{
	bStartSlowMotion = true;
}

void UMainPlayerSlowMotionComponent::ClearSlowMotionStartEvent()
{
	SlowMotionStartEvent.Unbind();
}

void UMainPlayerSlowMotionComponent::ClearSlowMotionEndEvent()
{
	SlowMotionEndEvent.Unbind();
}

```

</div>
</details>

이렇게 만들어 준 후, 컴포넌트를 추가 하자.

```
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Slow", meta = (AllowPrivateAccess = "Treu"))
	class UMainPlayerSlowMotionComponent* SlowMotion;
```

그리고 이 전에 만들어둔 PerfectEvade을 변경해주자

```
void AMainPlayer::PerfectEvade()
{
	bIsPerfectEvade = false;
	bCanEvadeAttack = true;

	SlowMotion->SlowMotionEndEvent.Unbind();
	SlowMotion->GetSlowMotionEndEvent().BindUFunction(this, FName("SlowDownEnd"));
	SlowMotion->StartSlowMotion();
}
```

그러고 테스트용 SlowDownEnd함수를 만들어주자

Delegate를 사용하기 위해서는 UFUNCTION을 꼭 붙여주자(1시간 넘게 몰라서 허헣ㅎ...)

```
UFUNCTION()
	void SlowDownEnd();
```
```
void AMainPlayer::SlowDownEnd()
{
	UE_LOG(LogTemp, Log, TEXT("SlowDown Dele"));
}
```

그러고 실행시키면

```
LogTemp: SlowDown Dele
```
잘 나온다.

