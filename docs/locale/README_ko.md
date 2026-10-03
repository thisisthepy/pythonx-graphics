# pythonx-graphics

Kotlin Multiplatform 위의 Python 앱을 위한 선언형 2D·3D 그래픽스입니다. 2D는 Compose, 3D는 Filament로
그리고, 사람뿐 아니라 AI 에이전트도 다룰 수 있게 장면을 설계합니다.

[English](../../README.md)

- **2D와 3D가 같은 모델입니다.** pythonx-compose UI처럼 함수, `state(...)`, 재선언으로 장면을
  선언합니다. 3D 장면은 노드가 Filament 엔티티인 Compose 컴포지션입니다.
- **프레임 루프는 네이티브에서 돕니다.** 움직임, 물리, 행동을 데이터로 선언하고, Python은 매 프레임이
  아니라 이벤트가 있을 때만 실행됩니다.
- **에이전트를 위한 설계입니다.** 모든 장면은 타입이 있는 문서입니다. 에이전트가 장면을 설명받고,
  질의하고, 엔티티 ID 버퍼로 픽셀과 연결하고, 검증되고 되돌릴 수 있는 트랜잭션으로 수정하고, 창 없이
  렌더링하고, 재생할 수 있습니다.

상태: 스펙 단계입니다. 아직 코드는 없습니다. [`docs/INTENT.md`](../INTENT.md)와
[`docs/SPEC.md`](../SPEC.md)를 읽어 주세요.

배포되면 이렇게 설치합니다.

```bash
uv add --prerelease allow pythonx-graphics
```

Apache-2.0 라이선스입니다.
