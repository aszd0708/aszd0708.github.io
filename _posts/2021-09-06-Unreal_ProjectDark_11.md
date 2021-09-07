---
layout: post
title:  "ProjectDark - 11 - EnemyCharacterBase"
date:   2021-09-06
excerpt: "ProjectDark - 11 - EnemyCharacterBase"
tag:
- Unreal
comments: false
---

만들 적의 기본적인 베이스다

AI는 유한 상태 모델로 만들것이다.

상태는 대기, 타겟이 있을때, 타겟에게 움직일때, 공격  
크게 4개로 나눌 것이고 만약, 적 마다 다른 상태가 있을때 그 상태를 그 캐릭터 클래스에 구현해 줄 것이다.

그리고 FSM은 Enter, Tick, Exit로 구현해 각 상태에서 빠저나가는 것까지 구현할 것이다.

<details>
<summary style="color:green">EnemyCharacterBase.h(EnumClass, Strcut)</summary>
<div markdown="1">

```
USTRUCT(Atomic, BlueprintType)
struct FEnemyInfo
{
	GENERATED_USTRUCT_BODY()

public:
	UPROPERTY(VisibleAnywhere, BlueprintReadWrite)
	float MaxHp;
	UPROPERTY(VisibleAnywhere, BlueprintReadWrite)
	float CurrentHp;
};

UENUM(BlueprintType)
enum class EEnemyState : uint8
{
	ES_Idle UMETA(DisplayName = "Idle"),
	ES_Targeting UMETA(DisplayName = "Targeting"),
	ES_MoveToTarget UMETA(DisplayName = "MoveToTarget"),
	ES_Attack UMETA(DisplayName = "Attack")
};

UENUM(BlueprintType)
enum class EFSMState : uint8 
{
	FSM_Enter UMETA(DisplayName = "Enter"),
	FSM_Tick UMETA(DisplayName = "Tick"),
	FSM_Exit UMETA(DisplayName = "Exit")
};
```

</div>
</details>

<details>
<summary style="color:green">EnemyCharacterBase.h</summary>
<div markdown="1">

```
UCLASS(Abstract)
class ENEMYCODES_API AEnemyCharacterBase : public ACharacter
{
	GENERATED_BODY()

public:
	// Sets default values for this character's properties
	AEnemyCharacterBase();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;


private:
	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, Category = "FSM", meta = (AllowPrivateAccess = "True"))
	EFSMState FSMState;
protected:
	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, Category = "FSM", meta = (AllowPrivateAccess = "True"))
	EEnemyState CurrentState;

private:
	void FSM();
	void FSMEnter();
	void FSMTick();
	void FSMExit();
	void SetCurrentState(const EEnemyState& State);

#pragma region Idle
public:
	virtual void IdleEnter();
	virtual void Idle();
	virtual void IdleExit();
#pragma endregion

#pragma region Targeting
public:
	virtual void TargetingEnter();
	virtual void Targeting();
	virtual void TargetingExit();
#pragma endregion

#pragma region MoveToTarget
public:
	virtual void MoveToTargetEnter();
	virtual void MoveToTarget();
	virtual void MoveToTargetExit();
#pragma endregion

#pragma region Attack
public:
	virtual void AttackEnter();
	virtual void Attack();
	virtual void AttackExit();
#pragma endregion
};
```

</div>
</details>

원래 추상 클래스로 만들려고 했는데 안됀다 ㅠㅠ 아무리 찾아봐도 = 0 을 붙이면 안되는지 안나왔지만 암튼 안돼서 그냥 가상함수로 만들어 준다.

<details>
<summary style="color:green">EnemyCharacterBase.cpp</summary>
<div markdown="1">

```

#include "EnemyCharacterBase.h"

AEnemyCharacterBase::AEnemyCharacterBase()
{
	PrimaryActorTick.bCanEverTick = true;

	CurrentState = EEnemyState::ES_Idle;
	FSMState = EFSMState::FSM_Enter;
}

void AEnemyCharacterBase::BeginPlay()
{
	Super::BeginPlay();

}

void AEnemyCharacterBase::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	FSM();
}

void AEnemyCharacterBase::FSM()
{
	switch (FSMState)
	{
	case EFSMState::FSM_Enter:
		FSMEnter();
		break;

	case EFSMState::FSM_Tick:
		FSMTick();
		break;

	case EFSMState::FSM_Exit:
		FSMExit();
		break;
	}
}

void AEnemyCharacterBase::FSMEnter()
{
	switch (CurrentState)
	{
	case EEnemyState::ES_Idle:
		IdleEnter();
		break;
	case EEnemyState::ES_Targeting:
		TargetingEnter();
		break;
	case EEnemyState::ES_MoveToTarget:
		MoveToTargetEnter();
		break;
	case EEnemyState::ES_Attack:
		AttackEnter();
		break;
	}

	FSMState = EFSMState::FSM_Tick;
}

void AEnemyCharacterBase::FSMTick()
{
	switch (CurrentState)
	{
	case EEnemyState::ES_Idle:
		Idle();
		break;
	case EEnemyState::ES_Targeting:
		Targeting();
		break;
	case EEnemyState::ES_MoveToTarget:
		MoveToTarget();
		break;
	case EEnemyState::ES_Attack:
		Attack();
		break;
	}
}

void AEnemyCharacterBase::FSMExit()
{
	switch (CurrentState)
	{
	case EEnemyState::ES_Idle:
		IdleExit();
		break;
	case EEnemyState::ES_Targeting:
		TargetingExit();
		break;
	case EEnemyState::ES_MoveToTarget:
		MoveToTargetExit();
		break;
	case EEnemyState::ES_Attack:
		AttackExit();
		break;
	}
}

void AEnemyCharacterBase::SetCurrentState(const EEnemyState& State)
{
	if (CurrentState == State) { return; }

	FSMExit();
	CurrentState = State;
	FSMState = EFSMState::FSM_Enter;
}
```

</div>
</details>

FSM에서 처음 들어갈 때, FSM_Enter로 만들어 주고  
반복문에서 Enter, Tick순으로 반복한다. 그리고 현재 State를 변경할 때,  
현재 State의 Exit를 호출, 현재의 State를 변경해주고 FSM을 Enter로 만들어준다.