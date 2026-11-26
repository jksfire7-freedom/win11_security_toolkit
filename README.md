[README.md](https://github.com/user-attachments/files/35484876/README.md)
# Windows 11 보안 점검·강화 도구

PySide6 기반 GUI. 취약한 Windows 11 보안 설정을 점검하고,
- **On/Off 스위치로 해결 가능한 항목** → 스위치 한 번으로 적용/원복
- **스위치로 해결 불가능한 항목** → 현재 상태 점검 + 단계별 가이드(설정 앱 바로가기 포함)

## 파일 구성
| 파일 | 설명 |
|---|---|
| `win11_security_toolkit.py` | 프로그램 동작 로직 (진입점) – 화면은 `ui/*.ui` 를 불러와 사용 |
| `ui/main_window.ui` | 메인 창 (헤더/보안 점수 Bar, 툴바, 필터 안내, 탭, 로그) + 전체 스타일시트(QSS) |
| `ui/item_card.ui` | 항목 카드 1개 (제목/위험도 배지/설명/상태/스위치/가이드 버튼) |
| `ui/guide_dialog.ui` | 수동 가이드 창 |
| `ui_loader.py` | `.ui` 파일 로더 (위젯 objectName → `self.<objectName>`, 이름 충돌 검사) |
| `toggle_switch.py` | On/Off 토글 스위치 위젯 (`.ui` 에서 승격 위젯으로 사용) |
| `report_builder.py` | 보고서 생성 (HTML/CSS, Markdown/텍스트) – GUI 와 분리 |
| `seccore.py` | 공통 유틸, 레지스트리(Reg), 백업 저장소(Store), 항목 모델(Item) |
| `handlers.py` | PowerShell/서비스/감사 등 레지스트리 외 항목의 점검·적용·복원 |
| `items_auto.py` | On/Off 자동 항목 정의 |
| `items_guides.py` | 수동 가이드 항목 정의 |
| `sysinfo.py` | 점검 대상 PC 정보 수집 (보고서용) |
| `catalog.py` | 자동 항목 + 가이드를 합친 전체 목록 |
| `version_info.txt` | exe 속성 정보 (파일/제품 버전 2.0.0.0, 저작권 K.S Ju) |
| `Win11SecToolkit.spec` | **빌드 설정(단일 기준)** – `--onefile --windowed --uac-admin`, `ui`·`app.ico` 포함 |
| `build.bat` | 위 `.spec` 으로 PyInstaller 빌드 (`build.bat nopip` = pip 설치 생략) |

## 실행 / 빌드
```bat
pip install -r requirements.txt
python win11_security_toolkit.py     :: 개발 중 실행
build.bat                            :: dist\Win11SecToolkit.exe 생성
build.bat nopip                      :: pip 설치/업그레이드 생략 (오프라인·이미 준비된 환경)
```

## 개인 정보 보호 항목 (v1.1)
- **On/Off 16개**: 진단 데이터 최소화, DiagTrack 중지, 광고 ID, 맞춤형 추천, 위치, 카메라/마이크 차단, 활동 기록, 클립보드 기록, Recall, 입력 개인화, 웹(Bing) 검색 전송, 검색/최근 문서 기록, 잠금 화면 알림, 오류 보고
- **수동 가이드 7개**: 계정/동기화, 앱별 권한, 진단 데이터 삭제, MS 개인 정보 대시보드, DNS 암호화(DoH), 브라우저 설정, PC 폐기 전 데이터 삭제
- 툴바의 **분류 필터**로 개인 정보 보호 항목만 모아 볼 수 있고, 상단에 개인 정보 보호 적용 현황이 표시됩니다.
- 사용자(HKCU) 설정은 **프로그램을 실행한 계정** 기준으로 적용됩니다. (다른 관리자 계정으로 승격 실행하면 그 계정에 적용됨)
- 카메라/마이크 차단 등 기능에 영향이 있는 항목은 경고가 표시되고 일괄 적용에서 제외됩니다.

## 동작 방식
- 설정 변경에는 **관리자 권한**이 필요합니다. (exe는 manifest로 자동 요청, 스크립트 실행 시 재시작 안내)
- 적용 직전의 **원래 값은 `%ProgramData%\Win11SecToolkit\backup.json`** 에 저장되며, 스위치를 Off 하면 그 값으로 복원됩니다.
- 로그: `%ProgramData%\Win11SecToolkit\toolkit.log`
- 호환성 영향이 있는 항목은 적용 전 경고창이 뜨고, "안전한 항목 일괄 적용"에서는 제외됩니다.
- 변경 전 "복원 지점" 버튼으로 시스템 복원 지점을 만들 수 있습니다.
- Windows가 아닌 환경에서는 UI 확인용 데모 모드로 실행됩니다.

## 화면(GUI) 수정 – Qt Designer
```bat
pyside6-designer ui\main_window.ui     :: 또는 item_card.ui / guide_dialog.ui
```
- 저장 후 `python win11_security_toolkit.py` 로 바로 확인할 수 있습니다. (빌드 단계 추가 없음, 실행 시 `.ui` 를 읽음)
- 코드가 참조하는 위젯 이름(objectName)은 **바꾸지 마세요.** (예: `bar`, `summary`, `search`, `btn_scan`, `tabs`, `sw`, `state_lbl` …) 위치·크기·문구·색은 자유롭게 바꿀 수 있습니다.
- 파란 강조 버튼은 동적 속성 `primary = true` 로 지정됩니다. (스타일시트 `QPushButton[primary="true"]`)
- 코드에서 정하는 값: 항목 수만큼의 카드, 분류 목록, 탭 제목의 개수, 업데이트 버튼의 버전 문구, 데모/읽기 전용 안내 배너 문구.
- `item_card.ui` 의 `ToggleSwitch` 는 승격 위젯입니다. Designer 에서는 일반 버튼으로 보이는 것이 정상입니다.

## 항목 추가 방법 (`items_auto.py` / `items_guides.py`)
레지스트리 항목은 `Item(...)` 에 `regs=[R(키, 값이름, 보안값, off=복원기본값)]` 만 추가하면 됩니다.
PowerShell 기반은 `check_fn / apply_fn / restore_fn` 을 지정하고, 수동 항목은 `guide=Guide(...)` 를 지정하세요.

## 버전 / 업데이트 (GitHub Releases API)
상단 **업데이트** 버튼은 GitHub API(`GET https://api.github.com/repos/{소유자}/{저장소}/releases/latest`)로 최신 릴리스를 조회합니다. 조회는 백그라운드로 처리되어 창이 멈추지 않습니다. (타임아웃 8초, 인증 불필요)

**최초 설정 (1회)**
1. `win11_security_toolkit.py` 의 `GITHUB_REPO = "OWNER/REPO"` 를 본인 **공개** 저장소(`소유자/저장소`)로 수정하고 빌드합니다.
2. 필요하면 `RELEASE_ASSET_NAME` 을 릴리스에 올릴 exe 이름과 맞춥니다. (기본 `Win11SecToolkit.exe`)

**새 버전 배포 순서**
1. `APP_VERSION` 과 `version_info.txt` 의 버전 값을 올리고 빌드합니다.
2. GitHub 저장소 → Releases → *Draft a new release* 에서 태그를 `v1.01` 형식으로 만들고, `dist\Win11SecToolkit.exe` 를 첨부해 **Publish** 합니다. (릴리스 본문은 '변경 사항'으로 표시됩니다)
3. 이후 사용자가 업데이트 버튼을 누르면:
   - 최신 태그가 현재 버전보다 높으면 → 새 버전 안내(변경 사항 포함) + **다운로드**/릴리스 페이지/나중에
   - 같거나 낮으면 → "현재 최신 버전을 사용 중입니다"
   - 릴리스에 exe 첨부가 없으면 → 릴리스 페이지를 엽니다

**안전·예외 처리**
- API 응답의 링크는 `https` 이면서 `github.com` / `githubusercontent.com` 도메인일 때만 사용합니다. (아니면 릴리스 페이지로 대체)
- 프로그램은 파일을 자동 설치하지 않고 **브라우저로 다운로드만** 엽니다.
- 오프라인, 저장소/릴리스 없음(404), API 요청 한도 초과(IP당 시간당 60회), 태그 형식 오류 등은 사유를 안내하고 릴리스 페이지 열기를 제안합니다.
- 비공개 저장소는 인증이 필요해 지원하지 않습니다.

## 보고서
'보고서 저장'에서 저장 형식을 고를 수 있습니다. 확장명 없이 이름만 입력하면 선택한 형식의 확장명이 붙습니다.
- **HTML 웹 페이지 (*.html)**: 브라우저에서 바로 열리는 보고서입니다. 점수 막대, 상태별 색상 배지, 어댑터별 네트워크 카드, 다크 모드, 모바일 화면, 인쇄(PDF 저장)를 지원합니다. 파일 하나로 완결되며 외부 파일/스크립트를 쓰지 않고, 모든 값은 이스케이프됩니다. 저장 후 브라우저로 열지 묻습니다.
- **Markdown (*.md) / Text (*.txt)**: 메모장·Markdown 뷰어용. UTF-8(BOM) + Windows 줄바꿈으로 저장됩니다.

모든 형식의 맨 위에 **점검 대상 PC 정보**가 들어갑니다: 컴퓨터 이름, 도메인/워크그룹, 계정, 계정 SID, 관리자 실행 여부, OS/버전, 활성 어댑터별 MAC·IPv4·IPv6·기본 게이트웨이·DNS 서버, 프로그램 버전. 계정·IP 등 식별 정보가 포함되므로 외부 공유 시 주의하세요. (프로그램을 실행한 계정 기준이며, 다른 관리자 계정으로 승격 실행하면 그 계정으로 표시됩니다)

## 시작 시 점검 화면
- 프로그램 시작(및 '다시 검사') 시 상단 녹색 Bar에 `전체 보안 상태 점검 중! - 잠시 기다려주세요!` 메시지가 점멸하고 Bar가 진행률만큼 채워집니다.
- 점검이 끝나 Bar가 100%로 표시될 때까지 창 안의 모든 버튼·검색/필터 컨트롤·항목 스위치·가이드 버튼이 비활성화됩니다. 완료 후 Bar는 최종 보안 점수로 바뀝니다.

## 2.0 변경 요약
- **원복 신뢰도**: SMBv1·Guest 는 원래 상태를 기억해 원래 꺼져 있던 것은 켜지 않습니다. 서비스 중지/시작·감사 정책은 실패하면 항목 실패로 표시하고, 원래 중지/감사 상태를 기억합니다. ASR 은 규칙별 원래 동작(감사/경고/해제)까지 복원합니다.
- **백업**: 이미 양호한 항목도 최초 원본을 백업하고(이미 백업이 있으면 최초 원본 유지) 원래 양호했던 항목은 Off 시 기존처럼 기본 해제 값으로 되돌리거나 값을 삭제해 보호를 해제하며, `backup.json` 은 임시 파일 → `os.replace` 로 원자적으로 저장합니다. 손상된 파일은 `.corrupt` 로 보관합니다.
- **점검 정확도**: 최신 패치 판정에 Windows Update 설치 기록·마지막 설치 시각을 함께 사용합니다. 허용 기준이 있는 항목은 툴팁에 '양호 판정 기준'이 표시됩니다.
- **종료 안정성**: 작업/업데이트 스레드가 끝날 때까지 안전하게 기다린 뒤 닫고(한 번 더 닫으면 강제 종료), 완료 타이머도 정리합니다.
- **UX**: 필터 결과가 현재 탭에 없으면 안내와 '다른 탭으로 이동' 버튼, 점수 산정 기준 툴팁, 일괄 적용 확인창(재부팅 표시·복원 지점 권고), 업데이트 창에 다운로드 파일·SHA-256 안내.
