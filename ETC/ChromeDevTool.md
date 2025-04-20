# 크롬 개발자 도구 가이드

## Elements 탭

1. **실시간으로 수정 가능**

   - **사용 방법**:

     1. Elements 패널에서 수정할 요소 선택
     2. 요소를 더블 클릭하거나 우클릭 후 "Edit as HTML" 선택
     3. HTML 코드 직접 수정 후 Enter 키 또는 바깥쪽 클릭으로 변경 적용
     4. 속성만 변경하려면 속성 이름이나, 값을 더블 클릭하여 수정 가능

   - **사용 목적**: 페이지 레이아웃, 콘텐츠, 스타일을 실시간으로 수정하여 즉시 결과 확인

2. **'$0'~'$4'까지의 전역 변수에 최근 클릭한 Node 저장**

   - **사용 방법**:

     1. Elements 패널에서 DOM 요소 클릭 (최대 5개까지 기록)
     2. Console 패널로 전환
     3. Console에서 $0 입력하여 가장 최근 선택 요소 접근 ($1~$4는 이전 선택 요소)
     4. $0.classList, $0.getAttribute('src') 등으로 속성 확인 가능
     5. 예: `$0.style.color = 'red'`로 직접 조작 가능

   - **사용 목적**: 선택한 DOM 요소에 빠르게 접근하여 속성 확인 및 조작

3. **엘리먼트에 hover/focus를 강제 적용**

   - **사용 방법**:

     1. Elements 패널에서 상태를 적용할 요소 선택
     2. 우측 Styles 패널에서 :hov 버튼(또는 :toggle 버튼) 클릭
     3. 표시된 체크박스 목록에서 원하는 상태(hover, active, focus, visited) 선택
     4. 해당 상태가 적용된 스타일을 실시간으로 확인
     5. 체크박스를 해제하면 원래 상태로 돌아감

   - **사용 목적**: hover, active, focus 등 다양한 상태의 스타일을 테스트

4. **엘리먼트에 변경을 일으키는 스크립트에 Breakpoint 걸기**

   - **사용 방법**:

     1. Elements 패널에서 대상 요소 선택 후 우클릭
     2. "Break on" 메뉴 선택
     3. 원하는 옵션 선택:
        - "Subtree modifications": 하위 요소 변경 시 중단
        - "Attribute modifications": 속성 변경 시 중단
        - "Node removal": 요소 제거 시 중단
     4. 이벤트 발생 시 Sources 패널로 자동 전환되며 해당 코드에서 중단
     5. 브레이크포인트 비활성화: Elements > DOM Breakpoints 패널에서 체크 해제

   - **사용 목적**: DOM 변경, 속성 변경 등이 발생할 때 자동으로 디버깅 시작

5. **실시간 CSS 스타일 수정**

   - **사용 방법**:

     1. Elements 패널에서 요소 선택
     2. 우측 Styles 패널에서:
        - 기존 속성 값 클릭하여 직접 수정
        - 속성 이름/값 사이 공간 클릭하여 새 속성 추가
        - 속성 좌측 체크박스로 속성 일시 비활성화
        - filter 텍스트 상자로 속성 필터링
        - color picker, shadow editor 등 내장 도구 사용
     3. 변경사항은 즉시 페이지에 적용됨
     4. 새 스타일 규칙 추가는 + 버튼 클릭

   - **사용 목적**: CSS 변경사항을 즉시 확인하여 디자인 작업 효율화

6. **페이지의 모든 이벤트 리스너 확인**

   - **사용 방법**:

     1. Elements 패널에서 요소 선택
     2. 우측 패널에서 Event Listeners 탭 클릭
     3. 이벤트 유형별로 그룹화된 리스너 목록 확인 (click, mouseout 등)
     4. 리스너 옆 > 화살표 클릭하여 세부 정보 펼치기
     5. "Remove" 버튼으로 리스너 제거 가능
     6. 옵션:
        - "Ancestors" 체크박스: 상위 요소의 이벤트도 표시
        - "Framework listeners" 체크박스: 프레임워크(React 등)의 리스너 표시
     7. 이벤트 핸들러 코드 클릭 시 Sources 패널로 이동하여 코드 확인 가능

   - **사용 목적**: 특정 요소에 연결된 이벤트와 처리 함수 식별 및 디버깅

## Console 탭

1. **컨텍스트 선택 영역**

   - **사용 방법**:

     1. Console 패널 상단의 실행 컨텍스트 드롭다운 메뉴 클릭
     2. 실행 컨텍스트 옵션 선택:
        - top (기본 페이지)
        - iframe 목록 (페이지 내 iframe)
        - 웹 워커 (백그라운드 스레드)
        - 서비스 워커
        - 확장 프로그램 컨텍스트
     3. 콘솔에 입력하는 모든 명령은 선택한 컨텍스트에서 실행됨
     4. 각 컨텍스트는 독립적인 변수 스코프와 전역 객체를 가짐

   - **사용 목적**: iframe, 워커 등 다양한 컨텍스트에서 코드 실행 가능

2. **Live Expression 생성 버튼**

   - **사용 방법**:

     1. Console 패널 상단의 눈 모양 아이콘(Create live expression) 클릭
     2. 모니터링할 JavaScript 표현식 입력 (예: `document.querySelector('.count').innerText`)
     3. 입력 완료 후 Enter 키 또는 바깥쪽 클릭
     4. 표현식의 값이 실시간으로 표시되며 값 변경 시 즉시 업데이트
     5. 표현식 수정: 표현식 클릭
     6. 삭제: 표현식 우측의 X 버튼 클릭
     7. 여러 개의 Live Expression 추가 가능

   - **사용 목적**: 전역에서 접근 가능한 상태값을 지속적으로 모니터링

3. **Filter 영역**

   - **사용 방법**:

     1. Console 패널 상단의 필터 텍스트 상자 클릭
     2. 필터링 옵션:
        - 텍스트 입력: 로그 메시지 내 텍스트 검색
        - 정규식 사용: /pattern/ 형태로 입력
        - 레벨 필터: 상단 드롭다운에서 error, warning, info 등 선택
        - 부정 필터: - 접두사 사용 (예: -error)
        - 고급 필터: url: 접두사로 특정 URL의 로그만 표시
     3. 필터 지우기: 텍스트 박스 우측의 X 버튼 클릭
     4. 여러 필터 조합 가능 (예: "error -network")

   - **사용 목적**: 로그 메시지를 텍스트로 검색하여 필요한 정보만 표시

4. **로깅 유틸리티**

   - **사용 방법**:

     1. Console 패널에서 다음 메서드 사용:
        - `console.log(obj)`: 기본 로깅
        - `console.error(obj)`: 빨간색 오류 메시지
        - `console.warn(obj)`: 노란색 경고 메시지
        - `console.info(obj)`: 정보 메시지(info 아이콘)
        - `console.table(array/obj)`: 표 형식으로 데이터 표시
        - `console.group('그룹명')`, `console.groupEnd()`: 로그 그룹화
        - `console.time('라벨')`, `console.timeEnd('라벨')`: 실행 시간 측정
        - `console.trace()`: 스택 트레이스 출력
        - `console.assert(조건, 메시지)`: 조건이 거짓일 때만 메시지 출력
     2. 서식 지정: `console.log('%c텍스트', 'color:red; font-size:20px');`
     3. 변수 대체: `console.log('이름: %s, 나이: %d', 이름, 나이);`

   - **사용 목적**: 다양한 형식으로 데이터를 로깅하여 디버깅

5. **명령어 히스토리**

   - **사용 방법**:

     1. Console 패널에서 위/아래 화살표 키 사용
     2. 위 화살표: 이전에 실행한 명령어로 이동
     3. 아래 화살표: 다음 명령어로 이동
     4. 명령어 수정: 좌/우 화살표와 백스페이스 사용
     5. 명령어 실행: Enter 키
     6. 기록 지우기: Console.clear() 실행 또는 우클릭 > Clear console
     7. 탭 키로 자동 완성 사용 가능
     8. 여러 줄 입력: Shift+Enter로 줄바꿈

   - **사용 목적**: 이전에 실행한 명령어를 빠르게 재사용

## Source 탭

1. **Page 탭**

   - **사용 방법**:

     1. Sources 패널 열기
     2. 좌측 네비게이션 패널에서 Page 탭 선택
     3. 트리 구조로 표시된 파일 탐색:
        - 최상위 도메인: 현재 페이지의 리소스
        - 외부 도메인: 외부에서 로드된 리소스
     4. 파일 열기: 파일명 클릭
     5. 파일 검색: Ctrl+P (Mac: Cmd+P) 누른 후 파일명 입력
     6. 코드 내 검색: Ctrl+F (Mac: Cmd+F)
     7. 파일 내용 편집: 코드 영역 클릭 후 수정
     8. 변경사항 저장: Ctrl+S (Mac: Cmd+S)

   - **사용 목적**: 페이지에 로드된 모든 리소스(JS, CSS 등) 탐색 및 편집

2. **Overrides**

   - **사용 방법**:

     1. Sources 패널에서 Overrides 탭 선택
     2. "+ Select folder for overrides" 클릭
     3. 로컬 폴더 선택 (페이지 리소스를 저장할 위치)
     4. "Allow" 버튼 클릭하여 권한 부여
     5. Page 탭으로 돌아가 수정할 파일 열기
     6. 파일 수정 후 Ctrl+S (Mac: Cmd+S)로 저장
     7. 수정된 파일은 자동으로 Overrides 폴더에 저장되고 페이지 새로고침 후에도 유지됨
     8. 비활성화: Overrides 탭의 "Enable Local Overrides" 체크박스 해제

   - **사용 목적**: 개발자도구의 변경 사항을 새로고침 이후에도 유지

3. **Snippets**

   - **사용 방법**:

     1. Sources 패널의 좌측 네비게이션에서 Snippets 탭 선택
     2. "+ New snippet" 클릭하여 새 스니펫 생성
     3. 스니펫 이름 지정 (우클릭 > "Rename")
     4. 코드 영역에 JavaScript 코드 작성
     5. 실행 방법:
        - 우클릭 > "Run" 선택
        - Ctrl+Enter (Mac: Cmd+Enter) 단축키
        - 스니펫 우측의 실행 버튼(▶) 클릭
     6. 스니펫 관리:
        - 저장: Ctrl+S (Mac: Cmd+S)
        - 삭제: 우클릭 > "Remove"
        - 복제: 우클릭 > "Duplicate"
     7. 스니펫은 브라우저 프로필에 저장되어 세션 간에 유지됨

   - **사용 목적**: 테스트나 자주 사용하는 코드 스니펫을 저장 및 실행

4. **Breakpoint, Conditional Breakpoint, Logpoint**

   - **사용 방법**:

     1. Sources 패널에서 디버깅할 파일 열기
     2. 중단점 설정:
        - 일반 중단점: 라인 번호 클릭
        - 조건부 중단점: 라인 번호 우클릭 > "Add conditional breakpoint" > 조건식 입력 (예: `count > 5`)
        - 로그 포인트: 라인 번호 우클릭 > "Add logpoint" > 로그 메시지 입력 (예: `값: ${variable}`)
     3. 중단점 관리:
        - 활성화/비활성화: 중단점 체크박스 토글
        - 삭제: 중단점 우클릭 > "Remove breakpoint" 또는 중단점을 멀리 드래그
        - 모든 중단점 비활성화: 우측 패널의 Breakpoints 섹션에서 "Deactivate breakpoints" 버튼
        - 모든 중단점 삭제: 우클릭 > "Remove all breakpoints"
     4. 디버깅 중 코드 실행 제어는 디버거 툴바 사용

   - **사용 목적**: 특정 조건에서 코드 실행을 중단하거나 로그 출력

5. **Blackbox Script**

   - **사용 방법**:

     1. Sources 패널에서 제외할 스크립트 파일 열기
     2. 파일 내용 우클릭 > "Blackbox script" 선택
     3. 또는 우측 패널의 "Call Stack" 섹션에서:
        - 스택 항목 우클릭 > "Blackbox script"
     4. 일괄 설정: 설정(⚙️) > Preferences > Blackboxing > "Add pattern" 버튼 > 패턴 입력
        - 예: `node_modules/.*` (정규식 형태로 입력)
     5. 블랙박스 적용 효과:
        - 디버거가 해당 스크립트 내부에서 중단되지 않음
        - 콜 스택에서 해당 스크립트의 함수 호출이 축소됨
     6. 비활성화: 같은 메뉴에서 "Stop blackboxing script" 선택

   - **사용 목적**: 라이브러리 등 디버깅이 필요 없는 스크립트를 디버거에서 제외

6. **Debugger 툴바**

   - **사용 방법**:

     1. Sources 패널 상단의 디버거 툴바 사용
     2. 중단점 활성화된 상태에서 페이지 실행하면 툴바 활성화
     3. 주요 버튼 기능:
        - Resume script execution (F8): 다음 중단점까지 계속 실행
        - Step over next function call (F10): 현재 함수 내에서 다음 줄로 이동
        - Step into next function call (F11): 함수 내부로 진입
        - Step out of current function (Shift+F11): 현재 함수에서 빠져나옴
        - Step (F9): 다음 JavaScript 구문으로 이동
        - Deactivate breakpoints: 모든 중단점 일시 비활성화
        - Pause on exceptions: 예외 발생 시 중단
     4. 디버깅 중 변수 검사:
        - 변수에 마우스 오버하여 값 확인
        - Scope 패널에서 현재 스코프의 모든 변수 확인
        - Watch 패널에서 특정 표현식 모니터링

   - **사용 목적**: 코드 실행을 제어하며 단계별로 디버깅

## Network 탭

1. **Disable cache**

   - **사용 방법**: Network 패널에서 "Disable cache" 체크박스 활성화
   - **사용 목적**: 캐시된 리소스를 무시하고 항상 서버에서 새로 요청

2. **연결 상태(네트워크 종류) 변경**

   - **사용 방법**: 네트워크 속도 드롭다운(No throttling)에서 선택
   - **사용 목적**: 다양한 네트워크 환경(3G, 4G 등)에서 사이트 성능 테스트

3. **저장/불러오기**

   - **사용 방법**: 네트워크 요청 우클릭 > Save all as HAR
   - **사용 목적**: 네트워크 활동을 HAR 파일로 저장하여 분석 또는 공유

4. **하단 상태 표시 바**

   - **사용 방법**: Network 패널 하단의 요약 정보 확인
   - **사용 목적**: 총 요청 수, 전송된 데이터 양, 로드 시간 등 성능 지표 확인

5. **요청 필터링**
   - **사용 방법**: 상단의 필터 입력란 또는 미리 정의된 필터 사용
   - **사용 목적**: XHR, JS, CSS 등 특정 유형의 요청만 표시

## Performance 탭

1. **저장/불러오기**

   - **사용 방법**: 레코딩 후 "Save profile" 버튼 클릭
   - **사용 목적**: 성능 측정 데이터를 저장하여 나중에 분석하거나 팀과 공유

2. **메모리 프로파일**

   - **사용 방법**: 설정에서 "Memory" 체크박스 활성화 후 레코딩
   - **사용 목적**: 시간에 따른 메모리 사용량 추적, 메모리 누수 식별

3. **레코딩 시작/중지**

   - **사용 방법**: Record 버튼 클릭 후 작업 수행, Stop 버튼으로 종료
   - **사용 목적**: 사용자 상호작용 중 성능 데이터 수집

4. **CPU/네트워크 조절**
   - **사용 방법**: CPU 또는 Network 드롭다운에서 제한 선택
   - **사용 목적**: 다양한 하드웨어/네트워크 환경에서의 성능 시뮬레이션

## Memory 탭

1. **Heap snapshot**

   - **사용 방법**: "Take snapshot" 버튼 클릭
   - **사용 목적**: 메모리에 있는 객체의 스냅샷을 생성하여 참조 변수가 수거되지 않은 메모리 누수 확인

2. **Allocation instrumentation on timeline**

   - **사용 방법**: "Start" 버튼 클릭 후 타임라인 기록
   - **사용 목적**: 시간에 따른 JS 객체의 메모리 할당 패턴 분석

3. **Allocation Sampling**

   - **사용 방법**: "Start" 클릭 후 샘플링 기록
   - **사용 목적**: 메모리 할당을 담당하는 함수를 식별하여 최적화 대상 파악

4. **Select JavaScript VM instance**
   - **사용 방법**: 드롭다운에서 VM 인스턴스 선택
   - **사용 목적**: 여러 컨텍스트(메인 페이지, iframe 등)의 메모리 사용량 모니터링

## Application 탭

1. **로컬 스토리지/세션 스토리지 관리**

   - **사용 방법**: 좌측 네비게이션에서 Storage > Local/Session Storage 선택
   - **사용 목적**: 웹 스토리지에 저장된 데이터 확인, 수정, 삭제

2. **쿠키 관리**

   - **사용 방법**: Storage > Cookies 선택
   - **사용 목적**: 사이트 쿠키 확인, 수정, 삭제

3. **캐시 저장소 확인**

   - **사용 방법**: Cache > Cache Storage 선택
   - **사용 목적**: Service Worker 캐시 내용 확인

4. **Service Workers 관리**

   - **사용 방법**: Application > Service Workers 선택
   - **사용 목적**: 등록된 Service Worker 확인, 업데이트, 디버깅

5. **매니페스트 확인**
   - **사용 방법**: Application > Manifest 선택
   - **사용 목적**: PWA의 웹 앱 매니페스트 정보 확인

## 유용한 단축키

- **개발자 도구 열기**: F12 또는 Ctrl+Shift+I (Mac: Cmd+Opt+I)
- **요소 검사**: Ctrl+Shift+C (Mac: Cmd+Shift+C)
- **콘솔 열기**: Ctrl+Shift+J (Mac: Cmd+Opt+J)
- **검색**: Ctrl+F (Mac: Cmd+F)
- **패널 간 이동**: Ctrl+[ 또는 Ctrl+] (Mac: Cmd+[ 또는 Cmd+])
- **개발자 도구 독**: Ctrl+Shift+D (Mac: Cmd+Shift+D)

#browser #debugging #webdev #frontend
