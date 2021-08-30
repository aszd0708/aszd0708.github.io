---
layout: post
title:  "ProjectDark - 03"
date:   2021-08-25
excerpt: "ProjectDark - 03"
tag:
- C++
- Unreal
comments: false
---

# Weapon

각 무기를 만들었으니까 현재 장착되어 있는 무기의 콤보와 공격 모션을 재생할 수 있게 플레이어에게 새로운 명령어를 만들어주자

## MainPlayer Class

<details>
<summary style="color:green">MainPlayer.h</summary>
<div markdown="1">

```
#pragma region ATK

private:
	bool bAttacking;

	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Atk", meta = (AllowPrivateAccess = "True"))
	bool bComboAttachCheck = false;

	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Atk", meta = (AllowPrivateAccess = "True"))
	bool bComboAttack = false;

	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Atk", meta = (AllowPrivateAccess = "True"))
	int32 CurrentComboCount = 0;

	bool bLeftWeapon;

public:
	void ComboAttackCheck();

	void ComboAttachNotify();

	void AttackStart();

	void ComboAttack();

	void AttackEnd();

	void Attack();

	void AttackEndNotify();

	void AttackLeft();
	void AttackRight();

	void AttackStartLeft();
	void AttackStartRight();

	void AttackEndLeft();
	void AttackEndRight();

	void SetAttackMontageEnd();
#pragma endregion
```

</div>
</details>

<details>
<summary style="color:green">MainPlayer.cpp</summary>
<div markdown="1">

```
void AMainPlayer::ChangeWeapon()
{
	if (CurrentWeaponIndex < 0) { return; }
	EqipWeapon(CurrentWeaponIndex + 1);
}

void AMainPlayer::ComboAttackCheck()
{
	if (bComboAttack == true)
	{
		ComboAttack();
		bComboAttachCheck = false;
		bComboAttack = false;
	}
}

void AMainPlayer::ComboAttachNotify()
{
	bComboAttachCheck = true;
}

void AMainPlayer::Attack()
{
	if (CurrentWeapon == NULL) { return; }
	if (bAttacking == false)
	{
		if (bLeftWeapon)
		{
			EqipWeapon(0);
		}
		else
		{
			EqipWeapon(1);
		}

		UAnimInstance* AnimInatance = GetMesh()->GetAnimInstance();
		if (AnimInatance)
		{
			UAnimMontage* AttackMontage = CurrentWeapon->GetWeaponInfo().AtkMontage;
			if (AttackMontage)
			{
				AnimInatance->Montage_Play(AttackMontage);
				AnimInatance->Montage_JumpToSection("Combo1");
			}
			CurrentComboCount = 1;
		}
		bAttacking = true;
	}

	if (bComboAttachCheck == true)
	{
		bComboAttack = true;
	}
}

void AMainPlayer::AttackEndNotify()
{
	//SetAttackMontageEnd();
	bAttacking = false;
	bComboAttachCheck = false;
	CurrentComboCount = 1;

	UE_LOG(LogTemp, Log, TEXT("AttackEnd"));
}

void AMainPlayer::AttackLeft()
{
	if(Weapons[0] == NULL){ return;}

	bLeftWeapon = true;
	Attack();
}

void AMainPlayer::AttackRight()
{
	if (Weapons[1] == NULL) { return; }

	bLeftWeapon = false;
	Attack();
}

void AMainPlayer::AttackStartLeft()
{
	if(CurrentWeapon == NULL){return;}
	CurrentWeapon->DualAttackStart(CurrentComboCount, Status.Damage, EDualWeaponDirection::Left);
}

void AMainPlayer::AttackStartRight()
{
	if (CurrentWeapon == NULL) { return; }
	CurrentWeapon->DualAttackStart(CurrentComboCount, Status.Damage, EDualWeaponDirection::Right);
}

void AMainPlayer::AttackEndLeft()
{
	if (CurrentWeapon == NULL) { return; }
	CurrentWeapon->DualAttackEnd(EDualWeaponDirection::Left);
}

void AMainPlayer::AttackEndRight()
{
	if (CurrentWeapon == NULL) { return; }
	CurrentWeapon->DualAttackEnd(EDualWeaponDirection::Right);
}

void AMainPlayer::SetAttackMontageEnd()
{
	UAnimInstance* AnimInatance = GetMesh()->GetAnimInstance();
	if (AnimInatance)
	{
		AnimInatance->Montage_Stop(1.0f);
	}
}

void AMainPlayer::AttackStart()
{
	if(CurrentWeapon == NULL){return;}

	int Temp = CurrentComboCount;
	if (CurrentComboCount >= CurrentWeapon->GetWeaponInfo().MaxComboCount)
	{
		Temp = CurrentWeapon->GetWeaponInfo().MaxComboCount;
	}
	CurrentWeapon->AttackStart(Temp, Status.Damage);
}

void AMainPlayer::ComboAttack()
{	
	if(CurrentWeapon == NULL){return;}

	CurrentComboCount++;
	if (CurrentComboCount > CurrentWeapon->GetWeaponInfo().MaxComboCount)
	{
		return;
	}

	UAnimInstance* AnimInatance = GetMesh()->GetAnimInstance();
	if (AnimInatance)
	{
		if (bLeftWeapon)
		{
			EqipWeapon(0);
		}
		else
		{
			EqipWeapon(1);
		}

		UAnimMontage* AttackMontage = CurrentWeapon->GetWeaponInfo().AtkMontage;
		int CurrentComboTemp = CurrentComboCount;
		if (CurrentComboTemp >= CurrentWeapon->GetWeaponInfo().MaxComboCount)
		{
			CurrentComboTemp = CurrentComboCount;
		}
		if (AttackMontage)
		{
			FName SectionName;

			switch (CurrentComboTemp)
			{
			case 1:
				SectionName = FName(TEXT("Combo1"));
				break;

			case 2:
				SectionName = FName(TEXT("Combo2"));
				break;

			case 3:
				SectionName = FName(TEXT("Combo3"));
				break;

			case 4:
				SectionName = FName(TEXT("Combo4"));
				break;
			}
			AnimInatance->Montage_Play(AttackMontage);
			AnimInatance->Montage_JumpToSection(SectionName);
		}
	}
}

void AMainPlayer::AttackEnd()
{
	if (CurrentWeapon == NULL) { return; }
	CurrentWeapon->AttackEnd();
}
```

</div>
</details>

현재 콤보가 작동 되는지 확인하고 콤보가 있으면 재생 아니면 끝낸다.

각각 무기마다 적용해서 왼쪽 오른쪽 무기를 순서대로 누르면 각각 콤보가 나온다.

L Weapon Combo1 -> R Weapon Combo2 -> L Weapon Combo3 -> 끝

이런 식이다.