[README.md](https://github.com/user-attachments/files/35416918/README.md)
# Windows 11 보안 점검·강화 도구

PySide6 기반 GUI. 취약한 Windows 11 보안 설정을 점검하고,
- **On/Off 스위치로 해결 가능한 항목** → 스위치 한 번으로 적용/원복
- **스위치로 해결 불가능한 항목** → 현재 상태 점검 + 단계별 가이드(설정 앱 바로가기 포함)

## 파일 구성
| 파일 | 설명 |
|---|---|
| `win11_security_toolkit.py` | PySide6 UI (진입점) |
| `seccore.py` | 점검/적용/복원 로직 및 항목 정의 |
| `version_info.txt` | exe 속성 정보 (파일/제품 버전 1.00, 저작권 K.S Ju) |
| `build.bat` | PyInstaller 단일 exe 빌드 (`--onefile --windowed --uac-admin`) |

## 실행 / 빌드
```bat
pip install -r requirements.txt
python win11_security_toolkit.py     :: 개발 중 실행
build.bat                            :: dist\Win11SecToolkit.exe 생성
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

## 항목 추가 방법 (`seccore.py`)
레지스트리 항목은 `Item(...)` 에 `regs=[R(키, 값이름, 보안값, off=복원기본값)]` 만 추가하면 됩니다.
PowerShell 기반은 `check_fn / apply_fn / restore_fn` 을 지정하고, 수동 항목은 `guide=Guide(...)` 를 지정하세요.

## 버전 / 업데이트
- 버전 변경 시 `win11_security_toolkit.py`의 `APP_VERSION`, `version_info.txt`(filevers/prodvers/FileVersion/ProductVersion)를 함께 수정하세요.
- 상단 **업데이트** 버튼은 서버의 JSON 파일(`REMOTE_CONFIG_URL`, 현재 `https://naver.me/56Xxzjkl`)을 내려받아 `download_url` 값을 기본 브라우저로 엽니다. (조회는 백그라운드로 처리되어 창이 멈추지 않으며, 타임아웃 8초)
  ```json
  { "download_url": "https://새-다운로드-주소" }
  ```
  - 이 주소는 **JSON 파일을 직접 내려받는 주소**여야 합니다. 로그인/미리보기 같은 웹 페이지가 열리는 링크는 읽을 수 없습니다. (리다이렉트는 따라갑니다)
  - 서버 조회에 실패하면(오프라인, JSON 아님, 값 오류 등) 사유를 보여 주고, 확인 후 **폴백 링크**를 엽니다. 폴백 순서: exe 옆 `update.json`의 `download_url` → 코드의 `DEFAULT_UPDATE_URL`
  - `download_url` 은 `http(s)://` 주소만 허용됩니다.
  - JSON 주소 자체를 바꾸려면 `REMOTE_CONFIG_URL` 을 수정하고 다시 빌드하세요. (다운로드 링크는 서버 JSON만 고치면 재빌드 불필요)
  - `build.bat` 은 빌드 후 폴백용 `update.json` 을 `dist\` 폴더로 복사합니다.

## 시작 시 점검 화면
- 프로그램 시작(및 '다시 검사') 시 상단 녹색 Bar에 `전체 보안 상태 점검 중! - 잠시 기다려주세요!` 메시지가 점멸하고 Bar가 진행률만큼 채워집니다.
- 점검이 끝나 Bar가 100%로 표시될 때까지 창 안의 모든 버튼·검색/필터 컨트롤·항목 스위치·가이드 버튼이 비활성화됩니다. 완료 후 Bar는 최종 보안 점수로 바뀝니다.
