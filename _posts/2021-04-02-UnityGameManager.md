---
layout: post
title:  "UnityGameManagers"
date:   2021-04-02
excerpt: "각종 게임 제작에 사용했던 유틸리티"
project: true
tag:
- 
comments: false
---

# UnityGameManagers

## 소개

제가 직접 유니티를 활용해서 프로젝트를 진핼 할 때 진행할 프로젝트 내에서 사용하는 기능들을 바로 사용 할 수 있게 제작한 매니저들 입니다.

[Github](https://github.com/aszd0708/UnityGameManagers){:target="_blank"}

## Managers

### PoolingManager

SetPool 을 사용해서 오브젝트를 SetActive(false)를 시킨 다음 SetPool에 있던 PoolingName 으로 새로운 카테고리를 만들어 넣어줍니다.

 만약 카테고리가 있을경우 그 카테고리로 넣어줍니다.


그리고 GetPool을 사용해서 원하는 카테고리에 있는 오브젝트를 가져올 수 있음 만약 없을경우 null을 반환 합니다.


사용시 받아올때 null을 체크 한 뒤 null을 받게 되면 원하는 오브젝트 생성해서 사용 하면 됩니다.

#### SetPool
```
public void SetPool(GameObject setPoolObj, string typeName)
    {
        setPoolObj.transform.SetParent(CheckPoolType(typeName));
        setPoolObj.SetActive(false);
    }
```
SetPooling 을 사용하여 풀링 타입 검사 후 없으면 타입을 만들고 있으면 그 타입에 자식으로 만들어 넣습니다.

```
public void SetPool(GameObject setPoolObj, string typeName, float time)
    {
        StartCoroutine(SetPoolingDelay(setPoolObj, typeName, time));
    }

    private IEnumerator SetPoolingDelay(GameObject setPoolObj, string typeName, float time)
    {
        yield return new WaitForSeconds(time);
        SetPool(setPoolObj, typeName);
        yield break;
    }
```
뒤에 시간을 넣게 되면, 
 코루틴으로 타이머가 돈 뒤 풀링을 시켜줍니다.

#### GetPool
```
 public GameObject GetPool(string typeName, Transform parent = null)
    {
        GameObject getPoolObj;

        Transform pooledTypeParent = CheckGetPoolType(typeName);

        if (pooledTypeParent == null)
            return null;

        else if (pooledTypeParent.childCount <= 2)
            return null;

        getPoolObj = pooledTypeParent.GetChild(0).gameObject;
        getPoolObj.SetActive(true);
        getPoolObj.transform.SetParent(parent);
        getPoolObj.transform.localPosition = Vector3.zero;

        return getPoolObj;
    }
```
풀링 타입을 입력하면 맨 위에 있는 오브젝트를 꺼내줍니다.


### AudioManager

#### PlaySound
```
public void PlaySound(string clipName, Vector3 pos, Transform parent = null, float pitch = 1f)
    {
        //clips에 있는 모든 AudioClip을 순환하며, 전달받은 clipName과 같은 이름의 클립을 찾아줌
        foreach (AudioClip ac in clips)
        {
            if (ac.name == clipName)
            {
                //찾은 소리를 실제로 재생해달라고 패러미터 넘겨줌
                PlaySound(ac, pos, parent, pitch);
            }
        }
    }
```
파일 이름으로 플레이 하거나
이름들을 넘겨 랜덤으로 플레이를 해줍니다.

소리나는 위치, 부모로 둘 오브젝트, 피치를 변경하여 넣을 수 도 있습니다.

```
public void PlaySound(AudioClip audioClip, Vector3 pos, Transform parent = null, float pitch = 1f)
    {
        //오디오 재생용 프리팝 생성
        AudioSource audioInstance = Instantiate(audioPrefab, pos, Quaternion.identity);

        //소리를 내는 사물을 따라가야 하는 경우, 그 사물의 자식으로 넣어줌
        if (parent != null)
            audioInstance.transform.SetParent(parent);

        audioInstance.clip = audioClip;     //클립을 바꿔주고
        audioInstance.pitch = pitch;        //피치값 설정
        audioInstance.volume = PlayerPrefs.GetInt("EffectSound", 1);
        audioInstance.Play();               //재생

        Destroy(audioInstance.gameObject, audioClip.length);       //오디오클립을 재생 후 인스턴스 삭제
        PoolingManager.Instance.SetPool(audioInstance.gameObject, "Audio", audioClip.length);       // 만약 PoolingManager를 사용하고 있다면 위에 Destroy를 지운뒤 이것 사용
    }
```
실질적인 사운드를 플레이 하는 함수 입니다.

### PoolingManager

팝업 창이 나올때 눌러도 괜찮은 UI와 누르면 안되는 UI를 나눠 터치를 막습니다. 

현재 까지의 팝업창의 갯수가 0 초과일 경우에 UI터치를 막습니다.

```
 public int PopupCount
    {
        get => popupCount;
        set
        {
            popupCount = value;

            if (PopupCount < 0)
                PopupCount = 0;

            if (PopupCount > 0)
            {
                for (int i = 0; i < popupNotPanel.Length; i++)
                    popupNotPanel[i].SetActive(false);

                for (int i = 0; i < popupNotBtn.Length; i++)
                    popupNotBtn[i].enabled = false;

                switch(nowSceneKind)
                {
                    case SceneKind.STAGE:
                        stageTouchManager.CanTouch = false;
                        break;
                    case SceneKind.MAIN:

                        break;
                    case SceneKind.GATCHA:
                        break;
                    default: break;
                }
            }
            else
            {
                for (int i = 0; i < popupNotPanel.Length; i++)
                    popupNotPanel[i].SetActive(true);

                for (int i = 0; i < popupNotBtn.Length; i++)
                    popupNotBtn[i].enabled = true;

                switch (nowSceneKind)
                {
                    case SceneKind.STAGE:
                        stageTouchManager.CanTouch = true;
                        break;
                    case SceneKind.MAIN:
                        break;
                    case SceneKind.GATCHA:
                        break;
                    default: break;
                }
            }
        }
    }
```
팝업 카운트를 하나씩 늘려가면서 0이 아닐경우 터치를 제한합니다.

### ObserverPatternClass 
```
public abstract class AchievementSubject : Singleton<AchievementSubject>
{
    [SerializeField]
    protected List<AchievementObserver> observers = new List<AchievementObserver>();

    /// <summary>
    /// 옵저버 더하는 함수
    /// </summary>
    public abstract void AddObserver(AchievementObserver observer);

    /// <summary>
    /// 옵저버 제거하는 함수
    /// </summary>
    public abstract void RemoveObserver(AchievementObserver observer);

    /// <summary>
    /// 옵저버 검사 후 실행
    /// </summary>
    public abstract void Notify();
}
```
옵저버는 AchievementObserver를 상속해서 사용합니다.

조건 달성은 AchievementSubject을 상속해서 사용합니다.

```
public abstract class AchievementObserver : MonoBehaviour
{
    //public GooglePlayGameManager GPGM;

    public abstract void GetAcheivement();
}
```
특정한 조건에 옵저버들을 감시해야 할 사항이 생기면 AchievementSubject의 Notify()를 실행시키면 옵저버들의 조건을 검사 합니다.

그 뒤에 옵저버의 조건 달성 이벤트를 실행시켜줍니다.

### SaveManager

```
public abstract class SaveData<T> : MonoBehaviour
{
    
    private T data;

    /// <summary>
    /// 제네릭 자료형 데이터 프로퍼티
    /// </summary>
    public T Data { get => data; set => data = value; }

    /// <summary>
    /// 데이터 디폴트값으로 설정하는 함수
    /// </summary>
    public abstract void SetDefaultData();

    /// <summary>
    /// 이름으로 검색 후 수정
    /// 그런뒤 그 데이터 반환
    /// </summary>
    /// <param name="value"></param>
    /// <returns></returns>
    public abstract T EditData(string value);

    /// <summary>
    /// 인덱스로 검색 후 수정
    /// 그런뒤 그 데이터 반환
    /// </summary>
    /// <param name="index"></param>
    /// <returns></returns>
    public abstract T EditData(int index);
}
```
각각 DefaultData를 만들어 세이브 데이터가 없을때 호출 합니다.

```
public class SaveDataManager : DataManager<GameData>
{
    [Header("각 데이터 매니저들")]
    public PlayerDataManager playerDataManager;
    public AnimalDataManager animalDataManager;
    public ItemDataManager itemDataManager;
    public LetterDataManager letterDataManager;
    '
    '
    '
}
```
원하는 데이터들을 만들어 새로운 데이터 클래스를 제네릭으로 받고 상속하여 사용 합니다.

Letter,Player,Item,Animal은 직접 사용했습니다.

이렇게 사용하게 되면 여러가지 데이터들을 한번에 묶어서 파일을 저장할 수 있게 되어 관리하기 편해집니다.

### Finity State Machine

#### FSM State
```
    public abstract void OnEnter(GameObject obj = null);

    public abstract void OnExecute(GameObject obj = null);

    public abstract void OnExit(GameObject obj = null);
```

원하는 상태를 이 함수를 상속하여 제작 합니다.

OnEnter()는 이 상태가 처음 들어갈때 한번 실행되는 함수 입니다.

OnExecute() 는 이 상태가 들어왔을때 반복문에서 실행되는 함수 입니다.

OnExit() 는 이 상태가 나갈때 한번 실행되는 함수 입니다.

#### FSM Manager

##### SetState
```
    private Dictionary<string, FSMState> stateMachine = new Dictionary<string, FSMState>();
    protected Dictionary<string, FSMState> StateMachine { get => stateMachine; set => stateMachine = value; }
    
    public virtual void SetStateComponent()
    {
        Debug.Log("SetStateComponent");
    }
```

이부분에 Dictionanry로 원하는 FSMState를 상속한 컴포넌트를 넣어 키값 설정 합니다.

##### Update
```
    protected FSMState curState;
    protected FSMState CurState { get => curState; set => curState = value; }
public virtual void FSMUpdate()
    {
        if (curState != null)
        {
            curState.OnExecute(target.gameObject);
        }
    }
```
CurState 가 존재하면 반복문으로 실행시킵니다.

##### ChangeState
```
 public void ChangeState(FSMState changeState, GameObject obj = null)
    {
        if (obj == null) obj = gameObject;

        // 현재상태랑 바꾸려는 상태가 같다면 
        if (curState == changeState) { return; }
        // 바꾸려는 상태가 없다면
        if (changeState == null) { return; }
        // 현재 상태가 없지 않다면 Exit시킴
        if (curState != null) { curState.OnExit(obj); }

        // Exit후 현재 상태를 바꾸려는 상태로 전환 후 Enter실행
        curState = changeState;
        curState.OnEnter(obj);
    }
```
현재 상태의 OnExit()를 실생시킨 뒤 상태를 바꾸고 OnEnter()를 실행 시킵니다.

### Singleton

씬마다 무조건 1개 있어야 하며, 2개 이상 있을경우가 없고 자원을 공유 해야 하는 컴포넌트에서만 사용 부탁 드립니다.

만약 호출 빈도가 적거나 위에 원칙에 해당이 안되는 경우라면, 사용하지 않는 것이 더 좋습니다.

이 문서에는 PoolingManager, AudioManager, PopupManager, SaveData 정도에 사용했습니다.

위 4개의 컴포넌트는 자원을 공유해야 하는 컴포넌트 이기 때문에 사용했습니다.

### JsonManager

제이슨을 사용하는 경우에 제이슨 에 맞는 클래스를 제작 후 상속해서 사용 합니다.

Awake()에서 실행되며,

Awake()가 실행될 때 이 컴포넌트를 사용하는 경우가 있을 경우 유니티 엔진 내에서 스크립트 실행 순서를 바꿔줘야 에러 없이 실행됩니다.