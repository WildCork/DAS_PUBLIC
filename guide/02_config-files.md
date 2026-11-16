# 0단계: 시험 Config 파일 설정

DAS 자체는 특정 발사체나 장비 이름을 미리 알지 못하는 범용 해석기입니다 — "이번 시험에 어떤 장비가 있고 어떤 IP:Port를 쓰는지", "그 소켓 쌍의 트래픽을 어느 IDL로 해석할지", "그 payload 바이트를 어떤 필드로 나눌지"를 프로그램에 알려주는 것이 바로 이번 항목에서 다루는 세 종류의 텍스트 파일입니다. 이 정보가 하나도 없으면 PCAPNG를 불러와도 장치는 전부 "미확인 IP"로, 메시지는 전부 원시 바이트로만 보입니다 — 뒤에 나올 ①·② 단계는 결국 이 세 파일이 담는 내용을 화면에서 하나씩 입력하는 과정입니다.

**사용 순서**

1. IDL 파일부터 준비합니다 — 각 장비가 주고받는 payload의 구조(메시지 ID, 필드, 주기 여부)를 정의하는 파일로, 이것이 없으면 뒤의 두 파일이 가리킬 대상 자체가 없습니다.
2. IDL을 다 준비했다면 endpoints.lua로 장치·소켓과 그 소켓 쌍이 어느 IDL을 쓰는지를 한 표로 묶습니다 — 같은 구성으로 시험을 반복한다면 이 파일 하나만 불러오면 됩니다.
3. endpoints.lua 대신, 또는 화면에서 직접 등록한 내용을 그대로 보관하고 싶다면 DAS 설정 JSON으로 저장합니다 — 이쪽은 DAS가 직접 만들고 읽는 형식이라 처음부터 손으로 쓰기보다는 화면에서 등록한 뒤 저장 기능으로 만드는 것이 일반적입니다.
4. 세 파일 모두 아래 하위 항목에서 필수 규칙(없으면 파싱 실패·기능 무력화)과 선택 규칙(없으면 기본값으로 처리)을 나눠 확인할 수 있습니다.

**주의사항**

- 세 파일 모두 필수 제출물은 아닙니다 — ①·② 단계 화면에서 장치·소켓·연결·IDL을 하나씩 직접 등록해도 결과는 완전히 같으며, 구성 파일은 그 입력을 미리 텍스트로 준비해 "설정 불러오기" 한 번으로 재현하는 지름길일 뿐입니다.
- endpoints.lua와 IDL은 사람이 텍스트 편집기로 직접 작성하는 반면, DAS 설정 JSON은 프로그램이 저장/불러오기 기능으로 직접 다루는 형식이라 목적이 다릅니다 — 매번 같은 장치 구성으로 반복되는 시험이라면 endpoints.lua+IDL 조합을, 이미 화면에서 등록해 둔 내용을 다음에 그대로 이어 쓰고 싶다면 설정 저장/불러오기(JSON)를 쓰는 것이 자연스럽습니다.

## endpoints.lua 작성 규칙

devices 표에서 장치별 소켓(IP:Port)을 정의하고, protos 표에서 어느 소켓 쌍이 서로 통신하는지와 그 트래픽을 해석할 IDL 이름을 지정합니다 — 이 파일 하나가 ①단계의 장치/소켓 목록과 ②단계의 연결·IDL 배정을 통째로 대신합니다. 다만 실제 Lua 언어 전체를 해석하는 것이 아니라 return { devices = {...}, protos = {...} } 형태의 단순한 표만 알아보는 전용 파서이므로, 조건문·함수 등 진짜 Lua 문법을 쓰면 처음부터 읽지 못합니다.

**사용 순서**

1. [필수] 파일 전체가 return { devices = { ... }, protos = { ... } } 최상위 표 하나로 감싸여 있어야 합니다 — 이 틀 자체를 찾지 못하면 파일을 전혀 읽지 못합니다.

   ```
   return {
       devices = { ... },   -- 엔드포인트(장치·소켓) 정의
       protos  = { ... },   -- 소켓 쌍 ↔ IDL 매핑
   }
   ```

2. [필수] protos의 각 항목은 devices에 실제로 정의된 소켓 키를 2개 이상 나열해야 합니다 — devices에 없는 키나 유효한 키가 1개뿐인 항목은 그 프로토콜 전체가 통째로 건너뛰어져 연결이 하나도 만들어지지 않습니다.

   ```
   protos = {
       WCU_CMS = { "WCU_CMS_UDP", "CMS_1_UDP" },  -- 2개: OK, 링크 2개 생성
       MCU_MS  = { "MCU_MS_UDP",  "MS_1_UDP"  },  -- 2개: OK
   },
   ```

3. [필수] 같은 발신→수신 소켓 쌍을 서로 다른 protos 항목에서 서로 다른 IDL로 중복 지정하면 안 됩니다 — 이 경우에만 진단 창에 실제 오류(Error)가 뜨며, 어느 IDL로 해독해야 할지 프로그램이 판단할 수 없습니다.

   ```
   protos = {
       WCU_CMS  = { "WCU_CMS_UDP", "CMS_1_UDP" },  -- WCU_CMS.idl 사용
       WCU_CMS2 = { "WCU_CMS_UDP", "CMS_1_UDP" },  -- 같은 소켓 쌍인데
                                                   -- WCU_CMS2.idl 사용 → 오류!
   },
   ```

4. [사실상 필수] 소켓을 여러 개 쓰는 장비(예: 상태 보고 포트와 명령 포트가 다른 장비)는 devices 각 항목의 name 접두어를 통일해야 합니다 — 장치 그룹핑은 name의 첫 번째 밑줄(_) 앞부분만 보고 판단하므로, 접두어가 어긋나면 같은 장비인데도 서로 다른 장치로 쪼개져 등록됩니다.

   ```
   WCU_CMS_UDP = { name = "WCU_CMS", ip = "10.3.24.10", port = 30150, type = "UDP" },
   WCU_WCP_UDP = { name = "WCU_WCP", ip = "10.3.24.10", port = 30630, type = "UDP" },
   WCU_MIE_UDP = { name = "WCU_MIE", ip = "10.3.24.10", port = 4300,  type = "UDP" },
   -- 첫 밑줄(_) 앞부분이 모두 "WCU" → 소켓 3개를 가진 장치 "WCU" 하나로 합쳐짐
   ```


**주의사항**

- [선택] devices.name이 비어 있으면 표의 키 자체를 이름으로 쓰고, ip가 비어 있으면 빈 문자열로 등록되어 어떤 실제 트래픽과도 매칭되지 않으며, port가 없으면 0("모든 포트 허용")으로 처리되고, type이 없거나 UDP/TCP가 아니면 프로토콜 미상으로 남습니다 — 넷 다 파일을 읽는 데는 지장이 없지만 실제 매칭 정확도에는 직접 영향을 줍니다.

  ```
  CMS_1_UDP = { name = "CMS_1", ip = "10.3.24.100", port = 30150, type = "UDP" },
  --            ^^^^^^^^^^^^^^   ^^^^^^^^^^^^^^^^^^^   ^^^^^^^^^   ^^^^^^^^^^^
  --            이름 (비면 키 사용) IP (비면 매칭 불가)  포트(비면 0)  프로토콜(비면 미상)
  ```

- [선택] protos 키와 같은 이름의 .idl 파일(예: protos.WCU_CMS ↔ 같은 폴더의 WCU_CMS.idl)이 자동으로 연결됩니다 — 그 .idl 파일이 폴더에 없어도 연결 자체는 만들어지고, 다만 그 프로토콜의 메시지는 필드 해석 없이 원시 바이트로만 표시됩니다.

  ```
  protos = {
      WCU_CMS = { "WCU_CMS_UDP", "CMS_1_UDP" },
  },
  -- endpoints.lua와 같은 폴더에서 "WCU_CMS.idl"을 자동으로 찾아 연결합니다.
  ```

- 두 소켓 항목이 같은 IP:Port를 가리키면(오타로 인한 경우가 많음) 경고만 뜨고 파일은 계속 읽히므로, 진단 창에서 놓치지 않고 확인하는 것이 좋습니다.

  ```
  A_UDP = { name = "A", ip = "10.3.24.10", port = 30150, type = "UDP" },
  B_UDP = { name = "B", ip = "10.3.24.10", port = 30150, type = "UDP" },
  -- A_UDP와 B_UDP가 같은 IP:Port → 경고("오타가 아닌지 확인하세요")
  ```


## DAS 설정 JSON 형식

③단계 "설정 저장"으로 만들어지는, DAS 자신이 읽고 쓰는 저장 형식입니다 — 사람이 처음부터 손으로 작성하기보다는 화면에서 장치·연결·IDL·요청-응답 관계를 등록한 뒤 저장 기능으로 만들고, 나중에 "설정 불러오기"로 그대로 되살리는 용도가 기본입니다. PCAPNG 패킷 자체와 ⑤단계 성적서 점검항목은 이 파일에 포함되지 않습니다.

**사용 순서**

1. [필수] 최상위에 "version" 필드가 숫자로 있어야 하며 1 이상이어야 합니다 — 이 파일이 유일하게 하드 오류로 거부하는 조건으로, 없거나 형식이 다르면 "DAS가 저장한 JSON 설정 파일이 맞는지 확인하라"는 오류로 아예 불러오기가 실패합니다. 현재 버전이 저장하는 값은 2(요청-응답 관계 포함)이며, 1로 저장된 예전 파일도 관계 없는 상태로 계속 불러올 수 있습니다.

   ```
   {
     "version": 2,
     ...
   }
   ```

2. 그 외 devices/idlFiles/links/responseRelationships 배열의 개별 필드는 형식만 맞으면 되고, 정상적으로 저장된 파일이라면 이 규칙은 자동으로 지켜집니다 — 이 파일을 손으로 직접 고쳐 쓰는 일은 권장하지 않습니다.

   ```
   {
     "devices":  [ { "id": 1, "name": "WCU", "sockets": [ ... ] } ],
     "idlFiles": [ { "id": 1, "name": "WCU_CMS",
                      "path": "C:/DAS/Config/WCU_CMS.idl" } ],
     "links":    [ { "id": 1, "srcDevId": 1, "dstDevId": 2,
                      "idlId": 1, ... } ]
   }
   ```

3. ③ 시험 분석 설정에서 정의한 요청-응답 관계(④ 시각화 > 응답성이 읽는 것과 같은 정의)는 "responseRelationships" 배열에 저장됩니다 — 각 원소는 관계 id·색상과, 요청/응답 메시지를 각각 가리키는 request/response 객체(linkId·idlId·msgId· structName·variantKey·srcDevId·dstDevId·표시용 labelCache)로 구성됩니다. 관계에 별도 "이름"은 없습니다 — 표시명(labelCache)은 요청→응답 메시지 경로를 조합해 자동으로 만들어집니다.

   ```
   "responseRelationships": [
     { "id": 1, "color": "#4f9df8",
       "request":  { "linkId": 12, "idlId": 4, "msgId": 3,
                      "structName": "MIE_MCU_HATCH_STATUS", ... },
       "response": { "linkId": 18, "idlId": 7, "msgId": 9,
                      "structName": "WCU_WCP_STATUS", ... },
       "options": {} }
   ]
   ```


**주의사항**

- [선택] devices/idlFiles/links의 개별 필드(이름, IP:Port, 경로 등)는 비어 있거나 빠져 있어도 빈 값·0으로 채워질 뿐 파일 자체는 계속 읽힙니다. idlFiles에는 IDL의 절대 경로(path)와 설정 파일 기준 상대 경로(relativePath), 저장 당시 내용 해시(sha256)가 함께 기록되고, 최상위 idlPathMode(relative 또는 absolute)가 불러올 때 어느 경로를 먼저 찾을지 정합니다 — 먼저 찾을 경로에 파일이 없으면 다른 경로의 파일을 해시가 같을 때만 대신 씁니다. 두 경로 모두에 없으면 설정 파일 폴더와 그 하위 폴더에서 같은 이름의 파일을 찾고, 해시가 다르거나 없으면 쓸지 묻습니다.

  ```
  "idlPathMode": "relative",
  "idlFiles": [
    { "id": 1, "name": "WCU_CMS", "path": "C:/DAS/Config/WCU_CMS.idl",
      "relativePath": "WCU_CMS.idl", "sha256": "…" }
  ]
  ```

- 구버전 호환을 위해 소켓 정보가 없는 예전 형식(포트 없이 IP만 있는 addresses 배열)도 읽을 수 있지만, 새로 저장하면 항상 최신 소켓 형식으로 다시 저장됩니다.

  ```
  "devices": [
    { "id": 1, "name": "WCU", "addresses": [ "10.3.24.10" ] }
  ]
  ```

- 이 파일은 손으로 편집하도록 설계되지 않았습니다 — devices/idlFiles/links 배열은 서로의 id로 서로를 참조하므로, 직접 고치다 id가 어긋나면 링크가 엉뚱한 장치를 가리킬 수 있습니다. 구성을 바꿀 때는 화면에서 수정한 뒤 다시 저장하는 것이 안전합니다.

  ```
  "devices": [ { "id": 1, "name": "WCU", ... } ],
  "links":   [ { "srcDevId": 1, "dstDevId": 2, ... } ]
  ```

- responseRelationships가 가리키는 장치·연결·IDL·메시지를 나중에 지워도 그 관계 자체는 파일에서 자동으로 빠지지 않습니다 — 다시 불러오면 "참조 누락" 상태로 목록에 남고, 새 메시지로 다시 연결하거나 직접 삭제해야 사라집니다.

## IDL 파일 작성 규칙

payload 바이트를 어떤 순서·타입의 필드로 나눌지, 그리고 어떤 메시지 ID가 어느 구조체에 대응하는지를 정의하는 파일입니다 — IDL이 없으면 ④단계 시간선에 payload가 항상 원시 바이트로만 보이고, 주기성·연속성 그래프도 전혀 만들어지지 않습니다. 실제 C 구조체 문법과 비슷하지만 전체를 컴파일하는 것이 아니라, struct 선언과 // 주석에 붙은 몇 가지 표시(태그)만 알아보는 전용 파서입니다.

**사용 순서**

1. [필수] struct 선언이 최소 하나 있어야 하고, 그중 정확히 하나는 선언 줄 오른쪽에 // $()$ 주석이 붙은 "헤더 구조체"여야 합니다 — 이 태그가 하나도 없으면 파일 전체가 파싱에 실패합니다.

   ```
   struct WCU_CMS_HEADER        // $()$   ← 헤더 구조체로 지정하는 태그
   {
       unsigned char MsgId;     // $()$
       unsigned char CrcError : 1;
       unsigned char Reserved : 7;
   };
   ```

2. [필수] 그 헤더 구조체 안의 필드 하나에도 // $()$ 주석이 붙어 있어야 합니다 — 이 필드가 실행 중 실제로 "이 payload는 몇 번 메시지인가"를 읽는 자리이며, 없으면 역시 파싱에 실패합니다.

   ```
   struct WCU_CMS_HEADER        // $()$
   {
       unsigned char MsgId;     // $()$   ← 메시지 ID를 읽는 필드로 지정하는 태그
       unsigned char CrcError : 1;
       unsigned char Reserved : 7;
   };
   ```

3. [필수] 필드에 쓰는 타입은 char/short/int/long/float/double 계열의 정해진 이름이거나, 이 파일 안에서 이미 선언된 다른 struct 이름이어야 합니다 — 오타 등으로 정의되지 않은 타입을 쓰면 오류로 처리됩니다.

   ```
   struct WCU_CMS_ENGAGE           // $(1)$
   {
       WCU_CMS_HEADER      header;       // 이미 선언된 struct 이름
       int                 TargetId;     // 정해진 기본 타입 이름
       double              TargetRange;  // 정해진 기본 타입 이름
   };
   ```

4. [필수] struct가 서로를 필드로 참조할 때 자기 자신으로 되돌아오는 순환 참조가 있으면 안 되고, 비트필드는 struct 타입에 쓸 수 없으며 그 폭이 실제 타입의 전체 비트 수를 넘을 수 없고, 한 필드를 배열이면서 동시에 비트필드로 선언할 수도 없습니다 — 모두 오류로 처리되어 그 IDL은 쓸 수 없게 됩니다.

   ```
   struct A { B b; };            // A가 B를 포함
   struct B { A a; };            // B가 다시 A를 포함 → 순환 참조 오류!

   struct C {
       unsigned char flag : 1;   // OK: 기본 타입 비트필드
       B              bad  : 2;  // 오류: struct 타입은 비트필드로 쓸 수 없음
   };
   ```


**주의사항**

- //+PACK_SIZE=N은 없으면 1(패딩 없음)로 기본 처리되지만, 1이 아닌 값으로 명시하면 로드 자체가 오류로 거부됩니다 — 이 프로그램은 항상 패딩 없이(1바이트 정렬)만 해독하므로, 실제 프로토콜에 패딩이 있는 IDL은 값이 조용히 밀려서 잘못 해석되는 것보다 아예 로드를 막는 편이 안전합니다. //+MOST_BYTE=true|false는 없으면 true(빅엔디안)로 기본 처리됩니다.

  ```
  //+PACK_SIZE=1
  //+MOST_BYTE=true
  //PERIODIC=[2,3]
  ```

- [선택] //PERIODIC=[N,N,...] 선언으로 어떤 메시지 ID가 주기 메시지인지 지정합니다 — 없으면 모든 메시지가 비주기로 취급되어 ④단계 주기성·연속성 그래프 대상에서 빠집니다.

  ```
  //PERIODIC=[2,3]
  // 메시지 ID 2(WCU_CMS_STATUS_REPORT)와 3(WCU_CMS_ROUND_INFO)이 주기 메시지
  ```

- [선택] 메시지 구조체는 struct 선언 줄에 // $(N)$ 주석을 붙여 메시지 ID N에 대응시킵니다 — 이 태그가 하나도 없어도 파일 자체는 읽히지만, 그 IDL로는 어떤 메시지도 실제로 해독되지 않습니다. 이 메시지 구조체의 첫 번째 필드는 헤더 구조체 타입으로 시작하는 것이 좋습니다 — 디코더가 항상 payload 맨 앞부터 헤더를 읽기 때문에, 첫 필드가 헤더가 아니면 그 뒤 필드들의 위치가 밀려 잘못 해석됩니다.

  ```
  struct WCU_CMS_STATUS_REPORT    // $(2)$   ← 메시지 ID 2로 등록
  {
      WCU_CMS_HEADER      header;   // 첫 필드가 헤더 구조체 타입
      st_WCU_CMS_STATUS   status;
  };
  ```

- [선택] 필드 뒤 주석에 msb=값, bitsize=비트수(또는 lsb=값)와 unit="단위"를 붙이면 원시 정수값을 실제 물리량으로 환산해 ④단계 상세 화면과 연속성 그래프에 단위와 함께 보여줍니다 — 붙이지 않으면 원시 값 그대로 표시될 뿐, 파싱 자체에는 영향이 없습니다.

  ```
  struct WCU_CMS_ROUND_INFO       // $(3)$
  {
      WCU_CMS_HEADER header;
      unsigned int   RoundId;
      int            Altitude_raw;  // lsb=0.1, unit="m"
      int            Velocity_raw;  // lsb=0.1, unit="m/s"
  };
  // Altitude_raw=12345 → 상세 화면에는 "1234.5 m"로 환산되어 표시됩니다.
  ```

- 같은 이름의 struct를 두 번 선언하면 나중 선언이 앞의 것을 덮어쓰고, 같은 메시지 ID를 두 struct에 중복 태그하면 역시 나중 것이 사용됩니다 — 둘 다 오류가 아니라 경고로만 표시되므로 놓치기 쉽습니다.

  ```
  // 먼저 선언:
  struct WCU_CMS_ENGAGE // $(1)$
  { WCU_CMS_HEADER header; int TargetId; };

  // 같은 이름 재선언 → 앞의 정의를 덮어씀:
  struct WCU_CMS_ENGAGE // $(1)$
  { WCU_CMS_HEADER header; double NewField; };
  ```

- [선택] struct NAME {...}; 대신 C의 typedef struct 문법도 쓸 수 있습니다 — 익명이든 태그 이름이 붙어 있든, 여는 중괄호가 같은 줄이든 다음 줄이든 상관없습니다. 다만 태그 이름(아래 예의 MESSAGE_TAG)은 다른 필드의 타입으로 쓸 수 없습니다 — 실제로 쓰이는 이름은 항상 닫는 중괄호 뒤의 별칭(MESSAGE)입니다.

  ```
  typedef struct
  {
      unsigned char value;
  } MESSAGE;              // 이름은 여기(별칭)로 결정됨

  typedef struct MESSAGE_TAG {   // 태그 이름은 참고용, 타입으론 못 씀
      unsigned char value;
  } MESSAGE2;
  ```

- [선택] typedef struct의 // $()$ (헤더 태그)와 // $(N)$ (메시지ID 태그)는 여는 줄("typedef struct { // $(N)$")과 이름이 정해지는 닫는 줄("} 이름; // $(N)$") 중 어디에 붙여도 됩니다. 양쪽에 서로 다른 태그를 붙이면 닫는 줄의 것을 쓰고 경고를 띄웁니다.

  ```
  typedef struct { // $()$         ← 여는 줄에 붙여도 되고
      unsigned char MsgId; // $()$
  } HEADER;

  typedef struct
  {
      HEADER header;
      unsigned char value;
  } MESSAGE_A; // $(1)$           ← 닫는 줄에 붙여도 된다
  ```

- [선택] typedef struct 전체를 한 줄에 다 쓸 수도 있습니다 — 다만 "//"가 나오면 그 뒤로는 줄 전체가 주석 처리되는 C++ 규칙 때문에, 한 줄 선언 안에서는 필드별 주석($SWITCH$/$CASE(n)$/msb=.../"// $()$" 등)을 쓸 수 없습니다 — 필드별 주석이 필요하면 여러 줄 형태를 쓰세요. 태그 뒤에 이미 정의된 struct를 그대로 가리키는 "typedef struct TAG ALIAS;" 형태(본문 없이 별칭만)도 지원합니다 — 이때 TAG는 typedef struct의 태그 이름이 아니라, 그 앞에서 "struct TAG { ... };"로 먼저 정의된 이름이어야 합니다.

  ```
  struct ITEM { unsigned char a; unsigned short b; };

  // 한 줄 전체 선언 (필드별 주석 불가, 구조체 자체의 태그는 가능)
  typedef struct { unsigned char a; unsigned char b; } SIMPLE; // $(9)$

  // 순수 별칭 -- ITEM의 필드를 그대로 복제한 새 이름
  typedef struct ITEM ITEM_ALIAS;
  ```

- [선택] 필드 뒤 주석에 // $SWITCH$를 붙이면, 그 필드의 실제 값에 따라 같은 구조체 안의 // $CASE(n)$이 붙은 필드들 중 하나만 활성화되는 조건부 구조를 만들 수 있습니다 — CASE 필드는 SWITCH 필드와 붙어 있지 않아도 되고, 둘 사이에 일반 필드가 있어도 됩니다. 다만 한 구조체 안에는 $SWITCH$가 하나만 있을 수 있습니다. SWITCH 필드는 정수 타입(또는 정수 비트필드)이어야 하고, 배열·구조체·실수 타입에는 붙일 수 없습니다. CASE 값은 메시지 ID와 같은 문법(10진수 또는 0x 접두사 16진수)에 더해 선행 '-'로 음수도 쓸 수 있습니다 — 다만 SWITCH 필드가 비트필드면 부호 확장이 되지 않아 음수 CASE는 실제로 일치할 수 없으니(경고만 뜨고 로드는 됩니다) 부호 있는 일반 필드를 쓰세요.

  ```
  struct MESSAGE // $(1)$
  {
      HEADER  header;
      uint8_t mode;       // $SWITCH$
      TYPE_A  dataA;      // $CASE(1)$
      TYPE_B  dataB;      // $CASE(2)$
      uint16_t tail;
  };
  // mode==1이면 dataA만, mode==2면 dataB만 상세 화면/그래프에 나타남

  // CASE 필드는 흩어져 있어도 됩니다 -- 같은 구조체 안이기만 하면 됨
  struct MESSAGE2 // $(2)$
  {
      HEADER  header;
      uint8_t mode;       // $SWITCH$
      uint8_t common1;
      TYPE_A  dataA;      // $CASE(1)$
      uint8_t common2;
      TYPE_B  dataB;      // $CASE(2)$
  };
  ```

- [선택] $CASE(n)$ 필드는 C의 union이 아니라 "조건부 필드"입니다 — 모든 필드(SWITCH·CASE·일반 필드 포함)는 선언된 순서 그대로 자기 자리에서 디코딩되고, 그중 SWITCH 값과 일치하지 않는 CASE 필드만 바이트를 전혀 소비하지 않고 통째로 건너뜁니다. 그래서 CASE 필드 뒤에 오는 필드(위 예의 tail)의 실제 바이트 위치는 그 occurrence에서 어떤 CASE가 활성화되었는지에 따라 달라집니다 — dataA(1바이트)가 활성화되면 tail은 dataA 바로 뒤에서, dataB(2바이트)가 활성화되면 tail은 dataB 바로 뒤에서 시작합니다. 실제 SWITCH 값과 일치하는 CASE가 하나도 없어도 전체 해독은 실패하지 않습니다(그 필드들만 없는 것으로 처리되고, 이후 필드는 정상적으로 이어집니다). 활성화되지 않은 CASE는 상세 화면·그래프·필드 값 필터 어디에도 나타나지 않습니다 — 존재하지 않는 값처럼 취급됩니다.
- [선택] $SWITCH$/$CASE(n)$은 "추가 메시지 식별자"(같은 메시지 ID를 필드 값으로 세분화해서 보는 기능)와는 별개입니다 — 식별자는 이미 고정된 구조를 값으로만 나누어 보는 것이고, SWITCH/CASE는 payload의 바이트 구조 자체가 값에 따라 달라진다고 IDL에 선언하는 것입니다. 두 기능을 같은 메시지에 함께 써도 됩니다.

---

[← 전체 사용 순서](01_workflow.md) · [목차](../README.md#사용-가이드) · [① 장치 / IDL →](03_step1.md)
