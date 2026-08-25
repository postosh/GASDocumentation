# GASDocumentation

간단한 멀티플레이어 샘플 프로젝트와 함께 정리한 언리얼 엔진 5(Unreal Engine 5)의 게임플레이 어빌리티 시스템(GameplayAbilitySystem, GAS) 플러그인에 대한 가이드입니다. 본 문서는 공식 문서가 아니며, 이 프로젝트와 작성자는 Epic Games와 어떠한 제휴 관계도 없습니다. 본 정보의 정확성을 완전히 보증하지는 않습니다.

이 문서의 목표는 GAS의 주요 개념과 클래스를 설명하고, 저의 실무 경험을 바탕으로 한 추가적인 팁과 해설을 제공하는 것입니다. 커뮤니티의 GAS 사용자들 사이에는 공유되지 않은 실무 지식(암묵지)이 많이 존재하며, 제가 알고 있는 모든 지식을 여기에 공유하고자 합니다.

샘플 프로젝트와 문서는 **Unreal Engine 5.3** (UE5)을 기준으로 작성되었습니다. 구버전 언리얼 엔진용 브랜치도 존재하지만, 더 이상 유지보수되지 않으며 버그나 오래된 정보가 포함되어 있을 수 있습니다. 엔진 버전에 맞는 브랜치를 사용해 주시기 바랍니다.

[GASShooter](https://github.com/tranek/GASShooter)는 멀티플레이어 FPS/TPS를 위한 GAS의 고급 테크닉을 다루는 자매 샘플 프로젝트입니다.

언제나 가장 훌륭한 문서는 플러그인의 소스 코드 그 자체입니다.

<a name="table-of-contents"></a>
## 목차 (Table of Contents)

> 1. [GameplayAbilitySystem 플러그인 소개](#intro)
> 1. [샘플 프로젝트](#sp)
> 1. [GAS를 사용한 프로젝트 설정](#setup)
> 1. [개념 (Concepts)](#concepts)  
>    4.1 [어빌리티 시스템 컴포넌트 (Ability System Component)](#concepts-asc)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.1.1 [리플리케이션 모드 (Replication Mode)](#concepts-asc-rm)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.1.2 [설정 및 초기화 (Setup and Initialization)](#concepts-asc-setup)  
>    4.2 [게임플레이 태그 (Gameplay Tags)](#concepts-gt)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.2.1 [게임플레이 태그 변경 감지 (Responding to Changes in Gameplay Tags)](#concepts-gt-change)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.2.2 [플러그인 .ini 파일에서 게임플레이 태그 로드](#concepts-gt-loadfromplugin)  
>    4.3 [어트리뷰트 (Attributes)](#concepts-a)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.1 [어트리뷰트 정의 (Attribute Definition)](#concepts-a-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.2 [BaseValue vs CurrentValue](#concepts-a-value)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.3 [메타 어트리뷰트 (Meta Attributes)](#concepts-a-meta)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.4 [어트리뷰트 변경 감지 (Responding to Attribute Changes)](#concepts-a-changes)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.5 [파생 어트리뷰트 (Derived Attributes)](#concepts-a-derived)  
>    4.4 [어트리뷰트 세트 (Attribute Set)](#concepts-as)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.1 [어트리뷰트 세트 정의](#concepts-as-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2 [어트리뷰트 세트 설계](#concepts-as-design)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.1 [개별 어트리뷰트를 가진 서브컴포넌트](#concepts-as-design-subcomponents)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.2 [런타임에 AttributeSet 추가 및 제거](#concepts-as-design-addremoveruntime)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.3 [아이템 어트리뷰트 (무기 탄약 등)](#concepts-as-design-itemattributes)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.3.1 [아이템의 단순 float 변수](#concepts-as-design-itemattributes-plainfloats)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.3.2 [아이템에 `AttributeSet` 배치](#concepts-as-design-itemattributes-attributeset)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.3.3 [아이템에 `ASC` 배치](#concepts-as-design-itemattributes-asc)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.3 [어트리뷰트 정의 매크로](#concepts-as-attributes)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.4 [어트리뷰트 초기화](#concepts-as-init)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.5 [PreAttributeChange()](#concepts-as-preattributechange)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.6 [PostGameplayEffectExecute()](#concepts-as-postgameplayeffectexecute)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.7 [OnAttributeAggregatorCreated()](#concepts-as-onattributeaggregatorcreated)  
>    4.5 [게임플레이 이펙트 (Gameplay Effects)](#concepts-ge)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.1 [게임플레이 이펙트 정의](#concepts-ge-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.2 [게임플레이 이펙트 적용](#concepts-ge-applying)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.3 [게임플레이 이펙트 제거](#concepts-ga-removing)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.4 [게임플레이 이펙트 모디파이어 (Modifiers)](#concepts-ge-mods)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.4.1 [곱셈(Multiply) 및 나눗셈(Divide) 모디파이어](#concepts-ge-mods-multiplydivide)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.4.2 [모디파이어의 게임플레이 태그](#concepts-ge-mods-gameplaytags)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.5 [게임플레이 이펙트 스태킹 (중첩)](#concepts-ge-stacking)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.6 [어빌리티 부여 (Granted Abilities)](#concepts-ge-ga)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.7 [게임플레이 이펙트 태그](#concepts-ge-tags)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.8 [면역 (Immunity)](#concepts-ge-immunity)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.9 [게임플레이 이펙트 스펙 (Gameplay Effect Spec)](#concepts-ge-spec)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.9.1 [SetByCallers](#concepts-ge-spec-setbycaller)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.10 [게임플레이 이펙트 컨텍스트 (Gameplay Effect Context)](#concepts-ge-context)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.11 [모디파이어 크기 계산 (Modifier Magnitude Calculation - MMC)](#concepts-ge-mmc)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12 [게임플레이 이펙트 실행 계산 (Gameplay Effect Execution Calculation)](#concepts-ge-ec)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1 [실행 계산으로 데이터 전달](#concepts-ge-ec-senddata)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1.1 [SetByCaller](#concepts-ge-ec-senddata-setbycaller)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1.2 [백킹 데이터 어트리뷰트 계산 모디파이어](#concepts-ge-ec-senddata-backingdataattribute)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1.3 [백킹 데이터 임시 변수 계산 모디파이어](#concepts-ge-ec-senddata-backingdatatempvariable)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1.4 [게임플레이 이펙트 컨텍스트](#concepts-ge-ec-senddata-effectcontext)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.13 [커스텀 적용 조건 (Custom Application Requirement)](#concepts-ge-car)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.14 [비용 게임플레이 이펙트 (Cost Gameplay Effect)](#concepts-ge-cost)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.15 [쿨다운 게임플레이 이펙트 (Cooldown Gameplay Effect)](#concepts-ge-cooldown)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.15.1 [쿨다운 게임플레이 이펙트 잔여 시간 가져오기](#concepts-ge-cooldown-tr)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.15.2 [쿨다운 시작 및 종료 감지](#concepts-ge-cooldown-listen)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.15.3 [쿨다운 예측 (Predicting Cooldowns)](#concepts-ge-cooldown-prediction)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.16 [활성 게임플레이 이펙트의 지속 시간 변경](#concepts-ge-duration)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.17 [런타임에 동적 게임플레이 이펙트 생성](#concepts-ge-dynamic)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.18 [게임플레이 이펙트 컨테이너 (Gameplay Effect Containers)](#concepts-ge-containers)  
>    4.6 [게임플레이 어빌리티 (Gameplay Abilities)](#concepts-ga)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.1 [게임플레이 어빌리티 정의](#concepts-ga-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.1.1 [리플리케이션 정책 (Replication Policy)](#concepts-ga-definition-reppolicy)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.1.2 [서버의 원격 어빌리티 취소 준수 (Server Respects Remote Ability Cancellation)](#concepts-ga-definition-remotecancel)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.1.3 [입력 직접 리플리케이트 (Replicate Input Directly)](#concepts-ga-definition-repinputdirectly)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.2 [입력을 ASC에 바인딩](#concepts-ga-input)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.2.1 [어빌리티 활성화 없이 입력에 바인딩](#concepts-ga-input-noactivate)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.3 [어빌리티 부여 (Granting Abilities)](#concepts-ga-granting)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.4 [어빌리티 활성화 (Activating Abilities)](#concepts-ga-activating)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.4.1 [패시브 어빌리티 (Passive Abilities)](#concepts-ga-activating-passive)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.4.2 [활성화 실패 태그 (Activation Failed Tags)](#concepts-ga-activating-failedtags)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.5 [어빌리티 취소 (Canceling Abilities)](#concepts-ga-cancelabilities)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.6 [활성 어빌리티 가져오기 (Getting Active Abilities)](#concepts-ga-definition-activeability)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.7 [인스턴싱 정책 (Instancing Policy)](#concepts-ga-instancing)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.8 [넷 실행 정책 (Net Execution Policy)](#concepts-ga-net)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.9 [어빌리티 태그 (Ability Tags)](#concepts-ga-tags)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.10 [게임플레이 어빌리티 스펙 (Gameplay Ability Spec)](#concepts-ga-spec)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.11 [어빌리티로 데이터 전달](#concepts-ga-data)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.12 [어빌리티 비용 및 쿨다운 (Cost and Cooldown)](#concepts-ga-commit)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.13 [어빌리티 레벨업 (Leveling Up Abilities)](#concepts-ga-leveling)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.14 [어빌리티 세트 (Ability Sets)](#concepts-ga-sets)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.15 [어빌리티 배칭 (Ability Batching)](#concepts-ga-batching)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.16 [넷 보안 정책 (Net Security Policy)](#concepts-ga-netsecuritypolicy)  
>    4.7 [어빌리티 태스크 (Ability Tasks)](#concepts-at)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.7.1 [어빌리티 태스크 정의](#concepts-at-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.7.2 [커스텀 어빌리티 태스크](#concepts-at-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.7.3 [어빌리티 태스크 사용법](#concepts-at-using)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.7.4 [루트 모션 소스 어빌리티 태스크](#concepts-at-rms)  
>    4.8 [게임플레이 큐 (Gameplay Cues)](#concepts-gc)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.1 [게임플레이 큐 정의](#concepts-gc-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.2 [게임플레이 큐 트리거](#concepts-gc-trigger)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.3 [로컬 게임플레이 큐 (Local Gameplay Cues)](#concepts-gc-local)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.4 [게임플레이 큐 매개변수 (Parameters)](#concepts-gc-parameters)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.5 [게임플레이 큐 매니저 (Gameplay Cue Manager)](#concepts-gc-manager)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.6 [게임플레이 큐 발동 방지](#concepts-gc-prevention)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.7 [게임플레이 큐 배칭 (Gameplay Cue Batching)](#concepts-gc-batching)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.7.1 [수동 RPC](#concepts-gc-batching-manualrpc)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.7.2 [하나의 GE에 여러 GC 설정](#concepts-gc-batching-gcsonge)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.8 [게임플레이 큐 이벤트 (Gameplay Cue Events)](#concepts-gc-events)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.9 [게임플레이 큐 신뢰성 (Reliability)](#concepts-gc-reliability)  
>    4.9 [어빌리티 시스템 글로벌 (Ability System Globals)](#concepts-asg)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.9.1 [InitGlobalData()](#concepts-asg-initglobaldata)  
>    4.10 [예측 (Prediction)](#concepts-p)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.1 [예측 키 (Prediction Key)](#concepts-p-key)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.2 [어빌리티 내에서 새 예측 윈도우 생성](#concepts-p-windows)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.3 [액터 예측 스폰 (Predictively Spawning Actors)](#concepts-p-spawn)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.4 [GAS 예측의 미래](#concepts-p-future)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.5 [네트워크 예측 플러그인 (Network Prediction Plugin)](#concepts-p-npp)  
>    4.11 [타겟팅 (Targeting)](#concepts-targeting)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.1 [타겟 데이터 (Target Data)](#concepts-targeting-data)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.2 [타겟 액터 (Target Actors)](#concepts-targeting-actors)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.3 [타겟 데이터 필터 (Target Data Filters)](#concepts-target-data-filters)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.4 [게임플레이 어빌리티 월드 레티클 (World Reticles)](#concepts-targeting-reticles)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.5 [게임플레이 이펙트 컨테이너 타겟팅](#concepts-targeting-containers)  
> 1. [자주 구현되는 어빌리티 및 이펙트 예제](#cae)  
>    5.1 [기절 (Stun)](#cae-stun)  
>    5.2 [질주 (Sprint)](#cae-sprint)  
>    5.3 [정조준 (Aim Down Sights - ADS)](#cae-ads)  
>    5.4 [생명력 흡수 (Lifesteal)](#cae-ls)  
>    5.5 [클라이언트와 서버에서 동일한 난수 생성](#cae-random)  
>    5.6 [치명타 (Critical Hits)](#cae-crit)  
>    5.7 [중첩되지 않지만 가장 큰 수치만 대상에 적용되는 게임플레이 이펙트](#cae-nonstackingge)  
>    5.8 [게임 일시정지 중 타겟 데이터 생성](#cae-paused)  
>    5.9 [단일 버튼 상호작용 시스템 (One Button Interaction System)](#cae-onebuttoninteractionsystem)  
> 1. [GAS 디버깅](#debugging)  
>    6.1 [showdebug abilitysystem](#debugging-sd)  
>    6.2 [게임플레이 디버거 (Gameplay Debugger)](#debugging-gd)  
>    6.3 [GAS 로깅 (GAS Logging)](#debugging-log)  
> 1. [최적화 (Optimizations)](#optimizations)  
>    7.1 [어빌리티 배칭 (Ability Batching)](#optimizations-abilitybatching)  
>    7.2 [게임플레이 큐 배칭 (Gameplay Cue Batching)](#optimizations-gameplaycuebatching)  
>    7.3 [어빌리티 시스템 컴포넌트 리플리케이션 모드](#optimizations-ascreplicationmode)  
>    7.4 [어트리뷰트 프록시 리플리케이션 (Attribute Proxy Replication)](#optimizations-attributeproxyreplication)  
>    7.5 [ASC 지연 로딩 (ASC Lazy Loading)](#optimizations-asclazyloading)  
> 1. [개발 편의성(QoL) 제안](#qol)  
>    8.1 [게임플레이 이펙트 컨테이너 (Gameplay Effect Containers)](#qol-gameplayeffectcontainers)  
>    8.2 [ASC 델리게이트에 바인딩하는 블루프린트 AsyncTask](#qol-asynctasksascdelegates)  
> 1. [문제 해결 (Troubleshooting)](#troubleshooting)  
>    9.1 [`LogAbilitySystem: Warning: Can't activate LocalOnly or LocalPredicted ability %s when not local!`](#troubleshooting-notlocal)  
>    9.2 [`ScriptStructCache` 오류](#troubleshooting-scriptstructcache)  
>    9.3 [애니메이션 몽타주가 클라이언트에 리플리케이트되지 않음](#troubleshooting-replicatinganimmontages)  
>    9.4 [블루프린트 액터 복제 시 AttributeSet이 nullptr로 설정됨](#troubleshooting-duplicatingblueprintactors)  
>    9.5 [미해결 외부 기호 UEPushModelPrivate::MarkPropertyDirty(int,int)](#troubleshooting-unresolvedexternalsymbolmarkpropertydirty)  
>    9.6 [열거형 이름이 경로 이름으로 표현되는 문제](#troubleshooting-enumnamesarenowpathnames)  
> 1. [자주 사용되는 GAS 약어 모음](#acronyms)
> 1. [기타 참고 자료](#resources)  
>    11.1 [Epic Games 개발자 Dave Ratti와의 Q&A](#resources-daveratti)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;11.1.1 [커뮤니티 질문 1](#resources-daveratti-community1)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;11.1.2 [커뮤니티 질문 2](#resources-daveratti-community2)  
> 1. [GAS 변경 로그 (Changelog)](#changelog)  
>    * [5.3](#changelog-5.3)  
>    * [5.2](#changelog-5.2)  
>    * [5.1](#changelog-5.1)  
>    * [5.0](#changelog-5.0)  
>    * [4.27](#changelog-4.27)  
>    * [4.26](#changelog-4.26)  
>    * [4.25.1](#changelog-4.25.1)  
>    * [4.25](#changelog-4.25)  
>    * [4.24](#changelog-4.24)
          
<a name="intro"></a>
## 1. GameplayAbilitySystem 플러그인 소개
[공식 문서](https://docs.unrealengine.com/en-US/Gameplay/GameplayAbilitySystem/index.html) 발췌:
> 게임플레이 어빌리티 시스템(Gameplay Ability System)은 RPG나 MOBA 타이틀에서 흔히 볼 수 있는 유형의 어빌리티와 어트리뷰트를 구축하기 위한 유연성이 매우 뛰어난 프레임워크입니다. 게임 내 캐릭터가 사용할 액션이나 패시브 어빌리티, 이러한 액션의 결과로 다양한 어트리뷰트를 강화하거나 소모시키는 상태 효과를 만들 수 있으며, 액션 사용을 제어하기 위한 '쿨다운' 타이머나 자원 소모 비용을 구현하고, 레벨별로 어빌리티와 효과를 변경하거나 파티클 및 사운드 이펙트를 활성화하는 등의 작업이 가능합니다. 간단히 말해, 이 시스템은 단순한 점프부터 현대 RPG 및 MOBA의 복잡한 캐릭터 어빌리티 셋에 이르기까지 게임 내 어빌리티를 효율적으로 설계, 구현 및 네트워크 동기화할 수 있도록 지원합니다.

GameplayAbilitySystem 플러그인은 Epic Games에서 개발하여 Unreal Engine에 기본 포함되어 있습니다. 파라곤(Paragon), 포트나이트(Fortnite) 등 여러 상용 AAA 게임에서 실전 검증을 거쳤습니다.

이 플러그인은 싱글플레이 및 멀티플레이 게임에서 다음과 같은 기능들을 즉시 사용할 수 있도록 제공합니다:
* 선택적 비용(Cost) 및 쿨다운(Cooldown)을 가진 레벨 기반 캐릭터 어빌리티/스킬 구현 ([GameplayAbilities](#concepts-ga))
* 액터가 보유한 수치형 `Attributes` 조작 ([Attributes](#concepts-a))
* 액터에 상태 효과 적용 ([GameplayEffects](#concepts-ge))
* 액터에 `GameplayTags` 적용 ([GameplayTags](#concepts-gt))
* 시각/사운드 이펙트 스폰 ([GameplayCues](#concepts-gc))
* 위에 언급된 모든 요소의 네트워크 리플리케이션(Replication)

멀티플레이어 게임에서 GAS는 다음과 같은 요소들의 [클라이언트 측 예측(Client-side prediction)](#concepts-p)을 지원합니다:
* 어빌리티 활성화
* 애니메이션 몽타주 재생
* `Attributes` 변경
* `GameplayTags` 적용
* `GameplayCues` 스폰
* `CharacterMovementComponent`와 연동된 `RootMotionSource` 함수를 통한 이동

**GAS의 기초 설정은 반드시 C++로 진행해야 하지만**, `GameplayAbilities`와 `GameplayEffects`의 제작은 기획자/디자이너가 블루프린트에서 진행할 수 있습니다.

현재 GAS의 알려진 한계점 및 이슈:
* `GameplayEffect` 지연 시간 보정(Latency Reconciliation) 문제: 어빌리티 쿨다운을 예측할 수 없기 때문에, 쿨다운이 짧은 어빌리티의 경우 핑(레이턴시)이 높은 플레이어는 핑이 낮은 플레이어에 비해 발사/시전 속도가 느려집니다.
* `GameplayEffect`의 제거는 예측할 수 없습니다. 다만 반대 효과를 가진 `GameplayEffect`를 추가하여 사실상 효과를 상쇄/제거하도록 예측할 수는 있습니다. 하지만 이 방식이 항상 적절하거나 구현 가능한 것은 아니므로 여전히 이슈로 남아 있습니다.
* 보일러플레이트 템플릿, 멀티플레이어 예제 및 공식 문서가 부족합니다. 이 문서가 그 갈증을 해소하는 데 도움이 되기를 바랍니다!

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="sp"></a>
## 2. 샘플 프로젝트
이 문서에는 GameplayAbilitySystem 플러그인을 처음 접하지만 언리얼 엔진 자체에는 익숙한 개발자를 위한 멀티플레이어 3인칭 슈팅 샘플 프로젝트가 포함되어 있습니다. C++, 블루프린트, UMG, 리플리케이션 등 언리얼 엔진의 중급 주제들에 대한 지식이 이미 있다고 가정합니다. 이 프로젝트는 플레이어/AI 조종 영웅의 경우 `PlayerState` 클래스에 `AbilitySystemComponent` (`ASC`)를 두고, AI 조종 미니언의 경우 `Character` 클래스에 `ASC`를 배치하여 멀티플레이어 환경에 대응하는 3인칭 슈터의 기본 셋업 예시를 보여줍니다.

목표는 GAS의 기초를 명확히 전달하면서 실무에서 자주 요청되는 어빌리티들을 주석이 충실히 달린 코드로 보여주어 프로젝트를 최대한 간결하게 유지하는 것입니다. 초심자를 대상으로 하므로 [발사체 예측(Projectiles Prediction)](#concepts-p-spawn)과 같은 고급 주제는 다루지 않습니다.

샘플 프로젝트에서 다루는 개념들:
* `PlayerState`의 `ASC` vs `Character`의 `ASC`
* 리플리케이트되는 `Attributes`
* 리플리케이트되는 애니메이션 몽타주
* `GameplayTags`
* `GameplayAbilities` 내부 및 외부에서 `GameplayEffects` 적용 및 제거
* 방어력(Armor)으로 피해를 경감시켜 캐릭터의 체력을 감소시키는 대미지 적용
* `GameplayEffectExecutionCalculations` (실행 계산)
* 기절(Stun) 효과
* 사망 및 리스폰
* 서버의 어빌리티에서 액터(발사체) 스폰
* 정조준(ADS) 및 질주 시 로컬 플레이어의 이동 속도를 예측하여 변경
* 질주를 위해 스태미나를 지속적으로 소모
* 어빌리티 시전에 마나 사용
* 패시브 어빌리티 (Passive Abilities)
* `GameplayEffects` 스태킹 (중첩)
* 타겟 액터 (Targeting actors)
* 블루프린트로 제작된 `GameplayAbilities`
* C++로 제작된 `GameplayAbilities`
* 액터당 인스턴스화되는(Instanced per Actor) `GameplayAbilities`
* 인스턴스화되지 않는(Non-Instanced) `GameplayAbilities` (점프)
* 정적 `GameplayCues` (총 발사 투사체 충돌 파티클 이펙트)
* 액터 `GameplayCues` (질주 및 기절 파티클 이펙트)

영웅(Hero) 클래스는 다음과 같은 어빌리티를 가지고 있습니다:

| 어빌리티 (Ability)        | 입력 바인딩 (Input Bind) | 예측 여부 (Predicted) | C++ / 블루프린트 | 설명                                                                                                                                                                         |
| ------------------------- | ------------------------ | ---------------------- | ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Jump (점프)               | Space Bar                | 예 (Yes)               | C++              | 영웅이 점프합니다.                                                                                                                                                           |
| Gun (총 발사)             | 마우스 좌클릭            | 아니오 (No)            | C++              | 영웅의 총에서 발사체를 발사합니다. 애니메이션은 예측되지만 발사체 스폰 자체는 예측되지 않습니다.                                                                              |
| Aim Down Sights (정조준)  | 마우스 우클릭            | 예 (Yes)               | Blueprint        | 버튼을 누르고 있는 동안 영웅의 이동 속도가 느려지고 카메라가 줌인되어 총기를 더 정밀하게 조준할 수 있습니다.                                                                  |
| Sprint (질주)             | Left Shift               | 예 (Yes)               | Blueprint        | 버튼을 누르고 있는 동안 영웅이 스태미나를 소모하며 더 빠르게 달립니다.                                                                                                       |
| Forward Dash (전방 대시)  | Q                        | 예 (Yes)               | Blueprint        | 영웅이 스태미나를 소모하여 전방으로 돌진합니다.                                                                                                                              |
| Passive Armor Stacks      | 패시브 (Passive)         | 아니오 (No)            | Blueprint        | 4초마다 영웅이 방어력 중첩을 1스택씩 얻어 최대 4스택까지 쌓입니다. 피해를 입으면 방어력 스택이 1개 소모됩니다.                                                              |
| Meteor (메테오)           | R                        | 아니오 (No)            | Blueprint        | 플레이어가 지점을 지정하여 적들에게 운석을 떨어뜨려 피해를 입히고 기절시킵니다. 타겟팅은 예측되지만 메테오 스폰 자체는 서버에서 처리됩니다.                                 |

`GameplayAbilities`를 C++로 만들든 블루프린트로 만들든 상관없습니다. 여기서는 두 언어에서 각각 구현하는 방법을 보여주기 위해 혼합하여 사용했습니다.

미니언(Minion)은 사전 정의된 `GameplayAbilities`를 가지고 있지 않습니다. 레드 미니언은 체력 재생력이 더 높고, 블루 미니언은 시작 체력이 더 높습니다.

`GameplayAbility` 명명 규칙의 경우, 블루프린트로 로직이 작성된 어빌리티에는 `_BP` 접미사를 붙였습니다. 접미사가 없는 것은 C++로 로직이 작성되었음을 의미합니다.

**블루프린트 에셋 명명 접두사 (Blueprint Asset Naming Prefixes)**

| 접두사 (Prefix) | 에셋 타입 (Asset Type) |
| --------------- | ---------------------- |
| GA_             | GameplayAbility        |
| GC_             | GameplayCue            |
| GE_             | GameplayEffect         |

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="setup"></a>
## 3. GAS를 사용한 프로젝트 설정
GAS를 사용하는 프로젝트의 기본 설정 단계:
1. 언리얼 에디터의 플러그인 메뉴에서 **GameplayAbilitySystem** 플러그인을 활성화합니다.
1. `YourProjectName.Build.cs` 파일을 열어 `PrivateDependencyModuleNames`에 `"GameplayAbilities", "GameplayTags", "GameplayTasks"`를 추가합니다.
1. Visual Studio 프로젝트 파일을 새로고침/재생성(Refresh/Regenerate)합니다.
1. 4.24부터 5.2까지는 [`TargetData`](#concepts-targeting-data)를 사용하기 위해 `UAbilitySystemGlobals::Get().InitGlobalData()`를 반드시 호출해야 했습니다. 샘플 프로젝트는 `UAssetManager::StartInitialLoading()`에서 이를 호출합니다. 5.3부터는 이 함수가 자동으로 호출됩니다. 자세한 내용은 [`InitGlobalData()`](#concepts-asg-initglobaldata) 섹션을 참조하세요.

이것으로 GAS를 활성화하기 위한 모든 기본 설정이 끝났습니다. 이제 `Character`나 `PlayerState`에 [`ASC`](#concepts-asc)와 [`AttributeSet`](#concepts-as)을 추가하고, [`GameplayAbilities`](#concepts-ga)와 [`GameplayEffects`](#concepts-ge)를 만들어 사용하기 시작하면 됩니다!

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts"></a>

## 4. GAS 개념 (GAS Concepts)

#### 세부 섹션

> 4.1 [어빌리티 시스템 컴포넌트 (Ability System Component)](#concepts-asc)  
> 4.2 [게임플레이 태그 (Gameplay Tags)](#concepts-gt)  
> 4.3 [어트리뷰트 (Attributes)](#concepts-a)  
> 4.4 [어트리뷰트 세트 (Attribute Set)](#concepts-as)  
> 4.5 [게임플레이 이펙트 (Gameplay Effects)](#concepts-ge)  
> 4.6 [게임플레이 어빌리티 (Gameplay Abilities)](#concepts-ga)  
> 4.7 [어빌리티 태스크 (Ability Tasks)](#concepts-at)  
> 4.8 [게임플레이 큐 (Gameplay Cues)](#concepts-gc)  
> 4.9 [어빌리티 시스템 글로벌 (Ability System Globals)](#concepts-asg)  
> 4.10 [예측 (Prediction)](#concepts-p)

<a name="concepts-asc"></a>
### 4.1 어빌리티 시스템 컴포넌트 (Ability System Component)
AbilitySystemComponent (ASC)는 GAS의 핵심 심장부입니다. 이는 시스템과의 모든 상호작용을 처리하는 UActorComponent([UAbilitySystemComponent](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAbilitySystemComponent/index.html))입니다. [GameplayAbilities](#concepts-ga)를 사용하거나, [Attributes](#concepts-a)를 가지거나, [GameplayEffects](#concepts-ge)를 적용받고자 하는 모든 Actor는 반드시 ASC를 하나 부착해야 합니다. 이러한 객체들은 모두 ASC 내부에 상주하며, ASC에 의해 관리 및 리플리케이트됩니다 (단, Attributes는 해당 [AttributeSet](#concepts-as)에 의해 리플리케이트됨). 개발자는 필요에 따라 이를 서브클래싱할 수 있습니다(권장되지만 필수는 아님).

ASC가 부착된 Actor를 ASC의 OwnerActor(소유자 액터)라고 부릅니다. ASC의 물리적 표현을 담당하는 Actor를 AvatarActor(아바타 액터)라고 부릅니다. MOBA 게임의 단순 AI 미니언처럼 OwnerActor와 AvatarActor가 동일한 액터일 수도 있습니다. 반면 MOBA 게임의 플레이어 조종 영웅처럼 OwnerActor는 PlayerState이고 AvatarActor는 영웅의 Character 클래스인 경우처럼 서로 다른 액터일 수도 있습니다. 대부분의 액터는 자기 자신에 ASC를 둡니다. 하지만 캐릭터가 리스폰(Respawn)되어도 스폰 간에 Attributes나 GameplayEffects의 상태가 유지(영속성)되어야 한다면(예: MOBA의 영웅), ASC를 PlayerState에 두는 것이 이상적입니다.

**참고:** ASC를 PlayerState에 배치한 경우, PlayerState의 NetUpdateFrequency를 높여주어야 합니다. PlayerState의 기본 넷 업데이트 빈도는 매우 낮게 설정되어 있어 클라이언트에서 Attributes나 GameplayTags 등의 변경 사항이 반영될 때 딜레이나 랙이 발생할 수 있습니다. 포트나이트(Fortnite)에서도 사용하는 [Adaptive Network Update Frequency](https://docs.unrealengine.com/en-US/Gameplay/Networking/Actors/Properties/index.html#adaptivenetworkupdatefrequency)를 반드시 활성화하는 것을 권장합니다.

OwnerActor와 AvatarActor가 서로 다른 액터인 경우, 두 액터 모두 IAbilitySystemInterface 인터페이스를 구현해야 합니다. 이 인터페이스에는 반드시 오버라이드해야 하는 단 하나의 함수 UAbilitySystemComponent* GetAbilitySystemComponent() const가 있으며, 이는 자신의 ASC 포인터를 반환합니다. ASC들은 시스템 내부에서 서로 상호작용할 때 이 인터페이스 함수를 찾아 호출합니다.

ASC는 현재 활성화된 GameplayEffects를 FActiveGameplayEffectsContainer ActiveGameplayEffects에 보관합니다.

ASC는 자신에게 부여된 Gameplay Abilities를 FGameplayAbilitySpecContainer ActivatableAbilities에 보관합니다. ActivatableAbilities.Items를 순회(iteration)할 때마다, 루프문 바로 위에 반드시 ABILITYLIST_SCOPE_LOCK();을 추가하여 (어빌리티 제거 등으로 인해) 리스트가 도중에 변경되는 것을 방지해야 합니다. 스코프 내의 모든 ABILITYLIST_SCOPE_LOCK();은 AbilityScopeLockCount를 증가시키고 스코프를 벗어날 때 감소시킵니다. ABILITYLIST_SCOPE_LOCK(); 스코프 내부에서는 어빌리티 제거를 시도하지 마십시오 (어빌리티 제거 함수들은 내부적으로 AbilityScopeLockCount를 검사하여 리스트가 잠겨 있는 경우 제거를 방지합니다).

<a name="concepts-asc-rm"></a>
### 4.1.1 리플리케이션 모드 (Replication Mode)
ASC는 GameplayEffects, GameplayTags, GameplayCues를 리플리케이트하기 위해 Full, Mixed, Minimal 세 가지 리플리케이션 모드를 정의합니다. Attributes는 소속된 AttributeSet에 의해 리플리케이트됩니다.

| 리플리케이션 모드 (Replication Mode) | 사용 시점                                  | 설명                                                                                                                             |
| ------------------------------------ | ------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------- |
| Full                               | 싱글 플레이어 (Single Player)              | 모든 GameplayEffect가 모든 클라이언트에 리플리케이트됩니다.                                                                   |
| Mixed                              | 멀티플레이어, 플레이어 조종 Actors       | GameplayEffects는 소유 클라이언트(Owning Client)에만 리플리케이트됩니다. GameplayTags와 GameplayCues만 모두에게 리플리케이트됩니다. |
| Minimal                            | 멀티플레이어, AI 조종 Actors            | GameplayEffects는 누구에게도 리플리케이트되지 않습니다. GameplayTags와 GameplayCues만 모두에게 리플리케이트됩니다.        |

**참고:** Mixed 리플리케이션 모드는 OwnerActor의 Owner가 Controller일 것을 요구합니다. PlayerState의 Owner는 기본적으로 Controller이지만, Character는 그렇지 않습니다. PlayerState가 아닌 OwnerActor에서 Mixed 리플리케이션 모드를 사용하려면 OwnerActor에서 유효한 Controller를 인자로 하여 SetOwner()를 호출해야 합니다.

4.24 버전부터는 PossessedBy()가 Pawn의 소유자를 새로운 Controller로 자동 설정합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-asc-setup"></a>
### 4.1.2 설정 및 초기화 (Setup and Initialization)
ASC는 일반적으로 OwnerActor의 생성자에서 생성되며 명시적으로 리플리케이트되도록 설정됩니다. **이 작업은 반드시 C++로 작성해야 합니다.**

```c++
AGDPlayerState::AGDPlayerState()
{
	// 어빌리티 시스템 컴포넌트 생성 및 명시적 리플리케이션 설정
	AbilitySystemComponent = CreateDefaultSubobject<UGDAbilitySystemComponent>(TEXT("AbilitySystemComponent"));
	AbilitySystemComponent->SetIsReplicated(true);
	//...
}
```

ASC는 서버와 클라이언트 양쪽 모두에서 OwnerActor 및 AvatarActor 정보로 초기화되어야 합니다. 초기화는 Pawn의 Controller가 설정된 이후(빙의/Possession 이후)에 수행해야 합니다. 싱글플레이어 게임은 서버 경로만 신경 쓰면 됩니다.

ASC가 Pawn에 존재하는 플레이어 조종 캐릭터의 경우, 저는 일반적으로 서버에서는 Pawn의 PossessedBy() 함수에서 초기화하고, 클라이언트에서는 PlayerController의 AcknowledgePossession() 함수에서 초기화합니다.

```c++
void APACharacterBase::PossessedBy(AController * NewController)
{
	Super::PossessedBy(NewController);

	if (AbilitySystemComponent)
	{
		AbilitySystemComponent->InitAbilityActorInfo(this, this);
	}

	// ASC Mixed 모드 리플리케이션은 ASC Owner의 Owner가 Controller일 것을 요구합니다.
	SetOwner(NewController);
}
```

```c++
void APAPlayerControllerBase::AcknowledgePossession(APawn* P)
{
	Super::AcknowledgePossession(P);

	APACharacterBase* CharacterBase = Cast<APACharacterBase>(P);
	if (CharacterBase)
	{
		CharacterBase->GetAbilitySystemComponent()->InitAbilityActorInfo(CharacterBase, CharacterBase);
	}

	//...
}
```

ASC가 PlayerState에 존재하는 플레이어 조종 캐릭터의 경우, 서버에서는 Pawn의 PossessedBy() 함수에서 초기화하고, 클라이언트에서는 Pawn의 OnRep_PlayerState() 함수에서 초기화합니다. 이를 통해 클라이언트에 PlayerState가 확실히 존재하는 시점에 초기화할 수 있습니다.

```c++
// 서버 전용
void AGDHeroCharacter::PossessedBy(AController * NewController)
{
	Super::PossessedBy(NewController);

	AGDPlayerState* PS = GetPlayerState<AGDPlayerState>();
	if (PS)
	{
		// 서버에서 ASC 설정. 클라이언트는 OnRep_PlayerState()에서 수행
		AbilitySystemComponent = Cast<UGDAbilitySystemComponent>(PS->GetAbilitySystemComponent());

		// AI는 PlayerController를 가지지 않으므로 여기서 한 번 더 확실하게 초기화할 수 있습니다. PlayerController를 가진 영웅에게 두 번 초기화되어도 무방합니다.
		PS->GetAbilitySystemComponent()->InitAbilityActorInfo(PS, this);
	}
	
	//...
}
```

```c++
// 클라이언트 전용
void AGDHeroCharacter::OnRep_PlayerState()
{
	Super::OnRep_PlayerState();

	AGDPlayerState* PS = GetPlayerState<AGDPlayerState>();
	if (PS)
	{
		// 클라이언트용 ASC 설정. 서버는 PossessedBy에서 수행.
		AbilitySystemComponent = Cast<UGDAbilitySystemComponent>(PS->GetAbilitySystemComponent());

		// 클라이언트용 ASC Actor Info 초기화. 서버는 새로운 액터에 빙의할 때 ASC를 초기화합니다.
		AbilitySystemComponent->InitAbilityActorInfo(PS, this);
	}

	// ...
}
```

만약 LogAbilitySystem: Warning: Can't activate LocalOnly or LocalPredicted ability %s when not local! 오류 메시지가 나타난다면, 클라이언트에서 ASC를 초기화하지 않은 것입니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-gt"></a>
### 4.2 게임플레이 태그 (Gameplay Tags)
[FGameplayTags](https://docs.unrealengine.com/en-US/API/Runtime/GameplayTags/FGameplayTag/index.html)는 GameplayTagManager에 등록되는 Parent.Child.Grandchild... 형태의 계층적 이름입니다. 이 태그들은 객체의 상태를 분류하고 설명하는 데 매우 유용합니다. 예를 들어 캐릭터가 기절한 경우, 기절 지속 시간 동안 State.Debuff.Stun 게임플레이 태그를 부여할 수 있습니다.

기존에 bool 변수나 enum으로 처리하던 작업들을 GameplayTags로 대체하고, 객체가 특정 GameplayTags를 가지고 있는지 여부로 불리언 로직을 수행하게 될 것입니다.

객체에 태그를 부여할 때, 해당 객체에 ASC가 있다면 GAS가 상호작용할 수 있도록 주로 ASC에 태그를 추가합니다. UAbilitySystemComponent는 IGameplayTagAssetInterface를 구현하므로 자신이 소유한 GameplayTags에 접근하는 함수들을 제공합니다.

여러 개의 GameplayTag는 FGameplayTagContainer에 저장할 수 있습니다. GameplayTagContainers에는 효율적인 처리를 위한 최적화가 적용되어 있으므로 TArray<FGameplayTag>보다 GameplayTagContainer를 사용하는 것이 좋습니다. 태그는 표준 FName이지만, 프로젝트 설정에서 Fast Replication을 활성화하면 리플리케이션을 위해 FGameplayTagContainers 내에서 효율적으로 패킹(압축)될 수 있습니다. Fast Replication은 서버와 클라이언트가 동일한 GameplayTags 목록을 가지고 있어야 합니다. 이는 일반적인 게임 개발 환경에서 문제되지 않으므로 이 옵션을 활성화하는 것이 좋습니다. GameplayTagContainers는 순회를 위해 TArray<FGameplayTag>를 반환할 수도 있습니다.

FGameplayTagCountContainer에 저장된 GameplayTags는 해당 GameplayTag의 인스턴스 개수를 저장하는 TagMap을 가집니다. FGameplayTagCountContainer 내에 GameplayTag가 여전히 존재하더라도 해당 태그의 TagMapCount가 0일 수 있습니다. 디버깅 중에 ASC에 특정 GameplayTag가 남아 있는 것처럼 보이는 경우 이런 상황일 수 있습니다. HasTag(), HasMatchingTag() 및 유사한 함수들은 내부적으로 TagMapCount를 확인하여 GameplayTag가 없거나 TagMapCount가 0이면 false를 반환합니다.

GameplayTags는 사전에 DefaultGameplayTags.ini에 정의되어 있어야 합니다. 언리얼 엔진 에디터는 개발자가 DefaultGameplayTags.ini를 수동으로 편집하지 않고도 GameplayTags를 관리할 수 있도록 프로젝트 설정에 인터페이스를 제공합니다. GameplayTag 에디터에서는 GameplayTags의 생성, 이름 변경, 참조 검색 및 삭제가 가능합니다.

![Project Settings의 GameplayTag 에디터](https://github.com/tranek/GASDocumentation/raw/master/Images/gameplaytageditor.png)

GameplayTag 참조 검색을 실행하면 에디터에 친숙한 참조 뷰어(Reference Viewer) 그래프가 열려 해당 GameplayTag를 참조하는 모든 에셋을 보여줍니다. 그러나 해당 GameplayTag를 참조하는 C++ 클래스는 표시되지 않습니다.

GameplayTags의 이름을 변경하면 리디렉터(Redirect)가 생성되어 이전 GameplayTag를 참조하는 에셋들이 새 GameplayTag로 리디렉트될 수 있습니다. 개인적으로는 리디렉터 생성을 방지하기 위해 가능하면 새 GameplayTag를 생성하고, 모든 참조를 새 태그로 수동 변경한 뒤 이전 GameplayTag를 삭제하는 것을 선호합니다.

Fast Replication 외에도, GameplayTag 에디터에는 자주 리플리케이트되는 GameplayTags 목록을 지정하여 더욱 최적화할 수 있는 옵션이 있습니다.

GameplayTags는 GameplayEffect를 통해 추가된 경우 리플리케이트됩니다. ASC는 리플리케이트되지 않으며 수동으로 관리해야 하는 LooseGameplayTags를 추가할 수 있도록 허용합니다. 샘플 프로젝트에서는 체력이 0이 되었을 때 소유 클라이언트가 즉각 반응할 수 있도록 State.Dead에 LooseGameplayTag를 사용합니다. 리스폰 시 수동으로 TagMapCount를 다시 0으로 설정합니다. LooseGameplayTags를 다룰 때만 TagMapCount를 수동으로 조절해야 합니다. TagMapCount를 직접 건드리기보다는 UAbilitySystemComponent::AddLooseGameplayTag() 및 UAbilitySystemComponent::RemoveLooseGameplayTag() 함수를 사용하는 것이 바람직합니다.

C++에서 GameplayTag 참조 가져오기:
```c++
FGameplayTag::RequestGameplayTag(FName("Your.GameplayTag.Name"))
```

부모 또는 자식 GameplayTags를 가져오는 등 고급 GameplayTag 조작이 필요하다면 GameplayTagManager에서 제공하는 함수들을 확인하세요. GameplayTagManager에 접근하려면 GameplayTagManager.h를 include하고 UGameplayTagManager::Get().FunctionName 형태로 호출합니다. GameplayTagManager는 단순 문자열 조작 및 비교보다 빠른 처리를 위해 GameplayTags를 관계형 노드(부모, 자식 등)로 저장합니다.

GameplayTags 및 GameplayTagContainers에는 Meta = (Categories = "GameplayCue")라는 선택적 UPROPERTY 지정자를 사용할 수 있으며, 이는 블루프린트에서 부모 태그가 GameplayCue인 태그만 필터링하여 표시합니다. 이는 특정 GameplayTag나 GameplayTagContainer 변수가 GameplayCues 전용으로만 사용되어야 함을 알고 있을 때 유용합니다.

또는 FGameplayTag를 캡슐화하고 블루프린트에서 부모 태그가 GameplayCue인 태그만 자동 필터링해주는 FGameplayCueTag라는 별도의 구조체도 제공됩니다.

함수의 GameplayTag 매개변수를 필터링하려면 UFUNCTION 지정자 Meta = (GameplayTagFilter = "GameplayCue")를 사용하세요. 함수의 GameplayTagContainer 매개변수는 기본적으로 필터링되지 않습니다. 이를 지원하도록 엔진을 수정하고 싶다면 Engine\Plugins\Editor\GameplayTagsEditor\Source\GameplayTagsEditor\Private\SGameplayTagGraphPin.cpp의 SGameplayTagGraphPin::ParseDefaultValueData()에서 FilterString = UGameplayTagsManager::Get().GetCategoriesMetaFromField(PinStructType);를 호출하고 SGameplayTagGraphPin::GetListContent()에서 FilterString을 SGameplayTagWidget에 전달하는 방식을 확인해 보세요. Engine\Plugins\Editor\GameplayTagsEditor\Source\GameplayTagsEditor\Private\SGameplayTagContainerGraphPin.cpp에 있는 이들 함수의 GameplayTagContainer 버전은 메타 필드 프로퍼티를 확인하지 않고 필터를 전달합니다.

샘플 프로젝트는 GameplayTags를 광범위하게 활용합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-gt-change"></a>
### 4.2.1 게임플레이 태그 변경 감지 (Responding to Changes in Gameplay Tags)
ASC는 GameplayTags가 추가되거나 제거될 때 발동하는 델리게이트를 제공합니다. EGameplayTagEventType을 전달받아 GameplayTag가 추가/제거될 때만 발생할지, 아니면 TagMapCount의 모든 변경에 대해 발생할지 지정할 수 있습니다.

```c++
AbilitySystemComponent->RegisterGameplayTagEvent(FGameplayTag::RequestGameplayTag(FName("State.Debuff.Stun")), EGameplayTagEventType::NewOrRemoved).AddUObject(this, &AGDPlayerState::StunTagChanged);
```

콜백 함수는 GameplayTag와 새로운 TagCount를 매개변수로 받습니다.
```c++
virtual void StunTagChanged(const FGameplayTag CallbackTag, int32 NewCount);
```

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-gt-loadfromplugin"></a>
### 4.2.2 플러그인 .ini 파일에서 게임플레이 태그 로드
자체 GameplayTags가 포함된 .ini 파일이 있는 플러그인을 제작하는 경우, 플러그인의 StartupModule() 함수에서 해당 플러그인의 GameplayTag .ini 디렉터리를 로드할 수 있습니다.

예를 들어 언리얼 엔진에 포함된 CommonConversation 플러그인은 다음과 같이 처리합니다:

```c++
void FCommonConversationRuntimeModule::StartupModule()
{
	TSharedPtr<IPlugin> ThisPlugin = IPluginManager::Get().FindPlugin(TEXT("CommonConversation"));
	check(ThisPlugin.IsValid());
	
	UGameplayTagsManager::Get().AddTagIniSearchPath(ThisPlugin->GetBaseDir() / TEXT("Config") / TEXT("Tags"));

	//...
}
```

이렇게 하면 플러그인이 활성화되어 있을 때 엔진 시작 시 Plugins\CommonConversation\Config\Tags 디렉터리를 검색하여 그 안의 GameplayTags가 포함된 모든 .ini 파일을 프로젝트로 로드합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-a"></a>
### 4.3 어트리뷰트 (Attributes)

<a name="concepts-a-definition"></a>
#### 4.3.1 어트리뷰트 정의 (Attribute Definition)
Attributes는 [FGameplayAttributeData](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FGameplayAttributeData/index.html) 구조체로 정의되는 float 값입니다. 이는 캐릭터의 체력 수치, 캐릭터 레벨, 포션의 남은 충전 횟수에 이르기까지 무엇이든 표현할 수 있습니다. Actor에 속한 게임플레이 관련 수치형 값이라면 Attribute 사용을 고려해야 합니다. Attributes는 ASC가 변경 사항을 [예측(predict)](#concepts-p)할 수 있도록 일반적으로 [GameplayEffects](#concepts-ge)를 통해서만 수정되어야 합니다.

Attributes는 [AttributeSet](#concepts-as) 내부에서 정의되고 상주합니다. AttributeSet은 리플리케이션 대상으로 지정된 Attributes의 복제를 담당합니다. Attributes를 정의하는 방법은 [AttributeSets](#concepts-as) 섹션을 참조하세요.

**팁:** 에디터의 Attributes 목록에 특정 Attribute가 나타나지 않게 하려면 Meta = (HideInDetailsView) 프로퍼티 지정자를 사용할 수 있습니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-a-value"></a>
#### 4.3.2 BaseValue vs CurrentValue
Attribute는 BaseValue와 CurrentValue 두 가지 값으로 구성됩니다. BaseValue는 어트리뷰트의 영구적인(permanent) 기본값이며, CurrentValue는 BaseValue에 GameplayEffects로 인한 일시적인 수정치가 반영된 값입니다. 예를 들어, 캐릭터의 이동 속도(movespeed) Attribute의 BaseValue가 600 units/second라고 가정해 봅시다. 아직 이동 속도를 변경하는 GameplayEffects가 없다면 CurrentValue 역시 600 u/s입니다. 만약 캐릭터가 일시적인 50 u/s 이동 속도 버프를 받는다면, BaseValue는 여전히 600 u/s로 유지되는 반면 CurrentValue는 600 + 50이 되어 총 650 u/s가 됩니다. 이동 속도 버프가 만료되면 CurrentValue는 다시 원래의 BaseValue인 600 u/s로 복귀합니다.

GAS 초심자들은 종종 BaseValue를 Attribute의 최대값(Max Value)으로 혼동하여 그렇게 처리하려고 합니다. 이는 잘못된 접근 방식입니다. 변경될 수 있거나 어빌리티 및 UI에서 참조되는 Attributes의 최대값은 별도의 Attributes로 취급해야 합니다. 하드코딩된 최대/최소값의 경우 FAttributeMetaData를 포함한 DataTable을 정의하여 최대/최소값을 설정하는 방법이 있지만, 에픽의 구조체 상단 주석에서는 이를 "work in progress(작업 진행 중)"라고 명시하고 있습니다. 자세한 내용은 AttributeSet.h를 참조하세요. 혼란을 방지하기 위해 어빌리티나 UI에서 참조할 수 있는 최대값은 별도의 Attributes로 만들고, 오직 Attributes의 클램핑(Clamping, 범위 제한)에만 사용되는 하드코딩된 최대/최소값은 AttributeSet 내에 하드코딩된 float 상수로 정의하는 것을 권장합니다. Attributes의 클램핑은 CurrentValue 변경에 대해서는 [PreAttributeChange()](#concepts-as-preattributechange)에서, GameplayEffects로 인한 BaseValue 변경에 대해서는 [PostGameplayEffectExecute()](#concepts-as-postgameplayeffectexecute)에서 다룹니다.

BaseValue에 대한 영구적인 변경은 Instant GameplayEffects로부터 발생하며, Duration 및 Infinite GameplayEffects는 CurrentValue를 변경합니다. 주기적(Periodic) GameplayEffects는 즉시(Instant) GameplayEffects처럼 취급되어 BaseValue를 변경합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-a-meta"></a>
#### 4.3.3 메타 어트리뷰트 (Meta Attributes)
일부 Attributes는 다른 Attributes와 상호작용하기 위한 일시적인 값의 플레이스홀더(Placeholder)로 취급됩니다. 이를 Meta Attributes(메타 어트리뷰트)라고 부릅니다. 예를 들어, 우리는 일반적으로 대미지(Damage)를 Meta Attribute로 정의합니다. GameplayEffect가 우리의 체력(Health) Attribute를 직접 변경하는 대신, 대미지라는 Meta Attribute를 임시 전달자로 사용합니다. 이렇게 하면 대미지 수치가 [GameplayEffectExecutionCalculation](#concepts-ge-ec) 내에서 버프 및 디버프에 의해 수정될 수 있고, AttributeSet 내에서 최종적으로 체력 Attribute를 차감하기 전에 현재 쉴드(Shield) Attribute에서 대미지를 먼저 차감하는 등의 추가 조작을 수행할 수 있습니다. 대미지 Meta Attribute는 GameplayEffects 간에 지속되지 않으며 매번 새 효과에 의해 덮어씌워집니다. Meta Attributes는 일반적으로 리플리케이트되지 않습니다.

Meta Attributes는 대미지나 힐링 등에 대해 "얼마나 많은 대미지를 입혔는가?"와 "이 대미지로 무엇을 할 것인가?" 사이에 훌륭한 논리적 분리를 제공합니다. 이러한 논리적 분리 덕분에 우리의 Gameplay Effects와 Execution Calculations는 대상(Target)이 대미지를 어떻게 처리하는지 알 필요가 없습니다. 대미지 예시를 계속 들자면, Gameplay Effect는 대미지 양을 결정하고, AttributeSet은 그 대미지로 무엇을 할지 결정합니다. 특히 서브클래싱된 AttributeSets를 사용하는 경우 모든 캐릭터가 동일한 Attributes를 가지지 않을 수 있습니다. 기본 AttributeSet 클래스에는 체력 Attribute만 있을 수 있지만, 이를 상속받은 서브클래스 AttributeSet에는 쉴드 Attribute가 추가될 수 있습니다. 쉴드 Attribute를 가진 서브클래스 AttributeSet은 기본 AttributeSet 클래스와는 다른 방식으로 받은 대미지를 분배하게 됩니다.

Meta Attributes는 훌륭한 디자인 패턴이지만 필수는 아닙니다. 만약 모든 대미지 인스턴스에 오직 하나의 Execution Calculation만 사용되고 모든 캐릭터가 공유하는 단 하나의 Attribute Set 클래스만 존재한다면, Execution Calculation 내부에서 체력, 쉴드 등으로 대미지 분배를 처리하고 해당 Attributes를 직접 수정해도 무방합니다. 단지 유연성을 일부 희생하는 것일 뿐이며, 프로젝트 규모에 따라 괜찮을 수 있습니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-a-changes"></a>
#### 4.3.4 어트리뷰트 변경 감지 (Responding to Attribute Changes)
Attribute 변경 시점을 감지하여 UI나 기타 게임플레이를 업데이트하려면 UAbilitySystemComponent::GetGameplayAttributeValueChangeDelegate(FGameplayAttribute Attribute)를 사용합니다. 이 함수는 Attribute가 변경될 때마다 자동으로 호출되는 바인딩 가능한 델리게이트를 반환합니다. 이 델리게이트는 NewValue, OldValue, FGameplayEffectModCallbackData를 포함하는 FOnAttributeChangeData 매개변수를 제공합니다. **참고:** FGameplayEffectModCallbackData는 서버에서만 설정됩니다.

```c++
AbilitySystemComponent->GetGameplayAttributeValueChangeDelegate(AttributeSetBase->GetHealthAttribute()).AddUObject(this, &AGDPlayerState::HealthChanged);
```

```c++
virtual void HealthChanged(const FOnAttributeChangeData& Data);
```

샘플 프로젝트에서는 HUD를 업데이트하고 체력이 0에 도달했을 때 플레이어 사망을 처리하기 위해 GDPlayerState에서 Attribute 값 변경 델리게이트에 바인딩합니다.

이 기능을 AsyncTask로 래핑한 커스텀 블루프린트 노드가 샘플 프로젝트에 포함되어 있습니다. 이는 UI_HUD UMG 위젯에서 체력, 마나, 스태미나 값을 업데이트하는 데 사용됩니다. 이 AsyncTask는 수동으로 EndTask()를 호출할 때까지 계속 유지되며, 샘플 프로젝트에서는 UMG 위젯의 Destruct 이벤트에서 이를 호출합니다. AsyncTaskAttributeChanged.h/cpp를 참조하세요.

![어트리뷰트 변경 감지 BP 노드](https://github.com/tranek/GASDocumentation/raw/master/Images/attributechange.png)

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-a-derived"></a>
#### 4.3.5 파생 어트리뷰트 (Derived Attributes)
하나 이상의 다른 Attributes로부터 값의 일부 또는 전체를 파생시키는 Attribute를 만들려면 하나 이상의 Attribute Based 또는 [MMC](#concepts-ge-mmc) [Modifiers](#concepts-ge-mods)를 가진 Infinite GameplayEffect를 사용합니다. Derived Attribute(파생 어트리뷰트)는 종속된 Attribute가 업데이트될 때 자동으로 함께 업데이트됩니다.

Derived Attribute에 적용된 모든 Modifiers의 최종 계산 공식은 Modifier Aggregators의 공식과 동일합니다. 특정 순서로 계산이 이루어져야 하는 경우 MMC 내부에서 모든 계산을 처리하십시오.

```
((CurrentValue + Additive) * Multiplicitive) / Division
```

**참고:** PIE에서 멀티 클라이언트로 플레이할 경우 에디터 환경설정에서 Run Under One Process를 비활성화해야 합니다. 그렇지 않으면 첫 번째 클라이언트 이외의 클라이언트에서 독립 어트리뷰트가 업데이트될 때 Derived Attributes가 갱신되지 않습니다.

다음 예제에서는 TestAttrA = (TestAttrA + TestAttrB) * ( 2 * TestAttrC) 공식에 따라 TestAttrB와 TestAttrC 어트리뷰트로부터 TestAttrA의 값을 파생시키는 Infinite GameplayEffect를 보여줍니다. TestAttrA는 관련 어트리뷰트의 값이 업데이트될 때마다 자동으로 값을 재계산합니다.

![파생 어트리뷰트 예시](https://github.com/tranek/GASDocumentation/raw/master/Images/derivedattribute.png)

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-as"></a>
### 4.4 어트리뷰트 세트 (Attribute Set)

<a name="concepts-as-definition"></a>
#### 4.4.1 어트리뷰트 세트 정의
AttributeSet은 Attributes를 정의하고 보유하며, 이에 대한 변경을 관리합니다. 개발자는 [UAttributeSet](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAttributeSet/index.html)을 상속받아 서브클래스를 만들어야 합니다. OwnerActor의 생성자에서 AttributeSet을 생성하면 해당 액터의 ASC에 자동으로 등록됩니다. **이 작업은 반드시 C++로 작성해야 합니다.**

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-as-design"></a>
#### 4.4.2 어트리뷰트 세트 설계
하나의 ASC는 하나 또는 여러 개의 AttributeSet을 가질 수 있습니다. AttributeSet의 메모리 오버헤드는 무시할 수 있을 정도로 작으므로, 몇 개의 AttributeSet을 사용할지는 전적으로 개발자의 구조적 설계 판단에 달려 있습니다.

게임 내 모든 Actor가 공유하는 하나의 거대한 모놀리식(Monolithic) AttributeSet을 만들고, 필요한 어트리뷰트만 사용하고 사용하지 않는 어트리뷰트는 무시하는 방식도 허용됩니다.

또는 필요에 따라 액터에 선택적으로 추가할 수 있도록 Attributes를 논리적 그룹으로 묶어 여러 개의 AttributeSet을 구성할 수도 있습니다. 예를 들어 체력 관련 Attributes를 위한 AttributeSet, 마나 관련 Attributes를 위한 AttributeSet 등으로 나눌 수 있습니다. MOBA 게임에서 영웅은 마나가 필요하지만 미니언은 마나가 필요하지 않을 수 있습니다. 따라서 영웅에게는 마나 AttributeSet을 부여하고 미니언에게는 부여하지 않을 수 있습니다.

또한 AttributeSets를 상속(서브클래싱)하는 것도 액터가 가질 Attributes를 선택적으로 구성하는 또 다른 방법입니다. Attributes는 내부적으로 AttributeSet클래스명.Attribute이름 형태로 참조됩니다. AttributeSet을 상속받으면 부모 클래스의 모든 Attributes는 여전히 부모 클래스의 이름을 접두사로 가집니다.

여러 개의 AttributeSet을 둘 수 있지만, 하나의 ASC에 **동일한 클래스의 AttributeSet을 둘 이상 추가해서는 안 됩니다.** 동일한 클래스의 AttributeSet이 여러 개 존재하면 시스템은 어떤 AttributeSet을 사용해야 할지 알지 못해 임의로 하나를 선택하게 됩니다.

<a name="concepts-as-design-subcomponents"></a>
##### 4.4.2.1 개별 어트리뷰트를 가진 서브컴포넌트
개별적으로 파괴 가능한 방어구 파츠처럼 하나의 Pawn에 대미지를 입을 수 있는 컴포넌트가 여러 개 있는 경우, Pawn이 가질 수 있는 최대 컴포넌트 개수를 알고 있다면 하나의 AttributeSet에 해당 개수만큼의 체력 Attributes(예: DamageableCompHealth0, DamageableCompHealth1 등)를 만들어 각 파괴 가능 컴포넌트의 논리적 '슬롯'으로 표현하는 것을 권장합니다. 파괴 가능 컴포넌트 클래스 인스턴스에서 해당 슬롯 번호의 Attribute를 지정하면, GameplayAbilities나 [Executions](#concepts-ge-ec)에서 이를 읽어 어느 Attribute에 대미지를 적용할지 판단할 수 있습니다. 최대 개수보다 적거나 파괴 가능 컴포넌트가 전혀 없는 Pawn이라도 문제없습니다. AttributeSet에 특정 Attribute가 정의되어 있다고 해서 반드시 사용해야 하는 것은 아닙니다. 사용되지 않는 Attributes가 차지하는 메모리는 극히 미미합니다.

서브컴포넌트마다 매우 많은 Attributes가 필요하거나, 서브컴포넌트의 수가 무제한일 수 있거나, 서브컴포넌트가 분리되어 다른 플레이어가 주워 사용할 수 있는 경우(예: 무기), 또는 기타 이유로 위 방식이 적합하지 않다면 Attributes 대신 컴포넌트에 일반 기본 float 변수를 저장하는 방식으로 전환하는 것을 권장합니다. [아이템 어트리뷰트](#concepts-as-design-itemattributes) 섹션을 참조하세요.

<a name="concepts-as-design-addremoveruntime"></a>
##### 4.4.2.2 런타임에 AttributeSet 추가 및 제거
런타임 중에 ASC에 AttributeSets를 추가하거나 제거할 수 있습니다. 그러나 AttributeSets를 제거하는 작업은 위험할 수 있습니다. 예를 들어 서버보다 클라이언트에서 AttributeSet이 먼저 제거되었는데 어트리뷰트 값 변경이 클라이언트로 리플리케이트되면, 해당 Attribute가 소속 AttributeSet을 찾지 못해 게임이 크래시(비정상 종료)될 수 있습니다.

인벤토리에 무기를 추가할 때:
```c++
AbilitySystemComponent->GetSpawnedAttributes_Mutable().AddUnique(WeaponAttributeSetPointer);
AbilitySystemComponent->ForceReplication();
```

인벤토리에서 무기를 제거할 때:
```c++
AbilitySystemComponent->GetSpawnedAttributes_Mutable().Remove(WeaponAttributeSetPointer);
AbilitySystemComponent->ForceReplication();
```

<a name="concepts-as-design-itemattributes"></a>
##### 4.4.2.3 아이템 어트리뷰트 (무기 탄약 등)
Attributes를 가진 장착형 아이템(무기 탄약, 방어구 내구도 등)을 구현하는 데는 몇 가지 방법이 있습니다. 이러한 모든 접근 방식은 아이템 자체에 값을 직접 저장합니다. 이는 아이템의 수명 주기 동안 여러 플레이어가 번갈아 장착할 수 있는 아이템의 경우 반드시 필요합니다.

> 1. 아이템에 일반 float 변수 사용 (**권장**)
> 1. 아이템에 별도의 AttributeSet 배치
> 1. 아이템에 별도의 ASC 배치

<a name="concepts-as-design-itemattributes-plainfloats"></a>
###### 4.4.2.3.1 아이템의 단순 float 변수
Attributes 대신 아이템 클래스 인스턴스에 일반 float 값을 저장합니다. 포트나이트(Fortnite)와 [GASShooter](https://github.com/tranek/GASShooter)가 총기 탄약을 이 방식으로 처리합니다. 총기의 경우 최대 탄창 크기, 현재 탄창 내 탄약, 예비 탄약 등을 총기 인스턴스에 직접 리플리케이트되는 float 변수(COND_OwnerOnly)로 저장합니다. 무기들이 예비 탄약을 공유한다면 예비 탄약은 캐릭터의 공유 탄약 AttributeSet에 Attribute로 배치합니다(재장전 어빌리티는 Cost GE를 사용하여 예비 탄약에서 총기의 float 탄창 탄약으로 탄약을 채울 수 있습니다). 현재 탄창 탄약에 Attributes를 사용하지 않으므로, 총기의 float 변수를 대상으로 비용을 확인하고 차감하도록 UGameplayAbility의 일부 함수를 오버라이드해야 합니다. 어빌리티를 부여할 때 [GameplayAbilitySpec](https://github.com/tranek/GASDocumentation#concepts-ga-spec)의 SourceObject로 해당 총기를 지정하면 어빌리티 내부에서 해당 어빌리티를 부여한 총기 인스턴스에 쉽게 접근할 수 있습니다.

자동 사격 시 총기가 탄약 수량을 역으로 리플리케이트하여 로컬의 탄약 수량을 덮어써 버리는 현상을 방지하려면, PreReplication()에서 플레이어가 IsFiring 게임플레이 태그를 가지고 있는 동안 리플리케이션을 비활성화합니다. 사실상 여기서 자체적인 로컬 예측을 수행하는 것입니다.

```c++
void AGSWeapon::PreReplication(IRepChangedPropertyTracker& ChangedPropertyTracker)
{
	Super::PreReplication(ChangedPropertyTracker);

	DOREPLIFETIME_ACTIVE_OVERRIDE(AGSWeapon, PrimaryClipAmmo, (IsValid(AbilitySystemComponent) && !AbilitySystemComponent->HasMatchingGameplayTag(WeaponIsFiringTag)));
	DOREPLIFETIME_ACTIVE_OVERRIDE(AGSWeapon, SecondaryClipAmmo, (IsValid(AbilitySystemComponent) && !AbilitySystemComponent->HasMatchingGameplayTag(WeaponIsFiringTag)));
}
```

장점:
1. AttributeSets 사용에 따른 제약 사항들을 회피할 수 있음 (아래 설명 참조)

한계:
1. 기존의 GameplayEffect 워크플로우를 그대로 사용할 수 없음 (탄약 소모용 Cost GE 등)
1. 총기의 float 변수에 대해 탄약 비용을 확인하고 적용하기 위해 UGameplayAbility의 핵심 함수들을 오버라이드하는 추가 작업이 필요함

<a name="concepts-as-design-itemattributes-attributeset"></a>
###### 4.4.2.3.2 아이템에 AttributeSet 배치
아이템에 별도의 AttributeSet을 두고, [아이템을 플레이어 인벤토리에 추가할 때 플레이어의 ASC에 등록](#concepts-as-design-addremoveruntime)하는 방식도 동작은 하지만 몇 가지 치명적인 한계가 있습니다. 저는 [GASShooter](https://github.com/tranek/GASShooter) 초기 버전에서 무기 탄약에 이 방식을 적용해 보았습니다. 무기는 최대 탄창 크기, 현재 탄창 탄약, 예비 탄약 등의 Attributes를 무기 클래스에 상주하는 AttributeSet에 저장합니다. 무기들이 예비 탄약을 공유한다면 예비 탄약은 캐릭터의 공유 탄약 AttributeSet으로 옮깁니다. 서버에서 플레이어 인벤토리에 무기가 추가되면 무기는 자신의 AttributeSet을 플레이어의 ASC::SpawnedAttributes에 추가합니다. 그러면 서버가 이를 클라이언트로 리플리케이트합니다. 인벤토리에서 무기가 제거되면 ASC::SpawnedAttributes에서 해당 AttributeSet을 제거합니다.

AttributeSet이 OwnerActor가 아닌 다른 대상(예: 무기)에 상주할 경우, 처음에는 AttributeSet에서 컴파일 오류가 발생합니다. 해결 방법은 생성자 대신 BeginPlay()에서 AttributeSet을 생성하고, 무기에 IAbilitySystemInterface를 구현(무기를 플레이어 인벤토리에 추가할 때 ASC 포인터를 설정)하는 것입니다.

```c++
void AGSWeapon::BeginPlay()
{
	if (!AttributeSet)
	{
		AttributeSet = NewObject<UGSWeaponAttributeSet>(this);
	}
	//...
}
```

이 구현은 [GASShooter의 구버전 커밋](https://github.com/tranek/GASShooter/tree/df5949d0dd992bd3d76d4a728f370f2e2c827735)에서 확인할 수 있습니다.

장점:
1. 기존의 GameplayAbility 및 GameplayEffect 워크플로우를 그대로 활용 가능 (탄약 소모용 Cost GE 등)
1. 아이템 종류가 매우 적은 경우 설정이 단순함

한계:
1. 모든 무기 타입마다 새로운 AttributeSet 클래스를 만들어야 합니다. ASC는 구조상 특정 클래스의 AttributeSet 인스턴스를 하나만 정상적으로 다룰 수 있습니다. Attribute 변경 시 ASC의 SpawnedAttributes 배열에서 해당 AttributeSet 클래스의 첫 번째 인스턴스만 찾기 때문입니다. 동일한 AttributeSet 클래스의 추가 인스턴스는 무시됩니다.
1. AttributeSet 클래스당 하나의 인스턴스만 가능하다는 위 제약 때문에, 플레이어 인벤토리에 동일한 유형의 무기를 둘 이상 소지할 수 없습니다.
1. AttributeSet을 제거하는 작업이 위험합니다. GASShooter에서 플레이어가 로켓 자폭으로 사망했을 때 인벤토리에서 로켓 런처를 즉시 제거(해당 AttributeSet도 ASC에서 제거)했습니다. 이때 서버에서 로켓 런처의 탄약 Attribute가 변경되었다는 사실이 리플리케이트되어 도착했을 때, 클라이언트의 ASC에는 이미 해당 AttributeSet이 존재하지 않아 게임이 크래시되었습니다.

<a name="concepts-as-design-itemattributes-asc"></a>
###### 4.4.2.3.3 아이템에 ASC 배치
각 아이템마다 완전히 독립된 AbilitySystemComponent를 부착하는 것은 극단적인 접근 방식입니다. 저는 개인적으로 이를 구현해보지 않았으며 실제 프로젝트에서 사용된 사례도 본 적이 없습니다. 이를 정상 동작하게 만들려면 상당한 수준의 엔지니어링 작업이 필요할 것입니다.

> 동일한 소유자(Owner)를 가지지만 아바타(Avatar)가 다른 여러 개의 AbilitySystemComponent를 두는 것이 가능할까요? (예: 폰과 무기/아이템/투사체에 ASC가 있고, Owner는 모두 PlayerState로 설정된 경우)
> 
> 여기서 가장 먼저 부딪히는 문제는 소유자 액터에 IGameplayTagAssetInterface와 IAbilitySystemInterface를 구현하는 것입니다. 전자는 가능할 수도 있습니다. 모든 ASC로부터 태그를 모아 집계하면 됩니다 (단, 주의할 점은 HasAllMatchingGameplayTags 같은 검사가 여러 ASC 간의 집계를 통해서만 만족될 수 있다는 점입니다. 각 ASC에 호출을 전달하고 결과를 단순 OR 연산하는 것만으로는 충분하지 않을 수 있습니다). 하지만 후자는 훨씬 더 까다롭습니다. 어떤 ASC가 권위(Authoritative)를 가진 컴포넌트일까요? 누군가 GE를 적용하고자 할 때 어느 컴포넌트가 받아야 할까요? 이를 해결할 수 있을지도 모르지만, 이 부분이야말로 가장 어려운 문제가 될 것입니다. 즉 소유자 아래에 여러 ASC가 존재하는 구조가 됩니다.
> 
> 폰과 무기에 별도의 ASC를 두는 것 자체는 타당할 수 있습니다. 예를 들어 무기를 설명하는 태그와 소유한 폰을 설명하는 태그를 구분하는 경우입니다. 무기에 부여된 태그가 소유자에게도 "적용"되도록 하되 다른 것은 독립적으로 유지하는 구조(예: 어트리뷰트와 GE는 독립적이지만 소유자는 위에서 설명한 것처럼 소유 태그들을 집계하는 방식)는 합리적일 수 있습니다. 분명 구현 가능할 것입니다. 하지만 동일한 소유자를 가진 여러 ASC를 두는 것은 매우 위험하고 까다로워질 수 있습니다.
> 
> *Epic Games의 Dave Ratti 답변 ([커뮤니티 질문 #6](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89))*

장점:
1. 기존의 GameplayAbility 및 GameplayEffect 워크플로우를 그대로 활용 가능 (탄약 소모용 Cost GE 등)
1. AttributeSet 클래스 재사용 가능 (각 무기의 ASC에 하나씩 배치)

한계:
1. 요구되는 엔지니어링 비용을 예측하기 어려움
1. 실현 가능한 구조인지조차 불확실함

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-as-attributes"></a>
#### 4.4.3 어트리뷰트 정의 매크로
**Attributes는 반드시 C++로만 정의할 수 있으며**, AttributeSet의 헤더 파일에 선언합니다. 모든 AttributeSet 헤더 파일 상단에 다음 매크로 블록을 추가하는 것을 권장합니다. 이 매크로는 어트리뷰트에 대한 Getter와 Setter 함수를 자동으로 생성해 줍니다.

```c++
// AttributeSet.h의 매크로를 사용합니다.
#define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)
```

리플리케이트되는 체력(Health) 어트리뷰트는 다음과 같이 정의합니다:

```c++
UPROPERTY(BlueprintReadOnly, Category = "Health", ReplicatedUsing = OnRep_Health)
FGameplayAttributeData Health;
ATTRIBUTE_ACCESSORS(UGDAttributeSetBase, Health)
```

헤더 파일에 OnRep 함수도 함께 선언합니다:
```c++
UFUNCTION()
virtual void OnRep_Health(const FGameplayAttributeData& OldHealth);
```

AttributeSet의 .cpp 파일에서는 예측 시스템에서 사용되는 GAMEPLAYATTRIBUTE_REPNOTIFY 매크로를 사용하여 OnRep 함수를 구현합니다:
```c++
void UGDAttributeSetBase::OnRep_Health(const FGameplayAttributeData& OldHealth)
{
	GAMEPLAYATTRIBUTE_REPNOTIFY(UGDAttributeSetBase, Health, OldHealth);
}
```

마지막으로, GetLifetimeReplicatedProps에 Attribute를 등록해야 합니다:
```c++
void UGDAttributeSetBase::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
	Super::GetLifetimeReplicatedProps(OutLifetimeProps);

	DOREPLIFETIME_CONDITION_NOTIFY(UGDAttributeSetBase, Health, COND_None, REPNOTIFY_Always);
}
```

REPNOTIFY_Always는 (예측 기능 때문에) 로컬 값이 서버로부터 리플리케이트되어 내려온 값과 이미 동일하더라도 OnRep 함수를 트리거하도록 지시합니다. 기본 설정의 경우 로컬 값이 서버에서 온 값과 같으면 OnRep 함수가 트리거되지 않습니다.

Meta Attribute처럼 리플리케이트되지 않는 Attribute의 경우, OnRep 및 GetLifetimeReplicatedProps 등록 과정을 생략할 수 있습니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-as-init"></a>
#### 4.4.4 어트리뷰트 초기화
Attributes를 초기화(즉 BaseValue 및 이에 따른 CurrentValue를 초기 값으로 설정)하는 데는 여러 가지 방법이 있습니다. Epic Games는 즉시(Instant) GameplayEffect를 사용하는 것을 권장합니다. 샘플 프로젝트에서도 이 방법을 사용합니다.

Attributes를 초기화하는 즉시 GameplayEffect를 만드는 방법은 샘플 프로젝트의 GE_HeroAttributes 블루프린트를 참조하세요. 이 GameplayEffect의 적용은 C++에서 수행됩니다.

Attributes를 정의할 때 ATTRIBUTE_ACCESSORS 매크로를 사용했다면, AttributeSet에 각 어트리뷰트별 초기화 함수가 자동으로 생성되므로 C++에서 언제든지 자유롭게 호출할 수 있습니다.

```c++
// InitHealth(float InitialValue)는 ATTRIBUTE_ACCESSORS 매크로로 정의된 'Health' 어트리뷰트에 대해 자동 생성된 함수입니다.
AttributeSet->InitHealth(100.0f);
```

Attributes를 초기화하는 다른 방법들은 AttributeSet.h를 참조하세요.

**참고:** 4.24 이전 버전에서는 FAttributeSetInitterDiscreteLevels가 FGameplayAttributeData와 함께 동작하지 않았습니다. 이는 Attributes가 단순 float이던 시절에 만들어졌기 때문에 FGameplayAttributeData가 POD(Plain Old Data)가 아니라는 경고를 뱉었습니다. 이 문제는 4.24에서 수정되었습니다 (https://issues.unrealengine.com/issue/UE-76557).

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-as-preattributechange"></a>
#### 4.4.5 PreAttributeChange()
PreAttributeChange(const FGameplayAttribute& Attribute, float& NewValue)는 Attribute의 CurrentValue가 변경되기 직전에 호출되는 AttributeSet의 핵심 함수 중 하나입니다. 참조 매개변수인 NewValue를 통해 CurrentValue에 적용될 변경 값을 클램핑(범위 제한)하기에 가장 이상적인 장소입니다.

예를 들어, 샘플 프로젝트에서는 이동 속도(movespeed) 모디파이어를 다음과 같이 클램핑합니다:
```c++
if (Attribute == GetMoveSpeedAttribute())
{
	// 150 units/s 미만으로 느려질 수 없으며 1000 units/s를 초과하여 증가할 수 없음
	NewValue = FMath::Clamp<float>(NewValue, 150, 1000);
}
```
GetMoveSpeedAttribute() 함수는 우리가 AttributeSet.h에 추가했던 매크로 블록([어트리뷰트 정의 매크로](#concepts-as-attributes))에 의해 자동 생성된 것입니다.

이 함수는 Attribute Setter 함수(매크로 블록으로 정의됨)를 사용하든 [GameplayEffects](#concepts-ge)를 사용하든 관계없이 Attributes의 모든 변경에 의해 트리거됩니다.

**참고:** 여기서 수행되는 클램핑은 ASC 내부의 모디파이어 자체를 영구적으로 변경하지 않습니다. 모디파이어를 쿼리하여 반환되는 값만 변경할 뿐입니다. 즉, [GameplayEffectExecutionCalculations](#concepts-ge-ec)나 [ModifierMagnitudeCalculations](#concepts-ge-mmc)처럼 모든 모디파이어로부터 CurrentValue를 재계산하는 로직에서는 클램핑을 다시 구현해야 합니다.

**참고:** Epic의 PreAttributeChange() 관련 주석에 따르면, 이 함수를 게임플레이 이벤트 용도로 사용하지 말고 오직 클램핑 용도로만 사용할 것을 권장하고 있습니다. Attribute 변경에 따른 게임플레이 이벤트를 처리하는 권장 위치는 UAbilitySystemComponent::GetGameplayAttributeValueChangeDelegate(FGameplayAttribute Attribute) ([어트리뷰트 변경 감지](#concepts-a-changes))입니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-as-postgameplayeffectexecute"></a>
#### 4.4.6 PostGameplayEffectExecute()
PostGameplayEffectExecute(const FGameplayEffectModCallbackData & Data)는 즉시(Instant) [GameplayEffect](#concepts-ge)에 의해 Attribute의 BaseValue가 변경된 후에만 트리거됩니다. GameplayEffect로 인해 Attributes가 변경되었을 때 추가적인 조작을 수행하기에 적합한 위치입니다.

예를 들어 샘플 프로젝트에서는 최종 대미지 Meta Attribute를 여기서 체력(Health) Attribute에서 차감합니다. 만약 쉴드 Attribute가 존재한다면 체력에서 남은 대미지를 빼기 전에 먼저 쉴드에서 대미지를 차감할 것입니다. 샘플 프로젝트는 또한 피격 리액션 애니메이션 재생, 플로팅 대미지 숫자 표시, 처치자에게 경험치 및 골드 보상 지급 등을 처리하는 데 이 위치를 활용합니다. 설계상 대미지 Meta Attribute는 Attribute Setter가 아니라 항상 즉시 GameplayEffect를 통해서만 전달됩니다.

마나 및 스태미나처럼 즉시 GameplayEffect에 의해서만 BaseValue가 변경되는 다른 Attributes 역시 여기서 최대값 대응 Attributes에 맞춰 클램핑할 수 있습니다.

**참고:** PostGameplayEffectExecute()가 호출되는 시점에는 이미 Attribute 변경이 일어난 상태이지만, 아직 클라이언트로 리플리케이트되지는 않은 상태입니다. 따라서 여기서 값을 클램핑하더라도 클라이언트에 네트워크 업데이트가 두 번 발생하는 일은 없습니다. 클라이언트는 클램핑이 완료된 최종 값만 수신합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-as-onattributeaggregatorcreated"></a>
#### 4.4.7 OnAttributeAggregatorCreated()
OnAttributeAggregatorCreated(const FGameplayAttribute& Attribute, FAggregator* NewAggregator)는 이 세트의 Attribute에 대해 Aggregator가 생성될 때 트리거됩니다. 이를 통해 [FAggregatorEvaluateMetaData](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FAggregatorEvaluateMetaData/index.html)를 커스텀으로 설정할 수 있습니다. AggregatorEvaluateMetaData는 적용된 모든 [Modifiers](#concepts-ge-mods)를 기반으로 Attribute의 CurrentValue를 평가할 때 Aggregator에 의해 사용됩니다. 기본적으로 AggregatorEvaluateMetaData는 어떤 Modifiers가 유효한지(자격이 있는지) 결정하기 위해 Aggregator에서만 사용되며, 대표적인 예로 MostNegativeMod_AllPositiveMods가 있습니다. 이는 모든 양수(버프) Modifiers는 허용하되 음수(디버프) Modifiers는 가장 음수 폭이 큰(가장 강력한 둔화) 단 하나로 제한합니다. 이는 파라곤(Paragon)에서 여러 개의 이동 속도 둔화 효과가 동시에 걸려 있어도 가장 강력한 둔화 효과 하나만 적용하고 모든 이동 속도 증가 버프는 중첩 적용하기 위해 사용되었습니다. 유효 자격을 얻지 못한 Modifiers라도 ASC 상에는 여전히 존재하며, 단지 최종 CurrentValue로 집계(Aggregate)되지 않을 뿐입니다. 이후 조건이 변경되면(예: 가장 강력했던 음수 모디파이어가 만료된 경우 그 다음으로 강력했던 모디파이어가 유효 자격을 얻음) 나중에 다시 자격을 얻을 수 있습니다.

가장 강력한 음수 모디파이어 하나와 모든 양수 모디파이어만 허용하는 예제에서 AggregatorEvaluateMetaData를 사용하는 방법:

```c++
virtual void OnAttributeAggregatorCreated(const FGameplayAttribute& Attribute, FAggregator* NewAggregator) const override;
```

```c++
void UGSAttributeSetBase::OnAttributeAggregatorCreated(const FGameplayAttribute& Attribute, FAggregator* NewAggregator) const
{
	Super::OnAttributeAggregatorCreated(Attribute, NewAggregator);

	if (!NewAggregator)
	{
		return;
	}

	if (Attribute == GetMoveSpeedAttribute())
	{
		NewAggregator->EvaluationMetaData = &FAggregatorEvaluateMetaDataLibrary::MostNegativeMod_AllPositiveMods;
	}
}
```

조건 평가를 위한 커스텀 AggregatorEvaluateMetaData는 FAggregatorEvaluateMetaDataLibrary에 정적(static) 변수로 추가해야 합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ge"></a>
### 4.5 게임플레이 이펙트 (Gameplay Effects)

<a name="concepts-ge-definition"></a>
#### 4.5.1 게임플레이 이펙트 정의
[GameplayEffects](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayEffect/index.html) (GE)는 어빌리티가 자기 자신이나 타인의 [Attributes](#concepts-a) 및 [GameplayTags](#concepts-gt)를 변경하는 매개체(Vessel)입니다. 대미지나 힐과 같은 즉각적인 Attribute 변경을 일으킬 수도 있고, 이동 속도 증가 버프나 기절(Stun) 같은 장기적인 상태 이상 버프/디버프를 적용할 수도 있습니다. UGameplayEffect 클래스는 단일 게임플레이 이펙트를 정의하는 **데이터 전용(data-only)** 클래스로 설계되었습니다. 따라서 GameplayEffects에 추가적인 실행 로직을 넣어서는 안 됩니다. 일반적으로 기획자/디자이너는 UGameplayEffect의 다양한 블루프린트 자식 클래스를 생성하여 사용합니다.

GameplayEffects는 [Modifiers](#concepts-ge-mods)와 [Executions (GameplayEffectExecutionCalculation)](#concepts-ge-ec)을 통해 Attributes를 변경합니다.

GameplayEffects에는 Instant(즉시), Duration(지속 시간), Infinite(무한) 세 가지 지속 시간 유형이 있습니다.

또한 GameplayEffects는 [GameplayCues](#concepts-gc)를 추가하거나 실행할 수 있습니다. Instant GameplayEffect는 해당 GameplayCue GameplayTags에 대해 Execute를 호출하는 반면, Duration 또는 Infinite GameplayEffect는 Add 및 Remove를 호출합니다.

| 지속 시간 유형 (Duration Type) | GameplayCue 이벤트 | 사용 시점                                                                                                                                                                                                                                  |
| ------------------------------ | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Instant                      | Execute            | Attribute의 BaseValue에 대한 즉각적이고 영구적인 변경. GameplayTags는 단 1프레임조차 적용되지 않습니다.                                                                                                                            |
| Duration                     | Add & Remove       | Attribute의 CurrentValue에 대한 일시적인 변경 및 GameplayEffect 만료 시 또는 수동 제거 시 함께 제거되는 GameplayTags 적용. 지속 시간은 UGameplayEffect 클래스/블루프린트에 지정됩니다.                                         |
| Infinite                     | Add & Remove       | Attribute의 CurrentValue에 대한 일시적인 변경 및 GameplayEffect 제거 시 함께 제거되는 GameplayTags 적용. 자체적으로 만료되지 않으며 어빌리티나 ASC에 의해 수동으로 제거되어야 합니다.                                          |

Duration 및 Infinite GameplayEffects는 Period(주기)에 정의된 매 X초마다 Modifiers와 Executions를 적용하는 주기적 효과(Periodic Effects) 옵션을 가질 수 있습니다. 주기적 효과는 Attribute의 BaseValue를 변경하고 GameplayCues를 Execute한다는 점에서 Instant GameplayEffects처럼 취급됩니다. 이는 지속 대미지(DoT, Damage over Time) 효과 등에 유용합니다. **참고:** 주기적 효과는 [예측(Prediction)](#concepts-p)할 수 없습니다.

Duration 및 Infinite GameplayEffects는 적용된 이후에도 Ongoing Tag Requirements(진행 중 태그 조건)의 충족 여부에 따라 일시적으로 꺼지거나(Off) 켜질(On) 수 있습니다 ([게임플레이 이펙트 태그](#concepts-ge-tags)). GameplayEffect가 꺼지면 해당 모디파이어의 효과와 부여된 GameplayTags가 비활성화되지만 GameplayEffect 자체가 제거되지는 않습니다. 태그 조건을 다시 만족하여 켜지면 모디파이어와 태그가 다시 적용됩니다.

Duration 또는 Infinite GameplayEffect의 Modifiers를 수동으로 재계산해야 하는 경우(예: Attributes에서 오지 않는 데이터를 사용하는 MMC가 있는 경우), UAbilitySystemComponent::ActiveGameplayEffects.GetActiveGameplayEffect(ActiveHandle).Spec.GetLevel()을 통해 현재 레벨을 가져온 뒤 UAbilitySystemComponent::ActiveGameplayEffects.SetActiveGameplayEffectLevel(FActiveGameplayEffectHandle ActiveHandle, int32 NewLevel)을 동일한 레벨로 호출할 수 있습니다. 백킹 Attributes 기반 모디파이어는 기반 Attributes가 갱신될 때 자동으로 업데이트됩니다. SetActiveGameplayEffectLevel()에서 모디파이어를 업데이트하는 핵심 내부 함수들은 다음과 같습니다:

```C++
MarkItemDirty(Effect);
Effect.Spec.CalculateModifierMagnitudes();
// 이 함수는 private이므로 레벨을 현재 레벨로 재설정하지 않고 이 세 함수만 직접 호출할 수는 없습니다.
UpdateAllAggregatorModMagnitudes(Effect);
```

GameplayEffects는 일반적으로 인스턴스화(C++ new/CreateObject)되지 않습니다. 어빌리티나 ASC가 GameplayEffect를 적용하고자 할 때는 GameplayEffect의 ClassDefaultObject로부터 [GameplayEffectSpec](#concepts-ge-spec)을 생성합니다. 성공적으로 적용된 GameplayEffectSpecs는 FActiveGameplayEffect라는 새로운 구조체에 담겨 ASC가 ActiveGameplayEffects라는 특수 컨테이너 구조체에서 관리합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ge-applying"></a>
#### 4.5.2 게임플레이 이펙트 적용
GameplayEffects는 [GameplayAbilities](#concepts-ga) 및 ASC의 다양한 함수를 통해 적용될 수 있으며, 대개 ApplyGameplayEffectTo 형태의 이름을 가집니다. 이러한 다양한 함수들은 궁극적으로 대상(Target)의 UAbilitySystemComponent::ApplyGameplayEffectSpecToSelf()를 호출하는 편의 함수들입니다.

투사체 등 GameplayAbility 외부에서 GameplayEffects를 적용하려면 대상(Target)의 ASC를 얻어온 뒤 ApplyGameplayEffectToSelf 계열 함수 중 하나를 사용해야 합니다.

ASC에 Duration 또는 Infinite GameplayEffects가 적용되는 시점을 감지하려면 다음 델리게이트에 바인딩할 수 있습니다:
```c++
AbilitySystemComponent->OnActiveGameplayEffectAddedDelegateToSelf.AddUObject(this, &APACharacterBase::OnActiveGameplayEffectAddedCallback);
```
콜백 함수:
```c++
virtual void OnActiveGameplayEffectAddedCallback(UAbilitySystemComponent* Target, const FGameplayEffectSpec& SpecApplied, FActiveGameplayEffectHandle ActiveHandle);
```

서버는 리플리케이션 모드와 관계없이 항상 이 함수를 호출합니다. 자율 프록시(Autonomous Proxy)는 Full 및 Mixed 리플리케이션 모드에서 리플리케이트된 GameplayEffects에 대해서만 이를 호출합니다. 시뮬레이트 프록시(Simulated Proxy)는 Full [리플리케이션 모드](#concepts-asc-rm)에서만 이를 호출합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ga-removing"></a>
#### 4.5.3 게임플레이 이펙트 제거
GameplayEffects는 [GameplayAbilities](#concepts-ga) 및 ASC의 여러 함수를 통해 제거될 수 있으며 대개 RemoveActiveGameplayEffect 형태를 띱니다. 이러한 함수들은 결국 대상의 FActiveGameplayEffectsContainer::RemoveActiveEffects()를 호출하는 편의 함수들입니다.

GameplayAbility 외부에서 GameplayEffects를 제거하려면 대상의 ASC를 가져와 RemoveActiveGameplayEffect 함수 중 하나를 호출해야 합니다.

ASC에서 Duration 또는 Infinite GameplayEffects가 제거되는 시점을 감지하려면 델리게이트에 바인딩할 수 있습니다:
```c++
AbilitySystemComponent->OnAnyGameplayEffectRemovedDelegate().AddUObject(this, &APACharacterBase::OnRemoveGameplayEffectCallback);
```
콜백 함수:
```c++
virtual void OnRemoveGameplayEffectCallback(const FActiveGameplayEffect& EffectRemoved);
```

서버는 리플리케이션 모드와 관계없이 항상 이 함수를 호출합니다. 자율 프록시는 Full 및 Mixed 리플리케이션 모드에서 리플리케이트된 GameplayEffects에 대해 이를 호출합니다. 시뮬레이트 프록시는 Full [리플리케이션 모드](#concepts-asc-rm)에서만 이를 호출합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ge-mods"></a>
#### 4.5.4 게임플레이 이펙트 모디파이어 (Modifiers)
Modifiers는 Attribute를 변경하며, Attribute를 [예측적으로(predictively)](#concepts-p) 변경할 수 있는 유일한 방법입니다. 하나의 GameplayEffect는 0개 또는 여러 개의 Modifiers를 가질 수 있습니다. 각 Modifier는 지정된 연산(Operation)을 통해 단 하나의 Attribute만을 변경하는 책임을 집니다.

| 연산 (Operation) | 설명                                                                                   |
| ---------------- | -------------------------------------------------------------------------------------- |
| Add            | 결과를 Modifier가 지정한 Attribute에 더합니다. 감산의 경우 음수 값을 사용합니다.   |
| Multiply       | 결과를 Modifier가 지정한 Attribute에 곱합니다.                                     |
| Divide         | Modifier가 지정한 Attribute를 결과 값으로 나눕니다.                                |
| Override       | Modifier가 지정한 Attribute를 결과 값으로 덮어씁니다.                              |

Attribute의 CurrentValue는 BaseValue에 적용된 모든 Modifiers의 집계 결과입니다. 모디파이어들이 집계되는 공식은 GameplayEffectAggregator.cpp의 FAggregatorModChannel::EvaluateWithBase에 다음과 같이 정의되어 있습니다:
```c++
((InlineBaseValue + Additive) * Multiplicitive) / Division
```

모든 Override 모디파이어는 최종 값을 덮어쓰며, 가장 마지막에 적용된 Modifier가 우선권을 갖습니다.

**참고:** 백분율(퍼센트) 기반 변경의 경우 연산이 덧셈 이후에 수행되도록 반드시 Multiply 연산을 사용해야 합니다.

**참고:** [예측(Prediction)](#concepts-p)은 백분율 기반 변경 처리에 다소 어려움이 있습니다.

Modifiers에는 네 가지 유형이 있습니다: Scalable Float, Attribute Based, Custom Calculation Class, Set By Caller. 이들은 모두 어떤 float 값을 계산해 내며, 이 값은 연산 방식에 따라 대상 Attribute를 변경하는 데 사용됩니다.

| Modifier 유형            | 설명                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Scalable Float           | FScalableFloats는 행(Row)에 변수가 있고 열(Column)에 레벨이 있는 데이터 테이블(Data Table)을 가리킬 수 있는 구조체입니다. Scalable Floats는 어빌리티의 현재 레벨(또는 [GameplayEffectSpec](#concepts-ge-spec)에서 오버라이드된 레벨)에 해당하는 지정된 테이블 행의 값을 자동으로 읽어옵니다. 이 값은 계수(Coefficient)로 추가 조작될 수 있습니다. 데이터 테이블/행이 지정되지 않은 경우 기본값 1로 취급되므로 계수를 사용하여 모든 레벨에서 단일 고정 값으로 하드코딩할 수도 있습니다. ![ScalableFloat](https://github.com/tranek/GASDocumentation/raw/master/Images/scalablefloats.png)                                                                                                                            |
| Attribute Based          | Attribute Based 모디파이어는 Source(GameplayEffectSpec을 생성한 대상) 또는 Target(GameplayEffectSpec을 받은 대상)의 기반 Attribute의 CurrentValue 또는 BaseValue를 가져와 계수 및 계수 전/후 덧셈 값을 통해 추가 수정합니다. 스냅샷(Snapshotting)은 GameplayEffectSpec이 생성될 당시의 기반 어트리뷰트를 캡처하는 것을 의미하며, 스냅샷을 사용하지 않으면 GameplayEffectSpec이 적용(Application)될 때 어트리뷰트를 캡처합니다.                                                                                                                                                                                                                                                              |
| Custom Calculation Class | Custom Calculation Class는 복잡한 Modifiers에 대해 최고의 유연성을 제공합니다. 이 모디파이어는 [ModifierMagnitudeCalculation](#concepts-ge-mmc) 클래스를 사용하며 계산된 최종 float 값을 계수 및 전/후 덧셈으로 추가 조작할 수 있습니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| Set By Caller            | SetByCaller 모디파이어는 어빌리티나 GameplayEffectSpec을 생성한 주체가 런타임에 GameplayEffect 외부에서 GameplayEffectSpec에 직접 설정하는 값입니다. 예를 들어 플레이어가 어빌리티 충전을 위해 버튼을 누르고 있던 시간에 비례하여 대미지를 설정하고자 할 때 SetByCaller를 사용합니다. SetByCallers는 본질적으로 GameplayEffectSpec에 상주하는 TMap<FGameplayTag, float>입니다. 모디파이어는 단지 지정된 GameplayTag와 연결된 SetByCaller 값을 찾으라고 Aggregator에 지시할 뿐입니다. 모디파이어에서 사용되는 SetByCallers는 오직 GameplayTag 버전만 사용할 수 있으며 FName 버전은 여기서 비활성화되어 있습니다. Modifier가 SetByCaller로 설정되었는데 해당 태그의 값이 GameplayEffectSpec에 존재하지 않으면 런타임 오류를 발생시키고 0을 반환합니다. 이는 Divide 연산 시 문제가 될 수 있습니다. 자세한 사용법은 [SetByCallers](#concepts-ge-spec-setbycaller)를 참조하세요. |

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ge-mods-multiplydivide"></a>
##### 4.5.4.1 곱셈(Multiply) 및 나눗셈(Divide) 모디파이어
기본적으로 모든 Multiply 및 Divide Modifiers는 Attribute의 BaseValue에 곱하거나 나누기 전에 먼저 서로 더해집니다.

```c++
float FAggregatorModChannel::EvaluateWithBase(float InlineBaseValue, const FAggregatorEvaluateParameters& Parameters) const
{
	...
	float Additive = SumMods(Mods[EGameplayModOp::Additive], GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Additive), Parameters);
	float Multiplicitive = SumMods(Mods[EGameplayModOp::Multiplicitive], GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Multiplicitive), Parameters);
	float Division = SumMods(Mods[EGameplayModOp::Division], GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Division), Parameters);
	...
	return ((InlineBaseValue + Additive) * Multiplicitive) / Division;
	...
}
```

```c++
float FAggregatorModChannel::SumMods(const TArray<FAggregatorMod>& InMods, float Bias, const FAggregatorEvaluateParameters& Parameters)
{
	float Sum = Bias;

	for (const FAggregatorMod& Mod : InMods)
	{
		if (Mod.Qualifies())
		{
			Sum += (Mod.EvaluatedMagnitude - Bias);
		}
	}

	return Sum;
}
```
*GameplayEffectAggregator.cpp 발췌*

이 공식에서 Multiply와 Divide 모디파이어는 모두 1의 Bias(바이어스) 값을 가집니다 (Addition은 0의 Bias를 가짐). 따라서 다음과 같은 형태가 됩니다:

```
1 + (Mod1.Magnitude - 1) + (Mod2.Magnitude - 1) + ...
```

이 공식은 몇 가지 예상치 못한 결과를 낳습니다. 첫째, 이 공식은 모든 모디파이어를 BaseValue에 곱하거나 나누기 전에 먼저 서로 더합니다. 대부분의 사람들은 이들이 서로 곱해지거나 나눠질 것이라 기대합니다. 예를 들어 1.5 배율 모디파이어가 2개 있다면 대부분 1.5 x 1.5 = 2.25배가 될 것이라 생각합니다. 하지만 이 공식은 1.5들을 더하여 BaseValue에 2배를 곱합니다 (50% 증가 + 또 다른 50% 증가 = 100% 증가). 이는 GameplayPrediction.h에 나오는 예시처럼 기본 이동 속도 500에 10% 속도 버프가 적용되면 550이 되고, 여기에 또 다른 10% 버프가 추가되면 600이 되도록 하기 위해 설계된 것입니다.

둘째, 이 공식은 파라곤(Paragon)을 염두에 두고 설계되었기 때문에 사용할 수 있는 수치에 대해 문서화되지 않은 규칙들이 있습니다.

Multiply 및 Divide의 덧셈 집계 공식 규칙:
* (1 미만의 값이 최대 1개) AND (1 이상 2 미만 [1, 2) 범위의 값이 임의 개수)
* 또는 (2 이상의 값이 1개)

공식의 Bias는 기본적으로 [1, 2) 범위의 숫자의 정수부를 빼줍니다. 첫 번째 Modifier의 Bias는 시작 Sum 값(루프 전 Bias로 초기화됨)에서 상쇄되므로, 단일 값은 어떤 값이든 작동하며 1 미만의 값 1개는 [1, 2) 범위의 숫자들과 함께 정상 작동합니다.

Multiply 연산 예시:  
배율: 0.5  
1 + (0.5 - 1) = 0.5, 정상 동작  

배율: 0.5, 0.5  
1 + (0.5 - 1) + (0.5 - 1) = 0, 비정상 (기대값: 0.25 또는 1?). 배율 덧셈 공식에서 1 미만의 값이 여러 개 들어가는 것은 논리적으로 맞지 않습니다. 파라곤은 [Multiply 모디파이어에 대해 가장 큰 음수 값만 사용](#cae-nonstackingge)하도록 설계되었으므로 BaseValue에 곱해지는 1 미만의 값은 항상 최대 1개뿐이었습니다.

배율: 1.1, 0.5  
1 + (0.5 - 1) + (1.1 - 1) = 0.6, 정상 동작  

배율: 5, 5  
1 + (5 - 1) + (5 - 1) = 9, 비정상 (기대값: 10 또는 25). 항상 모디파이어들의 합 - 모디파이어 개수 + 1이 됩니다.

많은 게임에서는 Multiply 및 Divide 모디파이어들이 BaseValue에 적용되기 전에 서로 곱해지거나 나눠지기를 원할 것입니다. 이를 달성하려면 FAggregatorModChannel::EvaluateWithBase()의 **엔진 코드를 수정**해야 합니다.

```c++
float FAggregatorModChannel::EvaluateWithBase(float InlineBaseValue, const FAggregatorEvaluateParameters& Parameters) const
{
	...
	float Multiplicitive = MultiplyMods(Mods[EGameplayModOp::Multiplicitive], Parameters);
	float Division = MultiplyMods(Mods[EGameplayModOp::Division], Parameters);
	...

	return ((InlineBaseValue + Additive) * Multiplicitive) / Division;
}
```

```c++
float FAggregatorModChannel::MultiplyMods(const TArray<FAggregatorMod>& InMods, const FAggregatorEvaluateParameters& Parameters)
{
	float Multiplier = 1.0f;

	for (const FAggregatorMod& Mod : InMods)
	{
		if (Mod.Qualifies())
		{
			Multiplier *= Mod.EvaluatedMagnitude;
		}
	}

	return Multiplier;
}
```

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ge-mods-gameplaytags"></a>
##### 4.5.4.2 모디파이어의 게임플레이 태그
각 [모디파이어](#concepts-ge-mods)마다 SourceTags와 TargetTags를 설정할 수 있습니다. 이들은 GameplayEffect의 [Application Tag Requirements](#concepts-ge-tags)와 동일하게 동작합니다. 즉, 이 태그들은 이펙트가 처음 적용될 때만 고려됩니다. 따라서 주기적인 무한 이펙트의 경우, 이펙트의 최초 적용 시에만 고려되며 매 주기 실행마다 확인되지는 *않습니다*.

Attribute Based 모디파이어는 SourceTagFilter와 TargetTagFilter를 설정할 수도 있습니다. Attribute Based 모디파이어의 소스가 되는 어트리뷰트의 크기를 결정할 때, 이 필터들을 사용하여 해당 어트리뷰트에 적용된 특정 모디파이어들을 제외할 수 있습니다. 소스나 대상이 필터의 모든 태그를 가지고 있지 않은 모디파이어는 제외됩니다.

상세 원리: 소스 ASC와 타겟 ASC의 태그는 GameplayEffects에 의해 캡처됩니다. 소스 ASC 태그는 GameplayEffectSpec이 생성될 때 캡처되고, 타겟 ASC 태그는 이펙트가 실행될 때 캡처됩니다. 지속(Duration) 또는 무한(Infinite) 이펙트의 모디파이어가 적용될 자격이 있는지(즉 Aggregator가 적합한지) 결정할 때 이 필터들이 설정되어 있다면, 캡처된 태그들을 해당 필터와 비교합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ge-stacking"></a>
#### 4.5.5 게임플레이 이펙트 스태킹 (중첩)
기본적으로 GameplayEffects는 적용 시 이전에 이미 존재하던 동일 GameplayEffectSpec 인스턴스를 신경 쓰지 않고 새로운 GameplayEffectSpec 인스턴스를 계속 추가합니다. GameplayEffects는 새 인스턴스를 추가하는 대신 기존에 존재하는 GameplayEffectSpec의 스택 수(Stack Count)를 변경하도록 스태킹(중첩)을 설정할 수 있습니다. 스태킹은 Duration 및 Infinite GameplayEffects에서만 작동합니다.

스태킹에는 소스별 집계(Aggregate by Source)와 타겟별 집계(Aggregate by Target) 두 가지 유형이 있습니다.

| 스태킹 유형 (Stacking Type)   | 설명                                                                                                                                 |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| 소스별 집계 (Aggregate by Source) | 타겟에 적용된 각 소스 ASC마다 별도의 스택 인스턴스를 유지합니다. 각 소스는 타겟에 X개만큼의 스택을 개별적으로 적용할 수 있습니다. |
| 타겟별 집계 (Aggregate by Target) | 소스에 관계없이 타겟에 단 하나의 공유 스택 인스턴스만 존재합니다. 각 소스는 공유 스택 한도까지 스택을 쌓을 수 있습니다.              |

스택에는 만료(Expiration), 지속 시간 갱신(Duration Refresh), 주기 리셋(Period Reset)에 대한 정책도 함께 제공되며, GameplayEffect 블루프린트에서 유용한 툴팁 안내를 확인할 수 있습니다.

샘플 프로젝트에는 GameplayEffect 스택 변경을 감지하는 커스텀 블루프린트 노드가 포함되어 있습니다. HUD UMG 위젯은 이를 사용하여 플레이어가 보유한 패시브 방어력 스택 수를 업데이트합니다. 이 AsyncTask는 수동으로 EndTask()를 호출할 때까지 계속 유지되며 UMG 위젯의 Destruct 이벤트에서 이를 호출합니다. AsyncTaskEffectStackChanged.h/cpp를 참조하세요.

![GameplayEffect 스택 변경 감지 BP 노드](https://github.com/tranek/GASDocumentation/raw/master/Images/gestackchange.png)

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ge-ga"></a>
#### 4.5.6 어빌리티 부여 (Granted Abilities)
GameplayEffects는 ASC에 새로운 [GameplayAbilities](#concepts-ga)를 부여할 수 있습니다. 오직 Duration 및 Infinite GameplayEffects만 어빌리티를 부여할 수 있습니다.

이에 대한 일반적인 사용 사례는 넉백(Knockback)이나 당기기(Pull)처럼 다른 플레이어에게 특정 행동을 강제하고 싶을 때입니다. 대상에게 GameplayEffect를 적용하여 원하는 동작을 수행하는 자동 활성화 어빌리티(부여 시 자동 활성화되는 방법은 [패시브 어빌리티](#concepts-ga-activating-passive) 참조)를 부여할 수 있습니다.

디자이너는 GameplayEffect가 부여할 어빌리티, 부여할 레벨, [바인딩할 입력](#concepts-ga-input), 부여된 어빌리티의 제거 정책을 선택할 수 있습니다.

| 제거 정책 (Removal Policy)     | 설명                                                                                                                                           |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| Cancel Ability Immediately     | 대상에서 어빌리티를 부여했던 GameplayEffect가 제거되는 즉시 부여된 어빌리티를 취소하고 제거합니다.                                          |
| Remove Ability on End          | 부여된 어빌리티가 정상 종료될 때까지 실행을 허용한 후 대상에서 제거합니다.                                                                     |
| Do Nothing                     | 대상에서 GameplayEffect가 제거되어도 부여된 어빌리티에 영향을 주지 않습니다. 대상은 나중에 수동으로 제거될 때까지 영구적으로 어빌리티를 유지합니다. |

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ge-tags"></a>
#### 4.5.7 게임플레이 이펙트 태그
GameplayEffects는 여러 개의 [GameplayTagContainers](#concepts-gt)를 보유합니다. 디자이너는 각 카테고리별로 Added와 Removed GameplayTagContainers를 편집하며, 컴파일 시 그 결과가 Combined GameplayTagContainer에 표시됩니다. Added 태그는 부모 클래스에는 없었지만 이 GameplayEffect가 새로 추가하는 태그입니다. Removed 태그는 부모 클래스에는 있었지만 이 서브클래스에서는 제외하고자 하는 태그입니다.

| 카테고리 (Category)               | 설명                                                                                                                                                                                                                                                                                                                 |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Gameplay Effect Asset Tags        | GameplayEffect 자체에 붙는 태그입니다. 그 자체로 어떤 기능을 수행하지는 않으며, 해당 GameplayEffect를 식별하고 설명하는 용도로만 사용됩니다.                                                                                                                                                                   |
| Granted Tags                      | GameplayEffect에 상주하면서 GameplayEffect가 적용된 대상의 ASC에 부여되는 태그입니다. 대상에서 GameplayEffect가 제거되면 ASC에서도 제거됩니다. 이는 Duration 및 Infinite GameplayEffects에서만 작동합니다.                                                                                             |
| Ongoing Tag Requirements          | 이펙트가 적용된 후, 이 태그 조건이 충족되는지에 따라 GameplayEffect의 활성화(On)/비활성화(Off) 여부가 결정됩니다. GameplayEffect는 꺼져 있어도 적용 상태를 유지할 수 있습니다. 조건 불충족으로 꺼졌다가 나중에 다시 조건을 만족하면 GameplayEffect가 다시 켜지며 모디파이어를 재적용합니다. Duration/Infinite 전용입니다. |
| Application Tag Requirements      | GameplayEffect가 대상에 적용될 수 있는지를 결정하는 대상의 태그 조건입니다. 이 조건이 충족되지 않으면 GameplayEffect는 적용되지 않습니다.                                                                                                                                                                     |
| Remove Gameplay Effects with Tags | 이 GameplayEffect가 성공적으로 적용되었을 때, 대상에 존재하는 GameplayEffects 중 자신의 Asset Tags나 Granted Tags에 이 태그들을 하나라도 가지고 있는 이펙트들을 대상에서 제거합니다.                                                                                                                       |

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ge-immunity"></a>
#### 4.5.8 면역 (Immunity)
GameplayEffects는 [GameplayTags](#concepts-gt)를 기반으로 다른 GameplayEffects의 적용을 차단하는 면역(Immunity)을 부여할 수 있습니다. Application Tag Requirements 등의 다른 방식으로도 면역 효과를 얻을 수 있지만, 이 면역 시스템을 사용하면 면역으로 인해 이펙트가 차단되었을 때 발동하는 UAbilitySystemComponent::OnImmunityBlockGameplayEffectDelegate 델리게이트를 활용할 수 있습니다.

GrantedApplicationImmunityTags는 소스 ASC(소스 어빌리티가 있다면 해당 AbilityTags 포함)가 지정된 태그를 가지고 있는지 확인합니다. 특정 캐릭터나 소스로부터 오는 모든 GameplayEffects에 대해 태그 기반으로 면역을 부여하는 방식입니다.

Granted Application Immunity Query는 유입되는 GameplayEffectSpec을 검사하여 쿼리와 일치하는 경우 적용을 차단하거나 허용합니다.

이 쿼리들에 대한 자세한 설명은 GameplayEffect 블루프린트의 툴팁에서 확인할 수 있습니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ge-spec"></a>
#### 4.5.9 게임플레이 이펙트 스펙 (Gameplay Effect Spec)
[GameplayEffectSpec](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FGameplayEffectSpec/index.html) (GESpec)은 GameplayEffects의 인스턴스화된 표현으로 생각할 수 있습니다. 자신이 나타내는 GameplayEffect 클래스에 대한 참조, 생성 당시의 레벨, 생성자(Instigator)에 대한 정보를 담고 있습니다. 런타임 전에 디자이너가 미리 생성해 두어야 하는 GameplayEffects와 달리, GameplayEffectSpec은 런타임에 자유롭게 생성하고 수정할 수 있습니다. GameplayEffect를 적용할 때 GameplayEffect로부터 GameplayEffectSpec이 생성되며, 실제로 대상에 적용되는 것은 바로 이 스펙입니다.

GameplayEffectSpecs는 블루프린트에서 호출 가능한 UAbilitySystemComponent::MakeOutgoingSpec()을 통해 GameplayEffects로부터 생성됩니다. GameplayEffectSpecs를 생성 즉시 적용할 필요는 없습니다. 어빌리티에서 생성된 투사체에 GameplayEffectSpec을 전달한 뒤, 투사체가 나중에 적중한 대상에게 적용하도록 만드는 패턴이 매우 흔하게 사용됩니다. GameplayEffectSpecs가 성공적으로 적용되면 FActiveGameplayEffect라는 새로운 구조체를 반환합니다.

주요 GameplayEffectSpec 구성 요소:
* 이 스펙이 생성된 원본 GameplayEffect 클래스
* 이 GameplayEffectSpec의 레벨. 일반적으로 스펙을 생성한 어빌리티의 레벨과 동일하지만 다를 수도 있습니다.
* GameplayEffectSpec의 지속 시간. 기본적으로 원본 GameplayEffect의 지속 시간을 따르지만 다르게 변경할 수 있습니다.
* 주기적 효과를 위한 GameplayEffectSpec의 주기. 기본적으로 원본 GameplayEffect의 주기를 따르지만 변경 가능합니다.
* 이 GameplayEffectSpec의 현재 스택 수. 스택 상한은 원본 GameplayEffect에 정의되어 있습니다.
* 이 GameplayEffectSpec을 누가 생성했는지 알려주는 [GameplayEffectContextHandle](#concepts-ge-context).
* 스냅샷으로 인해 GameplayEffectSpec 생성 시점에 캡처된 Attributes.
* 원본 GameplayEffect가 부여하는 태그 외에, 이 GameplayEffectSpec이 대상에 추가로 부여하는 DynamicGrantedTags.
* 원본 GameplayEffect의 에셋 태그 외에, 이 GameplayEffectSpec이 추가로 가지는 DynamicAssetTags.
* SetByCaller TMaps.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ge-spec-setbycaller"></a>
##### 4.5.9.1 SetByCallers
SetByCallers를 사용하면 GameplayEffectSpec이 GameplayTag 또는 FName과 연결된 float 값을 담아 전달할 수 있습니다. 이 값들은 GameplayEffectSpec의 TMap<FGameplayTag, float> 및 TMap<FName, float>에 저장됩니다. 이는 GameplayEffect의 Modifiers로 사용되거나 float 값을 전달하는 범용 수단으로 활용될 수 있습니다. 어빌리티 내부에서 계산된 수치 데이터를 SetByCallers를 통해 [GameplayEffectExecutionCalculations](#concepts-ge-ec)나 [ModifierMagnitudeCalculations](#concepts-ge-mmc)로 전달하는 것이 일반적인 방식입니다.

| SetByCaller 사용처 | 참고 사항                                                                                                                                                                                                                                                                                                                                                                                                   |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Modifiers          | GameplayEffect 클래스에 미리 정의되어 있어야 합니다. GameplayTag 버전만 사용할 수 있습니다. GameplayEffect 클래스에 정의되었으나 GameplayEffectSpec에 해당 태그와 float 값 쌍이 없는 경우, GameplayEffectSpec 적용 시 런타임 오류가 발생하며 0을 반환합니다. 이는 Divide 연산 시 문제가 될 수 있습니다. [Modifiers](#concepts-ge-mods) 참조.                                                |
| 기타 위치            | 어디에도 미리 정의할 필요가 없습니다. GameplayEffectSpec에 존재하지 않는 SetByCaller를 읽으려 할 경우, 경고 출력 여부와 함께 개발자가 지정한 기본값을 반환하도록 할 수 있습니다.                                                                                                                                                                                                                      |

블루프린트에서 SetByCaller 값을 할당하려면 필요한 버전(GameplayTag 또는 FName)에 맞는 블루프린트 노드를 사용합니다:

![SetByCaller 할당](https://github.com/tranek/GASDocumentation/raw/master/Images/setbycaller.png)

블루프린트에서 SetByCaller 값을 읽으려면 블루프린트 함수 라이브러리에 커스텀 노드를 구현해야 합니다.

C++에서 SetByCaller 값을 할당하려면 필요한 버전의 함수를 사용합니다:

```c++
void FGameplayEffectSpec::SetSetByCallerMagnitude(FName DataName, float Magnitude);
```
```c++
void FGameplayEffectSpec::SetSetByCallerMagnitude(FGameplayTag DataTag, float Magnitude);
```

C++에서 SetByCaller 값을 읽으려면 필요한 버전의 함수를 사용합니다:

```c++
float GetSetByCallerMagnitude(FName DataName, bool WarnIfNotFound = true, float DefaultIfNotFound = 0.f) const;
```
```c++
float GetSetByCallerMagnitude(FGameplayTag DataTag, bool WarnIfNotFound = true, float DefaultIfNotFound = 0.f) const;
```

블루프린트에서의 오타 실수를 방지할 수 있으므로 FName 버전보다는 GameplayTag 버전을 사용하는 것을 권장합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ge-context"></a>
#### 4.5.10 게임플레이 이펙트 컨텍스트 (Gameplay Effect Context)
[GameplayEffectContext](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FGameplayEffectContext/index.html) 구조체는 GameplayEffectSpec의 인스티게이터(발동자) 및 [TargetData](#concepts-targeting-data)에 대한 정보를 담고 있습니다. 또한 [ModifierMagnitudeCalculations](#concepts-ge-mmc) / [GameplayEffectExecutionCalculations](#concepts-ge-ec), [AttributeSets](#concepts-as), [GameplayCues](#concepts-gc) 간에 임의의 커스텀 데이터를 전달하기 위해 서브클래싱하기에 매우 적합한 구조체입니다.

GameplayEffectContext를 상속(서브클래싱)하는 단계:

1. FGameplayEffectContext를 상속하는 구조체를 만듭니다.
1. FGameplayEffectContext::GetScriptStruct()를 오버라이드합니다.
1. FGameplayEffectContext::Duplicate()를 오버라이드합니다.
1. 새로운 데이터가 리플리케이트되어야 한다면 FGameplayEffectContext::NetSerialize()를 오버라이드합니다.
1. 부모 구조체 FGameplayEffectContext처럼 서브클래스에 대해 TStructOpsTypeTraits를 구현합니다.
1. 커스텀 [AbilitySystemGlobals](#concepts-asg) 클래스에서 AllocGameplayEffectContext()를 오버라이드하여 서브클래스의 새 객체를 반환하도록 합니다.

[GASShooter](https://github.com/tranek/GASShooter)에서는 서브클래싱된 GameplayEffectContext를 사용하여 GameplayCues에서 접근할 수 있는 TargetData를 추가하며, 특히 여러 적을 동시에 타격할 수 있는 샷건에 이를 활용합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**
<a name="concepts-ge-mmc"></a>
#### 4.5.11 모디파이어 크기 계산 (Modifier Magnitude Calculation - MMC)
[ModifierMagnitudeCalculations](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayModMagnitudeCalculation/index.html) (ModMagCalc 또는 MMC)는 GameplayEffects의 [Modifiers](#concepts-ge-mods)로 사용되는 강력한 클래스입니다. [GameplayEffectExecutionCalculations](#concepts-ge-ec)과 유사하게 동작하지만 기능적으로는 약간 더 제한적인 대신 가장 중요한 장점으로 **[예측(Prediction)](#concepts-p)이 가능**합니다. MMC의 유일한 목적은 CalculateBaseMagnitude_Implementation() 함수에서 float 값을 계산하여 반환하는 것입니다. 블루프린트와 C++ 모두에서 이 함수를 상속 및 오버라이드할 수 있습니다.

MMC는 Instant, Duration, Infinite, Periodic 등 모든 지속 시간 유형의 GameplayEffects에서 사용할 수 있습니다.

MMC의 강력함은 GameplayEffectSpec에 완전히 접근하여 GameplayTags와 SetByCallers를 읽으면서, GameplayEffect의 Source나 Target에 속한 임의의 개수의 Attributes 값을 캡처할 수 있다는 점에서 나옵니다. Attributes는 스냅샷(Snapshot)으로 캡처하거나 비스냅샷으로 캡처할 수 있습니다. 스냅샷된 Attributes는 GameplayEffectSpec이 생성될 때 캡처되는 반면, 스냅샷되지 않은 Attributes는 GameplayEffectSpec이 적용될 때 캡처되며 Infinite 및 Duration GameplayEffects의 경우 해당 Attribute가 변경될 때마다 크기가 자동으로 업데이트됩니다. 어트리뷰트를 캡처하면 ASC의 기존 모디파이어들로부터 CurrentValue를 재계산합니다. 이 재계산 과정에서는 AttributeSet의 [PreAttributeChange()](#concepts-as-preattributechange)가 **호출되지 않으므로**, 필요한 모든 클램핑은 MMC 내부에서 다시 수행해야 합니다.

| 스냅샷 (Snapshot) | 소스 / 타겟 (Source or Target) | GameplayEffectSpec 캡처 시점 | Infinite/Duration GE에서 어트리뷰트 변경 시 자동 업데이트 여부 |
| ----------------- | ------------------------------ | ------------------------------ | ------------------------------------------------------------------ |
| 예 (Yes)          | Source                         | 생성 시점 (Creation)           | 아니오 (No)                                                        |
| 예 (Yes)          | Target                         | 적용 시점 (Application)        | 아니오 (No)                                                        |
| 아니오 (No)       | Source                         | 적용 시점 (Application)        | 예 (Yes)                                                           |
| 아니오 (No)       | Target                         | 적용 시점 (Application)        | 예 (Yes)                                                           |

MMC에서 계산된 결과 float 값은 GameplayEffect의 Modifier 설정에서 계수(Coefficient) 및 전/후 덧셈 값을 통해 추가로 수정될 수 있습니다.

다음은 Target의 마나 Attribute를 캡처하여 독(Poison) 효과로 마나를 감소시키되, 대상의 남은 마나 비율과 대상이 가진 태그에 따라 감소량을 변경하는 MMC 예제입니다:

```c++
UPAMMC_PoisonMana::UPAMMC_PoisonMana()
{
	// 헤더에 FGameplayEffectAttributeCaptureDefinition ManaDef; 로 정의됨
	ManaDef.AttributeToCapture = UPAAttributeSetBase::GetManaAttribute();
	ManaDef.AttributeSource = EGameplayEffectAttributeCaptureSource::Target;
	ManaDef.bSnapshot = false;

	// 헤더에 FGameplayEffectAttributeCaptureDefinition MaxManaDef; 로 정의됨
	MaxManaDef.AttributeToCapture = UPAAttributeSetBase::GetMaxManaAttribute();
	MaxManaDef.AttributeSource = EGameplayEffectAttributeCaptureSource::Target;
	MaxManaDef.bSnapshot = false;

	RelevantAttributesToCapture.Add(ManaDef);
	RelevantAttributesToCapture.Add(MaxManaDef);
}

float UPAMMC_PoisonMana::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec & Spec) const
{
	// 적용할 버프/효과에 영향을 줄 수 있으므로 소스와 타겟의 태그를 수집합니다.
	const FGameplayTagContainer* SourceTags = Spec.CapturedSourceTags.GetAggregatedTags();
	const FGameplayTagContainer* TargetTags = Spec.CapturedTargetTags.GetAggregatedTags();

	FAggregatorEvaluateParameters EvaluationParameters;
	EvaluationParameters.SourceTags = SourceTags;
	EvaluationParameters.TargetTags = TargetTags;

	float Mana = 0.f;
	GetCapturedAttributeMagnitude(ManaDef, Spec, EvaluationParameters, Mana);
	Mana = FMath::Max<float>(Mana, 0.0f);

	float MaxMana = 0.f;
	GetCapturedAttributeMagnitude(MaxManaDef, Spec, EvaluationParameters, MaxMana);
	MaxMana = FMath::Max<float>(MaxMana, 1.0f); // 0으로 나누기 방지

	float Reduction = -20.0f;
	if (Mana / MaxMana > 0.5f)
	{
		// 대상의 마나가 절반 이상 남아 있다면 효과를 2배로 증가
		Reduction *= 2;
	}
	
	if (TargetTags->HasTagExact(FGameplayTag::RequestGameplayTag(FName("Status.WeakToPoisonMana"))))
	{
		// 대상이 PoisonMana에 취약한 태그를 가지고 있다면 효과를 2배로 증가
		Reduction *= 2;
	}
	
	return Reduction;
}
```

만약 MMC 생성자에서 RelevantAttributesToCapture에 FGameplayEffectAttributeCaptureDefinition을 추가하지 않은 상태로 어트리뷰트 캡처를 시도하면, 캡처 중 Spec이 누락되었다는 오류가 발생합니다. 어트리뷰트를 캡처할 필요가 없다면 RelevantAttributesToCapture에 아무것도 추가하지 않아도 됩니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ge-ec"></a>
#### 4.5.12 게임플레이 이펙트 실행 계산 (Gameplay Effect Execution Calculation)
[GameplayEffectExecutionCalculations](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayEffectExecutionCalculat-/index.html) (ExecutionCalculation, Execution (플러그인 소스 코드에서 자주 볼 수 있는 용어), 또는 ExecCalc)은 GameplayEffects가 ASC를 변경할 수 있는 가장 강력한 수단입니다. [ModifierMagnitudeCalculations](#concepts-ge-mmc)과 마찬가지로 Attributes를 캡처하고 선택적으로 스냅샷을 찍을 수 있습니다. 하지만 MMC와 달리 둘 이상의 Attributes를 동시에 변경할 수 있으며 프로그래머가 원하는 거의 모든 임의의 로직을 실행할 수 있습니다. 이러한 막강한 성능과 유연성에 대한 대가로, ExecCalc는 **[예측(Prediction)](#concepts-p)할 수 없으며 반드시 C++로 구현**해야 합니다.

ExecutionCalculations는 오직 Instant 및 Periodic GameplayEffects에서만 사용할 수 있습니다. 이름에 'Execute'라는 단어가 들어간 요소들은 대개 이 두 가지 유형의 GameplayEffects를 가리킵니다.

스냅샷은 GameplayEffectSpec이 생성될 때 어트리뷰트를 캡처하고, 비스냅샷은 GameplayEffectSpec이 적용될 때 캡처합니다. 어트리뷰트를 캡처하면 ASC의 기존 모디파이어들로부터 CurrentValue를 재계산합니다. 이 재계산 과정에서는 AbilitySet의 [PreAttributeChange()](#concepts-as-preattributechange)가 **호출되지 않으므로**, 필요한 모든 클램핑은 여기서 다시 구현해야 합니다.

| 스냅샷 (Snapshot) | 소스 / 타겟 (Source or Target) | GameplayEffectSpec 캡처 시점 |
| ----------------- | ------------------------------ | ------------------------------ |
| 예 (Yes)          | Source                         | 생성 시점 (Creation)           |
| 예 (Yes)          | Target                         | 적용 시점 (Application)        |
| 아니오 (No)       | Source                         | 적용 시점 (Application)        |
| 아니오 (No)       | Target                         | 적용 시점 (Application)        |

어트리뷰트 캡처를 설정하기 위해, 우리는 Epic의 ActionRPG 샘플 프로젝트가 제시한 패턴에 따라 어트리뷰트 캡처 방식과 정의를 보관하는 구조체를 선언하고 구조체의 생성자에서 단일 인스턴스를 생성하는 방식을 따릅니다. 각 ExecCalc마다 이러한 구조체를 하나씩 가지게 됩니다. **참고:** 이 구조체들은 동일한 네임스페이스를 공유하므로 각각 고유한 이름을 가져야 합니다. 구조체 이름을 동일하게 사용하면 어트리뷰트 캡처 시 오동작(엉뚱한 어트리뷰트의 값을 캡처함)이 발생합니다.

Local Predicted, Server Only, Server Initiated [GameplayAbilities](#concepts-ga)의 경우, ExecCalc는 오직 서버에서만 호출됩니다.

Source와 Target의 여러 어트리뷰트를 읽어 복잡한 공식에 따라 받는 대미지를 계산하는 것이 ExecCalc의 가장 일반적인 예입니다. 포함된 샘플 프로젝트에는 GameplayEffectSpec의 [SetByCaller](#concepts-ge-spec-setbycaller)에서 대미지 값을 읽어온 뒤 Target에서 캡처한 방어력(Armor) Attribute를 기반으로 대미지를 경감시키는 단순한 ExecCalc가 구현되어 있습니다. GDDamageExecCalculation.cpp/.h를 참조하세요.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ge-ec-senddata"></a>
##### 4.5.12.1 실행 계산으로 데이터 전달
어트리뷰트를 캡처하는 것 외에도 ExecutionCalculation으로 데이터를 전달하는 데는 몇 가지 방법이 있습니다.

<a name="concepts-ge-ec-senddata-setbycaller"></a>
###### 4.5.12.1.1 SetByCaller
[GameplayEffectSpec에 설정된 모든 SetByCallers](#concepts-ge-spec-setbycaller)는 ExecutionCalculation에서 직접 읽을 수 있습니다.

```c++
const FGameplayEffectSpec& Spec = ExecutionParams.GetOwningSpec();
float Damage = FMath::Max<float>(Spec.GetSetByCallerMagnitude(FGameplayTag::RequestGameplayTag(FName("Data.Damage")), false, -1.0f), 0.0f);
```

<a name="concepts-ge-ec-senddata-backingdataattribute"></a>
###### 4.5.12.1.2 백킹 데이터 어트리뷰트 계산 모디파이어
GameplayEffect에 고정 수치를 하드코딩하여 전달하고 싶다면, 캡처된 Attributes 중 하나를 기반 데이터(backing data)로 사용하는 CalculationModifier를 통해 전달할 수 있습니다.

다음 스크린샷 예시에서는 캡처된 Damage Attribute에 50을 더하고 있습니다. 이를 Override로 설정하여 하드코딩된 값만 전달받도록 할 수도 있습니다.

![백킹 데이터 어트리뷰트 계산 모디파이어](https://github.com/tranek/GASDocumentation/raw/master/Images/calculationmodifierbackingdataattribute.png)

ExecutionCalculation은 어트리뷰트를 캡처할 때 이 값을 읽어옵니다.

```c++
float Damage = 0.0f;
// ExecutionCalculation 아래의 CalculationModifier로 대미지 GE에 설정된 선택적 대미지 값을 캡처합니다.
ExecutionParams.AttemptCalculateCapturedAttributeMagnitude(DamageStatics().DamageDef, EvaluationParameters, Damage);
```

<a name="concepts-ge-ec-senddata-backingdatatempvariable"></a>
###### 4.5.12.1.3 백킹 데이터 임시 변수 계산 모디파이어
GameplayEffect에 고정 수치를 하드코딩하여 전달하고 싶다면, C++에서 임시 변수(Temporary Variable) 또는 트랜지언트 애그리게이터(Transient Aggregator)라고 부르는 방식을 사용하는 CalculationModifier를 통해 전달할 수 있습니다. 이 임시 변수는 특정 GameplayTag와 연결됩니다.

다음 스크린샷 예시에서는 Data.Damage 게임플레이 태그를 사용하여 Temporary Variable에 50을 더하고 있습니다.

![백킹 데이터 임시 변수 계산 모디파이어](https://github.com/tranek/GASDocumentation/raw/master/Images/calculationmodifierbackingdatatempvariable.png)

ExecutionCalculation의 생성자에서 기반 Temporary Variables를 추가합니다:

```c++
ValidTransientAggregatorIdentifiers.AddTag(FGameplayTag::RequestGameplayTag("Data.Damage"));
```

ExecutionCalculation은 어트리뷰트 캡처 함수와 유사한 전용 캡처 함수를 사용하여 이 값을 읽습니다.

```c++
float Damage = 0.0f;
ExecutionParams.AttemptCalculateTransientAggregatorMagnitude(FGameplayTag::RequestGameplayTag("Data.Damage"), EvaluationParameters, Damage);
```

<a name="concepts-ge-ec-senddata-effectcontext"></a>
###### 4.5.12.1.4 게임플레이 이펙트 컨텍스트
[GameplayEffectSpec의 커스텀 GameplayEffectContext](#concepts-ge-context)를 통해 ExecutionCalculation으로 임의의 데이터를 전달할 수 있습니다.

ExecutionCalculation에서는 FGameplayEffectCustomExecutionParameters로부터 EffectContext에 접근할 수 있습니다.

```c++
const FGameplayEffectSpec& Spec = ExecutionParams.GetOwningSpec();
FGSGameplayEffectContext* ContextHandle = static_cast<FGSGameplayEffectContext*>(Spec.GetContext().Get());
```

GameplayEffectSpec이나 EffectContext의 내용을 수정해야 하는 경우:

```c++
FGameplayEffectSpec* MutableSpec = ExecutionParams.GetOwningSpecForPreExecuteMod();
FGSGameplayEffectContext* ContextHandle = static_cast<FGSGameplayEffectContext*>(MutableSpec->GetContext().Get());
```

ExecutionCalculation에서 GameplayEffectSpec을 수정할 때는 주의를 기울여야 합니다. GetOwningSpecForPreExecuteMod()의 주석을 확인하세요.

```c++
/** Non const access. Be careful with this, especially when modifying a spec after attribute capture. */
FGameplayEffectSpec* GetOwningSpecForPreExecuteMod() const;
```

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ge-car"></a>
#### 4.5.13 커스텀 적용 조건 (Custom Application Requirement)
[CustomApplicationRequirement](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayEffectCustomApplication-/index.html) (CAR) 클래스는 GameplayEffect의 단순 GameplayTag 검사를 넘어 디자이너에게 이펙트 적용 여부에 대한 고급 제어 권한을 제공합니다. 블루프린트에서는 CanApplyGameplayEffect()를 오버라이드하고, C++에서는 CanApplyGameplayEffect_Implementation()을 오버라이드하여 구현할 수 있습니다.

CAR 사용 예시:
* Target이 특정 수치 이상의 Attribute를 보유하고 있어야 함
* Target이 특정 GameplayEffect의 스택을 일정 개수 이상 가지고 있어야 함

CAR는 이 GameplayEffect 인스턴스가 이미 대상에 존재하는지 확인하고, 새 인스턴스를 적용하는 대신 기존 인스턴스의 [지속 시간을 연장/변경](#concepts-ge-duration)하는 등의 고급 작업도 수행할 수 있습니다 (CanApplyGameplayEffect()에서 false 반환).

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ge-cost"></a>
#### 4.5.14 비용 게임플레이 이펙트 (Cost Gameplay Effect)
[GameplayAbilities](#concepts-ga)는 어빌리티의 소모 비용으로 사용하도록 특별히 설계된 선택적 GameplayEffect를 가집니다. 비용이란 ASC가 GameplayAbility를 활성화하기 위해 보유해야 하는 Attribute 수치입니다. 만약 GA가 Cost GE의 조건을 감당할 수 없다면 활성화되지 않습니다. 이 Cost GE는 Attributes를 차감하는 하나 이상의 Modifiers를 가진 Instant GameplayEffect여야 합니다. 기본적으로 Cost GEs는 예측(Predicted)되도록 설계되어 있으며, 이 기능을 유지하려면 ExecutionCalculations를 사용하지 않는 것이 좋습니다. 복잡한 비용 계산에는 MMCs를 사용하는 것이 완벽하게 권장됩니다.

처음 시작할 때는 비용이 있는 GA마다 고유한 Cost GE를 하나씩 만들게 될 것입니다. 더 고급 기법은 여러 GA에서 하나의 Cost GE를 공유하고, Cost GE로부터 생성된 GameplayEffectSpec을 GA별 데이터(비용 수치는 GA에 정의됨)로 동적으로 수정하는 것입니다. **이 방식은 Instanced 어빌리티에서만 작동합니다.**

Cost GE를 재사용하는 두 가지 기법:

1. **MMC 사용.** 가장 쉬운 방법입니다. GameplayEffectSpec에서 가져올 수 있는 GameplayAbility 인스턴스로부터 비용 값을 읽어오는 [MMC](#concepts-ge-mmc)를 작성합니다.

```c++
float UPGMMC_HeroAbilityCost::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec & Spec) const
{
	const UPGGameplayAbility* Ability = Cast<UPGGameplayAbility>(Spec.GetContext().GetAbilityInstance_NotReplicated());

	if (!Ability)
	{
		return 0.0f;
	}

	return Ability->Cost.GetValueAtLevel(Ability->GetAbilityLevel());
}
```

이 예제에서 비용 값은 제가 GameplayAbility 자식 클래스에 추가한 FScalableFloat 변수입니다.
```c++
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cost")
FScalableFloat Cost;
```

![MMC를 활용한 Cost GE](https://github.com/tranek/GASDocumentation/raw/master/Images/costmmc.png)

2. **UGameplayAbility::GetCostGameplayEffect() 오버라이드.** 이 함수를 오버라이드하여 GameplayAbility의 비용 값을 읽어 [런타임에 동적으로 GameplayEffect를 생성](#concepts-ge-dynamic)합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ge-cooldown"></a>
#### 4.5.15 쿨다운 게임플레이 이펙트 (Cooldown Gameplay Effect)
[GameplayAbilities](#concepts-ga)는 어빌리티의 재사용 대기시간(쿨다운)으로 사용하도록 특별히 설계된 선택적 GameplayEffect를 가집니다. 쿨다운은 활성화 후 해당 어빌리티를 다시 활성화할 수 있을 때까지의 시간을 결정합니다. GA가 아직 쿨다운 중이라면 활성화될 수 없습니다. 이 Cooldown GE는 Modifiers가 없으며, GameplayEffect의 GrantedTags에 GameplayAbility별 또는 슬롯별(교체 가능한 어빌리티들이 쿨다운을 공유하는 슬롯 시스템의 경우) 고유한 GameplayTag("Cooldown Tag")를 가진 Duration GameplayEffect여야 합니다. GA는 실제로 Cooldown GE의 존재 여부가 아니라 이 Cooldown Tag의 존재 여부를 검사합니다. 기본적으로 Cooldown GEs는 예측(Predicted)되도록 설계되어 있으므로 ExecutionCalculations를 사용하지 않는 것을 권장합니다. 복잡한 쿨다운 계산에는 MMCs 사용이 권장됩니다.

처음 시작할 때는 쿨다운이 있는 GA마다 고유한 Cooldown GE를 하나씩 만들게 됩니다. 더 고급 기법은 여러 GA에서 하나의 Cooldown GE를 공유하고, Cooldown GE로부터 생성된 GameplayEffectSpec을 GA별 데이터(쿨다운 지속 시간 및 Cooldown Tag는 GA에 정의됨)로 동적으로 수정하는 것입니다. **이 방식은 Instanced 어빌리티에서만 작동합니다.**

Cooldown GE를 재사용하는 두 가지 기법:

1. **[SetByCaller](#concepts-ge-spec-setbycaller) 사용.** 가장 쉬운 방법입니다. 공유 Cooldown GE의 지속 시간을 GameplayTag를 사용하는 SetByCaller로 설정합니다. GameplayAbility 서브클래스에서 지속 시간을 위한 float / FScalableFloat, 고유한 Cooldown Tag를 위한 FGameplayTagContainer, 그리고 Cooldown Tag와 Cooldown GE 태그의 합집합 반환 포인터로 사용할 임시 FGameplayTagContainer를 정의합니다.
```c++
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FScalableFloat CooldownDuration;

UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FGameplayTagContainer CooldownTags;

// GetCooldownTags()에서 포인터를 반환할 임시 컨테이너입니다.
// 우리 CooldownTags와 Cooldown GE의 쿨다운 태그의 합집합이 됩니다.
UPROPERTY(Transient)
FGameplayTagContainer TempCooldownTags;
```

그런 다음 UGameplayAbility::GetCooldownTags()를 오버라이드하여 우리 Cooldown Tags와 기존 Cooldown GE 태그들의 합집합을 반환합니다.
```c++
const FGameplayTagContainer * UPGGameplayAbility::GetCooldownTags() const
{
	FGameplayTagContainer* MutableTags = const_cast<FGameplayTagContainer*>(&TempCooldownTags);
	MutableTags->Reset(); // CDO의 TempCooldownTags에 기록하므로 어빌리티 쿨다운 태그 변경(다른 슬롯 이동 등)에 대비해 초기화합니다.
	const FGameplayTagContainer* ParentTags = Super::GetCooldownTags();
	if (ParentTags)
	{
		MutableTags->AppendTags(*ParentTags);
	}
	MutableTags->AppendTags(CooldownTags);
	return MutableTags;
}
```

마지막으로 UGameplayAbility::ApplyCooldown()을 오버라이드하여 쿨다운 GameplayEffectSpec에 Cooldown Tags를 주입하고 SetByCaller 값을 설정합니다.
```c++
void UPGGameplayAbility::ApplyCooldown(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo * ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo) const
{
	UGameplayEffect* CooldownGE = GetCooldownGameplayEffect();
	if (CooldownGE)
	{
		FGameplayEffectSpecHandle SpecHandle = MakeOutgoingGameplayEffectSpec(CooldownGE->GetClass(), GetAbilityLevel());
		SpecHandle.Data.Get()->DynamicGrantedTags.AppendTags(CooldownTags);
		SpecHandle.Data.Get()->SetSetByCallerMagnitude(FGameplayTag::RequestGameplayTag(FName(  OurSetByCallerTag  )), CooldownDuration.GetValueAtLevel(GetAbilityLevel()));
		ApplyGameplayEffectSpecToOwner(Handle, ActorInfo, ActivationInfo, SpecHandle);
	}
}
```

이 그림에서 쿨다운의 지속 시간 Modifier는 Data.Cooldown 데이터 태그를 가진 SetByCaller로 설정되어 있습니다. Data.Cooldown이 위 코드의 OurSetByCallerTag에 해당합니다.

![SetByCaller를 활용한 Cooldown GE](https://github.com/tranek/GASDocumentation/raw/master/Images/cooldownsbc.png)

2. **[MMC](#concepts-ge-mmc) 사용.** Cooldown GE의 지속 시간에 SetByCaller를 사용하는 대신, 지속 시간을 Custom Calculation Class로 설정하고 새로 만들 MMC를 지정한다는 점을 제외하면 위 설정과 동일합니다.
```c++
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FScalableFloat CooldownDuration;

UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FGameplayTagContainer CooldownTags;

// GetCooldownTags()에서 포인터를 반환할 임시 컨테이너입니다.
UPROPERTY(Transient)
FGameplayTagContainer TempCooldownTags;
```

동일하게 UGameplayAbility::GetCooldownTags()를 오버라이드합니다.
```c++
const FGameplayTagContainer * UPGGameplayAbility::GetCooldownTags() const
{
	FGameplayTagContainer* MutableTags = const_cast<FGameplayTagContainer*>(&TempCooldownTags);
	MutableTags->Reset();
	const FGameplayTagContainer* ParentTags = Super::GetCooldownTags();
	if (ParentTags)
	{
		MutableTags->AppendTags(*ParentTags);
	}
	MutableTags->AppendTags(CooldownTags);
	return MutableTags;
}
```

UGameplayAbility::ApplyCooldown()을 오버라이드하여 Cooldown Tags를 주입합니다.
```c++
void UPGGameplayAbility::ApplyCooldown(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo * ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo) const
{
	UGameplayEffect* CooldownGE = GetCooldownGameplayEffect();
	if (CooldownGE)
	{
		FGameplayEffectSpecHandle SpecHandle = MakeOutgoingGameplayEffectSpec(CooldownGE->GetClass(), GetAbilityLevel());
		SpecHandle.Data.Get()->DynamicGrantedTags.AppendTags(CooldownTags);
		ApplyGameplayEffectSpecToOwner(Handle, ActorInfo, ActivationInfo, SpecHandle);
	}
}
```

MMC 구현:
```c++
float UPGMMC_HeroAbilityCooldown::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec & Spec) const
{
	const UPGGameplayAbility* Ability = Cast<UPGGameplayAbility>(Spec.GetContext().GetAbilityInstance_NotReplicated());

	if (!Ability)
	{
		return 0.0f;
	}

	return Ability->CooldownDuration.GetValueAtLevel(Ability->GetAbilityLevel());
}
```

![MMC를 활용한 Cooldown GE](https://github.com/tranek/GASDocumentation/raw/master/Images/cooldownmmc.png)

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ge-cooldown-tr"></a>
##### 4.5.15.1 쿨다운 게임플레이 이펙트 잔여 시간 가져오기
```c++
bool APGPlayerState::GetCooldownRemainingForTag(FGameplayTagContainer CooldownTags, float & TimeRemaining, float & CooldownDuration)
{
	if (AbilitySystemComponent && CooldownTags.Num() > 0)
	{
		TimeRemaining = 0.f;
		CooldownDuration = 0.f;

		FGameplayEffectQuery const Query = FGameplayEffectQuery::MakeQuery_MatchAnyOwningTags(CooldownTags);
		TArray< TPair<float, float> > DurationAndTimeRemaining = AbilitySystemComponent->GetActiveEffectsTimeRemainingAndDuration(Query);
		if (DurationAndTimeRemaining.Num() > 0)
		{
			int32 BestIdx = 0;
			float LongestTime = DurationAndTimeRemaining[0].Key;
			for (int32 Idx = 1; Idx < DurationAndTimeRemaining.Num(); ++Idx)
			{
				if (DurationAndTimeRemaining[Idx].Key > LongestTime)
				{
					LongestTime = DurationAndTimeRemaining[Idx].Key;
					BestIdx = Idx;
				}
			}

			TimeRemaining = DurationAndTimeRemaining[BestIdx].Key;
			CooldownDuration = DurationAndTimeRemaining[BestIdx].Value;

			return true;
		}
	}

	return false;
}
```

**참고:** 클라이언트에서 쿨다운 잔여 시간을 쿼리하려면 클라이언트가 리플리케이트된 GameplayEffects를 수신할 수 있어야 합니다. 이는 해당 ASC의 [리플리케이션 모드](#concepts-asc-rm)에 따라 달라집니다.

<a name="concepts-ge-cooldown-listen"></a>
##### 4.5.15.2 쿨다운 시작 및 종료 감지
쿨다운 시작 시점을 감지하려면 AbilitySystemComponent->OnActiveGameplayEffectAddedDelegateToSelf에 바인딩하여 Cooldown GE가 적용되는 시점에 반응하거나, AbilitySystemComponent->RegisterGameplayTagEvent(CooldownTag, EGameplayTagEventType::NewOrRemoved)에 바인딩하여 Cooldown Tag가 추가되는 시점에 반응할 수 있습니다. Cooldown GE 적용 시점에 반응하는 것을 추천합니다. 이를 적용한 GameplayEffectSpec에도 접근할 수 있으므로 해당 Cooldown GE가 로컬에서 예측된 것인지 서버의 보정본인지 판별할 수 있기 때문입니다.

쿨다운 종료 시점을 감지하려면 AbilitySystemComponent->OnAnyGameplayEffectRemovedDelegate()에 바인딩하여 Cooldown GE가 제거되는 시점에 반응하거나, AbilitySystemComponent->RegisterGameplayTagEvent(CooldownTag, EGameplayTagEventType::NewOrRemoved)에 바인딩하여 Cooldown Tag가 제거되는 시점에 반응할 수 있습니다. 여기서는 Cooldown Tag 제거 시점에 반응하는 것을 권장합니다. 서버의 보정된 Cooldown GE가 도착할 때 로컬에서 예측되었던 이전 Cooldown GE가 제거되면서 아직 쿨다운 중임에도 OnAnyGameplayEffectRemovedDelegate()가 조기 발동할 수 있기 때문입니다. 반면 Cooldown Tag는 예측 Cooldown GE 제거 및 서버의 보정 Cooldown GE 적용 과정에서도 끊김 없이 유지됩니다.

**참고:** 클라이언트에서 GameplayEffect 추가 및 제거를 감지하려면 클라이언트가 리플리케이트된 GameplayEffects를 수신할 수 있어야 합니다. 이는 해당 ASC의 [리플리케이션 모드](#concepts-asc-rm)에 따라 결정됩니다.

샘플 프로젝트에는 쿨다운 시작과 종료를 감지하는 커스텀 블루프린트 노드가 포함되어 있습니다. HUD UMG 위젯은 이를 사용하여 Meteor(운석) 어빌리티의 남은 쿨다운 시간을 업데이트합니다. 이 AsyncTask는 수동으로 EndTask()를 호출할 때까지 유지되며 UMG 위젯의 Destruct 이벤트에서 이를 호출합니다. [AsyncTaskCooldownChanged.h/cpp](Source/GASDocumentation/Private/Characters/Abilities/AsyncTaskCooldownChanged.cpp)를 참조하세요.

![쿨다운 변경 감지 BP 노드](https://github.com/tranek/GASDocumentation/raw/master/Images/cooldownchange.png)

<a name="concepts-ge-cooldown-prediction"></a>
##### 4.5.15.3 쿨다운 예측 (Predicting Cooldowns)
현재 GAS에서는 쿨다운을 완벽하게 클라이언트에서 예측할 수 없습니다. 로컬에서 예측된 Cooldown GE가 적용될 때 UI 쿨다운 타이머를 시작할 수는 있지만, GameplayAbility의 실제 쿨다운은 서버측 쿨다운 잔여 시간에 종속됩니다. 플레이어의 핑(지연시간)에 따라 로컬 예측 쿨다운이 만료되었더라도 서버에서는 여전히 어빌리티가 쿨다운 중일 수 있으며, 이로 인해 서버의 쿨다운이 끝날 때까지 어빌리티의 즉각적인 재활성화가 차단됩니다.

샘플 프로젝트는 로컬 예측 쿨다운이 시작될 때 Meteor 어빌리티의 UI 아이콘을 비활성화(회색 처리)하고, 서버의 보정된 Cooldown GE가 도착했을 때 쿨다운 타이머를 시작하는 방식으로 이를 처리합니다.

이로 인한 게임플레이적 결과는 지연 시간이 높은 플레이어가 지연 시간이 낮은 플레이어에 비해 쿨다운이 짧은 어빌리티의 연사 속도가 떨어져 불리해진다는 점입니다. 포트나이트(Fortnite)는 총기류에 쿨다운 GameplayEffects를 사용하지 않고 자체적인 커스텀 장부 관리 로직을 구현하여 이 문제를 회피했습니다.

진정한 의미의 쿨다운 예측(로컬 쿨다운이 만료되면 서버가 아직 쿨다운 중이라도 플레이어가 GameplayAbility를 활성화할 수 있도록 허용)은 Epic이 [GAS의 향후 개선](#concepts-p-future)에서 언젠가 구현하고자 하는 목표입니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ge-duration"></a>
#### 4.5.16 활성 게임플레이 이펙트의 지속 시간 변경
Cooldown GE나 임의의 Duration GameplayEffect의 잔여 시간을 변경하려면, GameplayEffectSpec의 Duration을 변경하고, StartServerWorldTime, CachedStartServerWorldTime, StartWorldTime을 업데이트한 뒤 CheckDuration()으로 지속 시간 검사를 다시 실행해야 합니다. 서버에서 이를 수행하고 FActiveGameplayEffect를 더티(Dirty) 상태로 표시하면 변경 사항이 클라이언트로 리플리케이트됩니다.
**참고:** 이 방식은 const_cast를 수반하며 Epic이 의도한 지속 시간 변경 공식 방식이 아닐 수 있지만, 지금까지 매우 잘 동작하고 있습니다.

```c++
bool UPAAbilitySystemComponent::SetGameplayEffectDurationHandle(FActiveGameplayEffectHandle Handle, float NewDuration)
{
	if (!Handle.IsValid())
	{
		return false;
	}

	const FActiveGameplayEffect* ActiveGameplayEffect = GetActiveGameplayEffect(Handle);
	if (!ActiveGameplayEffect)
	{
		return false;
	}

	FActiveGameplayEffect* AGE = const_cast<FActiveGameplayEffect*>(ActiveGameplayEffect);
	if (NewDuration > 0)
	{
		AGE->Spec.Duration = NewDuration;
	}
	else
	{
		AGE->Spec.Duration = 0.01f;
	}

	AGE->StartServerWorldTime = ActiveGameplayEffects.GetServerWorldTime();
	AGE->CachedStartServerWorldTime = AGE->StartServerWorldTime;
	AGE->StartWorldTime = ActiveGameplayEffects.GetWorldTime();
	ActiveGameplayEffects.MarkItemDirty(*AGE);
	ActiveGameplayEffects.CheckDuration(Handle);

	AGE->EventSet.OnTimeChanged.Broadcast(AGE->Handle, AGE->StartWorldTime, AGE->GetDuration());
	OnGameplayEffectDurationChange(*AGE);

	return true;
}
```

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ge-dynamic"></a>
#### 4.5.17 런타임에 동적 게임플레이 이펙트 생성
런타임에 동적 GameplayEffects를 생성하는 것은 고급 주제입니다. 자주 사용할 필요는 없습니다.

C++에서 런타임에 처음부터 동적으로 생성할 수 있는 것은 오직 Instant GameplayEffects뿐입니다. Duration 및 Infinite GameplayEffects는 리플리케이트될 때 존재하지 않는 GameplayEffect 클래스 정의를 찾기 때문에 런타임에 동적으로 생성할 수 없습니다. 이러한 기능이 필요하다면 에디터에서 통상적으로 하듯이 아키타입 GameplayEffect 클래스를 미리 만든 다음, 런타임에 해당 GameplayEffectSpec 인스턴스를 필요한 값으로 커스터마이징해야 합니다.

런타임에 생성된 Instant GameplayEffects는 [로컬 예측(local predicted)](#concepts-p) GameplayAbility 내부에서도 호출될 수 있습니다. 다만 이러한 동적 생성이 예상치 못한 부작용을 일으킬 수 있는지는 아직 완전히 검증되지 않았습니다.

##### 예제

샘플 프로젝트는 캐릭터가 사망 타격을 입었을 때 AttributeSet에서 처치자에게 골드와 경험치를 지급하기 위해 동적 즉시 GE를 하나 생성합니다.

```c++
// 현상금(골드/XP) 지급을 위한 동적 즉시 Gameplay Effect 생성
UGameplayEffect* GEBounty = NewObject<UGameplayEffect>(GetTransientPackage(), FName(TEXT("Bounty")));
GEBounty->DurationPolicy = EGameplayEffectDurationType::Instant;

int32 Idx = GEBounty->Modifiers.Num();
GEBounty->Modifiers.SetNum(Idx + 2);

FGameplayModifierInfo& InfoXP = GEBounty->Modifiers[Idx];
InfoXP.ModifierMagnitude = FScalableFloat(GetXPBounty());
InfoXP.ModifierOp = EGameplayModOp::Additive;
InfoXP.Attribute = UGDAttributeSetBase::GetXPAttribute();

FGameplayModifierInfo& InfoGold = GEBounty->Modifiers[Idx + 1];
InfoGold.ModifierMagnitude = FScalableFloat(GetGoldBounty());
InfoGold.ModifierOp = EGameplayModOp::Additive;
InfoGold.Attribute = UGDAttributeSetBase::GetGoldAttribute();

Source->ApplyGameplayEffectToSelf(GEBounty, 1.0f, Source->MakeEffectContext());
```

두 번째 예제는 로컬 예측 GameplayAbility 내에서 생성된 런타임 GameplayEffect를 보여줍니다. 사용 시 주의를 요합니다 (코드 내 주석 참조)!

```c++
UGameplayAbilityRuntimeGE::UGameplayAbilityRuntimeGE()
{
	NetExecutionPolicy = EGameplayAbilityNetExecutionPolicy::LocalPredicted;
}

void UGameplayAbilityRuntimeGE::ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData)
{
	if (HasAuthorityOrPredictionKey(ActorInfo, &ActivationInfo))
	{
		if (!CommitAbility(Handle, ActorInfo, ActivationInfo))
		{
			EndAbility(Handle, ActorInfo, ActivationInfo, true, true);
		}

		// 런타임에 GE 생성
		UGameplayEffect* GameplayEffect = NewObject<UGameplayEffect>(GetTransientPackage(), TEXT("RuntimeInstantGE"));
		GameplayEffect->DurationPolicy = EGameplayEffectDurationType::Instant; // 런타임 GE는 Instant만 동작함

		// MyAttribute를 42로 덮어쓰는 간단한 스케일러블 플로트 모디파이어 추가
		// 실제 환경에서는 TriggerEventData를 통해 전달된 정보를 사용합니다.
		const int32 Idx = GameplayEffect->Modifiers.Num();
		GameplayEffect->Modifiers.SetNum(Idx + 1);
		FGameplayModifierInfo& ModifierInfo = GameplayEffect->Modifiers[Idx];
		ModifierInfo.Attribute.SetUProperty(UMyAttributeSet::GetMyModifiedAttribute());
		ModifierInfo.ModifierMagnitude = FScalableFloat(42.f);
		ModifierInfo.ModifierOp = EGameplayModOp::Override;

		// GE 적용

		// GE 클래스 기본 객체(CDO)로부터 GESpec을 생성하려는 ASC의 기본 동작을 방지하기 위해 여기서 GESpec을 직접 생성합니다.
		// 동적 GE이므로 기본 방식으로 생성하면 기본 GameplayEffect 클래스로 GESpec이 만들어져 모디파이어가 유실됩니다.
		// 주의: 여기서 사용된 일종의 편법(hack)이 다른 문제를 일으킬 수 있는지는 알려지지 않았습니다!
		// GE가 스펙의 UPROPERTY로 지정되어 있으므로 스펙이 존재하는 동안 가비지 컬렉터에 의해 수거되는 것을 방지합니다.
		FGameplayEffectSpec* GESpec = new FGameplayEffectSpec(GameplayEffect, {}, 0.f); // 핸들 내부의 shared ptr가 수명을 관리하므로 "new" 사용
		ApplyGameplayEffectSpecToOwner(Handle, ActorInfo, ActivationInfo, FGameplayEffectSpecHandle(GESpec));
	}
	EndAbility(Handle, ActorInfo, ActivationInfo, false, false);
}
```

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ge-containers"></a>
#### 4.5.18 게임플레이 이펙트 컨테이너 (Gameplay Effect Containers)
Epic의 [Action RPG 샘플 프로젝트](https://www.unrealengine.com/marketplace/en-US/product/action-rpg)는 FGameplayEffectContainer라는 구조체를 구현했습니다. 이는 바닐라 GAS 기본 엔진 코드에는 포함되어 있지 않지만, GameplayEffects와 [TargetData](#concepts-targeting-data)를 함께 묶어 담기에 매우 유용한 구조체입니다. GameplayEffects로부터 GameplayEffectSpecs를 생성하고 GameplayEffectContext의 기본값을 설정하는 등의 반복 작업을 자동화해 줍니다. GameplayAbility에서 GameplayEffectContainer를 생성하여 스폰된 투사체에 넘겨주는 구조가 매우 깔끔하고 직관적입니다. 본 문서의 샘플 프로젝트는 바닐라 GAS에서 순수하게 작업하는 방식을 보여주기 위해 GameplayEffectContainers를 구현하지 않았지만, 개인 프로젝트에 이를 검토하고 도입하는 것을 강력히 추천합니다.

SetByCallers 등을 추가하기 위해 GameplayEffectContainers 내부의 GESpecs에 접근하려면, FGameplayEffectContainer를 Break(분해)한 후 GESpecs 배열에서 인덱스를 통해 해당 GESpec 참조에 접근합니다. 이 경우 접근하고자 하는 GESpec의 인덱스를 사전에 알고 있어야 합니다.

![GameplayEffectContainer를 사용한 SetByCaller](https://github.com/tranek/GASDocumentation/raw/master/Images/gecontainersetbycaller.png)

GameplayEffectContainers는 또한 효율적인 [타겟팅 수단](#concepts-targeting-containers)도 선택적으로 포함하고 있습니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ga"></a>
### 4.6 게임플레이 어빌리티 (Gameplay Abilities)

<a name="concepts-ga-definition"></a>
#### 4.6.1 게임플레이 어빌리티 정의
[`GameplayAbilities`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/UGameplayAbility/index.html) (`GA`)는 게임 내에서 `Actor`가 수행할 수 있는 모든 액션이나 스킬을 의미합니다. 질주하면서 총을 쏘는 것처럼 동시에 둘 이상의 `GameplayAbility`가 활성화될 수 있습니다. 블루프린트 또는 C++로 제작할 수 있습니다.

`GameplayAbilities`의 사용 예시:
* 점프 (Jumping)
* 질주 (Sprinting)
* 총기 발사 (Shooting a gun)
* 매 X초마다 패시브로 공격 방어
* 포션 사용
* 문 열기
* 자원 채집
* 건물 건설

`GameplayAbilities`로 구현하지 말아야 할 것들:
* 기본적인 이동 입력 (WASD 등)
* 일부 UI 상호작용 - 상점에서 아이템을 구매하는 동작에 `GameplayAbility`를 사용하지 마십시오.

이것들은 절대적인 규칙이 아니라 제 경험에 따른 권장 사항입니다. 프로젝트의 기획과 구현 방식에 따라 달라질 수 있습니다.

`GameplayAbilities`에는 어트리뷰트 변경 수치나 어빌리티 기능을 레벨별로 변경할 수 있는 레벨(Level) 기능이 기본 제공됩니다.

`GameplayAbilities`는 [`Net Execution Policy`](#concepts-ga-net)(넷 실행 정책)에 따라 소유 클라이언트 및/또는 서버에서 실행되며, **시뮬레이트 프록시(Simulated Proxies)에서는 실행되지 않습니다**. `Net Execution Policy`는 `GameplayAbility`가 로컬에서 [예측(Predicted)](#concepts-p)될지 여부를 결정합니다. 또한 [선택적인 비용 및 쿨다운 `GameplayEffects`](#concepts-ga-commit)에 대한 기본 동작이 내장되어 있습니다. `GameplayAbilities`는 이벤트 대기, 어트리뷰트 변경 대기, 플레이어의 타겟 지정 대기, `Root Motion Source`를 통한 `Character` 이동처럼 시간의 경과에 따라 진행되는 액션을 위해 [`AbilityTasks`](#concepts-at)를 사용합니다. **시뮬레이트 클라이언트는 `GameplayAbilities`를 직접 실행하지 않습니다.** 대신 서버가 어빌리티를 실행할 때, 시뮬레이트 프록시에서 시각적으로 재생되어야 하는 요소(예: 애니메이션 몽타주)는 `AbilityTasks`를 통해 리플리케이트되거나 RPC로 전송되며, 사운드나 파티클 같은 외형적 연출은 [`GameplayCues`](#concepts-gc)를 통해 동기화됩니다.

모든 `GameplayAbilities`는 게임플레이 로직이 구현된 `ActivateAbility()` 함수를 오버라이드하게 됩니다. `GameplayAbility`가 완료되거나 취소될 때 실행되는 추가 로직은 `EndAbility()`에 추가할 수 있습니다.

단순한 `GameplayAbility`의 흐름도:
![Simple GameplayAbility Flowchart](https://github.com/tranek/GASDocumentation/raw/master/Images/abilityflowchartsimple.png)


더 복잡한 `GameplayAbility`의 흐름도:
![Complex GameplayAbility Flowchart](https://github.com/tranek/GASDocumentation/raw/master/Images/abilityflowchartcomplex.png)

복잡한 어빌리티는 서로 상호작용(활성화, 취소 등)하는 여러 개의 `GameplayAbilities`를 조합하여 구현할 수 있습니다.

<a name="concepts-ga-definition-reppolicy"></a>
##### 4.6.1.1 리플리케이션 정책 (Replication Policy)
이 옵션은 사용하지 마십시오. 이름이 오해의 소지가 있으며 필요하지 않습니다. [`GameplayAbilitySpecs`](#concepts-ga-spec)는 기본적으로 서버에서 소유 클라이언트로 리플리케이트됩니다. 위에서 언급했듯이, **`GameplayAbilities`는 시뮬레이트 프록시에서 실행되지 않습니다**. 시뮬레이트 프록시에 시각적 변화를 전달하기 위해 `AbilityTasks`와 `GameplayCues`를 사용하여 리플리케이트하거나 RPC를 전송합니다. Epic의 Dave Ratti는 [향후 이 옵션을 제거하고 싶다](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)고 밝힌 바 있습니다.

<a name="concepts-ga-definition-remotecancel"></a>
##### 4.6.1.2 서버의 원격 어빌리티 취소 준수 (Server Respects Remote Ability Cancellation)
이 옵션은 대개 문제를 일으킵니다. 이는 클라이언트의 `GameplayAbility`가 취소되거나 자연 종료되어 끝났을 때, 서버 버전의 어빌리티가 아직 완료되지 않았더라도 강제로 종료시킴을 의미합니다. 특히 핑(지연시간)이 높은 플레이어가 로컬 예측 `GameplayAbilities`를 사용할 때 서버가 조기 종료되는 심각한 문제가 발생할 수 있습니다. 일반적으로 이 옵션은 비활성화하는 것이 좋습니다.

<a name="concepts-ga-definition-repinputdirectly"></a>
##### 4.6.1.3 입력 직접 리플리케이트 (Replicate Input Directly)
이 옵션을 켜면 입력 누름(Press) 및 뗌(Release) 이벤트가 항상 서버로 직접 리플리케이트됩니다. Epic은 이를 사용하는 대신 [입력을 `ASC`에 바인딩](#concepts-ga-input)한 경우 기존 입력 관련 [`AbilityTasks`](#concepts-at)에 내장된 `Generic Replicated Events`를 활용할 것을 권장합니다.

Epic의 소스 코드 주석:
```c++
/** Direct Input state replication. These will be called if bReplicateInputDirectly is true on the ability and is generally not a good thing to use. (Instead, prefer to use Generic Replicated Events). */
UAbilitySystemComponent::ServerSetInputPressed()
```

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ga-input"></a>
#### 4.6.2 입력을 ASC에 바인딩 (Binding Input to the ASC)
`ASC`를 사용하면 입력 액션을 컴포넌트에 직접 바인딩하고, 어빌리티를 부여할 때 해당 입력을 `GameplayAbilities`에 할당할 수 있습니다. `GameplayAbilities`에 할당된 입력 액션은 키를 눌렀을 때 `GameplayTag` 조건이 충족되면 해당 `GameplayAbilities`를 자동으로 활성화합니다. 할당된 입력 액션은 입력에 반응하는 내장 `AbilityTasks`를 사용하는 데 필수적입니다.

`GameplayAbilities` 활성화에 할당되는 일반 입력 외에도, `ASC`는 범용 `Confirm`(확인) 및 `Cancel`(취소) 입력도 수용합니다. 이 특수 입력들은 [`Target Actors`](#concepts-targeting-actors) 확인이나 취소 등을 위해 `AbilityTasks`에서 사용됩니다.

입력을 `ASC`에 바인딩하려면 먼저 입력 액션 이름을 byte로 변환하는 열거형(Enum)을 만들어야 합니다. 열거형의 이름은 프로젝트 설정의 입력 액션 이름과 정확히 일치해야 합니다. `DisplayName`은 달라도 상관없습니다.

샘플 프로젝트 코드:
```c++
UENUM(BlueprintType)
enum class EGDAbilityInputID : uint8
{
	// 0 None
	None			UMETA(DisplayName = "None"),
	// 1 Confirm
	Confirm			UMETA(DisplayName = "Confirm"),
	// 2 Cancel
	Cancel			UMETA(DisplayName = "Cancel"),
	// 3 LMB
	Ability1		UMETA(DisplayName = "Ability1"),
	// 4 RMB
	Ability2		UMETA(DisplayName = "Ability2"),
	// 5 Q
	Ability3		UMETA(DisplayName = "Ability3"),
	// 6 E
	Ability4		UMETA(DisplayName = "Ability4"),
	// 7 R
	Ability5		UMETA(DisplayName = "Ability5"),
	// 8 Sprint
	Sprint			UMETA(DisplayName = "Sprint"),
	// 9 Jump
	Jump			UMETA(DisplayName = "Jump")
};
```

`ASC`가 `Character`에 상주하는 경우 `SetupPlayerInputComponent()`에서 `ASC` 바인딩 함수를 호출합니다:
```c++
// AbilitySystemComponent에 바인딩
FTopLevelAssetPath AbilityEnumAssetPath = FTopLevelAssetPath(FName("/Script/GASDocumentation"), FName("EGDAbilityInputID"));
AbilitySystemComponent->BindAbilityActivationToInputComponent(PlayerInputComponent, FGameplayAbilityInputBinds(FString("ConfirmTarget"),
	FString("CancelTarget"), AbilityEnumAssetPath, static_cast<int32>(EGDAbilityInputID::Confirm), static_cast<int32>(EGDAbilityInputID::Cancel)));
```

`ASC`가 `PlayerState`에 상주하는 경우, `SetupPlayerInputComponent()` 실행 시점에 클라이언트에 아직 `PlayerState`가 리플리케이트되지 않았을 수 있는 잠재적 경쟁 조건(Race Condition)이 존재합니다. 따라서 `SetupPlayerInputComponent()`와 `OnRep_PlayerState()` 두 곳 모두에서 입력 바인딩을 시도하는 것을 권장합니다. `PlayerController`가 클라이언트에 `InputComponent`를 생성하는 `ClientRestart()`를 지시하기 전에 `PlayerState`가 먼저 리플리케이트되면 `Actor`의 `InputComponent`가 null일 수 있으므로 `OnRep_PlayerState()` 단독으로도 충분하지 않습니다. 샘플 프로젝트는 boolean 플래그를 사용하여 두 위치에서 모두 바인딩을 시도하되 실제로는 단 한 번만 바인딩되도록 안전하게 구현했습니다.

**참고:** 샘플 프로젝트에서 열거형의 `Confirm`과 `Cancel`은 프로젝트 설정의 입력 액션 이름(`ConfirmTarget`, `CancelTarget`)과 일치하지 않지만, `BindAbilityActivationToInputComponent()`에서 둘 사이의 매핑을 명시적으로 전달하므로 문제없이 동작합니다. 이 두 특수 입력은 매핑을 직접 제공하므로 이름이 달라도 되지만 같아도 무방합니다. 열거형의 다른 모든 일반 입력은 프로젝트 설정의 입력 액션 이름과 반드시 일치해야 합니다.

하나의 고정된 입력으로만 활성화되는 `GameplayAbilities`(예: MOBA처럼 항상 동일한 "슬롯"에 존재하는 어빌리티)의 경우, `UGameplayAbility` 서브클래스에 변수를 추가하여 해당 입력을 정의하고 어빌리티를 부여할 때 `ClassDefaultObject`로부터 이를 읽어오는 방식을 선호합니다.

<a name="concepts-ga-input-noactivate"></a>
##### 4.6.2.1 어빌리티 활성화 없이 입력에 바인딩
키를 눌렀을 때 `GameplayAbilities`가 자동 활성화되는 것은 원치 않지만, `AbilityTasks`에서 사용할 수 있도록 입력 바인딩은 유지하고 싶다면 `UGameplayAbility` 서브클래스에 기본값이 `true`인 `bActivateOnInput` bool 변수를 추가하고 `UAbilitySystemComponent::AbilityLocalInputPressed()`를 오버라이드하면 됩니다.

```c++
void UGSAbilitySystemComponent::AbilityLocalInputPressed(int32 InputID)
{
	// 이 InputID가 GenericConfirm/Cancel로 오버로드되어 있고 콜백이 바인딩되어 있다면 입력을 소비합니다.
	if (IsGenericConfirmInputBound(InputID))
	{
		LocalInputConfirm();
		return;
	}

	if (IsGenericCancelInputBound(InputID))
	{
		LocalInputCancel();
		return;
	}

	// ---------------------------------------------------------

	ABILITYLIST_SCOPE_LOCK();
	for (FGameplayAbilitySpec& Spec : ActivatableAbilities.Items)
	{
		if (Spec.InputID == InputID)
		{
			if (Spec.Ability)
			{
				Spec.InputPressed = true;
				if (Spec.IsActive())
				{
					if (Spec.Ability->bReplicateInputDirectly && IsOwnerActorAuthoritative() == false)
					{
						ServerSetInputPressed(Spec.Handle);
					}

					AbilitySpecInputPressed(Spec);

					// InputPressed 이벤트를 발동합니다. 여기서 리플리케이트되지는 않으며 수신 대기 중인 대상이 있다면 서버로 리플리케이트할 수 있습니다.
					InvokeReplicatedEvent(EAbilityGenericReplicatedEvent::InputPressed, Spec.Handle, Spec.ActivationInfo.GetActivationPredictionKey());
				}
				else
				{
					UGSGameplayAbility* GA = Cast<UGSGameplayAbility>(Spec.Ability);
					if (GA && GA->bActivateOnInput)
					{
						// 어빌리티가 활성 상태가 아니므로 활성화를 시도합니다.
						TryActivateAbility(Spec.Handle);
					}
				}
			}
		}
	}
}
```

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ga-granting"></a>
#### 4.6.3 어빌리티 부여 (Granting Abilities)
`ASC`에 `GameplayAbility`를 부여(Grant)하면 `ASC`의 `ActivatableAbilities` 목록에 추가되어, [`GameplayTag` 조건](#concepts-ga-tags)이 충족될 때 언제든지 해당 `GameplayAbility`를 활성화할 수 있게 됩니다.

우리는 서버에서 `GameplayAbilities`를 부여하며, 서버는 자동으로 소유 클라이언트에게 [`GameplayAbilitySpec`](#concepts-ga-spec)을 리플리케이트합니다. 다른 클라이언트나 시뮬레이트 프록시는 `GameplayAbilitySpec`을 수신하지 않습니다.

샘플 프로젝트는 `Character` 클래스에 `TArray<TSubclassOf<UGDGameplayAbility>>`를 두고 게임 시작 시 이를 읽어 어빌리티를 부여합니다:
```c++
void AGDCharacterBase::AddCharacterAbilities()
{
	// 서버에서만 어빌리티를 부여합니다.
	if (Role != ROLE_Authority || !AbilitySystemComponent.IsValid() || AbilitySystemComponent->bCharacterAbilitiesGiven)
	{
		return;
	}

	for (TSubclassOf<UGDGameplayAbility>& StartupAbility : CharacterAbilities)
	{
		AbilitySystemComponent->GiveAbility(
			FGameplayAbilitySpec(StartupAbility, GetAbilityLevel(StartupAbility.GetDefaultObject()->AbilityID), static_cast<int32>(StartupAbility.GetDefaultObject()->AbilityInputID), this));
	}

	AbilitySystemComponent->bCharacterAbilitiesGiven = true;
}
```

이 `GameplayAbilities`를 부여할 때 `UGameplayAbility` 클래스, 어빌리티 레벨, 바인딩된 입력, 그리고 `SourceObject`(`ASC`에 이 어빌리티를 부여한 주체)를 포함하여 `GameplayAbilitySpecs`를 생성합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ga-activating"></a>
#### 4.6.4 어빌리티 활성화 (Activating Abilities)
`GameplayAbility`에 입력 액션이 할당되어 있다면, 해당 입력을 눌렀을 때 태그 조건이 충족되면 자동으로 활성화됩니다. 하지만 이것이 항상 최선의 활성화 방식은 아닙니다. `ASC`는 `GameplayTag`, `GameplayAbility` 클래스, `GameplayAbilitySpec` 핸들, 그리고 이벤트 기반 활성화 등 4가지 추가 활성화 방식을 제공합니다. 이벤트로 어빌리티를 활성화하면 [이벤트와 함께 데이터 페이로드(Payload)를 전달](#concepts-ga-data)할 수 있습니다.

```c++
UFUNCTION(BlueprintCallable, Category = "Abilities")
bool TryActivateAbilitiesByTag(const FGameplayTagContainer& GameplayTagContainer, bool bAllowRemoteActivation = true);

UFUNCTION(BlueprintCallable, Category = "Abilities")
bool TryActivateAbilityByClass(TSubclassOf<UGameplayAbility> InAbilityToActivate, bool bAllowRemoteActivation = true);

bool TryActivateAbility(FGameplayAbilitySpecHandle AbilityToActivate, bool bAllowRemoteActivation = true);

bool TriggerAbilityFromGameplayEvent(FGameplayAbilitySpecHandle AbilityToTrigger, FGameplayAbilityActorInfo* ActorInfo, FGameplayTag Tag, const FGameplayEventData* Payload, UAbilitySystemComponent& Component);

FGameplayAbilitySpecHandle GiveAbilityAndActivateOnce(const FGameplayAbilitySpec& AbilitySpec, const FGameplayEventData* GameplayEventData);
```

이벤트로 `GameplayAbility`를 활성화하려면 `GameplayAbility` 내에 `Triggers`(트리거)가 설정되어 있어야 합니다. `GameplayTag`를 할당하고 `GameplayEvent` 옵션을 선택합니다. 이벤트를 전송하려면 `UAbilitySystemBlueprintLibrary::SendGameplayEventToActor(AActor* Actor, FGameplayTag EventTag, FGameplayEventData Payload)` 함수를 사용합니다.

`GameplayAbility` `Triggers`는 `GameplayTag`가 추가되거나 제거될 때 어빌리티가 활성화되도록 설정할 수도 있습니다.

**참고:** 블루프린트에서 이벤트로부터 `GameplayAbility`를 활성화할 때는 반드시 `ActivateAbilityFromEvent` 노드를 사용해야 합니다.

**참고:** 패시브 어빌리티처럼 상시 실행되어야 하는 어빌리티가 아니라면, `GameplayAbility`가 끝나는 시점에 반드시 `EndAbility()`를 호출하는 것을 잊지 마십시오.

**로컬 예측(locally predicted)** `GameplayAbilities`의 활성화 시퀀스:
1. **소유 클라이언트**가 `TryActivateAbility()`를 호출합니다.
1. `InternalTryActivateAbility()`를 호출합니다.
1. `CanActivateAbility()`를 호출하여 `GameplayTag` 조건 충족 여부, 비용 감당 가능 여부, 쿨다운 상태가 아닌지, 현재 활성화된 다른 인스턴스가 없는지 확인하고 반환합니다.
1. `CallServerTryActivateAbility()`를 호출하고 자체 생성한 `Prediction Key`(예측 키)를 전달합니다.
1. `CallActivateAbility()`를 호출합니다.
1. Epic에서 "상투적인 초기화 작업(boilerplate init stuff)"이라고 부르는 `PreActivate()`를 호출합니다.
1. `ActivateAbility()`를 호출하여 마침내 어빌리티를 로컬에서 활성화합니다.

**서버**가 `CallServerTryActivateAbility()`를 수신:
1. `ServerTryActivateAbility()`를 호출합니다.
1. `InternalServerTryActivateAbility()`를 호출합니다.
1. `InternalTryActivateAbility()`를 호출합니다.
1. `CanActivateAbility()`를 호출하여 태그 조건, 비용, 쿨다운, 중복 인스턴스 여부를 검사합니다.
1. 성공 시 `ClientActivateAbilitySucceed()`를 호출하여 서버에 의해 활성화가 확인되었음을 알리고 `ActivationInfo`를 업데이트하며 `OnConfirmDelegate` 델리게이트를 브로드캐스트합니다 (입력 확인 Confirm과는 다름).
1. `CallActivateAbility()`를 호출합니다.
1. `PreActivate()`를 호출합니다.
1. `ActivateAbility()`를 호출하여 서버에서 어빌리티를 활성화합니다.

서버에서 검증 중 활성화가 실패하면 서버는 즉시 `ClientActivateAbilityFailed()`를 호출하여 클라이언트의 `GameplayAbility`를 강제 종료하고 예측되었던 모든 변경 사항을 롤백(되돌림)합니다.

<a name="concepts-ga-activating-passive"></a>
##### 4.6.4.1 패시브 어빌리티 (Passive Abilities)
자동으로 활성화되어 지속적으로 동작하는 패시브 `GameplayAbilities`를 구현하려면, 어빌리티가 부여되고 `AvatarActor`가 설정될 때 자동 호출되는 `UGameplayAbility::OnAvatarSet()`을 오버라이드하고 그 안에서 `TryActivateAbility()`를 호출합니다.

커스텀 `UGameplayAbility` 클래스에 어빌리티가 부여될 때 자동 활성화되어야 하는지를 지정하는 `bool` 플래그를 추가하는 것을 추천합니다. 샘플 프로젝트는 패시브 방어력 스택 어빌리티에 이 방식을 사용합니다.

패시브 `GameplayAbilities`는 일반적으로 `Server Only`의 [`Net Execution Policy`](#concepts-ga-net)를 가집니다.

```c++
void UGDGameplayAbility::OnAvatarSet(const FGameplayAbilityActorInfo * ActorInfo, const FGameplayAbilitySpec & Spec)
{
	Super::OnAvatarSet(ActorInfo, Spec);

	if (bActivateAbilityOnGranted)
	{
		ActorInfo->AbilitySystemComponent->TryActivateAbility(Spec.Handle, false);
	}
}
```

Epic은 이 함수를 패시브 어빌리티를 시작하고 `BeginPlay` 성격의 작업을 수행하기에 가장 올바른 장소로 설명하고 있습니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ga-activating-failedtags"></a>
##### 4.6.4.2 활성화 실패 태그 (Activation Failed Tags)
어빌리티에는 활성화가 실패했을 때 그 이유를 알려주는 기본 로직이 내장되어 있습니다. 이를 활성화하려면 기본 실패 케이스에 대응하는 GameplayTags를 설정해야 합니다.

프로젝트에 다음 태그들(또는 팀의 명명 규칙에 맞춘 태그)을 추가합니다:
```
+GameplayTagList=(Tag="Activation.Fail.BlockedByTags",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.CantAffordCost",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.IsDead",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.MissingTags",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.Networking",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.OnCooldown",DevComment="")
```

그런 다음 [`GASDocumentation\Config\DefaultGame.ini`](https://github.com/tranek/GASDocumentation/blob/master/Config/DefaultGame.ini#L8-L13)에 추가합니다:
```
[/Script/GameplayAbilities.AbilitySystemGlobals]
ActivateFailIsDeadName=Activation.Fail.IsDead
ActivateFailCooldownName=Activation.Fail.OnCooldown
ActivateFailCostName=Activation.Fail.CantAffordCost
ActivateFailTagsBlockedName=Activation.Fail.BlockedByTags
ActivateFailTagsMissingName=Activation.Fail.MissingTags
ActivateFailNetworkingName=Activation.Fail.Networking
```

이제 어빌리티 활성화가 실패할 때마다 해당 대응 GameplayTag가 로그 출력 메시지에 포함되거나 `showdebug AbilitySystem` HUD 화면에 표시됩니다.
```
LogAbilitySystem: Display: InternalServerTryActivateAbility. Rejecting ClientActivation of Default__GA_FireGun_C. InternalTryActivateAbility failed: Activation.Fail.BlockedByTags
LogAbilitySystem: Display: ClientActivateAbilityFailed_Implementation. PredictionKey :109 Ability: Default__GA_FireGun_C
```

![showdebug AbilitySystem에 표시된 활성화 실패 태그](https://github.com/tranek/GASDocumentation/raw/master/Images/activationfailedtags.png)

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ga-cancelabilities"></a>
#### 4.6.5 어빌리티 취소 (Canceling Abilities)
`GameplayAbility` 내부에서 어빌리티를 취소하려면 `CancelAbility()`를 호출합니다. 이는 `EndAbility()`를 호출하며 `WasCancelled` 매개변수를 true로 설정합니다.

외부에서 `GameplayAbility`를 취소하기 위해 `ASC`는 몇 가지 함수를 제공합니다:

```c++
/** 지정된 어빌리티 CDO를 취소합니다. */
void CancelAbility(UGameplayAbility* Ability);	

/** 전달된 스펙 핸들이 가리키는 어빌리티를 취소합니다. */
void CancelAbilityHandle(const FGameplayAbilitySpecHandle& AbilityHandle);

/** 지정된 태그를 가진 모든 어빌리티를 취소합니다. Ignore 인스턴스는 취소하지 않습니다. */
void CancelAbilities(const FGameplayTagContainer* WithTags=nullptr, const FGameplayTagContainer* WithoutTags=nullptr, UGameplayAbility* Ignore=nullptr);

/** 태그에 상관없이 모든 어빌리티를 취소합니다. */
void CancelAllAbilities(UGameplayAbility* Ignore=nullptr);

/** 모든 어빌리티를 취소하고 남아 있는 인스턴스화된 어빌리티를 파괴합니다. */
virtual void DestroyActiveState();
```

**참고:** `Non-Instanced` `GameplayAbilities`가 존재하는 경우 `CancelAllAbilities`가 정상 작동하지 않는 현상을 발견했습니다. 인스턴스화되지 않은 어빌리티를 만나면 처리를 중단해 버리는 경향이 있습니다. 반면 `CancelAbilities`는 `Non-Instanced` 어빌리티를 더 안정적으로 처리하며, 샘플 프로젝트에서도 이 방식을 사용합니다(점프는 Non-Instanced 어빌리티입니다).

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ga-definition-activeability"></a>
#### 4.6.6 활성 어빌리티 가져오기 (Getting Active Abilities)
초심자들은 종종 변수를 설정하거나 취소하기 위해 "현재 활성화된 어빌리티를 어떻게 가져오나요?"라고 묻곤 합니다. 한 번에 둘 이상의 `GameplayAbility`가 활성화될 수 있으므로 단 하나의 "활성 어빌리티"라는 것은 존재하지 않습니다. 대신 `ASC`가 소유한 부여된 어빌리티 목록인 `ActivatableAbilities`를 순회하면서 찾고자 하는 [`Asset` 또는 `Granted` `GameplayTag`](#concepts-ga-tags)와 일치하는 어빌리티를 찾아야 합니다.

`UAbilitySystemComponent::GetActivatableAbilities()`는 순회할 수 있는 `TArray<FGameplayAbilitySpec>`을 반환합니다.

수동 순회 대신 `GameplayTagContainer`를 매개변수로 받아 검색을 도와주는 편의 함수도 제공됩니다. `bOnlyAbilitiesThatSatisfyTagRequirements` 매개변수는 태그 조건을 만족하여 지금 당장 활성화될 수 있는 `GameplayAbilitySpecs`만 반환하도록 필터링합니다. 예를 들어 무기 공격과 맨손 공격이라는 두 가지 기본 공격 어빌리티가 있을 때, 무기 장착 여부에 따른 태그 조건에 의해 올바른 어빌리티만 선택할 수 있습니다. 자세한 내용은 해당 함수의 주석을 참조하세요.
```c++
UAbilitySystemComponent::GetActivatableGameplayAbilitySpecsByAllMatchingTags(const FGameplayTagContainer& GameplayTagContainer, TArray < struct FGameplayAbilitySpec* >& MatchingGameplayAbilities, bool bOnlyAbilitiesThatSatisfyTagRequirements = true)
```

원하는 `FGameplayAbilitySpec`을 찾았다면 해당 스펙에 대해 `IsActive()`를 호출하여 활성 상태인지 확인할 수 있습니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ga-instancing"></a>
#### 4.6.7 인스턴싱 정책 (Instancing Policy)
`GameplayAbility`의 `Instancing Policy`는 어빌리티가 활성화될 때 인스턴스화되는 방식과 여부를 결정합니다.

| `Instancing Policy`     | 설명                                                                                             | 사용 예시                                                                                                                                                                                                                                           |
| ----------------------- | ------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Instanced Per Actor     | 각 `ASC`마다 어빌리티 인스턴스를 하나만 생성하고 활성화할 때마다 이를 재사용합니다.              | 가장 보편적으로 사용하게 될 정책입니다. 모든 어빌리티에 사용 가능하며 활성화 간에 상태를 유지할 수 있습니다. 활성화 간에 리셋되어야 하는 변수는 디자이너/프로그래머가 수동으로 초기화해야 합니다.                                             |
| Instanced Per Execution | 어빌리티가 활성화될 때마다 매번 새로운 `GameplayAbility` 인스턴스를 생성합니다.                   | 활성화할 때마다 변수가 완전히 초기화된다는 장점이 있습니다. 그러나 매 활성화마다 새 객체를 스폰하므로 `Instanced Per Actor`에 비해 성능이 떨어집니다. 샘플 프로젝트는 이를 사용하지 않습니다.                                                  |
| Non-Instanced           | 어빌리티가 `ClassDefaultObject`(CDO) 상에서 실행됩니다. 인스턴스가 전혀 생성되지 않습니다.        | 세 가지 정책 중 성능이 가장 뛰어나지만 기능적 제약이 매우 큽니다. 동적 변수를 가질 수 없고 `AbilityTask` 델리게이트에 바인딩할 수 없어 상태를 저장할 수 없습니다. MOBA나 미니언의 단순 기본 공격처럼 빈번히 실행되는 단순 어빌리티에 적합합니다. 샘플 프로젝트의 점프(Jump)가 `Non-Instanced`입니다. |

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ga-net"></a>
#### 4.6.8 넷 실행 정책 (Net Execution Policy)
`GameplayAbility`의 `Net Execution Policy`는 네트워크 상에서 누가 어떤 순서로 어빌리티를 실행할지 결정합니다.

| `Net Execution Policy` | 설명                                                                                                                                                                        |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Local Only`           | 어빌리티가 소유 클라이언트에서만 실행됩니다. 로컬 전용 외형 변경 등에 유용합니다. 싱글 플레이어 게임의 경우 `Server Only`를 사용해야 합니다.                              |
| `Local Predicted`      | `Local Predicted` 어빌리티는 소유 클라이언트에서 먼저 활성화된 후 서버에서 활성화됩니다. 서버 버전은 클라이언트가 잘못 예측한 모든 요소를 보정합니다. [예측(Prediction)](#concepts-p) 참조. |
| `Server Only`          | 어빌리티가 오직 서버에서만 실행됩니다. 패시브 `GameplayAbilities`는 일반적으로 `Server Only`입니다. 싱글 플레이어 게임에서도 이를 사용합니다.                              |
| `Server Initiated`     | `Server Initiated` 어빌리티는 서버에서 먼저 활성화된 후 소유 클라이언트에서 활성화됩니다. 실무에서 사용 빈도가 높지는 않습니다.                                             | 

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ga-tags"></a>
#### 4.6.9 어빌리티 태그 (Ability Tags)
`GameplayAbilities`는 내장된 로직과 연동되는 여러 `GameplayTagContainers`를 가집니다. 이 태그들은 리플리케이트되지 않습니다.

| `GameplayTag Container`     | 설명                                                                                                                                                        |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Ability Tags`              | `GameplayAbility` 자체가 소유하는 태그입니다. 해당 어빌리티를 식별하고 설명하는 용도로 사용됩니다.                                                          |
| `Cancel Abilities with Tag` | 이 `GameplayAbility`가 활성화될 때, 자신의 `Ability Tags`에 이 태그들을 가지고 있는 다른 `GameplayAbilities`를 취소시킵니다.                               |
| `Block Abilities with Tag`  | 이 `GameplayAbility`가 활성화되어 있는 동안, 자신의 `Ability Tags`에 이 태그들을 가지고 있는 다른 `GameplayAbilities`의 활성화를 차단(Block)합니다.         |
| `Activation Owned Tags`     | 이 `GameplayAbility`가 활성화되어 있는 동안 어빌리티 소유자(`ASC`)에게 부여되는 태그입니다 (리플리케이트되지 않음에 주의).                                   |
| `Activation Required Tags`  | 어빌리티 소유자가 이 태그들을 **모두** 가지고 있어야만 이 `GameplayAbility`를 활성화할 수 있습니다.                                                        |
| `Activation Blocked Tags`   | 어빌리티 소유자가 이 태그들 중 **하나라도** 가지고 있다면 이 `GameplayAbility`를 활성화할 수 없습니다.                                                     |
| `Source Required Tags`      | 소스(Source)가 이 태그들을 **모두** 가지고 있어야만 이 어빌리티를 활성화할 수 있습니다. 이벤트로 트리거된 어빌리티에서만 유효합니다.                       |
| `Source Blocked Tags`       | 소스(Source)가 이 태그들 중 **하나라도** 가지고 있다면 이 어빌리티를 활성화할 수 없습니다. 이벤트로 트리거된 어빌리티에서만 유효합니다.                   |
| `Target Required Tags`      | 대상(Target)이 이 태그들을 **모두** 가지고 있어야만 이 어빌리티를 활성화할 수 있습니다. 이벤트로 트리거된 어빌리티에서만 유효합니다.                       |
| `Target Blocked Tags`       | 대상(Target)이 이 태그들 중 **하나라도** 가지고 있다면 이 어빌리티를 활성화할 수 없습니다. 이벤트로 트리거된 어빌리티에서만 유효합니다.                   |

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ga-spec"></a>
#### 4.6.10 게임플레이 어빌리티 스펙 (Gameplay Ability Spec)
`GameplayAbilitySpec`은 `GameplayAbility`가 부여된 후 `ASC`에 상주하는 구조체로, 활성화 가능한 어빌리티 클래스, 레벨, 입력 바인딩, 그리고 어빌리티 클래스와 분리되어 관리되어야 하는 런타임 상태를 정의합니다.

서버에서 어빌리티가 부여되면 서버는 소유 클라이언트가 활성화할 수 있도록 `GameplayAbilitySpec`을 클라이언트로 리플리케이트합니다.

`GameplayAbilitySpec`을 활성화하면 해당 어빌리티의 `Instancing Policy`에 따라 인스턴스가 생성되거나(`Non-Instanced`의 경우 인스턴스 없이) 실행됩니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ga-data"></a>
#### 4.6.11 어빌리티로 데이터 전달
`GameplayAbilities`의 일반적인 패러다임은 `활성화(Activate) -> 데이터 생성(Generate Data) -> 적용(Apply) -> 종료(End)`입니다. 때로는 이미 존재하는 기존 데이터를 기반으로 동작해야 할 때가 있습니다. GAS는 `GameplayAbilities`로 외부 데이터를 전달하기 위한 몇 가지 옵션을 제공합니다:

| 전달 방식                                        | 설명                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 이벤트로 `GameplayAbility` 활성화                | 데이터 페이로드를 포함하는 이벤트로 `GameplayAbility`를 활성화합니다. 로컬 예측 어빌리티의 경우 이벤트 페이로드가 클라이언트에서 서버로 리플리케이트됩니다. 기존 변수에 맞지 않는 임의의 데이터는 두 개의 `Optional Object` 변수나 [`TargetData`](#concepts-targeting-data)를 활용할 수 있습니다. 단점은 입력 바인딩으로 직접 활성화할 수 없다는 점입니다. `UAbilitySystemBlueprintLibrary::SendGameplayEventToActor(AActor* Actor, FGameplayTag EventTag, FGameplayEventData Payload)` 함수를 사용합니다. |
| `WaitGameplayEvent` `AbilityTask` 사용           | 어빌리티 활성화 후 `WaitGameplayEvent` 태스크를 사용하여 데이터 페이로드를 가진 이벤트를 수신 대기합니다. 페이로드 구성 및 전송 과정은 이벤트 활성화와 동일합니다. 단점은 태스크 자체에서 이벤트가 리플리케이트되지 않으므로 `Local Only` 및 `Server Only` 어빌리티에서만 사용해야 한다는 점입니다. 필요하다면 페이로드를 리플리케이트하는 자체 커스텀 태스크를 작성할 수 있습니다.                                                                                                                |
| `TargetData` 사용                                | 커스텀 `TargetData` 구조체는 클라이언트와 서버 간에 임의의 데이터를 전달하는 매우 훌륭한 방법입니다.                                                                                                                                                                                                                                                                                                                                                                                                  |
| `OwnerActor` 또는 `AvatarActor`에 데이터 저장    | `OwnerActor`, `AvatarActor` 또는 참조할 수 있는 다른 객체에 리플리케이트되는 변수를 저장합니다. 가장 유연하며 입력 바인딩으로 활성화되는 어빌리티와 잘 맞습니다. 그러나 사용 시점에 데이터가 네트워크 패킷 손실 등으로 인해 제때 동기화되었음을 100% 보장하지 못하므로, 사전에 동기화 순서를 고려해야 합니다.                                                                                                                                                                                |

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ga-commit"></a>
#### 4.6.12 어빌리티 비용 및 쿨다운 (Cost and Cooldown)
`GameplayAbilities`에는 선택적 비용 및 쿨다운 기능이 기본 포함되어 있습니다. 비용은 어빌리티를 활성화하기 위해 `ASC`가 보유해야 하는 사전 정의된 `Attributes` 수치이며 즉시 `GameplayEffect`([`Cost GE`](#concepts-ge-cost))로 구현됩니다. 쿨다운은 만료될 때까지 어빌리티의 재활성화를 방지하는 타이머이며 지속 시간 `GameplayEffect`([`Cooldown GE`](#concepts-ge-cooldown))로 구현됩니다.

어빌리티가 `UGameplayAbility::Activate()`를 호출하기 전에 먼저 `UGameplayAbility::CanActivateAbility()`를 호출합니다. 이 함수는 소유 `ASC`가 비용을 감당할 수 있는지(`UGameplayAbility::CheckCost()`), 쿨다운 중이 아닌지(`UGameplayAbility::CheckCooldown()`)를 검사합니다.

어빌리티가 `Activate()`된 후, 언제든지 `UGameplayAbility::CommitAbility()`를 호출하여 비용과 쿨다운을 커밋(소모 및 적용)할 수 있습니다. 이는 내부적으로 `UGameplayAbility::CommitCost()`와 `UGameplayAbility::CommitCooldown()`을 호출합니다. 비용과 쿨다운을 서로 다른 시점에 적용하고 싶다면 각각 개별적으로 호출할 수도 있습니다. 비용과 쿨다운을 커밋할 때 `CheckCost()`와 `CheckCooldown()`이 다시 한 번 호출되며, 여기서 실패하면 어빌리티가 중단될 수 있습니다. 어빌리티 활성화 이후 커밋 시점 사이에 소유자의 어트리뷰트가 변경되어 비용이 부족해질 수 있기 때문입니다. 커밋 시점에 [예측 키(Prediction Key)](#concepts-p-key)가 유효하다면 비용과 쿨다운의 커밋은 [로컬 예측](#concepts-p)될 수 있습니다.

구현 세부사항은 [`CostGE`](#concepts-ge-cost) 및 [`CooldownGE`](#concepts-ge-cooldown)를 참조하세요.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ga-leveling"></a>
#### 4.6.13 어빌리티 레벨업 (Leveling Up Abilities)
어빌리티의 레벨을 올리는 데는 두 가지 일반적인 방법이 있습니다:

| 레벨업 방법                                      | 설명                                                                                                                                                                                                |
| ------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 어빌리티 제거(Ungrant) 후 새 레벨로 재부여(Regrant) | 서버에서 `ASC`로부터 `GameplayAbility`를 제거하고 다음 레벨로 다시 부여합니다. 당시 어빌리티가 활성화되어 있었다면 강제 종료됩니다.                                                                 |
| `GameplayAbilitySpec`의 레벨 증가                | 서버에서 `GameplayAbilitySpec`을 찾아 레벨을 올리고 더티(Dirty)로 표시하여 소유 클라이언트로 리플리케이트합니다. 이 방식은 어빌리티가 활성화되어 있더라도 실행을 종료시키지 않습니다.            |

두 방식의 주요 차이점은 레벨업 시점에 활성화되어 있는 어빌리티를 강제 취소할 것인지 여부입니다. 프로젝트에 따라 두 방식 모두 사용하게 될 것입니다. `UGameplayAbility` 서브클래스에 어떤 방식을 사용할지 지정하는 `bool` 플래그를 추가하는 것을 권장합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ga-sets"></a>
#### 4.6.14 어빌리티 세트 (Ability Sets)
`GameplayAbilitySets`는 캐릭터의 시작 `GameplayAbilities` 목록과 입력 바인딩, 그리고 이를 부여하는 로직을 담고 있는 편의성 `UDataAsset` 클래스입니다. 서브클래스에서 추가 속성이나 로직을 확장할 수도 있습니다. 파라곤(Paragon)에서는 각 영웅마다 부여할 어빌리티들을 묶은 `GameplayAbilitySet`을 보유했습니다.

개인적으로는 이 클래스가 필수적이지 않다고 생각합니다. 본 샘플 프로젝트는 `GDCharacterBase` 및 그 서브클래스 내부에서 `GameplayAbilitySets`의 모든 기능을 직접 처리합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ga-batching"></a>
#### 4.6.15 어빌리티 배칭 (Ability Batching)
기존의 `Gameplay Ability` 수명 주기는 클라이언트에서 서버로 최소 2~3개의 RPC를 전송합니다.

1. `CallServerTryActivateAbility()`
1. `ServerSetReplicatedTargetData()` (선택 사항)
1. `ServerEndAbility()`

만약 `GameplayAbility`가 단일 프레임의 원자적(Atomic) 그룹 내에서 이 모든 동작을 수행한다면, 이 2~3개의 RPC를 단 1개의 RPC로 묶어(Batch) 전송하도록 워크플로우를 최적화할 수 있습니다. GAS에서는 이 RPC 최적화를 `어빌리티 배칭(Ability Batching)`이라고 부릅니다. 어빌리티 배칭의 대표적인 사용 사례는 히트스캔(Hitscan) 총기입니다. 히트스캔 총기는 한 프레임의 원자적 그룹 내에서 활성화되고, 라인 트레이스를 수행하고, [`TargetData`](#concepts-targeting-data)를 서버로 전송하고, 어빌리티를 종료합니다. [GASShooter](https://github.com/tranek/GASShooter) 샘플 프로젝트가 히트스캔 총기에 이 테크닉을 적용한 예시를 보여줍니다.

단발(Semi-Automatic) 총기는 최적의 시나리오로 `CallServerTryActivateAbility()`, `ServerSetReplicatedTargetData()`(탄환 적중 결과), `ServerEndAbility()` 3개의 RPC를 단 1개의 RPC로 묶습니다.

연사(Full-Automatic)/점사(Burst) 총기는 첫 번째 탄환에 대해 `CallServerTryActivateAbility()`와 `ServerSetReplicatedTargetData()`를 1개의 RPC로 묶습니다. 이후 발사되는 각 탄환은 자체 `ServerSetReplicatedTargetData()` RPC로 전송됩니다. 마지막으로 사격이 멈추면 `ServerEndAbility()`가 별도의 RPC로 전송됩니다. 이는 최악의 시나리오이지만 첫 번째 탄환에서 여전히 1개의 RPC를 절약할 수 있습니다.

`ASC`에서 `Ability Batching`은 기본적으로 비활성화되어 있습니다. 어빌리티 배칭을 활성화하려면 `ShouldDoServerAbilityRPCBatch()`를 오버라이드하여 true를 반환하도록 합니다:

```c++
virtual bool ShouldDoServerAbilityRPCBatch() const override { return true; }
```

어빌리티 배칭이 활성화되면 배칭하려는 어빌리티를 활성화하기 전에 `FScopedServerAbilityRPCBatcher` 구조체를 생성해야 합니다. 이 특수 구조체는 스코프 내에서 뒤따라 실행되는 모든 어빌리티의 RPC 전송을 가로채서 배치 구조체에 패킹합니다. `FScopedServerAbilityRPCBatcher`가 스코프를 벗어날 때, `UAbilitySystemComponent::EndServerAbilityRPCBatch()`에서 이 배치 구조체를 서버로 자동 전송합니다. 서버는 `UAbilitySystemComponent::ServerAbilityRPCBatch_Internal(FServerAbilityRPCBatch& BatchInfo)`에서 배치 RPC를 수신합니다. `BatchInfo` 매개변수에는 어빌리티 종료 여부, 활성화 시점의 입력 누름 여부, 포함된 `TargetData` 등의 플래그가 들어 있습니다. 배칭이 정상 동작하는지 확인하기 위해 이 함수에 브레이크포인트를 걸어보면 좋습니다. 또는 `AbilitySystem.ServerRPCBatching.Log 1` 콘솔 변수를 사용하여 전용 배칭 로그를 활성화할 수도 있습니다.

이 메커니즘은 C++로만 구현 가능하며 `FGameplayAbilitySpecHandle`을 통해서만 어빌리티를 활성화할 수 있습니다.

```c++
bool UGSAbilitySystemComponent::BatchRPCTryActivateAbility(FGameplayAbilitySpecHandle InAbilityHandle, bool EndAbilityImmediately)
{
	bool AbilityActivated = false;
	if (InAbilityHandle.IsValid())
	{
		FScopedServerAbilityRPCBatcher GSAbilityRPCBatcher(this, InAbilityHandle);
		AbilityActivated = TryActivateAbility(InAbilityHandle, true);

		if (EndAbilityImmediately)
		{
			FGameplayAbilitySpec* AbilitySpec = FindAbilitySpecFromHandle(InAbilityHandle);
			if (AbilitySpec)
			{
				UGSGameplayAbility* GSAbility = Cast<UGSGameplayAbility>(AbilitySpec->GetPrimaryInstance());
				GSAbility->ExternalEndAbility();
			}
		}

		return AbilityActivated;
	}

	return AbilityActivated;
}
```

GASShooter는 단발 및 연사 총기 모두에 대해 `EndAbility()`를 직접 호출하지 않는 동일한 배칭 `GameplayAbility`를 재사용합니다 (플레이어 입력 및 발사 모드에 따른 배칭 어빌리티 호출을 관리하는 로컬 전용 어빌리티가 어빌리티 외부에서 이를 처리함). 모든 RPC가 `FScopedServerAbilityRPCBatcher` 스코프 내에서 발생해야 하므로, 관리하는 로컬 어빌리티가 `EndAbility()` 호출까지 함께 묶을지(단발), 묶지 않고 나중에 개별 RPC로 종료할지(연사) 지정할 수 있도록 `EndAbilityImmediately` 매개변수를 제공합니다.

GASShooter는 블루프린트 노드를 노출하여 로컬 전용 어빌리티가 배칭 어빌리티를 트리거할 수 있도록 지원합니다.

![배치 어빌리티 활성화](https://github.com/tranek/GASDocumentation/raw/master/Images/batchabilityactivate.png)

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-ga-netsecuritypolicy"></a>
#### 4.6.16 넷 보안 정책 (Net Security Policy)
`GameplayAbility`의 `NetSecurityPolicy`는 네트워크 상에서 어빌리티가 어디서 실행되어야 하는지를 결정합니다. 권한 없는 클라이언트가 제한된 어빌리티를 무단 실행하려는 시도로부터 시스템을 보호합니다.

| `NetSecurityPolicy`     | 설명                                                                                                                                          |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `ClientOrServer`        | 보안 요구사항이 없습니다. 클라이언트나 서버 모두 이 어빌리티의 실행 및 종료를 자유롭게 트리거할 수 있습니다.                                   |
| `ServerOnlyExecution`   | 클라이언트가 이 어빌리티의 실행을 요청하면 서버에 의해 무시됩니다. 단, 클라이언트가 서버에 어빌리티의 취소나 종료를 요청하는 것은 허용됩니다. |
| `ServerOnlyTermination` | 클라이언트가 이 어빌리티의 취소나 종료를 요청하면 서버에 의해 무시됩니다. 단, 어빌리티의 실행 요청은 허용됩니다.                             |
| `ServerOnly`            | 서버가 이 어빌리티의 실행과 종료를 모두 독점 제어합니다. 클라이언트의 모든 실행/종료 요청은 무시됩니다.                                       |

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-at"></a>

### 4.7 어빌리티 태스크 (Ability Tasks)

<a name="concepts-at-definition"></a>
### 4.7.1 어빌리티 태스크 정의
`GameplayAbilities`는 기본적으로 단 1프레임 내에서만 실행됩니다. 이것만으로는 유연한 구현에 한계가 있습니다. 시간의 경과에 따라 진행되는 액션을 수행하거나 나중에 발동되는 델리게이트에 응답하기 위해 우리는 `AbilityTasks`라는 지연 잠복 액션(Latent Action)을 사용합니다.

GAS에는 즉시 사용할 수 있는 다양한 `AbilityTasks`가 기본 제공됩니다:
* `RootMotionSource`를 사용하여 캐릭터를 이동시키는 태스크
* 애니메이션 몽타주를 재생하는 태스크
* `Attribute` 변경에 반응하는 태스크
* `GameplayEffect` 변경에 반응하는 태스크
* 플레이어 입력에 반응하는 태스크
* 그 외 다수

`UAbilityTask` 생성자에는 전체 게임에서 동시에 실행될 수 있는 `AbilityTasks`의 최대 개수가 1000개로 하드코딩되어 있습니다. RTS 게임처럼 월드에 수백 명의 캐릭터가 동시에 존재할 수 있는 게임의 `GameplayAbilities`를 설계할 때 이 점을 유의해야 합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-at-definition"></a>
### 4.7.2 커스텀 어빌리티 태스크 (Custom Ability Tasks)
실무에서는 자체적인 커스텀 `AbilityTasks`를 (C++로) 자주 만들게 됩니다. 샘플 프로젝트에는 두 가지 커스텀 `AbilityTasks`가 포함되어 있습니다:
1. `PlayMontageAndWaitForEvent`: 기본 제공되는 `PlayMontageAndWait`와 `WaitGameplayEvent` 태스크를 결합한 것입니다. 애니메이션 몽타주의 `AnimNotify`에서 발생한 게임플레이 이벤트를 몽타주를 시작했던 `GameplayAbility`로 전달받을 수 있게 해줍니다. 애니메이션 재생 도중 특정 타이밍에 액션을 트리거할 때 사용합니다.
1. `WaitReceiveDamage`: `OwnerActor`가 대미지를 받는 시점을 감지합니다. 패시브 방어력 스택 어빌리티는 영웅이 대미지를 입었을 때 방어력 스택을 하나 제거하기 위해 이를 사용합니다.

`AbilityTasks`의 주요 구성 요소:
* `AbilityTask`의 새 인스턴스를 생성하는 정적(static) 팩토리 함수
* `AbilityTask`가 목적을 달성했을 때 브로드캐스트하는 델리게이트들
* 메인 작업을 시작하고 외부 델리게이트에 바인딩하는 `Activate()` 함수
* 바인딩했던 외부 델리게이트 해제 등 정리를 담당하는 `OnDestroy()` 함수
* 바인딩된 외부 델리게이트에 대한 콜백 함수들
* 멤버 변수 및 내부 헬퍼 함수들

**참고:** 하나의 `AbilityTask`는 **오직 한 종류의 출력 델리게이트 시그니처만 선언**할 수 있습니다. 매개변수의 사용 여부와 관계없이 태스크 내의 모든 출력 델리게이트는 이와 동일한 타입이어야 합니다. 사용하지 않는 매개변수에는 기본값을 전달해야 합니다.

`AbilityTasks`는 원칙적으로 해당 `GameplayAbility`를 실행 중인 클라이언트 또는 서버에서만 실행됩니다. 그러나 `AbilityTask` 생성자에서 `bSimulatedTask = true;`를 설정하고, `virtual void InitSimulatedTask(UGameplayTasksComponent& InGameplayTasksComponent);`를 오버라이드하며, 관련 멤버 변수를 리플리케이트하도록 설정하면 시뮬레이트 클라이언트에서도 실행되도록 만들 수 있습니다. 이는 이동의 모든 세부 변경 사항을 패킷으로 리플리케이트하는 대신 전체 이동 태스크 자체를 시뮬레이트하고자 하는 이동 관련 `AbilityTasks` 같은 특수한 상황에서 유용합니다. 모든 `RootMotionSource` 태스크들이 이 방식을 사용합니다. `AbilityTask_MoveToLocation.h/.cpp`를 예시로 참조하세요.

`AbilityTask` 생성자에서 `bTickingTask = true;`를 설정하고 `virtual void TickTask(float DeltaTime);`을 오버라이드하면 태스크가 매 프레임 `Tick`을 수행하도록 할 수 있습니다. 프레임에 걸쳐 부드럽게 값을 보간(Lerp)해야 할 때 유용합니다. `AbilityTask_MoveToLocation.h/.cpp`를 참조하세요.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-at-using"></a>
### 4.7.3 어빌리티 태스크 사용법
C++에서 `AbilityTask`를 생성하고 활성화하는 방법 (`GDGA_FireGun.cpp` 발췌):
```c++
UGDAT_PlayMontageAndWaitForEvent* Task = UGDAT_PlayMontageAndWaitForEvent::PlayMontageAndWaitForEvent(this, NAME_None, MontageToPlay, FGameplayTagContainer(), 1.0f, NAME_None, false, 1.0f);
Task->OnBlendOut.AddDynamic(this, &UGDGA_FireGun::OnCompleted);
Task->OnCompleted.AddDynamic(this, &UGDGA_FireGun::OnCompleted);
Task->OnInterrupted.AddDynamic(this, &UGDGA_FireGun::OnCancelled);
Task->OnCancelled.AddDynamic(this, &UGDGA_FireGun::OnCancelled);
Task->EventReceived.AddDynamic(this, &UGDGA_FireGun::EventReceived);
Task->ReadyForActivation();
```

블루프린트에서는 `AbilityTask`용으로 생성된 블루프린트 노드를 배치하기만 하면 됩니다. 블루프린트에서는 `ReadyForActivation()`을 호출할 필요가 없습니다. 에디터의 `Engine/Source/Editor/GameplayTasksEditor/Private/K2Node_LatentGameplayTaskCall.cpp`가 이를 자동으로 호출해 줍니다. 또한 `K2Node_LatentGameplayTaskCall`은 태스크 클래스에 존재하는 경우 `BeginSpawningActor()`와 `FinishSpawningActor()`도 자동으로 호출합니다 (`AbilityTask_WaitTargetData` 참조). 다시 강조하자면, 이러한 자동 마법 처리는 블루프린트에서만 일어납니다. C++에서는 `ReadyForActivation()`, `BeginSpawningActor()`, `FinishSpawningActor()`를 수동으로 명시적 호출해야 합니다.

![블루프린트 WaitTargetData AbilityTask](https://github.com/tranek/GASDocumentation/raw/master/Images/abilitytask.png)

`AbilityTask`를 수동으로 취소하려면 블루프린트의 태스크 객체(`Async Task Proxy`) 또는 C++에서 `EndTask()`를 호출하면 됩니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-at-rms"></a>
### 4.7.4 루트 모션 소스 어빌리티 태스크
GAS에는 `CharacterMovementComponent`와 연동된 `Root Motion Sources`를 활용하여 넉백, 복잡한 점프, 당기기, 대시 등 캐릭터를 시간에 따라 이동시키는 다양한 `AbilityTasks`가 기본 제공됩니다.

**참고:** `RootMotionSource` `AbilityTasks`의 예측은 엔진 버전 4.19 및 4.25 이상에서 완벽하게 동작합니다. 4.20~4.24 엔진 버전에서는 예측에 버그가 존재했지만, 멀티플레이어 환경에서도 약간의 넷 보정(Net Correction)과 함께 기본 기능은 수행되며 싱글플레이어에서는 문제없이 작동합니다. 4.25의 [예측 수정 커밋](https://github.com/EpicGames/UnrealEngine/commit/94107438dd9f490e7b743f8e13da46927051bf33#diff-65f6196f9f28f560f95bd578e07e290c)을 체리픽하여 4.20~4.24 커스텀 엔진에 반영할 수도 있습니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-gc"></a>
### 4.8 게임플레이 큐 (Gameplay Cues)

<a name="concepts-gc-definition"></a>
#### 4.8.1 게임플레이 큐 정의
`GameplayCues` (`GC`)는 사운드 효과, 파티클 이펙트, 카메라 흔들림(Camera Shake) 등 게임플레이 로직과 무관한 시각적/청각적 연출을 실행합니다. `GameplayCues`는 일반적으로 리플리케이트되며(로컬에서 명시적으로 `Executed`, `Added`, `Removed`된 경우 제외) 예측이 가능합니다.

우리는 `ASC`를 통해 `GameplayCueManager`에 **반드시 `GameplayCue.`로 시작하는 부모 이름을 가진 `GameplayTag`**와 이벤트 유형(`Execute`, `Add`, `Remove`)을 전달하여 `GameplayCues`를 트리거합니다. `GameplayCueNotify` 객체 및 `IGameplayCueInterface`를 구현하는 액터들은 해당 `GameplayCue`의 `GameplayTag`(`GameplayCueTag`)를 기반으로 이 이벤트를 구독하여 처리합니다.

**참고:** 다시 한 번 강조하지만, `GameplayCue`용 태그는 반드시 `GameplayCue` 부모 태그로 시작해야 합니다. 예를 들어 유효한 태그는 `GameplayCue.A.B.C` 형태입니다.

`GameplayCueNotifies`에는 `Static`과 `Actor` 두 가지 클래스가 있습니다. 이들은 서로 다른 이벤트에 반응하며 서로 다른 유형의 `GameplayEffects`에 의해 트리거됩니다. 해당 이벤트를 오버라이드하여 로직을 작성합니다.

| `GameplayCue` 클래스                                                                                 | 이벤트             | 대응 `GameplayEffect` 유형 | 설명                                                                                                                                                                                                                                                                                 |
| ---------------------------------------------------------------------------------------------------- | ------------------ | -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [`GameplayCueNotify_Static`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayCueNotify_Static/index.html) | `Execute`          | `Instant` 또는 `Periodic`  | Static `GameplayCueNotifies`는 `ClassDefaultObject`(인스턴스 없음) 상에서 실행되며 탄환 충돌 이펙트처럼 일회성 단발 이펙트에 완벽합니다.                                                                                                                                           |
| [`GameplayCueNotify_Actor`](https://docs.unrealengine.com/en-US/BlueprintAPI/GameplayCueNotify/index.html)                           | `Add` 또는 `Remove` | `Duration` 또는 `Infinite` | Actor `GameplayCueNotifies`는 `Added`될 때 새로운 액터 인스턴스를 스폰합니다. 인스턴스화되므로 `Removed`될 때까지 시간에 걸쳐 지속적인 연출을 수행할 수 있습니다. 기반이 되는 `Duration`/`Infinite` `GameplayEffect`가 제거될 때 함께 종료되는 루핑 사운드나 파티클 효과에 적합합니다. 동시에 허용되는 인스턴스 수를 제어하는 옵션도 제공됩니다. |

`GameplayCueNotifies`는 기술적으로 모든 이벤트에 반응할 수 있지만, 위 표가 가장 일반적인 사용 패턴입니다.

**참고:** `GameplayCueNotify_Actor`를 사용할 때는 반드시 `Auto Destroy on Remove`를 체크해야 합니다. 그렇지 않으면 해당 `GameplayCueTag`에 대한 후속 `Add` 호출이 정상 동작하지 않습니다.

`ASC` [리플리케이션 모드](#concepts-asc-rm)가 `Full`이 아닐 때, 서버 플레이어(리슨 서버)에서는 `Add` 및 `Remove` `GC` 이벤트가 두 번 발생할 수 있습니다 (GE 적용 시 1번 + 클라이언트로의 Minimal NetMultiCast 시 1번). 그러나 `WhileActive` 이벤트는 여전히 한 번만 발생합니다. 클라이언트에서는 모든 이벤트가 항상 한 번만 발생합니다.

샘플 프로젝트에는 기절 및 질주 효과를 위한 `GameplayCueNotify_Actor`와 총기 투사체 충돌을 위한 `GameplayCueNotify_Static`이 포함되어 있습니다. 이 `GC`들은 `GE`를 통해 리플리케이트하는 대신 [로컬에서 직접 트리거](#concepts-gc-local)하여 성능을 더욱 최적화할 수 있습니다. 샘플 프로젝트는 초심자를 위해 가장 기본적인 방식을 보여주고 있습니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-gc-trigger"></a>
#### 4.8.2 게임플레이 큐 트리거
`GameplayEffect` 내부에서 성공적으로 적용되었을 때(태그 차단이나 면역이 없을 때) 발동할 모든 `GameplayCues`의 `GameplayTags`를 채워 넣습니다.

![GameplayEffect에서 트리거되는 GameplayCue](https://github.com/tranek/GASDocumentation/raw/master/Images/gcfromge.png)

`UGameplayAbility`는 블루프린트에서 `GameplayCues`를 `Execute`, `Add`, `Remove`할 수 있는 전용 노드를 제공합니다.

![GameplayAbility에서 트리거되는 GameplayCue](https://github.com/tranek/GASDocumentation/raw/master/Images/gcfromga.png)

C++에서는 `ASC`의 함수를 직접 호출할 수 있습니다 (또는 `ASC` 서브클래스에서 블루프린트로 노출):

```c++
/** GameplayCue를 독립적으로 실행합니다. 적중 결과 등을 전달하기 위해 선택적 effect context를 받습니다. */
void ExecuteGameplayCue(const FGameplayTag GameplayCueTag, FGameplayEffectContextHandle EffectContext = FGameplayEffectContextHandle());
void ExecuteGameplayCue(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

/** 지속형 게임플레이 큐를 추가합니다. */
void AddGameplayCue(const FGameplayTag GameplayCueTag, FGameplayEffectContextHandle EffectContext = FGameplayEffectContextHandle());
void AddGameplayCue(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

/** 지속형 게임플레이 큐를 제거합니다. */
void RemoveGameplayCue(const FGameplayTag GameplayCueTag);
	
/** GameplayEffect의 일부가 아닌 자체적으로 추가된 모든 GameplayCue를 제거합니다. */
void RemoveAllGameplayCues();
```

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-gc-local"></a>
#### 4.8.3 로컬 게임플레이 큐 (Local Gameplay Cues)
`GameplayAbilities`나 `ASC`에서 노출된 기본 `GameplayCue` 발동 함수들은 기본적으로 리플리케이트됩니다. 각 `GameplayCue` 이벤트는 멀티캐스트 RPC로 전달됩니다. 이는 매우 많은 RPC 트래픽을 유발할 수 있습니다. 또한 GAS는 넷 업데이트 주기당 동일한 `GameplayCue` RPC를 최대 2개로 제한합니다. 우리는 가능한 곳에서 로컬 `GameplayCues`를 사용하여 이를 방지합니다. 로컬 `GameplayCues`는 해당 개별 클라이언트에서만 `Execute`, `Add`, `Remove`됩니다.

로컬 `GameplayCues`를 사용할 수 있는 시나리오:
* 투사체 충돌 (Projectile impacts)
* 근접 공격 충돌 (Melee collision impacts)
* 애니메이션 몽타주에서 발동되는 `GameplayCues`

`ASC` 서브클래스에 추가해야 하는 로컬 `GameplayCue` 함수들:

```c++
UFUNCTION(BlueprintCallable, Category = "GameplayCue", Meta = (AutoCreateRefTerm = "GameplayCueParameters", GameplayTagFilter = "GameplayCue"))
void ExecuteGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

UFUNCTION(BlueprintCallable, Category = "GameplayCue", Meta = (AutoCreateRefTerm = "GameplayCueParameters", GameplayTagFilter = "GameplayCue"))
void AddGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

UFUNCTION(BlueprintCallable, Category = "GameplayCue", Meta = (AutoCreateRefTerm = "GameplayCueParameters", GameplayTagFilter = "GameplayCue"))
void RemoveGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);
```

```c++
void UPAAbilitySystemComponent::ExecuteGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters & GameplayCueParameters)
{
	UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::Executed, GameplayCueParameters);
}

void UPAAbilitySystemComponent::AddGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters & GameplayCueParameters)
{
	UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::OnActive, GameplayCueParameters);
	UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::WhileActive, GameplayCueParameters);
}

void UPAAbilitySystemComponent::RemoveGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters & GameplayCueParameters)
{
	UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::Removed, GameplayCueParameters);
}
```

만약 `GameplayCue`가 로컬에서 `Added`되었다면 반드시 로컬에서 `Removed`되어야 합니다. 리플리케이션을 통해 `Added`되었다면 리플리케이션을 통해 `Removed`되어야 합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-gc-parameters"></a>
#### 4.8.4 게임플레이 큐 매개변수 (Parameters)
`GameplayCues`는 추가 정보를 담고 있는 `FGameplayCueParameters` 구조체를 매개변수로 전달받습니다. `GameplayAbility`나 `ASC`의 함수에서 `GameplayCue`를 수동으로 트리거하는 경우, 전달할 `GameplayCueParameters` 구조체를 직접 채워야 합니다. `GameplayCue`가 `GameplayEffect`에 의해 트리거되는 경우, 다음 변수들이 `GameplayCueParameters` 구조체에 자동으로 채워집니다:

* AggregatedSourceTags
* AggregatedTargetTags
* GameplayEffectLevel
* AbilityLevel
* [EffectContext](#concepts-ge-context)
* Magnitude (`GameplayEffect`의 태그 컨테이너 상단 드롭다운에서 크기 측정용 어트리뷰트가 선택되어 있고 해당 어트리뷰트에 영향을 주는 모디파이어가 있는 경우)

`GameplayCueParameters` 구조체의 `SourceObject` 변수는 `GameplayCue`를 수동 트리거할 때 임의의 데이터를 넘기기에 좋은 위치입니다.

**참고:** `Instigator` 등 일부 변수는 이미 `EffectContext` 내에 존재할 수 있습니다. `EffectContext`는 월드에서 `GameplayCue`를 스폰할 위치를 위한 `FHitResult`도 포함할 수 있습니다. `EffectContext`를 서브클래싱하는 것은 특히 `GameplayEffect`에 의해 트리거되는 `GameplayCues`에 더 많은 데이터를 전달하는 훌륭한 방법입니다.

자세한 내용은 `GameplayCueParameters` 구조체를 채우는 [`UAbilitySystemGlobals`](#concepts-asg)의 세 함수를 참조하세요. 이 함수들은 가상(virtual) 함수이므로 오버라이드하여 더 많은 정보를 자동으로 채우도록 확장할 수 있습니다.

```c++
/** GameplayCue 매개변수 초기화 */
virtual void InitGameplayCueParameters(FGameplayCueParameters& CueParameters, const FGameplayEffectSpecForRPC &Spec);
virtual void InitGameplayCueParameters_GESpec(FGameplayCueParameters& CueParameters, const FGameplayEffectSpec &Spec);
virtual void InitGameplayCueParameters(FGameplayCueParameters& CueParameters, const FGameplayEffectContextHandle& EffectContext);
```

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-gc-manager"></a>
#### 4.8.5 게임플레이 큐 매니저 (Gameplay Cue Manager)
기본적으로 `GameplayCueManager`는 게임 디렉터리 전체를 스캔하여 `GameplayCueNotifies`를 찾고 게임 실행 시 모두 메모리에 로드합니다. `DefaultGame.ini` 설정을 통해 스캔 경로를 변경할 수 있습니다.

```
[/Script/GameplayAbilities.AbilitySystemGlobals]
GameplayCueNotifyPaths="/Game/GASDocumentation/Characters"
```

우리는 `GameplayCueManager`가 모든 `GameplayCueNotifies`를 스캔하여 찾아내기를 원하지만, 게임 시작 시 모든 에셋을 비동기 로드하는 것은 원치 않습니다. 그렇게 하면 해당 레벨에서 전혀 사용되지 않는 에셋이라 하더라도 모든 `GameplayCueNotify`와 이들이 참조하는 모든 사운드 및 파티클이 메모리에 올라가게 됩니다. 파라곤(Paragon)과 같은 대형 게임에서는 불필요한 수백 메가바이트의 에셋이 메모리를 차지하며 시작 시 끊김(Hitching) 및 프리징을 유발할 수 있습니다.

시작 시 모든 `GameplayCue`를 일괄 비동기 로드하는 대신, 게임플레이 중에 특정 `GameplayCue`가 처음 트리거될 때 비동기 로드하도록 변경할 수 있습니다. 이렇게 하면 메모리 낭비와 시작 시 프리징을 방지할 수 있지만, 플레이 도중 특정 큐가 처음 발동될 때 약간의 연출 딜레이가 발생할 수 있습니다. SSD 환경에서는 이러한 지연을 거의 체감할 수 없습니다. 언리얼 에디터 상에서는 파티클 시스템 컴파일로 인해 첫 로드 시 약간의 렉이 발생할 수 있으나, 이미 파티클이 컴파일되어 패키징된 빌드에서는 문제가 되지 않습니다.

먼저 `UGameplayCueManager`를 상속받은 서브클래스를 만들고, `DefaultGame.ini`에서 `AbilitySystemGlobals`가 우리 서브클래스를 사용하도록 지정합니다.

```
[/Script/GameplayAbilities.AbilitySystemGlobals]
GlobalGameplayCueManagerClass="/Script/ParagonAssets.PBGameplayCueManager"
```

커스텀 `UGameplayCueManager` 서브클래스에서 `ShouldAsyncLoadRuntimeObjectLibraries()`를 오버라이드합니다.

```c++
virtual bool ShouldAsyncLoadRuntimeObjectLibraries() const override
{
	return false;
}
```

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-gc-prevention"></a>
#### 4.8.6 게임플레이 큐 발동 방지
공격을 방어했을 때 대미지 `GameplayEffect`에 부착된 일반 피격 이펙트의 재생을 막고 방어 전용 이펙트를 재생하고 싶을 때처럼, `GameplayCues`의 발동을 차단하고 싶은 경우가 있습니다. [`GameplayEffectExecutionCalculations`](#concepts-ge-ec) 내부에서 `OutExecutionOutput.MarkGameplayCuesHandledManually()`를 호출한 뒤, 대상이나 소스의 `ASC`에 원하는 커스텀 `GameplayCue` 이벤트를 수동으로 전송함으로써 이를 달성할 수 있습니다.

특정 `ASC`에서 일체의 모든 `GameplayCues`가 발동되지 않도록 완전히 억제하려면 `AbilitySystemComponent->bSuppressGameplayCues = true;`를 설정할 수 있습니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-gc-batching"></a>
#### 4.8.7 게임플레이 큐 배칭 (Gameplay Cue Batching)
각각의 `GameplayCue` 트리거는 비신뢰(Unreliable) NetMulticast RPC입니다. 여러 `GC`가 동시에 발동되는 상황에서는 이를 1개의 RPC로 압축하거나 전송 데이터양을 줄여 대역폭을 절약하는 최적화 기법들이 존재합니다.

<a name="concepts-gc-batching-manualrpc"></a>
##### 4.8.7.1 수동 RPC
8발의 산탄 펠릿을 발사하는 샷건이 있다고 가정해 봅시다. 이는 8번의 트레이스 및 충돌 `GameplayCues`가 발생함을 의미합니다. [GASShooter](https://github.com/tranek/GASShooter)는 모든 트레이스 정보를 [`EffectContext`](#concepts-ge-ec)에 [`TargetData`](#concepts-targeting-data)로 담아 1개의 RPC로 묶는 방식을 취했습니다. 이렇게 하면 RPC 횟수를 8회에서 1회로 줄일 수 있지만, 단일 RPC에 실려 가는 데이터양(~500바이트)은 여전히 큽니다. 더 최적화된 방식은 충돌 위치들을 효율적으로 인코딩하거나 랜덤 시드(Random Seed) 번호만 전달하여 수신 측에서 충돌 지점을 재현/근사하는 커스텀 구조체를 RPC로 보내는 것입니다. 클라이언트는 이 커스텀 구조체를 받아 언팩한 후 [로컬 실행 `GameplayCues`](#concepts-gc-local)로 재생합니다.

동작 방식:
1. `FScopedGameplayCueSendContext`를 선언합니다. 이는 스코프를 벗어날 때까지 `UGameplayCueManager::FlushPendingCues()`의 호출을 억제하여 모든 `GameplayCues`를 큐에 보관합니다.
1. `UGameplayCueManager::FlushPendingCues()`를 오버라이드하여 특정 커스텀 `GameplayTag`를 기준으로 배칭 가능한 `GameplayCues`를 커스텀 구조체로 병합하고 클라이언트에 RPC를 보냅니다.
1. 클라이언트는 커스텀 구조체를 수신하여 언팩한 뒤 로컬 실행 `GameplayCues`로 변환합니다.

이 방식은 대미지 텍스트, 치명타 표시, 쉴드 파괴 표시, 결정타(치명타 처치) 표시 등 `GameplayCueParameters`에 맞지 않고 `EffectContext`를 복잡하게 만들고 싶지 않은 커스텀 파라미터가 필요할 때도 유용합니다.

참고: https://forums.unrealengine.com/development-discussion/c-gameplay-programming/1711546-fscopedgameplaycuesendcontext-gameplaycuemanager

<a name="concepts-gc-batching-gcsonge"></a>
##### 4.8.7.2 하나의 GE에 여러 GC 설정
`GameplayEffect`에 포함된 모든 `GameplayCues`는 이미 기본적으로 1개의 RPC로 함께 전송됩니다. 기본 설정에서 `UGameplayCueManager::InvokeGameplayCueAddedAndWhileActive_FromSpec()`은 `ASC`의 리플리케이션 모드와 무관하게 비신뢰 NetMulticast로 전체 `GameplayEffectSpec`(`FGameplayEffectSpecForRPC`로 변환됨)을 전송합니다. `GameplayEffectSpec`의 내용에 따라 대역폭 소모가 커질 수 있습니다. `AbilitySystem.AlwaysConvertGESpecToGCParams 1` 콘솔 변수를 설정하면 전체 스펙 대신 `FGameplayCueParameters` 구조체로 변환하여 전송하므로 대역폭을 최적화할 수 있습니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-gc-events"></a>
#### 4.8.8 게임플레이 큐 이벤트 (Gameplay Cue Events)
`GameplayCues`는 특정 `EGameplayCueEvents`에 응답합니다:

| `EGameplayCueEvent` | 설명                                                                                                                                                                                                                                                                          |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `OnActive`          | `GameplayCue`가 활성화(추가)될 때 호출됩니다.                                                                                                                                                                                                                                 |
| `WhileActive`       | `GameplayCue`가 활성 상태인 동안 호출됩니다 (중도 참가자 등). **이는 `Tick`이 아닙니다!** `GameplayCueNotify_Actor`가 추가되거나 리플리케이션 가시 범위(Relevancy)에 들어올 때 `OnActive`처럼 단 한 번 호출됩니다. `Tick()`이 필요하다면 액터이므로 자체 `Tick()`을 사용하십시오. |
| `Removed`           | `GameplayCue`가 제거될 때 호출됩니다. 이에 응답하는 블루프린트 함수는 `OnRemove`입니다.                                                                                                                                                                                        |
| `Executed`          | `GameplayCue`가 실행될 때 호출됩니다: 즉시 이펙트 또는 주기적 `Tick()` 실행. 이에 응답하는 블루프린트 함수는 `OnExecute`입니다.                                                                                                                                              |

`GameplayCue` 시작 시점에 발생해야 하지만 중도 참가자(Late joiner)가 놓쳐도 무방한 연출에는 `OnActive`를 사용합니다. 중도 참가자에게도 보여야 하는 지속적인 연출에는 `WhileActive`를 사용합니다. 예를 들어 MOBA에서 타워 폭발 `GameplayCue`가 있다면 최초 폭발 파티클과 사운드는 `OnActive`에 넣고, 폭발 후 바닥에 남아 지속되는 잔존 화염 파티클과 루핑 사운드는 `WhileActive`에 배치합니다. 이 경우 나중에 참가한 플레이어가 최초 폭발을 다시 볼 필요는 없지만 바닥의 지속 화염 효과는 보아야 하기 때문입니다. `OnRemove`는 `OnActive`와 `WhileActive`에서 생성된 모든 리소스를 정리해야 합니다. 액터가 `GameplayCueNotify_Actor`의 리플리케이션 범위에 들어올 때마다 `WhileActive`가 호출되고 범위를 벗어날 때마다 `OnRemove`가 호출됩니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-gc-reliability"></a>
#### 4.8.9 게임플레이 큐 신뢰성 (Reliability)
`GameplayCues`는 기본적으로 비신뢰(Unreliable) 네트워크 전송으로 간주되어야 하며 게임플레이에 직접적인 영향을 미치는 핵심 로직에는 부적합합니다.

**Executed `GameplayCues`:** 비신뢰 멀티캐스트로 적용되며 항상 비신뢰성입니다.

**`GameplayEffects`를 통해 적용된 `GameplayCues`:**
* 자율 프록시(Autonomous Proxy)는 `OnActive`, `WhileActive`, `OnRemove`를 신뢰성 있게 수신합니다.  
  `FActiveGameplayEffectsContainer::NetDeltaSerialize()`가 `UAbilitySystemComponent::HandleDeferredGameplayCues()`를 호출하여 `OnActive`와 `WhileActive`를 실행하며, 제거 시 `FActiveGameplayEffectsContainer::RemoveActiveGameplayEffectGrantedTagsAndModifiers()`가 `OnRemoved`를 호출합니다.
* 시뮬레이트 프록시(Simulated Proxy)는 `WhileActive`와 `OnRemove`를 신뢰성 있게 수신합니다.  
  `UAbilitySystemComponent::MinimalReplicationGameplayCues`의 복제가 `WhileActive`와 `OnRemove`를 호출합니다. `OnActive` 이벤트는 비신뢰 멀티캐스트로 전달됩니다.

**`GameplayEffect` 없이 단독 적용된 `GameplayCues`:**
* 자율 프록시는 `OnRemove`만 신뢰성 있게 수신합니다.  
  `OnActive`와 `WhileActive` 이벤트는 비신뢰 멀티캐스트로 호출됩니다.
* 시뮬레이트 프록시는 `WhileActive`와 `OnRemove`를 신뢰성 있게 수신합니다.  
  `OnActive` 이벤트는 비신뢰 멀티캐스트로 호출됩니다.

만약 `GameplayCue`에서 특정 연출이 반드시 '신뢰성(Reliable)' 있게 동기화되어야 한다면, 이를 `GameplayEffect`를 통해 적용하고 `WhileActive`에서 FX를 생성하며 `OnRemove`에서 FX를 제거하도록 구성하십시오.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-asg"></a>
### 4.9 어빌리티 시스템 글로벌 (Ability System Globals)
[`AbilitySystemGlobals`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAbilitySystemGlobals/index.html) 클래스는 GAS에 대한 글로벌 전역 정보를 보관합니다. 대부분의 변수는 `DefaultGame.ini`에서 설정할 수 있습니다. 일반적으로 이 클래스와 직접 상호작용할 일은 많지 않지만 그 존재를 알고 있어야 합니다. [`GameplayCueManager`](#concepts-gc-manager)나 [`GameplayEffectContext`](#concepts-ge-context) 등을 서브클래싱할 때 `AbilitySystemGlobals`를 통해 등록해야 합니다.

`AbilitySystemGlobals`를 서브클래싱하려면 `DefaultGame.ini`에 클래스 이름을 지정합니다:
```
[/Script/GameplayAbilities.AbilitySystemGlobals]
AbilitySystemGlobalsClassName="/Script/ParagonAssets.PAAbilitySystemGlobals"
```

<a name="concepts-asg-initglobaldata"></a>
#### 4.9.1 InitGlobalData()
UE 4.24부터 5.2 사이 버전에서는 [`TargetData`](#concepts-targeting-data)를 사용하기 위해 `UAbilitySystemGlobals::Get().InitGlobalData()`를 반드시 호출해야 했습니다. 그렇지 않으면 `ScriptStructCache` 관련 오류가 발생하고 클라이언트 연결이 끊어집니다. 이 함수는 프로젝트에서 단 한 번만 호출하면 됩니다. 포트나이트는 `UAssetManager::StartInitialLoading()`에서 호출하고 파라곤은 `UEngine::Init()`에서 호출했습니다. 샘플 프로젝트처럼 `UAssetManager::StartInitialLoading()`에 배치하는 것이 가장 좋습니다. `TargetData` 관련 오류를 방지하기 위해 프로젝트에 복사해 넣어야 할 기본 보일러플레이트 코드입니다. 5.3부터는 이 함수가 엔진에서 자동으로 호출됩니다.

`AbilitySystemGlobals`의 `GlobalAttributeSetDefaultsTableNames`를 사용할 때 크래시가 발생하는 경우, 포트나이트처럼 `AssetManager`나 `GameInstance`에서 `UAbilitySystemGlobals::Get().InitGlobalData()`를 조금 더 늦은 타이밍에 호출해야 할 수 있습니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-p"></a>

### 4.10 예측 (Prediction)
GAS는 클라이언트 측 예측(Client-side Prediction)을 기본 지원하지만, 모든 것을 예측하지는 않습니다. GAS에서 클라이언트 측 예측이란 클라이언트가 `GameplayAbility`를 활성화하고 `GameplayEffects`를 적용하기 위해 서버의 승인을 기다릴 필요가 없음을 의미합니다. 클라이언트는 서버가 승인해 줄 것을 "예측"하고 `GameplayEffects`를 적용할 대상(타겟)을 예측합니다. 그 후 서버는 네트워크 지연 시간(Latency) 뒤에 `GameplayAbility`를 실행하고 클라이언트의 예측이 맞았는지 틀렸는지를 클라이언트에 통보합니다. 만약 클라이언트의 예측이 틀렸다면(오예측/Misprediction), 클라이언트는 잘못 예측했던 변경 사항들을 서버 상태에 맞춰 "롤백(Rollback)"합니다.

GAS 관련 예측에 대한 가장 명확하고 권위 있는 원본 소스는 플러그인 소스 코드의 `GameplayPrediction.h`입니다.

Epic의 기본 철학은 **"문제가 되지 않는 최소한의 것만 예측하라(Only predict what you can get away with)"**입니다. 예를 들어 파라곤과 포트나이트는 대미지(Damage)를 예측하지 않습니다. 이들 게임은 애초에 예측이 불가능한 [`ExecutionCalculations`](#concepts-ge-ec)로 대미지를 처리했을 가능성이 높습니다. 그렇다고 대미지 같은 요소를 예측해서는 안 된다는 뜻은 아닙니다. 여러분의 프로젝트에서 이를 시도하여 만족스러운 결과를 얻었다면 그것으로 훌륭합니다.

> ... 저희 역시 "모든 것을 매끄럽고 완벽하게 자동 예측한다"는 식의 솔루션을 전적으로 지지하지는 않습니다. 저희는 여전히 플레이어 예측을 최소한으로 유지하는 것(즉, 문제가 생기지 않는 선에서 최소한의 것만 예측하는 것)이 최선이라고 생각합니다.
> 
> *새로운 [네트워크 예측 플러그인(Network Prediction Plugin)](#concepts-p-npp)에 대한 Epic의 Dave Ratti 코멘트*

**예측 가능한 항목:**
> * 어빌리티 활성화 (Ability activation)
> * 트리거된 이벤트 (Triggered Events)
> * `GameplayEffect` 적용:
>    * 어트리뷰트 수정 (예외: Execution은 현재 예측되지 않으며, Attribute Modifier만 예측됨)
>    * 게임플레이 태그(GameplayTag) 수정
> * 게임플레이 큐(GameplayCue) 이벤트 (예측 GE 내부 및 독립 실행 모두)
> * 애니메이션 몽타주 재생
> * 이동 (언리얼 엔진의 `UCharacterMovement`에 내장됨)

**예측 불가능한 항목:**
> * `GameplayEffect` 제거 (Removal)
> * `GameplayEffect` 주기적 효과 (Periodic Effects / DoT 틱)

*`GameplayPrediction.h` 발췌*

`GameplayEffect`의 적용은 예측할 수 있지만, `GameplayEffect`의 제거는 예측할 수 없습니다. 이 한계를 우회하는 한 가지 방법은 `GameplayEffect`를 제거하고 싶을 때 그 반대 효과를 주는 이펙트를 예측 적용하는 것입니다. 예를 들어 40% 이동 속도 둔화가 걸려 있을 때, 40% 이동 속도 증가 버프를 예측 적용한 뒤 두 `GameplayEffects`를 동시에 제거하는 방식입니다. 하지만 이 방식이 모든 시나리오에 적합한 것은 아니며, `GameplayEffect` 제거 예측에 대한 엔진 차원의 지원은 여전히 필요합니다. Epic의 Dave Ratti는 [향후 GAS 버전](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)에서 이를 추가하고 싶다고 밝혔습니다.

`GameplayEffects`의 제거를 예측할 수 없기 때문에, `GameplayAbility`의 쿨다운을 완벽하게 예측할 수 없으며 이에 대한 반대 이펙트 우회책도 존재하지 않습니다. 서버가 리플리케이트한 `Cooldown GE`가 클라이언트에 존재하게 되며, 이를 우회하려는 시도(예: `Minimal` 리플리케이션 모드 사용)는 서버에 의해 거부됩니다. 즉, 지연 시간(핑)이 높은 클라이언트는 서버에 쿨다운 시작을 알리고 서버의 `Cooldown GE` 제거를 수신하는 데 더 오랜 시간이 걸립니다. 결과적으로 핑이 높은 플레이어는 핑이 낮은 플레이어에 비해 쿨다운이 짧은 어빌리티의 발사 속도(연사력)가 떨어져 불리해집니다. 포트나이트는 `Cooldown GEs` 대신 자체적인 커스텀 장부 관리 로직을 사용하여 이 문제를 회피했습니다.

대미지 예측과 관련하여, 초심자들이 GAS를 시작할 때 가장 먼저 시도하는 것 중 하나이지만 저는 개인적으로 대미지 예측을 권장하지 않습니다. 특히 **사망(Death) 예측은 절대로 추천하지 않습니다.** 대미지를 예측할 수는 있지만 매우 까다롭습니다. 대미지 적용을 잘못 예측하면 플레이어 눈앞에서 적의 체력 바가 다시 차오르는 현상이 발생합니다. 사망을 예측했다가 빗나간 경우 캐릭터가 래그돌(Ragdoll) 상태로 쓰러졌다가 서버 보정을 받고 벌떡 일어나 다시 총을 쏘는 매우 어색하고 당혹스러운 상황이 연출될 수 있습니다.

**참고:** 자신에게 적용되는 어트리뷰트 변경 즉시(Instant) `GameplayEffects`(예: `Cost GEs`)는 매끄럽게 예측되지만, 다른 캐릭터에게 즉시 어트리뷰트 변경을 예측 적용하면 대상의 어트리뷰트에 순간적인 튐(Blip/오차) 현상이 나타날 수 있습니다. 예측된 `Instant` `GameplayEffects`는 예측 실패 시 롤백될 수 있도록 내부적으로 `Infinite` `GameplayEffects`처럼 취급됩니다. 서버의 `GameplayEffect`가 도착하여 적용되는 순간 일시적으로 두 개의 동일한 `GameplayEffect`가 공존하여 모디파이어가 찰나의 순간 동안 2번 적용되거나 전혀 적용되지 않는 상태가 될 수 있습니다. 곧 스스로 보정되지만 플레이어의 눈에 순간적인 튐이 보일 수 있습니다.

GAS의 예측 구현이 해결하고자 하는 6가지 핵심 과제:
> 1. "내가 이 동작을 해도 되는가? (Can I do this?)" 예측을 위한 기본 프로토콜.
> 2. "실행 취소 (Undo)" 예측이 실패했을 때 부작용(Side Effect)을 되돌리는 방법.
> 3. "재실행 방지 (Redo)" 로컬에서 예측 실행했으나 서버로부터 다시 리플리케이트되어 오는 부작용이 중복 재생되는 것을 방지하는 방법.
> 4. "완전성 (Completeness)" 모든 부작용을 /정말로/ 빠짐없이 예측했는지 확인하는 방법.
> 5. "의존성 (Dependencies)" 종속적인 예측 및 일련의 연쇄 예측 이벤트를 관리하는 방법.
> 6. "오버라이드 (Override)" 서버가 소유/리플리케이트하는 상태를 클라이언트가 예측적으로 덮어쓰는 방법.

*`GameplayPrediction.h` 발췌*

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-p-key"></a>
#### 4.10.1 예측 키 (Prediction Key)
GAS의 예측은 클라이언트가 `GameplayAbility`를 활성화할 때 생성하는 정수 식별자인 `Prediction Key`(예측 키)라는 개념을 기반으로 동작합니다.

* 클라이언트는 `GameplayAbility`를 활성화할 때 예측 키를 생성합니다. 이것이 `Activation Prediction Key`(활성화 예측 키)입니다.
* 클라이언트는 `CallServerTryActivateAbility()`를 통해 이 예측 키를 서버로 전송합니다.
* 클라이언트는 해당 예측 키가 유효한 동안 자신이 적용하는 모든 `GameplayEffects`에 이 예측 키를 부여합니다.
* 클라이언트의 예측 키가 스코프를 벗어납니다. 동일 어빌리티 내에서 추가적인 효과를 예측하려면 새로운 [스코프 예측 윈도우(Scoped Prediction Window)](#concepts-p-windows)가 필요합니다.


* 서버는 클라이언트로부터 예측 키를 수신합니다.
* 서버는 자신이 적용하는 모든 `GameplayEffects`에 이 예측 키를 동일하게 부여합니다.
* 서버는 이 예측 키를 클라이언트로 다시 리플리케이트합니다.


* 클라이언트는 적용에 사용된 예측 키가 포함된 `GameplayEffects`를 서버로부터 리플리케이트받습니다. 서버에서 온 `GameplayEffect`가 클라이언트가 동일 예측 키로 적용했던 `GameplayEffect`와 일치한다면 정확하게 예측된 것입니다. 클라이언트가 자신의 예측본을 제거할 때까지 대상에는 일시적으로 2개의 `GameplayEffect` 사본이 존재합니다.
* 클라이언트는 서버로부터 예측 키를 다시 수신합니다. 이것이 `Replicated Prediction Key`입니다. 이 예측 키는 이제 만료(Stale)된 것으로 표시됩니다.
* 클라이언트는 이제 만료된 리플리케이트 예측 키로 생성했던 **모든** `GameplayEffects`를 로컬에서 제거합니다. 서버가 리플리케이트한 `GameplayEffects`는 유지됩니다. 클라이언트가 추가했으나 서버로부터 일치하는 리플리케이트 버전을 받지 못한 모든 `GameplayEffects`는 오예측(Misprediction)된 것으로 판명되어 롤백됩니다.

예측 키는 활성화 예측 키로부터 시작되는 `GameplayAbilities` 내의 단일 원자적 명령 그룹 "윈도우(창)" 동안에만 유효함이 보장됩니다. 쉽게 말해 **단 1프레임 동안만 유효**하다고 생각할 수 있습니다. 지연 액션인 `AbilityTasks`의 콜백은 태스크 내부에 새로운 [스코프 예측 윈도우](#concepts-p-windows)를 생성하는 동기화 지점(Synch Point)이 내장되어 있지 않는 한 더 이상 유효한 예측 키를 가지지 못합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-p-windows"></a>
#### 4.10.2 어빌리티 내에서 새 예측 윈도우 생성
`AbilityTasks`의 콜백에서 추가적인 액션을 예측하려면 새로운 Scoped Prediction Key를 가진 새로운 `Scoped Prediction Window`(스코프 예측 윈도우)를 생성해야 합니다. 이를 클라이언트와 서버 간의 **동기화 지점(Synch Point)**이라고 부르기도 합니다. 모든 입력 관련 태스크처럼 일부 `AbilityTasks`에는 새로운 스코프 예측 윈도우를 생성하는 기능이 내장되어 있어 태스크 콜백의 원자적 코드에서 유효한 예측 키를 사용할 수 있습니다. 반면 `WaitDelay` 같은 태스크는 콜백용 새 스코프 예측 윈도우 생성 로직이 내장되어 있지 않습니다. `WaitDelay`처럼 지원되지 않는 태스크 이후에 액션을 예측해야 한다면 `OnlyServerWait` 옵션이 설정된 `WaitNetSync` `AbilityTask`를 사용하여 수동으로 처리해야 합니다.

클라이언트가 `OnlyServerWait` 설정의 `WaitNetSync`를 만나면, 어빌리티의 활성화 예측 키를 기반으로 새로운 Scoped Prediction Key를 생성하여 서버로 RPC를 전송하고 자신이 적용하는 새 `GameplayEffects`에 이를 부여합니다. 서버가 `OnlyServerWait` 설정의 `WaitNetSync`를 만나면 클라이언트로부터 새 Scoped Prediction Key를 수신할 때까지 실행을 일시 대기합니다. 이 새로운 예측 키는 활성화 예측 키와 동일한 과정(이펙트에 부여 -> 클라이언트로 복제 -> 만료 처리)을 거칩니다. Scoped Prediction Key는 스코프를 벗어날 때까지 유효합니다. 따라서 지연 작업이 아닌 단일 프레임의 원자적 연산만 새 예측 키를 사용할 수 있습니다.

필요에 따라 스코프 예측 윈도우를 얼마든지 생성할 수 있습니다.

자체 커스텀 `AbilityTasks`에 동기화 지점 기능을 추가하고 싶다면 입력 관련 태스크들이 내부적으로 `WaitNetSync` 코드를 어떻게 구현하고 있는지 참고하세요.

**참고:** `WaitNetSync`를 사용하면 클라이언트로부터 응답을 받을 때까지 서버의 `GameplayAbility` 실행이 블로킹(대기)됩니다. 악의적인 유저가 새 예측 키 전송을 의도적으로 지연시키는 방식으로 악용할 소지가 있습니다. Epic은 `WaitNetSync`를 매우 아껴서 사용하며, 보안이 중요하다면 클라이언트 응답이 없어도 일정 지연 시간 후 자동 진행되는 커스텀 태스크 작성을 권장합니다.

샘플 프로젝트는 질주(Sprint) `GameplayAbility`에서 스태미나 비용을 예측 적용할 수 있도록 매번 비용을 적용할 때마다 `WaitNetSync`를 사용하여 새로운 스코프 예측 윈도우를 생성합니다. 이상적으로 비용과 쿨다운을 적용할 때는 유효한 예측 키를 확보해야 합니다.

소유 클라이언트에서 예측된 `GameplayEffect`가 두 번 중복 실행되는 현상이 발생한다면, 예측 키가 만료되어 "재실행(Redo)" 문제가 발생한 것입니다. `GameplayEffect`를 적용하기 직전에 `OnlyServerWait` 설정의 `WaitNetSync` 태스크를 배치하여 새로운 예측 키를 발급받으면 대개 해결됩니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-p-spawn"></a>
#### 4.10.3 액터 예측 스폰 (Predictively Spawning Actors)
클라이언트에서 액터를 예측 스폰하는 것은 고급 주제입니다. GAS는 이를 처리하는 기능을 기본 제공하지 않습니다 (`SpawnActor` 태스크는 서버에서만 액터를 스폰함). 핵심 개념은 클라이언트와 서버 양쪽 모두에서 리플리케이트되는 `Actor`를 스폰하는 것입니다.

스폰되는 액터가 단순 시각 연출용이거나 게임플레이 기능을 갖지 않는다면, 간단한 해결책은 해당 액터의 `IsNetRelevantFor()` 함수를 오버라이드하여 서버 액터가 소유 클라이언트로 리플리케이트되는 것을 차단하는 것입니다. 소유 클라이언트는 로컬에서 스폰한 버전을 보고, 서버 및 다른 클라이언트는 서버가 리플리케이트한 버전을 봅니다.

```c++
bool APAReplicatedActorExceptOwner::IsNetRelevantFor(const AActor * RealViewer, const AActor * ViewTarget, const FVector & SrcLocation) const
{
	return !IsOwnedBy(ViewTarget);
}
```

스폰된 액터가 대미지를 예측해야 하는 투사체처럼 게임플레이에 직접적인 영향을 미친다면 본 문서의 범위를 벗어나는 고급 네트워크 동기화 로직이 필요합니다. Epic Games의 GitHub에 공개된 UnrealTournament의 발사체 예측 스폰 방식을 참고하세요. 소유 클라이언트에서만 스폰되는 더미(Dummy) 투사체를 두고 서버의 리플리케이트 투사체와 동기화시키는 방식을 사용합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-p-future"></a>
#### 4.10.4 GAS 예측의 미래
`GameplayPrediction.h`에는 향후 `GameplayEffect`의 제거 및 주기적 `GameplayEffects`에 대한 예측 기능을 추가할 수 있다고 언급되어 있습니다.

Epic의 Dave Ratti는 지연 시간이 높은 플레이어가 불리해지는 쿨다운 예측의 `지연 시간 조정(latency reconciliation)` 문제를 해결하는 데 관심을 표명했습니다.

Epic의 새로운 [`Network Prediction` 플러그인](#concepts-p-npp)은 기존의 `CharacterMovementComponent`처럼 GAS와 완벽하게 상호 운용될 것으로 기대됩니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-p-npp"></a>
#### 4.10.5 네트워크 예측 플러그인 (Network Prediction Plugin)
Epic은 기존 `CharacterMovementComponent`를 대체하기 위한 새로운 `Network Prediction` 플러그인 개발을 시작했습니다. 이 플러그인은 아직 초기 단계이지만 언리얼 엔진 GitHub에서 얼리 액세스로 확인할 수 있습니다. 향후 어느 엔진 버전에서 실험적 베타로 공식 도입될지는 미정입니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-targeting"></a>
### 4.11 타겟팅 (Targeting)

<a name="concepts-targeting-data"></a>
#### 4.11.1 타겟 데이터 (Target Data)
[`FGameplayAbilityTargetData`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/FGameplayAbilityTargetData/index.html)는 네트워크를 통해 전달되도록 설계된 타겟팅 데이터를 위한 범용 구조체입니다. `TargetData`는 일반적으로 `AActor`/`UObject` 참조, `FHitResults`, 그리고 위치/방향/원점 정보 등을 담습니다. 그러나 이를 상속(서브클래싱)하면 [`GameplayAbilities`에서 클라이언트와 서버 간에 데이터를 주고받기 위한](#concepts-ga-data) 임의의 커스텀 데이터를 무엇이든 담을 수 있습니다. 기본 구조체인 `FGameplayAbilityTargetData`는 직접 사용하기보다는 상속하여 사용하도록 설계되었습니다. GAS에는 `GameplayAbilityTargetTypes.h`에 몇 가지 서브클래스 구조체가 기본 제공됩니다.

`TargetData`는 일반적으로 [`Target Actors`](#concepts-targeting-actors)에 의해 생성되거나 **수동으로 생성**되며, [`EffectContext`](#concepts-ge-context)를 통해 [`AbilityTasks`](#concepts-at) 및 [`GameplayEffects`](#concepts-ge)에서 사용됩니다. `EffectContext`에 포함됨으로써 [`Executions`](#concepts-ge-ec), [`MMCs`](#concepts-ge-mmc), [`GameplayCues`](#concepts-gc), 그리고 [`AttributeSet`](#concepts-as)의 백엔드 함수들이 `TargetData`에 자유롭게 접근할 수 있습니다.

우리는 보통 `FGameplayAbilityTargetData`를 직접 전달하지 않고, 내부에 `FGameplayAbilityTargetData` 포인터들의 TArray를 보유한 [`FGameplayAbilityTargetDataHandle`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/FGameplayAbilityTargetDataHandle/index.html)을 사용합니다. 이 중간 래퍼 구조체는 `TargetData`의 다형성(Polymorphism)을 지원합니다.

`FGameplayAbilityTargetData` 상속 예제:
```c++
USTRUCT(BlueprintType)
struct MYGAME_API FGameplayAbilityTargetData_CustomData : public FGameplayAbilityTargetData
{
    GENERATED_BODY()
public:

    FGameplayAbilityTargetData_CustomData()
    { }

    UPROPERTY()
    FName CoolName = NAME_None;

    UPROPERTY()
    FPredictionKey MyCoolPredictionKey;

    // FGameplayAbilityTargetData의 모든 자식 구조체에 필수 구현
    virtual UScriptStruct* GetScriptStruct() const override
    {
    	return FGameplayAbilityTargetData_CustomData::StaticStruct();
    }

	// FGameplayAbilityTargetData의 모든 자식 구조체에 필수 구현
    bool NetSerialize(FArchive& Ar, class UPackageMap* Map, bool& bOutSuccess)
    {
	    // 엔진이 FName 및 FPredictionKey에 대한 NetSerialize를 이미 구현해 두었습니다.
        CoolName.NetSerialize(Ar, Map, bOutSuccess);
        MyCoolPredictionKey.NetSerialize(Ar, Map, bOutSuccess);
        bOutSuccess = true;
        return true;
    }
};

template<>
struct TStructOpsTypeTraits<FGameplayAbilityTargetData_CustomData> : public TStructOpsTypeTraitsBase2<FGameplayAbilityTargetData_CustomData>
{
	enum
	{
        WithNetSerializer = true // FGameplayAbilityTargetDataHandle 넷 직렬화가 동작하기 위해 필수적입니다.
	};
};
```

타겟 데이터를 핸들에 추가하는 방법:
```c++
UFUNCTION(BlueprintPure)
FGameplayAbilityTargetDataHandle MakeTargetDataFromCustomName(const FName CustomName)
{
	// 타겟 데이터 생성
	// 핸들이 소멸될 때 자동으로 메모리를 정리하고 삭제합니다.
	// 핸들에 추가하지 않으면 메모리 누수가 발생할 수 있으므로 같은 프레임 내에서 항상 핸들에 추가하는 것이 안전합니다!
	FGameplayAbilityTargetData_CustomData* MyCustomData = new FGameplayAbilityTargetData_CustomData();
	MyCustomData->CoolName = CustomName;
	
	// 블루프린트용 핸들 래퍼 생성
	FGameplayAbilityTargetDataHandle Handle;
	// 핸들에 타겟 데이터 추가
	Handle.Add(MyCustomData);
	return Handle;
}
```

핸들의 타겟 데이터에서 값을 꺼내올 때는 타입 안전성 검사(Type Safety Checking)가 필요합니다. 핸들로부터 타겟 데이터를 꺼내는 유일한 방법이 일반 C/C++ 캐스팅인데, 이는 타입 안전하지 않아 객체 슬라이싱(Object Slicing)이나 크래시를 유발할 수 있기 때문입니다. 타입 검사에는 주로 두 가지 방법이 사용됩니다:
- 게임플레이 태그(Gameplay Tags): 기본 부모 타입으로 캐스팅하여 태그를 확인한 뒤 일치하는 파생 클래스로 캐스팅하는 방식.
- 스크립트 구조체 & 정적 구조체 비교: `FGameplayAbilityTargetData`의 `GetScriptStruct()`를 호출하여 대상 구조체의 `StaticStruct()`와 직접 비교하는 방식 (모든 자식 구조체가 `GetScriptStruct`를 의무 구현하므로 가능한 강력한 이점).

타입 검사 함수 예제:
```c++
UFUNCTION(BlueprintPure)
FName GetCoolNameFromTargetData(const FGameplayAbilityTargetDataHandle& Handle, const int Index)
{   
    // 참고: '::Get(int32 Index)' 함수에는 두 가지 버전이 있습니다:
    // 1) 'const FGameplayAbilityTargetData*'를 반환하는 const 버전 (데이터 읽기용)
    // 2) 'FGameplayAbilityTargetData*'를 반환하는 non-const 버전 (데이터 수정용)
    FGameplayAbilityTargetData* Data = Handle.Get(Index); // 인덱스 유효성 검사를 자동으로 수행합니다.
    
    if(Data == nullptr)
    {
       	return NAME_None;
    }
    
    // 타입 안전성 검사 단계입니다. static_cast는 타입 검사를 하지 않으므로 이 확인이 필수적입니다.
    if(Data->GetScriptStruct() == FGameplayAbilityTargetData_CustomData::StaticStruct())
    {
        // 올바른 타입임을 확인했으므로 안전하게 캐스팅합니다.
        FGameplayAbilityTargetData_CustomData* CustomData = static_cast<FGameplayAbilityTargetData_CustomData*>(Data);    
        return CustomData->CoolName;
    }
    return NAME_None;
}
```

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-targeting-actors"></a>
#### 4.11.2 타겟 액터 (Target Actors)
`GameplayAbilities`는 월드로부터 타겟팅 정보를 시각화하고 캡처하기 위해 `WaitTargetData` `AbilityTask`를 사용하여 [`TargetActors`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/AGameplayAbilityTargetActor/index.html)를 스폰합니다. `TargetActors`는 현재 타겟 대상을 시각적으로 표시하기 위해 선택적으로 [`GameplayAbilityWorldReticles`](#concepts-targeting-reticles)를 사용할 수 있습니다. 타겟팅이 확인(Confirm)되면 정보가 [`TargetData`](#concepts-targeting-data)로 반환되어 `GameplayEffects`로 전달될 수 있습니다.
 
`TargetActors`는 `AActor` 기반이므로 스태틱 메시나 데칼 등 **어디를** **어떻게** 조준하고 있는지 시각화하기 위한 모든 종류의 렌더링 컴포넌트를 가질 수 있습니다. 스태틱 메시는 캐릭터가 건설할 구조물의 배치 위치를 미리 보여주는 데 사용할 수 있습니다. 데칼은 지면에 광역 효과(AoE) 범위를 표시하는 데 사용할 수 있습니다. 샘플 프로젝트는 Meteor(운석) 어빌리티의 피해 범위를 지면에 데칼로 표시하기 위해 [`AGameplayAbilityTargetActor_GroundTrace`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/AGameplayAbilityTargetActor_Grou-/index.html)를 사용합니다. [GASShooter](https://github.com/tranek/GASShooter)의 히트스캔 총기처럼 즉시 라인 트레이스하는 경우 시각적 요소를 전혀 표시하지 않을 수도 있습니다.

이들은 기본 라인 트레이스나 콜리전 오버랩을 사용하여 타겟팅 정보를 캡처하고, `TargetActor` 구현에 따라 그 결과를 `FHitResults` 또는 `AActor` 배열 형태의 `TargetData`로 변환합니다. `WaitTargetData` `AbilityTask`는 `TEnumAsByte<EGameplayTargetingConfirmation::Type> ConfirmationType` 매개변수를 통해 타겟팅 확인 시점을 제어합니다. `Instant` 확인 모드가 **아닌** 경우, `TargetActor`는 일반적으로 `Tick()`에서 트레이스/오버랩을 수행하고 구현에 따라 위치를 `FHitResult`로 업데이트합니다. `Tick()`에서 트레이스를 수행하지만 리플리케이트되지 않으며 동시에 여러 개가 실행되지 않으므로 일반적으로 큰 부담은 아닙니다. 그러나 복잡한 계산을 수행하는 경우 성능에 따라 틱 레이트를 낮추는 것을 고려할 수 있습니다. `Instant` 확인 모드의 경우 `TargetActor`는 즉시 스폰되어 `TargetData`를 생성하고 곧바로 파괴되며 `Tick()`은 호출되지 않습니다.

| `EGameplayTargetingConfirmation::Type` | 타겟 확인 시점                                                                                                                                                                                                                                                                          |
| -------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Instant`                              | 별도의 추가 입력 대기 없이 즉시 타겟팅이 발생합니다.                                                                                                                                                                                                                                   |
| `UserConfirmed`                        | 플레이어가 [어빌리티에 바인딩된 `Confirm` 입력](#concepts-ga-input)을 누르거나 `UAbilitySystemComponent::TargetConfirm()`을 호출하여 타겟팅을 확정할 때 발생합니다. 바인딩된 `Cancel` 입력을 누르거나 `UAbilitySystemComponent::TargetCancel()`을 호출하면 타겟팅이 취소됩니다. |
| `Custom`                               | 어빌리티가 `UGameplayAbility::ConfirmTaskByInstanceName()`을 호출하여 타겟 데이터 준비 완료 시점을 직접 결정합니다. 취소는 `UGameplayAbility::CancelTaskByInstanceName()`을 호출합니다.                                                                                              |
| `CustomMulti`                          | 어빌리티가 `UGameplayAbility::ConfirmTaskByInstanceName()`을 호출하여 타겟팅을 확정합니다. 데이터가 생성되어도 `AbilityTask`가 종료되지 않고 계속 유지됩니다.                                                                                                                           |

모든 `TargetActor`가 모든 확인 유형을 지원하는 것은 아닙니다. 예를 들어 `AGameplayAbilityTargetActor_GroundTrace`는 `Instant` 확인을 지원하지 않습니다.

`WaitTargetData` 태스크는 매 활성화마다 새로운 `TargetActor` 인스턴스를 스폰하고 태스크 종료 시 파괴합니다. `WaitTargetDataUsingActor` 태스크는 이미 스폰된 액터를 전달받지만 여전히 태스크 종료 시 액터를 파괴합니다. 자동소총처럼 타겟 데이터를 연속으로 쉼 없이 생성해야 하는 환경에서는 매번 액터를 스폰/파괴하는 것이 비효율적일 수 있습니다. GASShooter는 액터를 파괴하지 않고 재사용할 수 있는 커스텀 [`AGameplayAbilityTargetActor`](https://github.com/tranek/GASShooter/blob/master/Source/GASShooter/Public/Characters/Abilities/GSGATA_Trace.h) 및 새로운 [`WaitTargetDataWithReusableActor`](https://github.com/tranek/GASShooter/blob/master/Source/GASShooter/Public/Characters/Abilities/AbilityTasks/GSAT_WaitTargetDataUsingActor.h) 태스크를 구현하여 최적화했습니다.

`TargetActors`는 기본적으로 리플리케이트되지 않지만, 다른 플레이어에게 조준 지점을 보여주어야 한다면 리플리케이트되도록 설정할 수 있습니다. `WaitTargetData` 태스크에는 서버와 RPC로 통신하는 기능이 내장되어 있습니다. `ShouldProduceTargetDataOnServer` 속성이 `false`이면, 클라이언트는 타겟 확인 시 `UAbilityTask_WaitTargetData::OnTargetDataReadyCallback()`에서 `CallServerSetReplicatedTargetData()`를 통해 서버로 `TargetData`를 RPC 전송합니다. `ShouldProduceTargetDataOnServer`가 `true`이면, 클라이언트는 `EAbilityGenericReplicatedEvent::GenericConfirm` 확인 이벤트만 RPC로 전송하고 서버가 RPC를 수신했을 때 직접 트레이스/오버랩을 수행하여 서버에서 타겟 데이터를 생성합니다. 클라이언트가 타겟팅을 취소하면 `EAbilityGenericReplicatedEvent::GenericCancel` RPC를 서버로 보냅니다. 클라이언트가 서버로 `TargetData`를 보낼 때는 치트 방지를 위해 서버에서 데이터 검증을 수행하는 것이 좋습니다. 서버에서 직접 데이터를 생성하면 이 문제는 해결되지만 클라이언트 측에서 오예측이 발생할 가능성이 있습니다.

사용하는 `AGameplayAbilityTargetActor` 서브클래스에 따라 노드에 노출되는 주요 매개변수:

| 주요 `TargetActor` 매개변수 | 설명                                                                                                                                                                                                               |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Debug                       | `true`인 경우 논-쉬핑(Non-Shipping) 빌드에서 트레이스/오버랩 시 디버그 정보를 화면에 그립니다. Non-Instant 모드에서는 매 틱마다 디버그 그리기가 호출됩니다.                                                       |
| Filter                      | [선택] 트레이스/오버랩 시 타겟 대상에서 특정 액터를 제외(필터링)하는 특수 구조체입니다. 플레이어 자신의 폰을 제외하거나 특정 클래스만 타겟팅하도록 제한할 때 사용합니다 ([타겟 데이터 필터](#concepts-target-data-filters) 참조). |
| Reticle Class               | [선택] `TargetActor`가 스폰할 `AGameplayAbilityWorldReticle`의 서브클래스입니다.                                                                                                                                   |
| Reticle Parameters          | [선택] 레티클 설정 구조체입니다 ([월드 레티클](#concepts-targeting-reticles) 참조).                                                                                                                               |
| Start Location              | 트레이스가 시작될 위치를 정의하는 특수 구조체입니다. 주로 플레이어 시점, 총구 위치, 폰의 위치 등이 사용됩니다.                                                                                                    |

기본 `TargetActor` 클래스에서 액터는 트레이스/오버랩 범위 내에 직접 들어와 있을 때만 유효한 타겟이 됩니다. 조준을 돌리면 타겟에서 제외됩니다. 마지막 유효 타겟을 계속 기억하고 유지(Lock-on)하고 싶다면 커스텀 `TargetActor`에 영속 타겟(Persistent Targets) 기능을 추가해야 합니다. GASShooter는 유도 로켓 조준을 위해 이 방식을 사용합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-target-data-filters"></a>
#### 4.11.3 타겟 데이터 필터 (Target Data Filters)
`Make GameplayTargetDataFilter`와 `Make Filter Handle` 노드를 사용하여 플레이어의 폰을 제외하거나 특정 클래스만 필터링할 수 있습니다. 더 복잡한 고급 필터링이 필요하다면 `FGameplayTargetDataFilter`를 상속받아 `FilterPassesForActor` 함수를 오버라이드합니다.

```c++
USTRUCT(BlueprintType)
struct GASDOCUMENTATION_API FGDNameTargetDataFilter : public FGameplayTargetDataFilter
{
	GENERATED_BODY()

	/** 액터가 필터를 통과하여 타겟팅 대상이 될 경우 true 반환 */
	virtual bool FilterPassesForActor(const AActor* ActorToBeFiltered) const override;
};
```

그러나 이 구조체는 `FGameplayTargetDataFilterHandle`을 요구하는 `Wait Target Data` 노드에 바로 연결되지 않습니다. 서브클래스를 수용할 수 있는 커스텀 `Make Filter Handle` 라이브러리 함수를 만들어야 합니다:

```c++
FGameplayTargetDataFilterHandle UGDTargetDataFilterBlueprintLibrary::MakeGDNameFilterHandle(FGDNameTargetDataFilter Filter, AActor* FilterActor)
{
	FGameplayTargetDataFilter* NewFilter = new FGDNameTargetDataFilter(Filter);
	NewFilter->InitializeFilterContext(FilterActor);

	FGameplayTargetDataFilterHandle FilterHandle;
	FilterHandle.Filter = TSharedPtr<FGameplayTargetDataFilter>(NewFilter);
	return FilterHandle;
}
```

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-targeting-reticles"></a>
#### 4.11.4 게임플레이 어빌리티 월드 레티클 (World Reticles)
[`AGameplayAbilityWorldReticles`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/AGameplayAbilityWorldReticle/index.html) (`Reticles`)은 Non-Instant 방식으로 타겟팅할 때 **누구를** 조준하고 있는지를 시각적으로 표시해 줍니다. `TargetActors`가 레티클의 스폰 및 소멸 수명 주기를 관리합니다. `Reticles`는 `AActor`이므로 모든 시각 컴포넌트를 사용할 수 있으며, [GASShooter](https://github.com/tranek/GASShooter)처럼 스크린 공간 UMG 위젯을 표시하는 `WidgetComponent`를 사용하는 것이 보편적입니다. `TargetActors`는 대개 매 `Tick()`마다 레티클의 위치를 타겟의 위치로 업데이트합니다.

GASShooter는 유도 로켓 보조 어빌리티의 락온(Lock-on) 타겟을 표시하기 위해 레티클을 사용합니다. 적 위의 빨간색 표시가 `Reticle`입니다.
![GASShooter의 레티클](https://github.com/tranek/GASDocumentation/raw/master/Images/gameplayabilityworldreticle.png)

`Reticles`에는 디자이너가 블루프린트에서 구현할 수 있는 여러 이벤트들이 기본 제공됩니다:

```c++
/** bIsTargetValid 값이 변경될 때마다 호출 */
UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void OnValidTargetChanged(bool bNewValue);

/** bIsTargetAnActor 값이 변경될 때마다 호출 */
UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void OnTargetingAnActor(bool bNewValue);

UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void OnParametersInitialized();

UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void SetReticleMaterialParamFloat(FName ParamName, float value);

UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void SetReticleMaterialParamVector(FName ParamName, FVector value);
```

`Reticles`는 `TargetActor`가 제공하는 [`FWorldReticleParameters`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/FWorldReticleParameters/index.html)를 설정값으로 사용할 수 있습니다. 기본 구조체는 `FVector AOEScale` 변수 하나만 제공합니다. 커스텀 `TargetActor`를 작성할 경우 커스텀 파라미터 구조체를 정의하여 서브클래스 레티클에 전달할 수 있습니다.

`Reticles`는 기본적으로 리플리케이트되지 않지만, 다른 플레이어에게 내가 누구를 타겟팅하고 있는지 보여주고 싶다면 리플리케이트되도록 설정할 수 있습니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="concepts-targeting-containers"></a>
#### 4.11.5 게임플레이 이펙트 컨테이너 타겟팅
[`GameplayEffectContainers`](#concepts-ge-containers)는 [`TargetData`](#concepts-targeting-data)를 생성하는 효율적인 옵션을 기본 제공합니다. 이 타겟팅은 `EffectContainer`가 클라이언트와 서버에 적용될 때 즉시 일어납니다. 타겟팅 객체의 CDO 상에서 실행되므로 `Actors` 스폰/파괴가 없어 [`TargetActors`](#concepts-targeting-actors)보다 훨씬 효율적이지만, 플레이어 입력을 기다리거나 확인/취소할 수 없고 클라이언트에서 서버로 데이터를 전송할 수 없습니다 (양쪽에서 동시 생성). 즉시 라인 트레이스나 구체 오버랩에 매우 적합합니다. Epic의 [Action RPG 샘플 프로젝트](https://www.unrealengine.com/marketplace/en-US/product/action-rpg)는 컨테이너와 함께 어빌리티 소유자 타겟팅, 이벤트로부터 타겟 데이터 추출, 그리고 플레이어 전방 구체 트레이스 등을 구현했습니다. C++ 또는 블루프린트에서 `URPGTargetType`을 상속받아 커스텀 타겟팅 타입을 만들 수 있습니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="cae"></a>

## 5. 자주 구현되는 어빌리티 및 이펙트 예제 (Commonly Implemented Abilities and Effects)

<a name="cae-stun"></a>
### 5.1 기절 (Stun)
일반적으로 기절 효과가 걸리면 캐릭터의 활성화된 모든 `GameplayAbilities`를 취소하고, 새로운 `GameplayAbility` 활성화를 차단하며, 기절 지속 시간 동안 이동을 금지하고자 합니다. 샘플 프로젝트의 Meteor(운석) `GameplayAbility`는 적중 대상에게 기절을 적용합니다.

대상의 활성 어빌리티를 취소하려면 기절 [`GameplayTag`가 추가될 때](#concepts-gt-change) `AbilitySystemComponent->CancelAbilities()`를 호출합니다.

기절 상태 동안 새 어빌리티가 활성화되는 것을 막으려면 `GameplayAbilities`의 [`Activation Blocked Tags` `GameplayTagContainer`](#concepts-ga-tags)에 기절 `GameplayTag`를 등록합니다.

기절 상태 동안 이동을 방지하려면 `CharacterMovementComponent`의 `GetMaxSpeed()` 함수를 오버라이드하여 소유자가 기절 `GameplayTag`를 가지고 있을 때 0을 반환하도록 합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="cae-sprint"></a>
### 5.2 질주 (Sprint)
샘플 프로젝트는 `Left Shift` 키를 누르고 있는 동안 더 빠르게 달리는 질주 구현 예제를 제공합니다.

이동 속도 증가는 네트워크를 통해 서버로 플래그를 전송하여 `CharacterMovementComponent`에 의해 예측 처리됩니다. 자세한 내용은 `GDCharacterMovementComponent.h/cpp`를 참조하세요.

`GA`는 `Left Shift` 입력을 처리하고, `CMC`에 질주 시작과 정지를 지시하며, `Left Shift`가 눌려 있는 동안 스태미나를 예측 차감합니다. 자세한 내용은 `GA_Sprint_BP`를 참조하세요.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="cae-ads"></a>
### 5.3 정조준 (Aim Down Sights - ADS)
샘플 프로젝트는 질주와 완전히 동일한 방식으로 처리하되, 이동 속도를 증가시키는 대신 감소시킵니다.

이동 속도를 예측하여 감소시키는 세부 구현은 `GDCharacterMovementComponent.h/cpp`를 참조하세요.

입력 처리 세부사항은 `GA_AimDownSight_BP`를 참조하세요. 정조준에는 스태미나 비용이 소모되지 않습니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="cae-ls"></a>
### 5.4 생명력 흡수 (Lifesteal)
저는 대미지 [`ExecutionCalculation`](#concepts-ge-ec) 내부에서 생명력 흡수를 처리합니다. `GameplayEffect`는 `Effect.Damage.CanLifesteal`과 같은 `GameplayTag`를 가집니다. `ExecutionCalculation`은 `GameplayEffectSpec`이 해당 태그를 가지고 있는지 확인합니다. 태그가 존재하면 `ExecutionCalculation`은 회복할 체력 수치를 모디파이어로 갖는 [동적 즉시(Instant) `GameplayEffect`](#concepts-ge-dynamic)를 생성하여 공격자의 `Source` `ASC`에 다시 적용합니다.

```c++
if (SpecAssetTags.HasTag(FGameplayTag::RequestGameplayTag(FName("Effect.Damage.CanLifesteal"))))
{
	float Lifesteal = Damage * LifestealPercent;

	UGameplayEffect* GELifesteal = NewObject<UGameplayEffect>(GetTransientPackage(), FName(TEXT("Lifesteal")));
	GELifesteal->DurationPolicy = EGameplayEffectDurationType::Instant;

	int32 Idx = GELifesteal->Modifiers.Num();
	GELifesteal->Modifiers.SetNum(Idx + 1);
	FGameplayModifierInfo& Info = GELifesteal->Modifiers[Idx];
	Info.ModifierMagnitude = FScalableFloat(Lifesteal);
	Info.ModifierOp = EGameplayModOp::Additive;
	Info.Attribute = UPAAttributeSetBase::GetHealthAttribute();

	SourceAbilitySystemComponent->ApplyGameplayEffectToSelf(GELifesteal, 1.0f, SourceAbilitySystemComponent->MakeEffectContext());
}
```

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="cae-random"></a>
### 5.5 클라이언트와 서버에서 동일한 난수 생성
탄환 반동이나 탄퍼짐(Spread) 처리를 위해 `GameplayAbility` 내부에서 "랜덤(난수)" 값을 생성해야 할 때가 있습니다. 이때 클라이언트와 서버는 모두 동일한 난수를 생성해야 합니다. 이를 위해서는 `GameplayAbility` 활성화 시점에 양쪽의 `랜덤 시드(Random Seed)`를 동일하게 맞춰야 합니다. 클라이언트가 오예측하여 난수 시퀀스가 서버와 어긋날 경우를 대비해 매 활성화마다 난수 시드를 재설정하는 것이 좋습니다.

| 시드 설정 방법                                        | 설명                                                                                                                                                                                                                                                                                                              |
| ---------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 활성화 예측 키(Activation Prediction Key) 사용       | `GameplayAbility` 활성화 예측 키는 int16 정수이며 `Activation()` 시점에 클라이언트와 서버 양쪽 모두에서 동기화되어 사용 가능함이 보장됩니다. 이를 양쪽의 `랜덤 시드`로 사용할 수 있습니다. 단점은 게임이 시작될 때마다 예측 키가 항상 0부터 시작하여 동일하게 증가하므로 매 매치마다 정확히 동일한 난수 시퀀스가 나온다는 점입니다. |
| 어빌리티 활성화 시 이벤트 페이로드로 시드 전달       | 이벤트로 어빌리티를 활성화하고, 클라이언트가 생성한 랜덤 시드를 리플리케이트되는 이벤트 페이로드에 담아 서버로 전달합니다. 더 완전한 무작위성을 제공하지만, 클라이언트가 매번 동일한 시드값만 보내도록 핵(변조)을 쓰기 쉽습니다. 또한 입력 바인딩으로 어빌리티를 직접 활성화할 수 없게 됩니다.              |

난수 오차 범위가 작다면 플레이어들이 매 경기마다 시퀀스가 동일하다는 사실을 눈치채기 어려우므로 활성화 예측 키를 `랜덤 시드`로 사용하는 것만으로도 충분합니다. 보안이 매우 중요한 복잡한 시스템이라면 서버가 예측 키를 생성하거나 시드를 생성하여 이벤트 페이로드로 보내는 `Server Initiated` 어빌리티를 사용하는 것이 더 안전합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="cae-crit"></a>
### 5.6 치명타 (Critical Hits)
치명타 역시 대미지 [`ExecutionCalculation`](#concepts-ge-ec) 내부에서 처리합니다. `GameplayEffect`에 `Effect.CanCrit` 같은 태그를 두고, `ExecutionCalculation`이 이 태그를 검사합니다. 태그가 존재하면 공격자(`Source`)에서 캡처한 치명타 확률 `Attribute`에 따라 난수를 생성하고, 성공 시 공격자의 치명타 대미지 `Attribute`를 합산합니다. 대미지를 예측하지 않으므로 `ExecCalc`는 서버에서만 실행되며, 클라이언트와 서버의 난수 생성기를 동기화할 필요가 없습니다. 만약 `MMC`를 사용하여 예측적으로 대미지 계산을 수행하려 한다면 `GameplayEffectSpec->GameplayEffectContext->GameplayAbilityInstance`를 통해 `랜덤 시드` 참조를 가져와야 합니다.

[GASShooter](https://github.com/tranek/GASShooter)의 헤드샷 구현을 참고하세요. 난수 확률에 의존하지 않고 `FHitResult`의 본(Bone) 이름을 검사한다는 점을 제외하면 동일한 개념입니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="cae-nonstackingge"></a>
### 5.7 중첩되지 않지만 가장 큰 수치만 대상에 적용되는 게임플레이 이펙트
파라곤(Paragon)의 이동 속도 둔화(Slow) 효과는 중첩되지 않았습니다. 각 슬로우 이펙트는 평소처럼 적용되어 개별 지속 시간을 유지했지만, 그중 가장 수치가 큰(가장 강력한) 슬로우 효과 하나만 실제로 캐릭터에 영향을 주었습니다. GAS는 `AggregatorEvaluateMetaData`를 통해 이 기능을 기본 지원합니다. 세부 구현은 [`AggregatorEvaluateMetaData()`](#concepts-as-onattributeaggregatorcreated)를 참조하세요.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="cae-paused"></a>
### 5.8 게임 일시정지 중 타겟 데이터 생성
`WaitTargetData` `AbilityTask`로 플레이어의 [`TargetData`](#concepts-targeting-data) 입력을 기다리는 동안 게임을 일시정지(Pause)해야 한다면, 일반 일시정지 대신 콘솔 명령 `slomo 0`을 사용하는 것을 추천합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="cae-onebuttoninteractionsystem"></a>
### 5.9 단일 버튼 상호작용 시스템 (One Button Interaction System)
[GASShooter](https://github.com/tranek/GASShooter)는 플레이어가 'E' 키를 누르거나 길게 눌러 아군 부활, 무기 상자 열기, 미닫이문 개폐 등의 다양한 오브젝트와 상호작용할 수 있는 단일 버튼 상호작용 시스템을 구현하고 있습니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="debugging"></a>
## 6. GAS 디버깅 (Debugging GAS)
GAS 관련 문제를 디버깅할 때 주로 다음과 같은 내용들을 확인하고 싶어집니다:
> * "현재 내 어트리뷰트 값은 얼마인가?"
> * "현재 내가 어떤 게임플레이 태그들을 가지고 있는가?"
> * "현재 나에게 적용되어 있는 게임플레이 이펙트는 무엇인가?"
> * "나에게 어떤 어빌리티들이 부여되어 있고, 어떤 것이 실행 중이며, 어떤 것이 활성화 차단되어 있는가?"

GAS는 런타임에 이러한 질문에 답할 수 있는 두 가지 훌륭한 기능을 제공합니다 - [`showdebug abilitysystem`](#debugging-sd) 콘솔 명령과 [`GameplayDebugger`](#debugging-gd) 연동.

**팁:** 언리얼 엔진은 C++ 코드를 적극적으로 최적화하므로 깊은 코드 추적 시 디버깅이 어려울 수 있습니다. Visual Studio의 솔루션 구성을 `DebugGame Editor`로 설정해도 변수 검사나 코드 트레이싱이 건너뛰어지는 경우, 해당 함수를 `UE_DISABLE_OPTIMIZATION` 및 `UE_ENABLE_OPTIMIZATION` 매크로로 감싸서 모든 최적화를 임시 해제할 수 있습니다. (디버깅이 끝나면 매크로를 반드시 제거하세요!)

```c++
UE_DISABLE_OPTIMIZATION
void MyClass::MyFunction(int32 MyIntParameter)
{
	// 디버깅할 코드
}
UE_ENABLE_OPTIMIZATION
```

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="debugging-sd"></a>
### 6.1 showdebug abilitysystem
인게임 콘솔창에 `showdebug abilitysystem`을 입력합니다. 이 기능은 3개의 "페이지"로 나뉘어 있습니다. 3개 페이지 모두 현재 보유한 `GameplayTags`를 표시합니다. 페이지를 전환하려면 콘솔에 `AbilitySystem.Debug.NextCategory`를 입력합니다.

첫 번째 페이지는 모든 `Attributes`의 `CurrentValue`를 표시합니다:
![showdebug abilitysystem 1페이지](https://github.com/tranek/GASDocumentation/raw/master/Images/showdebugpage1.png)

두 번째 페이지는 현재 적용된 모든 `Duration` 및 `Infinite` `GameplayEffects`, 스택 수, 부여하는 `GameplayTags` 및 `Modifiers`를 표시합니다:
![showdebug abilitysystem 2페이지](https://github.com/tranek/GASDocumentation/raw/master/Images/showdebugpage2.png)

세 번째 페이지는 부여된 모든 `GameplayAbilities`, 현재 실행 중 여부, 활성화 차단 여부, 그리고 현재 실행 중인 `AbilityTasks`의 상태를 표시합니다:
![showdebug abilitysystem 3페이지](https://github.com/tranek/GASDocumentation/raw/master/Images/showdebugpage3.png)

디버깅 대상 액터(녹색 직육면체로 표시됨)를 변경하려면 `PageUp` 키(또는 `NextDebugTarget` 콘솔 명령)로 다음 대상, `PageDown` 키(또는 `PreviousDebugTarget` 콘솔 명령)로 이전 대상을 선택합니다.

**참고:** 선택된 디버그 액터에 따라 어빌리티 시스템 정보가 갱신되도록 하려면 `DefaultGame.ini`의 `AbilitySystemGlobals`에 다음과 같이 설정해야 합니다:
```
[/Script/GameplayAbilities.AbilitySystemGlobals]
bUseDebugTargetFromHud=true
```

**참고:** `showdebug abilitysystem`이 정상 작동하려면 GameMode에 실제 HUD 클래스가 지정되어 있어야 합니다. 그렇지 않으면 명령을 찾지 못하고 "Unknown Command" 오류를 반환합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="debugging-gd"></a>
### 6.2 게임플레이 디버거 (Gameplay Debugger)
GAS는 엔진의 게임플레이 디버거(Gameplay Debugger) 기능을 지원합니다. 키보드의 아포스트로피(`'`) 키를 눌러 게임플레이 디버거를 활성화합니다. 넘패드 `3`번을 눌러 Abilities 카테고리를 활성화합니다. 넘패드가 없는 노트북의 경우 프로젝트 설정에서 단축키를 변경할 수 있습니다.

게임플레이 디버거는 **다른** 캐릭터에 걸려 있는 `GameplayTags`, `GameplayEffects`, `GameplayAbilities`를 확인하고자 할 때 유용합니다. 화면 중앙에 있는 캐릭터를 자동으로 타겟팅하며, 에디터의 월드 아웃라이너에서 선택하거나 대상을 바라본 상태로 `'` 키를 다시 눌러 대상을 변경할 수 있습니다.

![Gameplay Debugger](https://github.com/tranek/GASDocumentation/raw/master/Images/gameplaydebugger.png)

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="debugging-log"></a>
### 6.3 GAS 로깅 (GAS Logging)
GAS 소스 코드에는 다양한 상세도(Verbosity) 수준으로 출력되는 수많은 로깅 구문(`ABILITY_LOG()` 등)이 포함되어 있습니다. 기본 상세도는 `Display` 수준이며, 그보다 상세한 로그는 기본적으로 콘솔에 출력되지 않습니다.

특정 로그 카테고리의 상세도를 변경하려면 콘솔에 다음과 같이 입력합니다:

```
log [카테고리] [상세도]
```

예를 들어 `ABILITY_LOG()` 구문을 모두 출력하려면 콘솔에 다음과 같이 입력합니다:
```
log LogAbilitySystem VeryVerbose
```

기본값으로 되돌리려면:
```
log LogAbilitySystem Display
```

모든 로그 카테고리 목록을 보려면:
```
log list
```

주요 GAS 관련 로그 카테고리:

| 로그 카테고리 (Logging Category) | 기본 상세도 수준 (Default Verbosity) |
| -------------------------------- | ----------------------------------- |
| LogAbilitySystem                 | Display                             |
| LogAbilitySystemComponent        | Log                                 |
| LogGameplayCueDetails            | Log                                 |
| LogGameplayCueTranslator         | Display                             |
| LogGameplayEffectDetails         | Log                                 |
| LogGameplayEffects               | Display                             |
| LogGameplayTags                  | Log                                 |
| LogGameplayTasks                 | Log                                 |
| VLogAbilitySystem                | Display                             |

자세한 내용은 [언리얼 로깅 관련 Wiki](https://unrealcommunity.wiki/logging-lgpidy6i)를 참조하세요.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="optimizations"></a>
## 7. 최적화 (Optimizations)

<a name="optimizations-abilitybatching"></a>
### 7.1 어빌리티 배칭 (Ability Batching)
단일 프레임 내에서 활성화되고, 선택적으로 서버에 `TargetData`를 전송하며, 즉시 종료되는 [`GameplayAbilities`](#concepts-ga)는 [2~3개의 RPC를 1개로 압축하도록 배칭](#concepts-ga-batching)할 수 있습니다. 히트스캔 총기류에 널리 사용됩니다.

<a name="optimizations-gameplaycuebatching"></a>
### 7.2 게임플레이 큐 배칭 (Gameplay Cue Batching)
많은 수의 [`GameplayCues`](#concepts-gc)를 동시에 전송해야 한다면 [하나의 RPC로 배칭](#concepts-gc-batching)하는 것을 고려하세요. 목표는 RPC 전송 횟수를 줄이고 대역폭 전송량을 최소화하는 것입니다.

<a name="optimizations-ascreplicationmode"></a>
### 7.3 어빌리티 시스템 컴포넌트 리플리케이션 모드
기본적으로 [`ASC`](#concepts-asc)는 [`Full Replication Mode`](#concepts-asc-rm)로 설정되어 있습니다. 이는 모든 [`GameplayEffects`](#concepts-ge)를 모든 클라이언트에 복제합니다 (싱글 플레이어에서는 무방함). 멀티플레이어 게임에서는 플레이어 소유 ASC는 `Mixed Replication Mode`로, AI 조종 캐릭터는 `Minimal Replication Mode`로 설정해야 합니다. 이렇게 하면 플레이어 캐릭터에 걸린 GE는 해당 소유 클라이언트에게만 복제되고, AI 캐릭터에 걸린 GE는 클라이언트로 전혀 복제되지 않아 대역폭을 크게 절약할 수 있습니다. [`GameplayTags`](#concepts-gt)와 [`GameplayCues`](#concepts-gc)는 모드와 관계없이 정상 복제됩니다.

<a name="optimizations-attributeproxyreplication"></a>
### 7.4 어트리뷰트 프록시 리플리케이션 (Attribute Proxy Replication)
포트나이트 배틀로얄(FNBR)처럼 플레이어가 매우 많은 대규모 게임에서는 상시 리플리케이션되는(Always-relevant) `PlayerState` 상에 수많은 [`ASCs`](#concepts-asc)가 존재하여 수많은 [`Attributes`](#concepts-a)를 복제하게 됩니다. 이 병목을 해결하기 위해 포트나이트는 `PlayerState::ReplicateSubobjects()`에서 **시뮬레이트된 플레이어 조종 프록시**에 대해 ASC 및 [`AttributeSets`](#concepts-as)의 복제를 아예 비활성화했습니다. (자율 프록시와 AI 폰은 정상 복제됨). 대신 플레이어의 `Pawn`에 리플리케이트되는 프록시 구조체를 두고 어트리뷰트 변경을 전달받아 로컬 ASC로 밀어 넣는 방식을 사용했습니다. 이를 통해 어트리뷰트 복제가 `Pawn`의 가시거리(Relevancy)와 `NetUpdateFrequency`를 활용할 수 있게 됩니다.

<a name="optimizations-asclazyloading"></a>
### 7.5 ASC 지연 로딩 (ASC Lazy Loading)
포트나이트 배틀로얄은 월드에 수많은 파괴 가능 액터(나무, 건물 등)가 존재하며 각각 `ASC`를 가지고 있습니다. 이는 메모리 부담이 될 수 있습니다. FNBR은 액터가 플레이어에게 처음 피격당할 때 `ASC`를 동적으로 지연 생성(Lazy Loading)함으로써 매치 동안 피해를 입지 않은 오브젝트들의 메모리 낭비를 줄였습니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="qol"></a>
## 8. 개발 편의성(QoL) 제안 (Quality of Life Suggestions)

<a name="qol-gameplayeffectcontainers"></a>
### 8.1 게임플레이 이펙트 컨테이너 (Gameplay Effect Containers)
[GameplayEffectContainers](#concepts-ge-containers)는 [`GameplayEffectSpecs`](#concepts-ge-spec), [`TargetData`](#concepts-targeting-data), [간단한 타겟팅](#concepts-targeting-containers) 등을 사용하기 쉬운 단일 구조체로 묶어줍니다. 어빌리티에서 스폰한 투사체에 스펙을 넘겨주고 나중에 충돌 시 적용하도록 구성할 때 매우 편리합니다.

<a name="qol-asynctasksascdelegates"></a>
### 8.2 ASC 델리게이트에 바인딩하는 블루프린트 AsyncTask
UMG 위젯 UI를 제작할 때 기획자/디자이너 친화적인 작업 속도를 위해, UMG 블루프린트 그래프에서 `ASC`의 변경 델리게이트에 직접 바인딩할 수 있는 C++ 기반 Blueprint AsyncTask 노드를 제작하세요. 유일한 주의점은 위젯이 파괴될 때 `EndTask()`를 호출하여 수동으로 정리해주어야 메모리 누수를 방지할 수 있다는 점입니다. 샘플 프로젝트에 세 가지 예제가 포함되어 있습니다.

어트리뷰트 변경 감지:
![어트리뷰트 변경 감지 BP 노드](https://github.com/tranek/GASDocumentation/raw/master/Images/attributeschange.png)

쿨다운 변경 감지:
![쿨다운 변경 감지 BP 노드](https://github.com/tranek/GASDocumentation/raw/master/Images/cooldownchange.png)

GE 스택 변경 감지:
![GE 스택 변경 감지 BP 노드](https://github.com/tranek/GASDocumentation/raw/master/Images/gestackchange.png)

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="troubleshooting"></a>
## 9. 문제 해결 (Troubleshooting)

<a name="troubleshooting-notlocal"></a>
### 9.1 `LogAbilitySystem: Warning: Can't activate LocalOnly or LocalPredicted ability %s when not local!`
[클라이언트에서 ASC를 초기화](#concepts-asc-setup)해야 합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="troubleshooting-scriptstructcache"></a>
### 9.2 `ScriptStructCache` 오류
[`UAbilitySystemGlobals::InitGlobalData()`](#concepts-asg-initglobaldata)를 호출해야 합니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="troubleshooting-replicatinganimmontages"></a>
### 9.3 애니메이션 몽타주가 클라이언트에 리플리케이트되지 않음
[GameplayAbilities](#concepts-ga)에서 `PlayMontage` 노드 대신 `PlayMontageAndWait` 블루프린트 노드를 사용하고 있는지 확인하세요. 이 [AbilityTask](#concepts-at)는 `ASC`를 통해 몽타주를 자동 복제하지만 `PlayMontage` 노드는 복제하지 않습니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="troubleshooting-duplicatingblueprintactors"></a>
### 9.4 블루프린트 액터 복제 시 AttributeSet이 nullptr로 설정됨
언리얼 엔진에는 기존 블루프린트 액터 클래스를 복제(Duplicate)하여 새 클래스를 만들었을 때 클래스의 `AttributeSet` 포인터가 nullptr로 설정되는 [엔진 버그](https://issues.unrealengine.com/issue/UE-81109)가 있습니다. 몇 가지 우회 방법이 있습니다. 클래스에 전용 `AttributeSet` 포인터를 직접 두지 않고(헤더에 포인터 선언 및 생성자 `CreateDefaultSubobject` 호출 생략), `PostInitializeComponents()`에서 `ASC`에 `AttributeSets`를 직접 추가하는 방법으로 해결할 수 있습니다. 리플리케이트된 `AttributeSets`는 여전히 `ASC`의 `SpawnedAttributes` 배열에 보관됩니다:

```c++
void AGDPlayerState::PostInitializeComponents()
{
	Super::PostInitializeComponents();

	if (AbilitySystemComponent)
	{
		AbilitySystemComponent->AddSet<UGDAttributeSetBase>();
		// ... 기타 보유한 다른 AttributeSets
	}
}
```

이 경우 [매크로로 만든 `AttributeSet`의 함수를 호출](#concepts-as-attributes)하는 대신 `ASC`의 함수를 통해 값을 읽고 씁니다.

```c++
/** 어트리뷰트의 현재(최종) 값을 반환 */
float GetNumericAttribute(const FGameplayAttribute &Attribute) const;

/** 어트리뷰트의 기본값을 설정 */
void SetNumericAttributeBase(const FGameplayAttribute &Attribute, float NewBaseValue);
```

`GetHealth()` 구현 예시:
```c++
float AGDPlayerState::GetHealth() const
{
	if (AbilitySystemComponent)
	{
		return AbilitySystemComponent->GetNumericAttribute(UGDAttributeSetBase::GetHealthAttribute());
	}

	return 0.0f;
}
```

체력 어트리뷰트 초기화 예시:
```c++
const float NewHealth = 100.0f;
if (AbilitySystemComponent)
{
	AbilitySystemComponent->SetNumericAttributeBase(UGDAttributeSetBase::GetHealthAttribute(), NewHealth);
}
```

`ASC`는 `AttributeSet` 클래스당 최대 하나의 인스턴스만 수용한다는 점을 기억하세요.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="troubleshooting-unresolvedexternalsymbolmarkpropertydirty"></a>
### 9.5 미해결 외부 기호 UEPushModelPrivate::MarkPropertyDirty(int,int)
다음과 같은 컴파일 링커 오류가 발생하는 경우:

```
error LNK2019: unresolved external symbol "__declspec(dllimport) void __cdecl UEPushModelPrivate::MarkPropertyDirty(int,int)" (__imp_?MarkPropertyDirty@UEPushModelPrivate@@YAXHH@Z) referenced in function "public: void __cdecl FFastArraySerializer::IncrementArrayReplicationKey(void)" (?IncrementArrayReplicationKey@FFastArraySerializer@@QEAAXXZ)
```

이는 `FFastArraySerializer`에서 `MarkItemDirty()`를 호출할 때(예: 쿨다운 지속 시간 업데이트 등) 발생합니다.

```c++
ActiveGameplayEffects.MarkItemDirty(*AGE);
```

원인은 `WITH_PUSH_MODEL`이 여러 위치에서 상충되게 정의되는 현상 때문입니다. `PushModelMacros.h`에서는 0으로 정의되는 반면 다른 여러 곳에서는 1로 정의됩니다. `PushModel.h`에서는 1로 인식하지만 `PushModel.cpp`에서는 0으로 인식하게 됩니다.

해결 방법은 프로젝트의 `Build.cs`에서 `PublicDependencyModuleNames`에 `"NetCore"`를 추가하는 것입니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="troubleshooting-enumnamesarenowpathnames"></a>
### 9.6 열거형 이름이 경로 이름으로 표현되는 문제
다음과 같은 컴파일러 경고가 발생하는 경우:

```
warning C4996: 'FGameplayAbilityInputBinds::FGameplayAbilityInputBinds': Enum names are now represented by path names. Please use a version of FGameplayAbilityInputBinds constructor that accepts FTopLevelAssetPath. Please update your code to the new API before upgrading to the next release, otherwise your project will no longer compile.
```

UE 5.1부터 `BindAbilityActivationToInputComponent()` 생성자에서 `FString`을 사용하는 방식이 Deprecated 되었습니다. 대신 `FTopLevelAssetPath`를 전달해야 합니다.

이전 방식:
```c++
AbilitySystemComponent->BindAbilityActivationToInputComponent(InputComponent, FGameplayAbilityInputBinds(FString("ConfirmTarget"),
	FString("CancelTarget"), FString("EGDAbilityInputID"), static_cast<int32>(EGDAbilityInputID::Confirm), static_cast<int32>(EGDAbilityInputID::Cancel)));
```

신규 방식:
```c++
FTopLevelAssetPath AbilityEnumAssetPath = FTopLevelAssetPath(FName("/Script/GASDocumentation"), FName("EGDAbilityInputID"));
AbilitySystemComponent->BindAbilityActivationToInputComponent(InputComponent, FGameplayAbilityInputBinds(FString("ConfirmTarget"),
	FString("CancelTarget"), AbilityEnumAssetPath, static_cast<int32>(EGDAbilityInputID::Confirm), static_cast<int32>(EGDAbilityInputID::Cancel)));
```

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="acronyms"></a>
## 10. 자주 사용되는 GAS 약어 모음 (Common GAS Acronyms)

| 명칭 (Name)                                                                                            | 약어 (Acronyms)     |
| ------------------------------------------------------------------------------------------------------ | ------------------- |
| AbilitySystemComponent                                                                                 | ASC                 |
| AbilityTask                                                                                            | AT                  |
| [Epic의 Action RPG 샘플 프로젝트](https://www.unrealengine.com/marketplace/en-US/product/action-rpg)  | ARPG, ARPG Sample   |
| CharacterMovementComponent                                                                             | CMC                 |
| GameplayAbility                                                                                        | GA                  |
| GameplayAbilitySystem                                                                                  | GAS                 |
| GameplayCue                                                                                            | GC                  |
| GameplayEffect                                                                                         | GE                  |
| GameplayEffectExecutionCalculation                                                                     | ExecCalc, Execution |
| GameplayTag                                                                                            | Tag, GT             |
| ModifierMagnitudeCalculation                                                                           | ModMagCalc, MMC     |

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="resources"></a>

## 11. 기타 참고 자료 (Other Resources)
* [에픽게임즈 공식 문서](https://docs.unrealengine.com/en-US/Gameplay/GameplayAbilitySystem/index.html)
* 엔진 소스 코드!
   * 특히 `GameplayPrediction.h`
* [에픽게임즈 Lyra 샘플 프로젝트](https://unrealengine.com/marketplace/en-US/learn/lyra)
* [에픽게임즈 Action RPG 샘플 프로젝트](https://www.unrealengine.com/marketplace/en-US/product/action-rpg)
* [Unreal Slackers 디스코드](https://unrealslackers.org/)의 `#gameplay-ability-system` 전용 채널
   * 고정(Pinned) 메시지 확인 필수
* [Dan 'Pan'의 GAS 리소스 GitHub 저장소](https://github.com/Pantong51/GASContent)
* [SabreDartStudios의 YouTube 튜토리얼 영상](https://www.youtube.com/channel/UCCFUhQ6xQyjXDZ_d6X_H_-A)

<a name="resources-daveratti"></a>
### 11.1 에픽게임즈 Dave Ratti와의 Q&A

<a name="resources-daveratti-community1"></a>
#### 11.1.1 커뮤니티 질문 세션 1
[Unreal Slackers 디스코드 서버 커뮤니티의 GAS 관련 질문에 대한 Dave Ratti의 답변](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89):

1. **`GameplayAbilities` 외부에서 또는 어빌리티와 무관하게 필요할 때 즉시(On-demand) 스코프 예측 윈도우를 생성할 수 있는 방법이 있나요? 예를 들어, 발사 후 망각(Fire-and-forget) 방식의 투사체가 적에게 적중했을 때 대미지 `GameplayEffect`를 로컬에서 예측하려면 어떻게 해야 하나요?**

> `PredictionKey` 시스템은 본래 이러한 목적으로 설계된 것이 아닙니다. 근본적으로 이 시스템은 클라이언트가 예측 액션을 시작하고, 키를 통해 서버에 이를 알린 다음, 클라이언트와 서버 양쪽이 동일한 로직을 실행하면서 해당 예측 키와 관련된 예측 부작용(Side Effects)을 연결하는 방식으로 작동합니다. 예를 들어 "내가 지금 어빌리티를 예측 활성화하고 있다"라거나 "내가 타겟 데이터를 생성했으므로 WaitTargetData 태스크 이후의 어빌리티 그래프 부분을 예측 실행하겠다" 같은 식입니다.
>
> 이 패턴에서 `PredictionKey`는 서버를 한 번 "왕복(Bounce)"하여 `UAbilitySystemComponent::ReplicatedPredictionKeyMap`(리플리케이트되는 프로퍼티)을 통해 클라이언트로 돌아옵니다. 키가 서버에서 다시 복제되어 돌아오면, 클라이언트는 로컬에서 예측했던 모든 부작용(`GameplayCues`, `GameplayEffects`)을 취소/정리할 수 있습니다. 서버에서 복제된 진짜 버전이 *도착해 있을 것*이며, 도착하지 않았다면 오예측(Misprediction)이었던 것입니다. 여기서 예측 부작용을 언제 되돌릴지 정확한 시점을 아는 것이 결정적입니다. 너무 빠르면 화면에 공백(Gaps)이 생기고, 너무 늦으면 효과가 중복(Double)되어 나타납니다. (참고로 이는 지속 시간 기반 GameplayEffect나 루핑 GameplayCue처럼 상태를 가지는 예측에 해당합니다. 즉발성 "버스트" GameplayCue나 Instant GameplayEffect는 "되돌려지거나" 롤백되지 않으며, 연관된 예측 키가 있다면 클라이언트에서 단순히 중복 실행을 건너뜁니다).
>
> 핵심을 다시 강조하자면, 예측 액션은 서버가 스스로 단독 실행하는 것이 아니라 클라이언트가 지시했을 때만 수행하는 것이어야 한다는 점입니다. 따라서 서버가 클라이언트의 요청을 받았을 때만 수행하도록 약속된 작업이 아니라면, "원할 때 키를 임의로 생성해 서버에 알리고 무언가를 실행한다"는 일반적인 방식은 성립할 수 없습니다.
>
> 원래 질문인 발사 후 망각 투사체로 돌아가자면, 파라곤과 포트나이트 모두 GameplayCue를 사용하는 투사체 액터 클래스를 가지고 있습니다. 그러나 우리는 이를 처리할 때 Prediction Key 시스템을 사용하지 않습니다. 대신 **비복제 게임플레이 큐(Non-Replicated GameplayCues)**라는 개념을 사용합니다. 로컬에서만 발동되고 서버에서는 완전히 건너뛰는 GameplayCue들입니다. 본질적으로 이들은 `UGameplayCueManager::HandleGameplayCue`에 대한 직접 호출입니다. `UAbilitySystemComponent`를 거치지 않으므로 예측 키 검사나 조기 반환이 일어나지 않습니다.
>
> 비복제 GameplayCue의 단점은 말 그대로 복제되지 않는다는 점입니다. 따라서 해당 함수를 호출하는 코드 경로가 모든 플레이어 환경에서 올바르게 실행되도록 보장하는 것은 투사체 클래스/블루프린트의 몫입니다. 우리는 투사체 생성 시점(`BeginPlay`), 폭발, 벽/캐릭터 충돌 등의 큐를 가지고 있습니다.
>
> 이러한 이벤트들은 이미 클라이언트 측에서 자체 생성되므로 비복제 게임플레이 큐를 호출하는 것은 큰 문제가 되지 않았습니다. 복잡한 블루프린트는 다소 까다로울 수 있으며, 작성자가 어느 로직이 어디서 실행되는지 명확히 이해하고 있어야 합니다.

2. **로컬 예측 `GameplayAbility`에서 스코프 예측 윈도우를 만들기 위해 `OnlyServerWait` 설정의 `WaitNetSync` `AbilityTask`를 사용할 때, 서버가 클라이언트의 예측 키 RPC를 기다리는 점을 악용하여 패킷 전송을 의도적으로 지연시키는 방식으로 어빌리티 타이밍을 조작하는 치팅(Lag Switch 등)이 가능하지 않나요? 파라곤이나 포트나이트에서 이것이 문제가 된 적이 있었는지, 에픽은 이를 어떻게 해결했는지 궁금합니다.**

> 네, 충분히 타당한 우려입니다. 서버에서 실행되는 어빌리티 블루프린트 중 클라이언트의 "신호(Signal)"를 기다리는 모든 로직은 랙 스위치(Lag Switch) 유형의 어뷰징에 잠재적으로 취약합니다.
>
> 파라곤에는 `UAbilityTask_WaitTargetData`와 유사한 커스텀 타겟팅 태스크가 있었습니다. 이 태스크에서 우리는 즉시 타겟팅 모드에 대해 타임아웃(Timeout), 즉 클라이언트를 기다리는 "최대 지연 시간(Max Delay)"을 두었습니다. 타겟팅 모드가 사용자의 확인(버튼 입력)을 기다리는 중이라면 플레이어가 시간을 들일 수 있으므로 무시되지만, 즉시 확인되는 어빌리티의 경우 일정 시간만 기다린 후 A) 서버 측에서 타겟 데이터를 직접 생성하거나 B) 어빌리티를 강제 취소했습니다.
>
> 하지만 사용 빈도가 극히 드물었던 `WaitNetSync`에 대해서는 이러한 메커니즘을 별도로 구현하지 않았습니다.
>
> 포트나이트의 경우 이런 식의 구조를 전혀 사용하지 않는 것으로 알고 있습니다. 포트나이트의 무기 어빌리티들은 포트나이트 전용 단일 RPC로 특수 배칭(Batch) 처리됩니다. 즉, 어빌리티 활성화, 타겟 데이터 전달, 어빌리티 종료를 단 1개의 RPC로 묶어 처리하므로 배틀로얄에서 무기 어빌리티는 본질적으로 이 취약점에 노출되지 않습니다.
>
> 제 생각에 이는 시스템 전반 차원에서 해결할 수 있는 문제이지만, 당장 에픽이 직접 수정할 계획은 없습니다. 우려되는 상황에 대비해 `WaitNetSync`를 수정하여 최대 지연 시간을 포함하도록 개선하는 것은 합리적인 작업이 될 것입니다.

3. **파라곤과 포트나이트는 어떤 `EGameplayEffectReplicationMode`를 사용했나요? 에픽이 권장하는 각 모드의 사용 기준은 무엇인가요?**

> 두 게임 모두 기본적으로 플레이어가 조종하는 캐릭터에는 **`Mixed`** 모드를 사용하고, AI가 조종하는 캐릭터(미니언, 정글 몬스터, 허스크 등)에는 **`Minimal`** 모드를 사용합니다. 멀티플레이어 게임에서 이 시스템을 사용하는 대부분의 개발자에게 권장하는 세팅이며, 프로젝트 초기에 설정할수록 좋습니다.
>
> 포트나이트는 여기서 한 걸음 더 나아간 최적화를 적용했습니다. 포트나이트는 시뮬레이트 프록시(Simulated Proxies)에 대해 `UAbilitySystemComponent`를 아예 복제하지 않습니다. 소유 플레이어 스테이트 클래스의 `::ReplicateSubobjects()` 내부에서 컴포넌트와 어트리뷰트 서브오브젝트 복제를 통째로 건너뜁니다. 대신 어빌리티 시스템 컴포넌트에서 꼭 필요한 최소한의 복제 데이터(어트리뷰트 값의 일부와 비트마스크로 복제되는 화이트리스트 태그 서브셋)만 폰(Pawn) 자체의 구조체로 밀어 넣습니다. 우리는 이를 "프록시(Proxy)"라고 부릅니다. 수신 측에서는 폰에서 복제된 이 프록시 데이터를 받아 플레이어 스테이트의 어빌리티 시스템 컴포넌트로 다시 밀어 넣습니다. 즉, FNBR의 모든 플레이어는 ASC를 가지고 있지만 네트워크로 직접 복제되지 않고, 폰의 미니멀 프록시 구조체를 통해 복제된 후 수신 측 ASC로 전달됩니다. 이는 A) 훨씬 적은 데이터양을 전송하고 B) 폰의 가시거리(Relevancy)를 활용할 수 있다는 큰 장점이 있습니다.
>
> 다만 이후 도입된 다른 서버 측 최적화(Replication Graph 등)를 고려할 때 이것이 여전히 필수적인지는 확신할 수 없으며, 유지보수 측면에서 가장 이상적인 패턴은 아닙니다.

4. **`GameplayPrediction.h`에 명시된 대로 `GameplayEffect`의 제거는 예측할 수 없는데, 지연 시간(Latency)으로 인한 이펙트 제거 지연 문제를 완화할 전략이 있나요? 예를 들어 이동 속도 둔화(Slow)를 해제할 때 현재는 서버가 GE 제거를 복제해 줄 때까지 기다려야 하므로 플레이어 캐릭터 위치가 순간이동(Snap)하는 현상이 발생합니다.**

> 정말 다루기 까다로운 문제이며 완벽한 정답은 없습니다. 우리는 일반적으로 허용 오차(Tolerance)와 스무딩(Smoothing)을 통해 이러한 문제들을 피해 갔습니다. 어빌리티 시스템과 캐릭터 무브먼트 시스템 간의 정밀한 동기화가 아직 이상적이지 않으며, 반드시 개선하고 싶은 부분이라는 점에 전적으로 동의합니다.
>
> 한때 GE의 예측 제거를 허용하는 작업을 보류함(Shelf)에 올려두고 연구했으나, 모든 예외 케이스를 해결하지 못한 채 다른 작업으로 넘어가야 했습니다. 하지만 이를 해결하더라도 캐릭터 무브먼트가 자체 세이브드 무브 버퍼(Saved Move Buffer)를 가지고 있어 어빌리티 시스템의 이속 변경 등을 알지 못하므로, GE 제거 예측 여부와 무관하게 보정 피드백 루프에 빠질 가능성은 여전히 존재합니다.
>
> 정말 절박한 상황이라면, 이동 속도 GE를 억제(Inhibit)하는 반대 성격의 GE를 예측 적용하는 방법을 시도해 볼 수 있습니다. 직접 적용해 본 적은 없으나 이론적으로 구상해 본 적은 있으며, 특정 유형의 문제 해결에 도움이 될 수 있습니다.

5. **파라곤과 포트나이트에서는 `AbilitySystemComponent`가 `PlayerState`에 위치하고 Action RPG 샘플에서는 `Character`에 위치함을 알고 있습니다. ASC가 어디에 위치해야 하며 `Owner`가 누가 되어야 하는지에 대한 에픽 내부의 규칙, 가이드라인 또는 권장 사항은 무엇인가요?**

> 일반적으로 **리스폰(Respawn)이 필요 없는 대상**은 Owner와 Avatar 액터를 동일하게 설정해야 합니다. AI 적, 건물, 월드 프랍 등이 여기에 해당합니다.
>
> 반면 **리스폰이 필요한 대상**은 리스폰 시 어빌리티 시스템 컴포넌트를 따로 저장/재생성/복구할 필요가 없도록 Owner와 Avatar를 분리해야 합니다. `PlayerState`는 모든 클라이언트에 복제되므로 가장 논리적인 선택입니다 (`PlayerController`는 소유 클라이언트에만 복제됨). 단점은 PlayerState가 상시 가시성(Always relevant)을 가지므로 100인 배틀로얄 같은 대규모 게임에서 문제가 될 수 있다는 점입니다 (질문 3번의 포트나이트 대응 방식 참조).

6. **동일한 소유자(Owner)를 가지지만 서로 다른 아바타(Avatar)를 가지는 여러 개의 `AbilitySystemComponent`를 두는 방식(예: Pawn과 무기/아이템/투사체에 각각 ASC를 두고 Owner는 모두 `PlayerState`로 설정)은 실현 가능한가요?**

> 가장 먼저 마주칠 문제는 소유 액터에서 `IGameplayTagAssetInterface`와 `IAbilitySystemInterface`를 구현하는 부분입니다. 전자는 모든 ASC의 태그를 취합(Aggregate)하는 방식으로 가능할 수 있습니다 (단, `HasAllMatchingGameplayTags`가 여러 ASC 간의 조합으로만 만족되는 경우 단순 OR 연산으로는 부족할 수 있으므로 주의해야 합니다). 하지만 후자는 훨씬 더 까다롭습니다: 어느 ASC가 권한(Authoritative)을 가진 주체인가요? 누군가 GE를 적용하고자 할 때 어느 ASC가 받아야 할까요? 해결할 수는 있겠지만, 하위에 여러 ASC를 거느린 단일 소유자 구조는 매우 다루기 힘들 것입니다.
>
> 하지만 Pawn과 무기에 독립적인 ASC를 두는 것 자체는 타당할 수 있습니다. 예를 들어 무기를 설명하는 태그와 폰을 설명하는 태그를 명확히 구분하는 경우입니다. 무기에 부여된 태그가 소유자에게도 "적용"되는 것으로 처리하는 식입니다 (어트리뷰트와 GE는 독립적이지만 소유자가 태그를 취합함). 가능은 하겠지만 동일한 Owner를 공유하는 다중 ASC는 복잡해질 위험이 큽니다.

7. **소유 클라이언트에서 로컬 예측 어빌리티의 쿨다운 지속 시간을 서버가 덮어쓰지 못하도록 막는 방법이 있나요? 높은 지연 시간 환경에서 로컬 쿨다운이 끝났을 때 클라이언트가 활성화를 "시도"할 수 있다면, 네트워크 패킷이 서버에 도달했을 즈음에는 서버의 쿨다운도 만료되어 정상 발동될 수 있을 것입니다. 그렇지 않으면 핑이 높은 클라이언트는 핑이 낮은 클라이언트에 비해 쿨다운이 짧은 기본 공격 등의 재발사 속도(연사력)가 현저히 떨어지는 불이익을 겪게 됩니다. 에픽은 파라곤의 기본 공격 등을 어떻게 설계하여 핑이 높은 유저도 동일한 속도로 공격할 수 있게 했나요?**

> 짧게 답하자면 이를 방지할 기본 방법은 없으며, 파라곤 역시 이 문제를 겪었습니다. 높은 지연 시간 연결에서는 기본 공격의 발사 속도(ROF)가 실제로 낮아졌습니다.
>
> 저는 GE 지속 시간을 계산할 때 지연 시간을 감안하는 "GE 조정(GE reconciliation)" 기능을 추가하여 해결을 시도한 적이 있습니다. 본질적으로 서버가 전체 GE 시간의 일부를 차감하여 클라이언트 측의 유효 GE 시간이 지연 시간에 관계없이 100% 일관되도록 만드는 방식이었습니다. 하지만 출시 가능한 수준의 안정성을 확보하지 못했고 프로젝트 일정이 촉박하여 완전히 해결하지 못했습니다.
>
> 포트나이트는 무기 발사 속도에 대해 자체적인 장부 관리(Bookkeeping)를 수행하며, 무기 쿨다운에 GE를 사용하지 않습니다. 게임에서 이 문제가 치명적이라면 이 방식을 추천합니다.

8. **GameplayAbilitySystem 플러그인에 대한 에픽의 로드맵은 무엇인가요? 2019년 이후 어떤 기능들을 추가할 계획인가요?**

> 우리는 현재 시스템이 전반적으로 상당히 안정적이라고 느끼고 있으며, 전담 인력이 주요 신기능을 개발하고 있지는 않습니다. 포트나이트나 UDN/풀 리퀘스트를 통한 버그 수정과 소소한 개선이 가끔 이루어지는 정도입니다.
>
> 장기적으로는 언젠가 "V2" 또는 대대적인 개편을 진행하게 될 것이라 생각합니다. 우리는 이 시스템을 만들면서 많은 것을 배웠고, 잘한 부분도 많지만 잘못 만든 부분도 많다고 느낍니다. 위에서 지적된 치명적인 결함들을 바로잡고 실수를 개선할 기회가 있기를 바랍니다.
>
> 만약 V2가 나온다면 업그레이드 경로를 제공하는 것이 최우선 과제가 될 것입니다. 포트나이트를 V1에 남겨둔 채 V2를 만들지는 않을 것이며, 수동 재작업이 일부 필요하더라도 최대한 자동 마이그레이션할 수 있는 절차를 마련할 것입니다.
>
> 우선순위가 높은 개선 사항들:
> * 캐릭터 무브먼트 시스템과의 더 나은 상호 운용성 및 클라이언트 예측 통합.
> * GE 제거 예측 (질문 4번)
> * GE 지연 시간 조정 (질문 7번)
> * RPC 배칭 및 프록시 구조체와 같은 일반화된 네트워크 최적화 지원.
>
> 더 일반적인 리팩터링 계획:
> * GE가 스프레드시트 값을 직접 참조하는 방식에서 벗어나, GE는 파라미터만 노출하고 상위 객체가 커브 테이블과 바인딩하도록 분리.
> * `UGameplayAbility`의 여러 "정책(Policy)" 축소. `ReplicationPolicy`와 `InstancingPolicy` 제거. `FGameplayAbilitySpec`을 상속 가능한 UObject로 변경하여 비인스턴스 어빌리티 역할을 수행하게 하고, `UGameplayAbility`는 순수 실행 인스턴스로 단순화.
> * 필터링된 GE 적용 컨테이너, 오버랩 볼륨 지원 등 자주 쓰이는 중간 수준의 구성 요소 기본 제공.
> * 보일러플레이트 코드 감소. 패시브 어빌리티나 기본 히트스캔 무기 등을 즉시 제공하는 별도 확장 라이브러리 모듈 제공.
> * `GameplayCues`를 어빌리티 시스템과 분리된 독립 모듈로 분리.
>
> 이는 저 개인의 의견이며 회사의 공식 약속은 아닙니다. 새로운 엔진 기술 이니셔티브가 도입될 때 어빌리티 시스템도 함께 업데이트될 기회가 올 것입니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="resources-daveratti-community2"></a>
#### 11.1.2 커뮤니티 질문 세션 2
커뮤니티 멤버 [iniside](https://github.com/iniside)와 Dave Ratti의 Q&A:

1. **디커플링된 고정 틱(Decoupled fixed ticking)에 대한 지원 계획이 있나요? 렌더링 스레드는 자유롭게 실행되면서 게임 스레드는 30/60fps 등으로 고정되기를 원합니다.**

> 렌더링 프레임 레이트와 게임 스레드 틱 레이트를 디커플링할 계획은 없습니다. 시스템의 복잡성과 이전 엔진 버전과의 하위 호환성 유지 문제로 인해 이 방향은 실현되기 어렵습니다.
>
> 대신 우리가 나아간 방향은 게임 스레드와 독립적으로 고정 틱 레이트로 실행되는 비동기 "물리 스레드(Physics Thread)"를 두는 것입니다. 고정 레이트로 실행되어야 하는 요소들은 여기서 실행되고, 게임 스레드와 렌더링은 기존 방식대로 동작할 수 있습니다.
>
> 네트워크 예측(Network Prediction) 플러그인은 독립 틱(Independent Ticking)과 고정 틱(Fixed Ticking) 모드를 지원합니다. 장기적 계획은 독립 틱을 가변 프레임 레이트의 게임 스레드에서 실행되는 전통적인 "클라이언트가 자신의 폰을 예측하는" 모델로 유지하고, 고정 틱은 비동기 물리를 사용하여 물리 오브젝트, 타 클라이언트/차량 등을 예측하는 데 활용하는 것입니다.

2. **Network Prediction 플러그인이 Ability System과 어떻게 통합될 계획인가요? 예를 들어 예측 키 대신 고정 프레임 기반 어빌리티 활성화가 도입되나요?**

> 네, 계획은 어빌리티 시스템의 예측 키를 완전히 재작성/제거하고 Network Prediction 구조체로 대체하는 것입니다.
>
> 핵심 아이디어는 ASC의 RPC에서 클라이언트->서버 간의 명시적인 Prediction Key 교환을 제거하는 것입니다. 스코프 예측 윈도우나 예측 키가 더 이상 존재하지 않으며, 모든 것이 NetworkPrediction 프레임에 앵커링됩니다. 중요한 것은 클라이언트와 서버가 특정 사건이 언제 일어났는지에 대해 프레임 단위로 완벽히 합의한다는 점입니다:
> * 어빌리티가 활성화/종료/취소된 시점
> * GameplayEffect가 적용/제거된 시점
> * 특정 프레임 X에서의 어트리뷰트 값

3. **GameplayMessages 플러그인이 다시 복구될 계획이 있나요? (게임 피처 및 모듈러 게임플레이와 연동되는 범용 이벤트 버스 디스패처)**

> 네, 어느 시점에 다시 도입될 것입니다. API가 아직 최종 확정되지 않았던 상태였습니다. 모듈러 게임플레이 디자인에 매우 유용할 것입니다.

4. **비동기 고정 물리 및 비동기 CharacterMovementComponent 작업에 대한 방향성은 어떠한가요?**

> 목표는 비동기 물리가 활성화된 상태에서 엔진은 가변 틱 레이트로 실행되면서 물리 및 "핵심" 게임플레이 시뮬레이션(캐릭터 이동, 차량, GAS 등)은 고정 레이트로 실행될 수 있도록 하는 것입니다.
>
> 현재 이를 활성화하기 위한 콘솔 변수:
> `p.DefaultAsyncDt=0.03333`  
> `p.RewindCaptureNumFrames=64`  
> 카오스(Chaos)는 물리 상태에 대한 보간을 제공하며, `p.AsyncInterpolationMultiplier` 변수로 이를 제어할 수 있습니다.
>
> 현재는 NetSerialize, ShouldReconcile, Interpolate 등을 수동으로 작성해야 하지만, 궁극적으로는 리플렉션 시스템을 활용하여 보일러플레이트 없이 직관적으로 사용할 수 있는 인터페이스를 제공할 예정입니다.

**[⬆ 맨 위로 이동](#table-of-contents)**

<a name="changelog"></a>
## 12. GAS 변경 이력 (GAS Changelog)

공식 언리얼 엔진 업그레이드 릴리즈 노트 및 문서화되지 않은 변경 사항들을 종합한 GAS의 주요 변경 내역입니다.

<a name="changelog-5.3"></a>
### 5.3
* 크래시 수정: 심리스 트래블(Seamless travel) 이후 Gameplay Cues를 적용하려 할 때 발생하던 크래시를 수정했습니다.
* 크래시 수정: 라이브 코딩(Live Coding) 사용 시 GlobalAbilityTaskCount로 인해 발생하던 크래시를 수정했습니다.
* 크래시 수정: `UAbilityTask_StartAbilityState` 등의 케이스에서 `UAbilityTask::OnDestroy`가 재귀 호출될 때 크래시가 발생하던 문제를 수정했습니다.
* 버그 수정: 자식 클래스에서 `Super::ActivateAbility`를 호출해도 안전합니다. 이전에는 내부에서 `CommitAbility`를 잘못 호출했습니다.
* 버그 수정: 다양한 유형의 `FGameplayEffectContext`를 올바르게 리플리케이트할 수 있도록 지원을 추가했습니다.
* 버그 수정: `FGameplayEffectContextHandle`이 "Actors"를 검색하기 전에 데이터 유효성을 먼저 검사하도록 수정했습니다.
* 버그 수정: Gameplay Ability System Target Data LocationInfo의 회전(Rotation) 값을 유지하도록 수정했습니다.
* 버그 수정: 유효한 PlayerController를 찾았을 때만 탐색을 중단하도록 수정했습니다.
* 버그 수정: `RemoveGameplayCue_Internal`에서 기본 파라미터 객체 대신 기존 `GameplayCueParameters`가 존재하는 경우 이를 사용하도록 수정했습니다.
* 버그 수정: `GameplayAbilityWorldReticle`이 TargetingActor 대신 소스 Actor를 바라보도록 수정했습니다.
* 버그 수정: `GiveAbilityAndActivateOnce` 호출 시 어빌리티 목록이 잠겨 있을 경우 전달된 트리거 이벤트 데이터를 캐시하도록 수정했습니다.
* 버그 수정: `FInheritedGameplayTags`가 저장을 기다리지 않고 `CombinedTags`를 즉시 업데이트하도록 지원을 추가했습니다.
* 버그 수정: `ShouldAbilityRespondToEvent`를 클라이언트 전용 코드 경로에서 서버와 클라이언트 양쪽 모두로 이동했습니다.
* 버그 수정: 커브 단순화로 인해 쿠킹된 빌드(Cooked Builds)에서 `FAttributeSetInitterDiscreteLevels`가 동작하지 않던 문제를 수정했습니다.
* 버그 수정: GameplayAbility에서 `CurrentEventData`를 올바르게 설정하도록 수정했습니다.
* 버그 수정: 잠재적 콜백을 실행하기 전에 `MinimalReplicationTags`가 올바르게 설정되도록 보장했습니다.
* 버그 수정: 인스턴스화된 GameplayAbility에서 `ShouldAbilityRespondToEvent`가 호출되지 않던 문제를 수정했습니다.
* 버그 수정: `gc.PendingKill`이 비활성화된 상태에서 Child Actor에서 실행 중인 Gameplay Cue Notify Actor가 메모리를 누수하지 않도록 수정했습니다.
* 버그 수정: 해시 충돌로 인해 GameplayCueManager에서 `GameplayCueNotify_Actor`가 유실되던 문제를 수정했습니다.
* 버그 수정: 액터에 게임플레이 태그가 없더라도 `WaitGameplayTagQuery`가 쿼리를 준수하도록 수정했습니다.
* 버그 수정: `PostAttributeChange` 및 `AttributeValueChangeDelegates`가 올바른 `OldValue`를 가지도록 수정했습니다.
* 버그 수정: 네이티브 코드에서 생성된 `FGameplayTagQuery` 구조체의 Query Description이 정상 표시되지 않던 문제를 수정했습니다.
* 버그 수정: 어빌리티 시스템 사용 시 `UAbilitySystemGlobals::InitGlobalData`가 반드시 호출되도록 보장했습니다. 이전에는 호출하지 않으면 시스템이 비정상 동작했습니다.
* 버그 수정: `UGameplayAbility::EndAbility`에서 애님 레이어를 링크/언링크할 때 발생하던 문제를 수정했습니다.
* 버그 수정: Ability System Component 함수가 Spec의 어빌리티 포인터를 사용 전 검사하도록 업데이트했습니다.
* 신규 기능: 더 복잡한 요구사항을 지정할 수 있도록 `FGameplayTagRequirements`에 `GameplayTagQuery` 필드를 추가했습니다.
* 신규 기능: `SourceTagQuery`를 보강하는 `FGameplayEffectQuery::SourceAggregateTagQuery`를 도입했습니다.
* 신규 기능: 콘솔 명령에서 Gameplay Abilities 및 Gameplay Effects를 직접 실행하고 취소할 수 있는 기능을 확장했습니다.
* 신규 기능: 어빌리티 블루프린트의 개발 방식과 사용 의도 정보를 보여주는 "Audit(감사)" 기능을 추가했습니다.
* 변경 사항: Instanced Per Actor 어빌리티의 경우 `OnAvatarSet`이 CDO 대신 기본 인스턴스(Primary Instance)에서 호출됩니다.
* 변경 사항: 동일한 어빌리티 그래프 내에서 `Activate Ability`와 `Activate Ability From Event`를 동시에 사용할 수 있도록 허용했습니다.
* 변경 사항: `AnimTask_PlayMontageAndWait`에 BlendOut 이벤트 이후 Completed 및 Interrupted를 허용하는 토글을 추가했습니다.
* 변경 사항: `ModMagnitudeCalc` 래퍼 함수들이 `const`로 선언되었습니다.
* 변경 사항: `FGameplayTagQuery::Matches`가 빈 쿼리에 대해 false를 반환합니다.
* 변경 사항: `FGameplayAttribute::PostSerialize`가 포함된 어트리뷰트를 검색 가능한 이름(Searchable Name)으로 등록하도록 업데이트했습니다.
* 변경 사항: `GetAbilitySystemComponent`의 기본 매개변수를 Self로 지정했습니다.
* 변경 사항: `AbilityTask_WaitTargetData`의 함수들을 `virtual`로 표시했습니다.
* 변경 사항: 사용되지 않는 `FGameplayAbilityTargetData::AddTargetDataToGameplayCueParameters` 함수를 제거했습니다.
* 변경 사항: 사용되지 않는 잔재 코드인 `GameplayAbility::SetMovementSyncPoint`를 제거했습니다.
* 변경 사항: 일부 GameplayEffect 기능을 선택적 컴포넌트로 분리 이동했습니다. 기존 콘텐츠는 PostCDOCompiled 시 컴포넌트를 사용하도록 자동 업데이트됩니다.

릴리즈 노트: https://docs.unrealengine.com/5.3/en-US/unreal-engine-5.3-release-notes/

<a name="changelog-5.2"></a>
### 5.2
* 버그 수정: `UAbilitySystemBlueprintLibrary::MakeSpecHandle` 함수의 크래시를 수정했습니다.
* 버그 수정: 서버에서 로컬 스폰된 논-컨트롤 폰(차량 등)이 원격 프록시로 잘못 간주되던 로직을 수정했습니다.
* 버그 수정: 서버에 의해 거부된 예측 인스턴스 어빌리티의 활성화 정보를 올바르게 설정하도록 수정했습니다.
* 버그 수정: 원격 인스턴스에서 GameplayCues가 고착되던 버그를 수정했습니다.
* 버그 수정: `WaitGameplayEvent` 호출을 체이닝할 때 발생하던 메모리 스톰프(Memory Stomp)를 수정했습니다.
* 버그 수정: 블루프린트에서 `GetOwnedGameplayTags()` 호출 시 동일 노드가 여러 번 실행되어도 이전 반환값이 잔류하지 않도록 수정했습니다.
* 버그 수정: 복제되지 않는 동적 객체 참조를 복제하려던 GameplayEffectContext 문제를 수정했습니다.
* 신규 기능: 데이터 기반 타겟팅 요청을 생성할 수 있는 [게임플레이 타겟팅 시스템(Gameplay Targeting System)](https://docs.unrealengine.com/en-US/gameplay-targeting-system-in-unreal-engine/)을 추가했습니다.
* 신규 기능: GameplayTag Queries에 대한 커스텀 직렬화 지원을 추가했습니다.
* 신규 기능: 파생된 `FGameplayEffectContext` 타입의 복제 지원을 추가했습니다.
* 신규 기능: 에셋의 게임플레이 어트리뷰트가 저장 시 검색 가능한 이름으로 등록되어 레퍼런스 뷰어에서 확인할 수 있게 되었습니다.
* 신규 기능: AbilitySystemComponent에 대한 기본 단위 테스트(Unit Tests)를 추가했습니다.
* 신규 기능: 어트리뷰트가 코어 리디렉트(Core Redirects)를 준수합니다. 코드에서 Attribute Set 및 어트리뷰트 이름을 변경하더라도 DefaultEngine.ini 리디렉트를 통해 기존 에셋이 정상 로드됩니다.
* 변경 사항: 코드에서 Gameplay Effect Modifier의 평가 채널(Evaluation Channel)을 변경할 수 있도록 허용했습니다.
* 변경 사항: 사용되지 않던 `FGameplayModifierInfo::Magnitude` 변수를 제거했습니다.

릴리즈 노트: https://docs.unrealengine.com/5.2/en-US/unreal-engine-5.2-release-notes/

<a name="changelog-5.1"></a>
### 5.1
* 버그 수정: 복제된 루즈(Loose) 게임플레이 태그가 소유자에게 복제되지 않던 문제를 수정했습니다.
* 버그 수정: 어빌리티가 적시에 가비지 컬렉션되는 것을 방해하던 AbilityTask 버그를 수정했습니다.
* 버그 수정: 특정 태그를 수신하여 활성화되는 어빌리티가 둘 이상일 때 목록의 첫 번째 어빌리티가 유효하지 않거나 권한이 없으면 활성화에 실패하던 문제를 수정했습니다.
* 버그 수정: Data Registries를 사용하는 GameplayEffects의 로드 경고를 수정하고 메시지를 개선했습니다.
* 버그 수정: ApplyGameplayEffectSpecToTarget 내부 잠금 중에 EndAbility가 호출될 경우 어빌리티가 고착되던 문제를 수정했습니다.
* 신규 기능: Gameplay Effects가 차단 어빌리티 태그(Blocked Ability Tags)를 추가할 수 있도록 지원합니다.
* 신규 기능: `WaitGameplayTagQuery` 노드를 추가했습니다 (AbilityTask 및 AbilityAsync 기반).
* 신규 기능: 논-쉬핑 빌드에서 콘솔 변수를 통한 AbilityTask 디버깅 기록 및 로그 출력을 기본 활성화했습니다.
* 신규 기능: `AbilitySystem.DebugAbilityTags`, `AbilitySystem.DebugBlockedTags`, `AbilitySystem.DebugAttribute` 디버그 명령을 추가했습니다.
* 신규 기능: 어트리뷰트의 디버그 문자열 표현을 가져오는 블루프린트 함수를 추가했습니다.
* 변경 사항: `FGameplayAbilitySpec/Def::SourceObject`를 약참조(Weak Reference)로 변경했습니다.
* 변경 사항: Ability Task 내의 Ability System Component 참조를 약포인터(Weak Pointer)로 변경하여 GC 수집이 가능하도록 했습니다.
* 변경 사항: 중복 열거형인 `EWaitGameplayTagQueryAsyncTriggerCondition`을 제거했습니다.
* 변경 사항: Gameplay Abilities의 활성화 실패 이유를 알 수 있도록 로깅을 강화했습니다.

릴리즈 노트: https://docs.unrealengine.com/5.1/en-US/unreal-engine-5.1-release-notes/

<a name="changelog-5.0"></a>
### 5.0
릴리즈 노트: https://docs.unrealengine.com/5.0/en-US/unreal-engine-5.0-release-notes/

<a name="changelog-4.27"></a>
### 4.27
* 크래시 수정: 시간에 따른 강도 모디파이어를 가진 지속 힘 루트 모션 태스크를 사용하는 어빌리티가 종료될 때 네트워크 클라이언트에서 발생하던 크래시를 수정했습니다.
* 버그 수정: GameplayCues 사용 시 에디터 로딩 시간 저하 문제를 수정했습니다.
* 버그 수정: 동일한 EffectLevel을 설정할 때 GameplayEffectsContainer의 `SetActiveGameplayEffectLevel`이 FastArray를 더티 표시하지 않도록 수정했습니다.
* 버그 수정: `K2_OnEndAbility`에서 `EndAbility`를 다시 호출하여 발생하던 무한 재귀 문제를 수정했습니다.
* 버그 수정: GameplayTags 작업의 스레드 안전성을 개선했습니다.
* 신규 기능: GameplayAbility의 `K2_CanActivateAbility`에 SourceObject를 노출했습니다.
* 신규 기능: 네이티브 C++ 게임플레이 태그인 `FNativeGameplayTag`를 도입하여 모듈 로드/언로드 시 태그가 안전하게 등록 및 해제되도록 지원합니다.
* 신규 기능: `GiveAbilityAndActivateOnce`에 FGameplayEventData 매개변수를 전달할 수 있도록 업데이트했습니다.
* 신규 기능: Data Registry 시스템으로부터 커브 테이블을 동적으로 조회할 수 있도록 ScalableFloats를 개선했습니다.
* 신규 기능: GameplayEffectSpec의 `SetContext` 호출 시 원본 캡처된 SourceTags를 보존하는 옵션을 추가했습니다.
* 신규 기능: 시퀀서(Sequencer)에 GameplayCueTrack 트랙을 추가하여 애니메이션 타임라인에서 큐를 직접 트리거할 수 있게 되었습니다.
* 변경 사항: `GameplayCueInterface`가 GameplayCueParameters 구조체를 참조(Reference)로 전달하도록 변경했습니다.
* 최적화: GameplayTag 테이블 로드 및 재생성 성능을 크게 개선했습니다.

릴리즈 노트: https://docs.unrealengine.com/en-US/WhatsNew/Builds/ReleaseNotes/4_27/

<a name="changelog-4.26"></a>
### 4.26
* GAS 플러그인이 더 이상 베타(Beta) 플래그가 아니며 정식 프로덕션 준비 상태로 전환되었습니다.
* 크래시 수정: 유효한 소스 선택 없이 태그를 추가할 때 발생하던 크래시를 수정했습니다.
* 버그 수정: 중첩 GE 적용 시 지속 시간이 리셋되지 않던 버그를 수정했습니다.
* 버그 수정: `CancelAllAbilities`가 비인스턴스 어빌리티만 취소하던 문제를 수정했습니다.
* 신규 기능: 어빌리티 커밋 함수에 선택적 태그 매개변수를 추가했습니다.
* 신규 기능: `PlayMontageAndWait` 태스크에 `StartTimeSeconds` 매개변수를 추가했습니다.
* 신규 기능: `FGameplayAbilitySpec`에 `DynamicAbilityTags` 컨테이너를 추가했습니다 (스펙과 함께 복제되며 소스 태그로 캡처됨).
* 신규 기능: `IsLocallyControlled` 및 `HasAuthority` 함수를 블루프린트에서 호출할 수 있게 되었습니다.
* 신규 기능: 블루프린트 노드의 게임플레이 어트리뷰트 핀에 리디렉터를 지원합니다.
* 신규 기능: 루트 모션 이동 태스크가 종료될 때 무브먼트 컴포넌트의 이동 모드를 태스크 시작 전 상태로 자동 복구합니다.

릴리즈 노트: https://docs.unrealengine.com/en-US/WhatsNew/Builds/ReleaseNotes/4_26/

<a name="changelog-4.25.1"></a>
### 4.25.1
* 버그 수정: UE-92787 Get Float Attribute 노드의 어트리뷰트 핀이 인라인 설정된 블루프린트 저장 시 크래시 수정.
* 버그 수정: UE-92810 인라인 변경된 인스턴스 편집 가능 태그 프로퍼티를 가진 액터 스폰 시 크래시 수정.

<a name="changelog-4.25"></a>
### 4.25
* `RootMotionSource` `AbilityTasks`의 예측 문제를 수정했습니다.
* [`GAMEPLAYATTRIBUTE_REPNOTIFY()`](#concepts-as-attributes) 매크로가 이제 이전 어트리뷰트 값(`OldValue`)을 추가 매개변수로 받습니다.
* `UGameplayAbility`에 [`NetSecurityPolicy`](#concepts-ga-netsecuritypolicy)를 추가했습니다.
* 크래시 수정: 태그 요구조건을 검사하기 전에 GameplayEffect 정의 유효성을 먼저 확인하도록 수정했습니다.
* 버그 수정: 블루프린트 디버거에서 멀티 뷰포트 시 GameplayEffects 태그가 복제되지 않던 문제를 수정했습니다.
* 버그 수정: `GiveAbilityAndActivateOnce`가 불일치하게 동작하던 버그를 수정했습니다.
* 신규 기능: `UGameplayAbility`에 `OnRemoveAbility` 함수를 추가했습니다.
* 신규 기능: 지정된 태그를 가진 모든 활성 GE를 조회하는 `GetActiveEffectsWithAllTags` 함수를 추가했습니다.
* API 변경: `AddDefaultSubobjectSet`이 Deprecated 되었으며 대신 `AddAttributeSetSubobject`를 사용해야 합니다.
* 신규 기능: Gameplay Abilities에서 몽타주를 재생할 Anim Instance를 직접 지정할 수 있게 되었습니다.

릴리즈 노트: https://docs.unrealengine.com/en-US/WhatsNew/Builds/ReleaseNotes/4_25/

<a name="changelog-4.24"></a>
### 4.24
* 컴파일 시 블루프린트 노드의 `Attribute` 변수가 `None`으로 리셋되던 문제를 수정했습니다.
* [`TargetData`](#concepts-targeting-data)를 사용하려면 [`UAbilitySystemGlobals::InitGlobalData()`](#concepts-asg-initglobaldata)를 반드시 호출해야 합니다. 그렇지 않으면 `ScriptStructCache` 오류가 발생하고 클라이언트 연결이 끊어집니다.
* `UGameplayAbility::MontageStop()` 함수가 `OverrideBlendOutTime` 매개변수를 올바르게 사용하도록 수정했습니다.
* `GameplayEffectExecutionCalculations`가 어트리뷰트 캡처 없이도 "임시 변수"에 대한 스코프 모디파이어를 지원할 수 있는 기능을 추가했습니다 (`ValidTransientAggregatorIdentifiers`).
* `APawn::PossessedBy()`가 폰의 소유자를 새로운 `Controller`로 설정하도록 수정했습니다. ASC가 폰에 있을 때 [Mixed 리플리케이션 모드](#concepts-asc-rm)에 필수적입니다.
* `FAttributeSetInitterDiscreteLevels`의 POD 데이터 관련 버그를 수정했습니다.

릴리즈 노트: https://docs.unrealengine.com/en-US/WhatsNew/Builds/ReleaseNotes/4_24/

**[⬆ 맨 위로 이동](#table-of-contents)**
