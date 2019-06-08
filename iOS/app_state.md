iOS App State
==
- Not Running
- Foreground
    - Inactice
    - Active
- Background
- Suspend

## Not Running
- App이 실행되기 전 또는 실행 후 종료된 상태

## ForeGround
- App이 실행되어 사용자에게 보여지는 상태
- 오직 하나의 App만 foreground 상태이다.
- inactive와 active로 나뉜다.
### Active
    말그대로 active 상태.
### Inactive
    foreground 상태에서 전화, 잠금, 멀티테스킹 스크린 등으로 앱이 액티브하지 않음

## Background
- Foreground 상태에서 홈스크린으로 이동한 상태. 
- background 전환 전에 호출된 task는 background에서도 돌아가지만, 전환 후 호출된 task는 foreground 상태로 전환 된 후에야 실행된다.
- background 상태에서 가능한 task들이 정해져 있다.
    - Audio
    - Location Update
    - Notifications

## Suspend
- Background상태에서 더 이상 작업수행 안함. 앱은 메모리에 남아서 Suspend 될 당시의 상태를 저장하고 있지만 cpu나 배터리는 소모하지 않는다.
- 메모리 부족 등의 상황에서 자동으로 종료 될 수 있다.

