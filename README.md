# SRT 화이트보드 애니메이션 Skill — 한국어판

> [geeklee/srt-whiteboard-animation](https://github.com/geeklee/srt-whiteboard-animation)(MIT)의 포크입니다.
> 한국어로 옮기고, 원본에 있던 렌더 크래시를 고쳤습니다. 그리기 알고리즘은 원본 그대로입니다.

## 원본 대비 바뀐 점

| 변경 | 내용 |
|---|---|
| **버그 수정** | 영역 안에 그릴 선이 하나도 없으면 렌더가 통째로 죽던 문제. `render_stream_whiteboard.py` 405줄에서 `_lay_ink`에 잉여 인자를 넘기고 있었다 (`TypeError`). 재현·수정·검증 완료 |
| **한국어화** | SKILL.md 전문, 콘솔 출력·도움말·에러 메시지, 내부 주석, 구역 편집기(`assets/preview.html`) UI. 코드 로직은 무변경(AST 동일) |
| **폰트 경로** | `render_annotation_preview.py`가 Windows 전용 폰트 경로만 봐서 macOS·리눅스에서 실행 불가였다. 크로스플랫폼 폴백 추가 |
| **손 이미지** | `assets/drawing-hand.png` 펜대에 새겨져 있던 원저작자 서명 제거 (원본은 `drawing-hand.orig.png`로 보존) |

라이선스는 MIT 그대로이고 `LICENSE`의 원저작자 저작권 표기를 유지합니다.

---

SRT 자막을 이야기 순서대로 그려지는 화이트보드 손그림 영상으로 바꿔 주는 Skill입니다. **구역별 마스크 연출**과 **스트리밍 펜 선 그리기**를 결합했습니다. 각 요소가 자막을 따라 차례로 등장하고, 펜 끝이 구역 안에서 연속으로 선을 긋고, 이어서 색을 조금씩 입힌 뒤 최종적으로 MP4로 내보냅니다.

지식 설명, 스토리 내레이션, 강의 자막, 숏폼 영상 대본을 따뜻한 미색(베이지) 종이 배경의 손그림 애니메이션으로 만들기에 적합합니다.

## 예시

**장면: 원숭이산 바나나 뺏기** — 자막의 이야기 순서에 따라 바위산과 작은 원숭이, 바나나를 뺏는 큰 원숭이, 구경하는 아이들을 차례로 그립니다.

![원숭이산 바나나 뺏기: SRT 화이트보드 애니메이션 데모](examples/scene-01-monkey-mountain-stream.gif)

원본 선화: [PNG 보기](examples/scene-01-monkey-mountain.png)

## 핵심 기능

- SRT 자막을 해석하고, 권장 길이 25–35초 단위로 장면 분할
- 먼저 분할표(스토리보드)와 그림 전략을 출력해, 한 장면에 핵심 의미 하나만 담기도록 보장
- 화면 좌표가 아니라 자막 이벤트 기준으로 요소의 그리기 순서를 의미 단위로 구성
- `annotation.json`으로 구역, 타이밍, 자막 연결, 겹침 보호 구역 관리
- 각 구역은 연속 스트리밍 펜 선 방식: 먼저 `ink`로 선화를 깔고, 그다음 `color`로 채색
- 브라우저 미리보기 도구에서 구역·순서·시간·자막 연결 조정 가능
- 장면별 렌더링과 여러 장면 병합을 지원, 완성된 MP4 출력

## 작동 방식

이 Skill의 핵심은 "자막이 주도하고, 단계마다 확인"입니다. 각 단계가 끝날 때마다 확인을 기다려서, 스토리보드·선화·주석(annotation)이 확정되기 전에 렌더링 비용을 낭비하지 않습니다:

1. SRT를 해석해 스토리보드와 그림 전략을 출력.
2. 확인 후 통일된 스타일의 선화를 생성.
3. 선화 확인 후, 자막과 원본 이미지를 결합해 주석(annotation)을 만들고 미리보기 도구에 로드.
4. 주석 확인 후, 구역·방향 검사용 이미지 생성.
5. 미리보기 도구에서 구역, 이야기 순서, 타이밍, 자막 연결을 조정하고 저장.
6. 최종 주석 확인 후, 장면별로 MP4 렌더링.
7. 여러 장면 프로젝트는 각 장면 완성본 확인 후 병합.

## 비주얼 규칙

- 따뜻한 미색 종이 배경: 권장 `#F5EBD7`
- 짙은 회색 스케치 선, 빨강·주황·파랑은 개념 강조용으로만 소량 사용
- 미니멀 손그림, 깨끗한 배경, 충분한 여백
- 장면 내 텍스트, 라벨, 사진 느낌, 3D 효과, 복잡한 질감은 사용하지 않음

## 설치와 환경

Skill에는 독립된 Python 가상환경 준비 스크립트가 포함되어 있습니다. 처음 실행할 때:

```bash
python scripts/prepare_env.py --check
python scripts/prepare_env.py
```

성공하면 첫 번째 명령이 `ENV_PY=<경로>`를 출력합니다. 이후 렌더링에는 그 인터프리터를 사용해 의존성을 격리하세요.

## 프로젝트 소재 구조

```text
assets/whiteboard/<프로젝트명>/
├── scene-01-<이름>.png
├── scene-01-<이름>.annotation.json
├── scene-01-<이름>-whiteboard.mp4
└── scene-01-<이름>-preview.mp4
```

이미지와 주석 파일은 이름이 같아야 합니다. 예: `scene-01-demo.png` ↔ `scene-01-demo.annotation.json`

## 주석(annotation) 형식

각 요소는 원본 이미지의 정수 픽셀 좌표를 사용하고, `sequence`, `subtitle`, `narrativeRole`로 자막의 이벤트와 연결합니다. 구역은 "장면 배경 → 핵심 인물/사물 → 동작이나 변화 → 반응/결과" 순서로 정렬해야 합니다.

```json
{
  "sceneId": "scene-01",
  "canvas": { "width": 1672, "height": 941 },
  "storyBasis": "작은 원숭이가 원숭이산 위에서 바나나를 들고 있고, 큰 원숭이가 바나나를 뺏고, 아이들이 옆에서 구경한다.",
  "sceneDurationMs": 9000,
  "elements": [
    {
      "id": "rockery",
      "label": "원숭이산 배경",
      "sequence": 1,
      "narrativeRole": "이야기의 장면 배경",
      "subtitle": "작은 원숭이가 원숭이산 꼭대기에 앉아 바나나를 들고 있다.",
      "type": "structure",
      "region": { "x": 20, "y": 120, "width": 540, "height": 780 },
      "reveal": {
        "direction": "top_to_bottom",
        "startMs": 300,
        "durationMs": 2600,
        "maskPaddingPx": 22,
        "protectedRegions": []
      },
      "handPath": { "start": [290, 130], "end": [290, 890], "easing": "easeInOut" }
    }
  ]
}
```

`direction`과 `handPath`는 미리보기 도구의 사각형 대리 표시용입니다. 최종 영상의 실제 펜 선은 스트리밍 그리기 엔진이 자동 생성합니다. 서로 가려지는 대상은, 먼저 그려지는 요소의 `protectedRegions`에 나중에 보여야 할 구역을 표시해서 뒤 내용이 미리 드러나지 않게 합니다.

## 자주 쓰는 명령

자막 해석 + 장면 분할 제안:

```bash
python scripts/parse_srt.py <자막.srt> --target-sec 30 --min-sec 25 --max-sec 35
```

구역 검사 이미지 생성:

```bash
python scripts/render_annotation_preview.py <이미지경로> <주석경로> <검사이미지출력경로>
```

`assets/preview.html`을 열고 "폴더 열기"로 장면 폴더를 불러오면 구역, 순서, 시간, 자막 연결을 편집할 수 있습니다.

단일 장면 렌더링:

```bash
<ENV_PY> scripts/render_stream_whiteboard.py <이미지경로> <주석경로> <출력.mp4> assets/drawing-hand.png \
  --ink-path grid --color-fill contour-wipe
```

여러 장면 병합:

```bash
<ENV_PY> scripts/merge_scenes.py --inputs 장면1.mp4 장면2.mp4 장면3.mp4 --output final.mp4
```

## 품질 체크

- 첫 프레임은 깨끗한 미색 종이 배경이고, 미리 드러난 선이 없어야 함
- `canvas`가 원본 이미지 크기와 일치하고, 모든 구역이 캔버스 안의 정수 픽셀 좌표여야 함
- `sequence`, `startMs`가 자막의 이야기 순서와 일치해야 함
- 중간 프레임에서, 아직 시작하지 않은 구역과 보호 구역이 미리 나타나면 안 됨
- 펜 끝이 현재 그려지는 선에 밀착해야 함. 선화가 깔끔하면 `--ink-path skeleton` 선택 가능
- 각 장면은 끝난 후 완성 화면에서 최소 0.5초 머무름. 병합 순서는 자막 스토리보드와 일치해야 함

## 저장소 구성

```text
srt-whiteboard-animation/
├── SKILL.md                          # 전체 워크플로와 제약
├── assets/
│   ├── drawing-hand.png              # 손 이미지 소재
│   ├── preview.html                  # 로컬 편집 미리보기 도구
├── examples/                         # README 예시 소재
├── scripts/
│   ├── parse_srt.py                  # 자막 해석 + 장면 분할 제안
│   ├── render_annotation_preview.py  # 주석 검사 이미지
│   ├── render_stream_whiteboard.py   # 스트리밍 펜 선 MP4 렌더러
│   ├── merge_scenes.py               # 여러 장면 병합
│   └── prepare_env.py                # 의존성 환경 준비
└── agents/openai.yaml                # Codex 메타데이터
```

## 기여

Issue나 Pull Request를 환영합니다. 그리기 로직을 건드리는 변경은, 실제 자막·주석·완성 영상으로 마스크 보호, 타이밍, 최종 화면을 검사해야 합니다.

## 라이선스

MIT License로 공개되어 있습니다. [LICENSE](LICENSE) 참고.

## 원작자

물고기 키우기를 좋아하는 아저씨 / AI Builder / AI 팀으로 1인 회사를 만드는 중.

抖音(더우인), B站(빌리빌리), 위챗 공식계정: 江哥是老登啊
