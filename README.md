# Cord: Marigold

<p align="center">
  <img src="ScreenShot/s1.jpg" width="85%" alt="메인 화면">
</p>

Unity로 제작한 3인칭 건슈팅 디펜스 게임입니다.

플레이어는 무기와 아이템을 활용해 기지를 방어하고, 스테이지를 클리어할 때마다 상점에서 무기와 능력을 강화하며 10스테이지까지 진행합니다.

---

## 프로젝트 소개

Cord: Marigold는 플레이어가 직접 사격하며 기지를 방어하는 3인칭 건슈팅 디펜스 게임입니다.

단발, 연발, 저격 사격과 로켓을 상황에 맞게 바꿔 사용할 수 있으며, 터렛을 설치하면 적의 공격 대상이 터렛으로 바뀝니다.

스테이지를 클리어하면 적 처치로 얻은 코인으로 무기 공격력, 플레이어 체력, 터렛, EMP, 로켓 등을 강화하고 다음 스테이지를 준비합니다.

| 항목 | 내용 |
| --- | --- |
| 플랫폼 | PC |
| 개발 엔진 | Unity |
| 개발 언어 | C# |
| 개발 기간 | 2025.03.01 ~ 2025.06.18 |
| 개발 인원 | 총 4명 |
| 팀 구성 | 아트 3명 / 기획·프로그래밍 1명 (본인) |

---

## Gameplay

| 전투 장면 | 상점 UI |
| :---: | :---: |
| <img src="ScreenShot/s2.jpg" width="100%" alt="전투 장면"> | <img src="ScreenShot/s3.jpg" width="100%" alt="상점 UI"> |

게임 플레이 영상은 아래 링크에서 확인할 수 있습니다.

[Cord: Marigold Gameplay Video](https://youtu.be/LIh10WImKrg)

---

## Download

게임 실행 파일은 아래 링크에서 다운로드할 수 있습니다.

- [Download for Windows](https://drive.google.com/file/d/19D6CXySd6lN7rgg42rRtS_mboQrVkVyU/view)
- [Download for macOS](https://drive.google.com/file/d/1VpC8uMO6C5oWmSdndyjzkfCf9CmK16RZ/view)

---

## My Role

기획과 클라이언트 프로그래밍을 혼자 담당했습니다.

- 단발·연발·저격 사격과 로켓 발사, 무기 교체, 탄약·재장전 처리
- 사격 여부에 따른 엄폐 판정과 플레이어·기지 피해 처리
- 스테이지별 적 생성 수·종류·능력치 조정 및 스테이지 진행
- 적의 공격·이동 반복 동작과 터렛 설치 시 공격 대상 전환
- 터렛, EMP, 로켓 아이템
- 스테이지 클리어 후 상점(강화 구매·되돌리기)
- HUD(체력, 탄약, 코인, 스테이지), 사운드 볼륨·화면 모드 설정, 점수 저장
- 튜토리얼 씬

---

# 주요 구현 내용

## 무기 사격 및 교체

`WeaponBase` 추상 클래스에 사격 시작·종료(`StartWeaponAction`, `StopWeaponAction`)와 재장전(`StartReload`)을 추상 메서드로 두고, 마우스 위치 기준 Raycast 사격(`Shoot`)을 기본 구현으로 두었습니다. 단발·연발·저격·로켓은 이 클래스를 상속해 각자 동작을 구현합니다.

`WeaponSwitchSystem`은 네 무기를 `WeaponBase` 배열로 들고 있으며, 숫자키(1·2·3, Q)와 마우스 휠로 무기를 바꿉니다. 휠 입력은 배열 인덱스를 나머지 연산으로 순환시켜 처리했습니다.

- **저격**: 버튼을 누르고 있는 동안 0.5초에 걸쳐 차지되고, 버튼을 떼면 차지 진행도에 따라 최대 2배 데미지로 발사합니다. 판정은 `SphereCast`로 가장 먼저 맞은 적 하나만 처리하고, 적중 시 EMP 쿨타임을 줄입니다.
- **무기별 상성**: 무기마다 추가 데미지를 주는 적 유형(태그)이 정해져 있으며, 상성 무기로 맞힌 적을 처치하면 조건에 따라 추가 코인을 지급합니다.
- **로켓**: 발사체를 생성해 직선으로 날린 뒤, 적과 충돌하거나 2초가 지나면 `OverlapSphere` 범위 안의 적에게 폭발 데미지를 줍니다. 보유 수는 최대 3개입니다.
- **재장전**: R 키 또는 탄약이 0이 되면 자동으로 시작됩니다.

**관련 코드** [WeaponBase.cs](https://github.com/Thispring/Cord-Marigold/blob/main/Script/Weapon/WeaponBase.cs) · [WeaponSwitchSystem.cs](https://github.com/Thispring/Cord-Marigold/blob/main/Script/Weapon/WeaponSwitchSystem.cs) · [WeaponSniperRifle.cs](https://github.com/Thispring/Cord-Marigold/blob/main/Script/Weapon/WeaponSniperRifle.cs) · [Rocket.cs](https://github.com/Thispring/Cord-Marigold/blob/main/Script/Weapon/Rocket.cs)

---

## 엄폐 판정과 플레이어·기지 피해

플레이어가 사격하지 않는 동안에는 엄폐 상태가 되고, 사격 중에는 엄폐가 풀립니다.

적 발사체는 플레이어가 엄폐 중이 아닐 때는 플레이어에게, 엄폐 중일 때는 기지에 피해를 줍니다. 플레이어나 기지의 체력이 0이 되면 패배 씬으로 전환합니다.

**관련 코드** [PlayerStatusManager.cs](https://github.com/Thispring/Cord-Marigold/blob/main/Script/Player/PlayerStatusManager.cs) · [EnemyProjectile.cs](https://github.com/Thispring/Cord-Marigold/blob/main/Script/Enemy/EnemyProjectile.cs) · [BaseStatus.cs](https://github.com/Thispring/Cord-Marigold/blob/main/Script/GameManager/BaseStatus.cs)

---

## 적 생성 및 스테이지 진행

스테이지가 시작되면 해당 스테이지에 나올 적 목록을 먼저 정한 뒤, 일정 주기로 생성합니다.

- **스테이지별 조정**: 스테이지가 오를수록 적의 최대 수와 동시 생성 수가 늘고 생성 주기가 짧아지며, 적의 체력과 공격력도 스테이지 번호에 따라 증가합니다.
- **엘리트 적**: 3스테이지부터 일정 확률로 등장하며, 한 스테이지에 최대 2마리(8스테이지 이후 3마리)로 제한했습니다.
- **생성 위치**: 적 크기(소형·중형·대형)별로 지정된 스폰 구역(`Bounds`) 안에서 무작위 위치를 뽑고, 이번 스테이지에서 먼저 배치된 위치들과 3 이상 떨어진 위치를 찾습니다. 50회 안에 찾지 못하면 마지막 위치에 배치해 무한 반복을 막았습니다.
- **생성 위치 표시**: 적이 나타나기 전 위치를 알려주는 표시 오브젝트는 직접 작성한 `MemoryPool` 클래스로 재사용합니다. 비활성 오브젝트를 꺼내 쓰고, 모두 사용 중이면 5개씩 추가 생성합니다.

스테이지의 적을 모두 처치하면 처치 수를 점수에 더하고, 게임을 일시 정지한 뒤 상점을 엽니다. 상점에서 게임을 재개하면 다음 스테이지 생성이 시작되며, 10스테이지를 클리어하면 승리 씬으로 전환합니다.

**관련 코드** [EnemyMemoryPool.cs](https://github.com/Thispring/Cord-Marigold/blob/main/Script/Enemy/EnemyMemoryPool.cs) · [MemoryPool.cs](https://github.com/Thispring/Cord-Marigold/blob/main/Script/MemoryPool.cs) · [StageManager.cs](https://github.com/Thispring/Cord-Marigold/blob/main/Script/GameManager/StageManager.cs)

---

## 적 행동 및 공격 대상 전환

적은 생성 후 대기 시간을 거친 뒤, 대상을 향해 두 번 공격하고 좌우 무작위 방향으로 최대 2초간 이동하는 동작을 반복합니다. 공격과 이동은 각각 코루틴으로 처리하며, 하나를 시작할 때 다른 하나를 중지합니다.

플레이어가 터렛을 설치하면 `TurretController`가 활성화된 적들에게 알려 공격 대상을 터렛으로 바꾸고, 터렛이 파괴되면 원래 대상으로 되돌립니다. 터렛이 설치된 뒤 생성된 적도 생성 시점에 활성화된 터렛을 대상으로 잡습니다.

EMP를 사용하면 모든 적이 일정 시간 동안 공격과 이동을 멈추고, 날아오던 적 발사체는 사라집니다.

**관련 코드** [EnemyFSM.cs](https://github.com/Thispring/Cord-Marigold/blob/main/Script/Enemy/EnemyFSM.cs) · [TurretController.cs](https://github.com/Thispring/Cord-Marigold/blob/main/Script/Item/TurretController.cs) · [EMP.cs](https://github.com/Thispring/Cord-Marigold/blob/main/Script/Item/EMP.cs)

---

## 상점 및 강화

스테이지를 클리어하면 열리는 상점에서 무기 3종 공격력, 체력 회복, 최대 체력, 기지 수리, 터렛 추가·강화, EMP 지속시간, 로켓 추가 등 10가지 항목을 구매할 수 있습니다.

강화 항목은 단계가 오를수록 가격이 올라갑니다. 구매 시 코인은 바로 차감하고 효과는 구매 횟수로만 기록해 두었다가, 게임을 재개할 때 한 번에 적용합니다. 그 전에는 되돌리기 버튼으로 상점을 열었을 때의 코인과 단계로 복구할 수 있습니다.

**관련 코드** [ShopManager.cs](https://github.com/Thispring/Cord-Marigold/blob/main/Script/GameManager/ShopManager.cs) · [StageManager.cs](https://github.com/Thispring/Cord-Marigold/blob/main/Script/GameManager/StageManager.cs)

---

> 이 저장소에는 포트폴리오 공개용으로 프로젝트의 스크립트 코드만 포함되어 있습니다.
>
> 게임 에셋과 전체 Unity 프로젝트 파일은 포함되어 있지 않습니다.
