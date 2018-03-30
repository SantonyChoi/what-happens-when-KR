...하면 무슨 일이 생길까
====================

이 저장소는 고전적인 인터뷰 질문인 "당신의 브라우저 주소창에 google.com을 치고 엔터를 누르면 어떤
일이 생길까요?"에 대해 답해보고자 합니다.

일반적인 얘길 하는 대신에, 우리는 가능한 한 세부적으로 이 질문에 답하고자 노력할 것입니다. 아무것도
빼먹지 않고 말입니다.

이 저장소는 공동 작업물이 되어야 합니다. 함께 파고들고 도움을 주시면 감사하겠습니다! 분명 수많은 디테일상의 실수들이
있으니, 여러분이 힘을 보태주시길 기다리겠습니다! Pull request를 보내주세요!

이 저작물은 `Creative Commons Zero`_ 라이센스를 따릅니다.

영문 원본은 `이 곳`_ 에서 확인하실 수 있습니다.

Table of Contents
====================

.. contents::
   :backlinks: none
   :local:

"g"키를 누르면
-----------

이번 절에서는 키보드의 물리적인 동작과 운영체제의 인터럽트에 대해 다룹니다. 하지만,
여기서 설명하지 않는 곳에서도 물론 무수히 많은 동작이 일어납니다. 우선 당신이 "g" 키를
누르면 해당 이벤트가 브라우저에 전달되고 자동완성 기능이 모두 활성화됩니다. 당신이
사용하는 브라우저의 알고리즘과 당신이 개인/익명 모드를 사용하는지에 따라 다양한 제안이
URL 창 아래에 드랍박스로 나타나죠. 대부분의 알고리즘은 결과를 검색 기록이나 즐겨찾기에
기반해 정렬합니다. 당신은 "google.com"을 입력할 것이기 때문에 상관없지만, 수많은
코드가 당신이 다 마치기 전에 동작하며, 매 키를 누를 때마다 제안이 선별됩니다. 아마
당신이 다 치기도 전에 "google.com"을 제안할지도 모르겠네요.

"엔터"키가 쏙 들어갑니다
------------------

명확히 설명드리기 위해, 키보드의 엔터키가 끝까지 눌러졌다고 해봅시다. 여기서, 엔터키에 할당된
전기 회로가 (직접적으로든 정전식으로든) 닫힙니다. 이것이 적은 양의 전류를 키보드에서부터
각 키 스위치 상태를 확인하는 논리 회로 소자에 흐르도록 하고, 빠르고 간헐적인 스위치 차단으로 인한
전기적 잡음을 디바운싱하며, 신호를 키코드 정수, 이 경우에는 13으로 변환해줍니다. 키보드 컨트롤러는 곧,
키코드를 인코딩해 컴퓨터로 전달합니다. 이것이 지금은 대부분 유니버설 시리얼 버스 (USB) 혹은
블루투스 연결을 통해 이루어지며, 과거에는 PS/2 혹은 ADB 연결에서 통용되던 방법입니다.

*USB 키보드의 경우:*

- 키보드의 USB 회로 소자는 컴퓨터 USB 호스트 컨트롤러의 핀 1로 제공되는 5V 전원으로 동작합니다.

- 생성된 키 코드는 키보드 내부 회로 소자내의 "endpoint"라고 불리는 레지스터에 저장됩니다.

- 호스트 USB 컨트롤러는 "endpoint"를 매 10ms(키보드에 따라 정의된 최소값) 이내의 시간마다
  폴링하며, 이를 통해 저장된 키 코드 값을 얻어냅니다.

- 이 값은 USB SIE (Serial Interface Engine) 으로 넘어가 저수준의 USB 프로토콜에 맞는,
  하나 혹은 그 이상의 USB 패킷으로 변환됩니다.

- 패킷들은 D+와 D- 핀 (가운데 있는 둘) 을 이용한 차동 신호를 통해 최대 1.5 Mb/s의
  속도로 전송되는데, HID (Human Interface Device) 디바이스는 늘 "저속 장치"로 분류되기
  때문입니다 (USB 2.0 compliance).

- 이 직렬 신호는 곧 컴퓨터의 호스트 USB 컨트롤러에서 디코딩되고, 컴퓨터의
  HID (Human Interface Device) 유니버설 키보드 디바이스 드라이버에 의해 변환됩니다.
  키 값은 이제 운영 체제의 하드웨어 추상화 레이어로 전달됩니다.


*가상 키보드의 경우 (터치 스크린 장치 등에 있는):*

- 사용자가 현대적인 정전식 터치 스크린에 손가락을 올리면, 작은 양의 전류가 손가락으로 흐릅니다.
  이것이 전도층의 정전기를 통해 회로를 완성시키고 스크린 위의 해당 지점에 전압 하강을 유도합니다.
  그러면 ``스크린 컨트롤러`` 는 키 입력의 좌표를 알리는 인터럽트를 발생시킵니다.

- 이제 모바일 운영체제는 현재 키 입력 이벤트의 초점을 자신의 GUI 요소 중 하나(여기서는 가상
  키보드 어플리케이션 버튼)에 알립니다.

- 가상 키보드는 이제 소프트웨어 인터럽트를 일으켜 '키 입력' 메시지를 OS에 되돌려줄 수 있습니다.

- 이 인터럽트는 현재 키 입력 이벤트의 초점을 알립니다.

인터럽트 발생 [키보드가 USB가 아닌 경우에]
---------------------------------

키보드는 인터럽트 요청 라인 (IRQ) 를 통해 신호를 보내는데, 이 라인은 인터럽트 컨트롤러에 의해
``인터럽트 벡터`` (정수 값) 에 연결되어 있습니다. CPU는 ``Interrupt Descriptor Table``
(IDT) 을 활용해 커널에서 제공된 함수들 (``인터럽트 핸들러``) 에 인터럽트 벡터를 연결하구요.
인터럽트가 도착하면, CPU는 IDT와 인터럽트 벡터를 살펴보고 적절한 핸들러를 실행합니다. 이에 따라서,
커널에 진입하게 됩니다.

(Windows에서) ``WM_KEYDOWN`` 메시지가 앱으로 전달되어요
-----------------------------------------------

HID 트랜스포트는 키 눌림 이벤트를 HID가 사용하는 형태의 스캔코드로 변환하는 ``KBDHID.sys``
드라이버에 전달합니다. 이 경우에 스캔코드는 ``VK_RETURN`` (``0x0D``)가 되죠.
``KBDHID.sys`` 드라이버는 ``KBDCLASS.sys`` (키보드 클래스 드라이버) 와 접속합니다.
이 드라이버는 모든 키보드와 키패드 입력의 안전한 처리를 담당합니다. 그리고는 (설치된 서드파티
키보드 필터로 메시지를 전달한 후에) ``Win32K.sys`` 를 호출합니다. 이 모든 일은
커널 모드에서 일어나죠.

``Win32K.sys`` 는 어떤 창이 활성화 돼 있는지를 ``GetForegroundWindow()`` API를 통해
알아냅니다. 이 API는 브라우저 주소창의 윈도우 핸들을 제공하겠네요. Windows의 "message pump"는
곧, ``SendMessage(hWnd, WM_KEYDOWN, VK_RETURN, lParam)`` 을 호출합니다.
``lParam`` 은 키눌림의 더 자세한 정보를 가리키는 비트마스크입니다: 반복 횟수(여기선 0),
진짜 스캔 코드 (OEM 별로 상이하지만, 보통은 ``VK_RETURN``), 특수키(alt, shift, ctrl 같은)가
함께 눌렸는지 (여기선 안 눌렸죠), 그리고 몇 가지 다른 상태에 대한 정보가 담겨있어요.

Windows의 ``SendMessage`` API는 특정한 창 핸들 (``hWnd``) 의 큐에 메시지를 추가하는 간단한
함수입니다. 그리고나서, ``hWnd`` 에 할당된 (``WindowProc`` 이라 불리는) 주 메시지 처리 함수가
큐에 있는 메시지들을 처리하기 위해 호출됩니다.

활성화 된 창 (``hWnd``) 은 실제로 편집을 제어하며 여기서의 ``WindowProc`` 은 ``WM_KEYDOWN``
메시지에 대한 메시지 핸들러를 갖게 됩니다. 이 코드는 ``SendMessage`` 로 전달된 세 번째 파라미터
(``wParam``) 를 들여다보는데요, 사용자가 엔터키를 쳤다는 걸 알려주는 게 ``VK_RETURN`` 이기
때문입니다.

(OS X에서) ``KeyDown`` NSEvent가 앱으로 전달되어요
--------------------------------------------

인터럽트 신호는 I/O Kit kext 키보드 드라이버에 인터럽트 이벤트를 발생시킵니다. 이 드라이버는 해당
신호를 OS X의 ``WindowServer`` 프로세스에 전달되는 키 코드로 변환합니다. 그 결과로서,
``WindowServer`` 는 어떠한 적절한 곳 (활성화 혹은 리스닝하는 곳과 같은 곳) 에라도 이벤트 큐가
들어있는 Mach의 포트를 통해 이벤트를 보내게 됩니다. 그리고 나면 이벤트는 이 큐에서,
``mach_ipc_dispatch`` 함수를 호출할 수 있는 권한을 가진 스레드에 의해 읽힙니다. 일련의 과정은
``NSApplication`` 메인 이벤트 루프에 의해, ``NSEventType`` 의 ``KeyDown`` 이라는
``NSEvent`` 를 통해 처리됩니다.

(GNU/Linux에서) Xorg 서버가 키코드를 listen해요.
------------------------------------------

그래픽이 제공되는 ``X 서버`` 를 사용할 땐, ``X`` 가 일반적인 이벤트 드라이버 ``evdev`` 를
키 눌림 확인에 활용합니다. 키코드를 스캔코드로 다시 맵핑하는 것은 ``X 서버`` 고유의 키맵과 룰에 따라
이뤄지고요. 키 눌림의 스캔코드 맵핑이 완료되면, ``X 서버`` 는 해당 문자를 ``윈도우 관리자``
(DWM, metacity, i3 등등) 에 전달하여, ``윈도우 관리자`` 가 활성화된 창에 문자를 보내게 하죠.
문자를 전달받은 창에서는 그래픽을 표현하는 API가 적절한 폰트 기호를 적절한 선택 영역에 찍어줍니다.

URL 파싱하기
---------

* 이제 브라우저는 URL (유일 자원 지시자) 을 담고 있는 아래의 정보를 가지고 있어요:

    - ``프로토콜``  "http"
        '하이퍼 텍스트 전송 규약'을 사용하시오

    - ``자원``  "/"
        메인 (인덱스) 페이지를 가져오시오


검색어일까 URL일까?
---------------

프로토콜이나 유효한 도메인 이름이 주어지지 않으면, 브라우저는 주소창에 놓인 텍스트를 브라우저의 기본 웹
검색엔진에 넘겨줍니다. 많은 경우에 이 URL에는 어떤 브라우저로부터 전달되었는지 검색엔진이 알 수 있게
해주는 특수한 부분 텍스트가 붙습니다.

호스트명에서 ASCII 아닌 유니코드 문자열 변환
-----------------------------------

* 브라우저는 호스트네임에서 ``a-z``, ``A-Z``, ``0-9``, ``-``, 혹은 ``.`` 아닌 문자들을
  확인합니다.

* 지금의 호스트명은 ``google.com`` 이기때문에 유니코드가 없지만, 있을 때에는 브라우저가 URL에서
  호스트명 부분에 `퓨니코드 (Punycode)`_ 인코딩을 하기도 합니다.

HSTS 리스트 점검
-------------

* 브라우저는 "미리 불러들인 HSTS (HTTP Strict Transport Security)" 리스트를 점검합니다. 이
  리스트는 HTTPS로만 연결되도록 요청한 웹사이트의 목록이죠.

* 웹사이트가 목록에 있다면, 브라우저는 요청을 HTTP 대신 HTTPS로 보내게 됩니다. 그렇지 않다면, 첫
  요청은 HTTP로 보내지구요. (웹사이트가 HSTS 목록에 *없더라도* 여전히 HSTS 정책을 사용할 수 있다는
  점을 알아두세요. 사용자의 첫 HTTP 요청에 대한 응답으로 사용자가 반드시 HTTPS 요청을 보내도록
  요구한다는 내용을 받게 되는 것이죠. 하지만, 이 단일 HTTP 요청이 잠재적으로 사용자를 `다운그레이드
  공격 (downgrade attack)`_ 에 취약하도록 할 수도 있고, 이 때문에 HSTS 목록이 현대적인
  웹 브라우저에 들어있는 것입니다.)

DNS 검색
-------

* 브라우저는 도메인이 캐시에 들어있는지 확인합니다. (크롬에서 DNS 캐시를 보려면,
  `chrome://net-internals/#dns <chrome://net-internals/#dns>`_ 으로 가보세요).
* 만약 못 찾으면, 브라우저는 검색을 하기 위해 (OS에 따라 상이하지만) ``gethostbyname`` 라이브러리
  함수를 호출합니다.
* ``gethostbyname`` 은 DNS를 통한 호스트명 확인을 시도하기 전에, 호스트명이 로컬의
  (`OS에 따라`_ 위치가 다른) hosts 파일에서 참조될 수 있는지 봅니다.
* ``gethostbyname`` 이 캐시와 ``hosts`` 파일 모두에서 호스트명을 못 찾으면, 곧 네트워크
  스택에서 정의된 DNS 서버에 요청을 보냅니다. 일반적으로 로컬 라우터나 인터넷 공급자의 캐시 DNS 서버로
  보내지죠.
* 만약 DNS 서버가 같은 서브넷에 존재한다면 이 네트워크 라이브러리는 DNS 서버에 대해 ``ARP 프로세스``
  를 거칩니다.
* 만약 DNS 서버가 다른 서브넷에 존재한다면, 네트워크 라이브러리는 기본 게이트웨이 IP에 대해
  ``ARP 프로세스`` 를 거칩니다.

ARP 프로세스
----------

ARP (주소 결정 프로토콜, Address Resolution Protocol) 브로드캐스트를 보내기 위해서는
네트워크 스택 라이브러리가 검색할 목적지 IP의 주소를 알아야 합니다. 또, ARP 브로드캐스트를 보내는 데
사용하는 인터페이스의 MAC 주소 역시 알아야 합니다.

가장 먼저, ARP 캐시가 목적지 IP의 ARP 항목을 가지고 있는지 점검합니다. 만약 캐시에 있다면 라이브러리
함수는 다음의 형태로 결과를 리턴합니다: 목적지 IP = MAC.

항목이 ARP 캐시에 없다면:

* 라우트 테이블을 검색해서 목적지 IP 주소가 로컬 라우트 테이블의 서브넷에 존재하는지 봅니다. 존재한다면,
  라이브러리가 그 서브넷에 속하는 인터페이스를 활용합니다. 없다면, 라이브러리는 우리 기본 게이트웨이의
  서브넷에 속하는 인터페이스를 활용합니다.

* 선택된 네트워크 인터페이스의 MAC 주소가 검색이 됩니다.

* 네트워크 라이브러리는 레이어 2 (`OSI 모델`_에서 데이터 링크 레이어) 를 통해 ARP 요청을 보냅니다:

``ARP Request``::

    Sender MAC: interface:mac:address:here
    Sender IP: interface.ip.goes.here
    Target MAC: FF:FF:FF:FF:FF:FF (Broadcast)
    Target IP: target.ip.goes.here

컴퓨터와 라우터 사이에 어떤 하드웨어가 있는지에 따라:

직접 연결시:

* 컴퓨터가 라우터에 직접 연결되어 있으면 라우터는 ``ARP Reply`` 를 회신합니다.(아래를 확인하세요)

허브:

* 컴퓨터가 허브에 연결되어 있으면, 허브가 ARP 요청을 모든 포트에 브로드캐스트합니다. 라우터가 동일한
  "Wire"에 연결되어 있으면, 허브가 ``ARP Reply`` 를 회신하게 되지요.(아래를 확인하세요)

스위치:

* 만약 컴퓨터가 스위치에 연결되어 있다면, 스위치가 자신의 로컬 CAM/MAC 테이블을 확인해 어떤 포트가
  지금 찾고자하는 MAC 주소를 가지고 있는지 봅니다. 스위치에 해당 MAC 주소가 없다면 ARP 요청을 모든
  포트에 다시 브로드캐스트 하게 되지요.

* 스위치가 MAC/CAM 테이블에서 해당 주소를 찾으면 ARP 요청을 해당 주소의 포트에 보냅니다.

* 라우터가 동일한 "wire"에 있다면, 스위치가 ``ARP Reply`` 를 회신합니다.(아래를 확인하세요)

``ARP Reply``::

    Sender MAC: target:mac:address:here
    Sender IP: target.ip.goes.here
    Target MAC: interface:mac:address:here
    Target IP: interface.ip.goes.here


이제 네트워크 라이브러리는 우리 DNS 서버나 DNS 프로세스를 재개할 수 있는 기본 게이트웨이 중 하나의
IP 주소를 갖고 있습니다:

* 53번 포트는 DNS 서버에 UDP 요청을 보내기 위해 열려 있습니다 (만약 응답 크기가 너무 크다면,
  TCP가 대신 사용되구요).
* 로컬/ISP의 DNS 서버가 해당 정보를 갖고 있지 않다면, 재귀적인 탐색이 수행되고 SOA가 도달해서
  해답이 되돌아올 때까지 DNS 서버 리스트를 타고 올라갑니다

소켓 열기
-------

브라우저가 목적지 서버의 IP 주소를 받으면, 거기서 호스트명과 포트 번호(HTTP 프로토콜에서 기본값 80,
HTTPS에서는 443)를 뽑아내어, ``socket`` 이라는 이름의 시스템 라이브러리를 호출하고 TCP 소켓 스트림
- ``AF_INET/AF_INET6`` 과 ``SOCK_STREAM`` - 을 요청합니다.

* 이 요청은 먼저 TCP 세그먼트가 제작되는 Transport 레이어로 전달됩니다. 목적지 포트는 헤더에
  더해지고, 출발지 포트는 커널의 동적 포트 범위 (리눅스의 ip_local_port_range) 에서 선택됩니다.

* 이 세그먼트는 추가적인 IP 헤더를 덧씌우는 Network 레이어로 보내집니다. 지금의 머신뿐 아니라 목적지
  서버의 IP 주소도 담아 패킷을 만들죠.

* 패킷은 곧 Link 레이어에 도착합니다. 머신 NIC의 MAC 주소에 게이트웨이(로컬 라우터)의 MAC 주소까지
  포함한 프레임 헤더가 더해지죠. 전과 마찬가지로, 커널이 게이트웨이의 MAC 주소를 모르면, ARP 쿼리를
  브로드캐스트 해서 찾아야합니다.

이 지점에서 패킷은 다음 중 하나로 전송될 준비를 마칩니다:

* `이더넷`_
* `와이파이`_
* `무선 통신 네트워크`_

대부분의 집이나 소규모 업체의 인터넷 연결에서 패킷은 컴퓨터로부터, 아마도 로컬 네트워크를 통해,
모뎀 (MOdulator/DEModulator) 으로 보내지고 이를 통해 디지털 신호인 1과 0이, 전화나 케이블, 혹은
무선 통신 연결 등으로 전달되기 적합한 아날로그 신호로 변환됩니다. 그 연결의 반대편에서는 아날로그 신호를
디지털 신호로 되돌려주는 또 다른 모뎀이 다음 ``네트워크 노드`` 가 출발지와 도착지를 분석할 수 있도록
해줍니다.

대부분의 큰 사업체나 몇몇 신축 단지에서는 데이터를 다음 ``네트워크 노드`` 까지 디지털로 직접 연결해주는
광케이블 및 다이렉트 이더넷 연결이 존재하기도 합니다.

결국, 패킷은 로컬 서브넷을 관리하는 라우터에 도착합니다. 거기서부터, 패킷은 자율 시스템 (AS) 의 보더
라우터까지, 다른 자율 시스템까지, 그리고 결국 목적지 서버까지 여행하게 되죠. 이 때 지나치는 각각의
라우터는 IP 헤더로부터 목적지 주소를 추출해내서 적절한 다음 단계가지 이어줍니다. IP 헤더 내의
Time to live (TTL) 영역은 라우터를 하나씩 지날 때마다 감소됩니다. TTL 영역이 0이 되거나 도달한
라우터의 큐에 (네트워크 혼잡과 같은 이유로) 자리가 없을 때 패킷은 드랍됩니다.

이 송수신 동작은 다음의 TCP 연결 흐름을 따라 여러 차례 일어납니다:

* 클라이언트가 초기 순서 번호 (ISN, Initial Sequence Number) 을 선택하고, ISN을 설정하는
  중임을 나타내는 SYN 비트가 set된 한 패킷을 서버로 보냅니다.

* 서버가 SYN을 수신하고 수용가능한 상태인지 확인합니다:
   * 서버가 자신의 initial sequence number를 고릅니다
   * 서버가 ISN 선택중임을 알리는 SYN 비트를 set합니다
   * 서버가 (클라이언트 ISN + 1) 을 ACK 영역에 붙이고 첫 번째 패킷을 확인했다고 알리는 ACK
     플래그를 추가합니다

* 클라이언트가 패킷을 하나 보내 연결을 확인해줍니다:
   * 자신의 ISN을 하나 올립니다
   * 수신자 확인 번호를 하나 올립니다
   * ACK 필드를 set합니다.

* 데이터가 다음과 같이 옮겨집니다:
   * 한 쪽에서 N개의 데이터 바이트를 보내면서, SEQ를 해당 숫자만큼 증가시킵니다
   * 반대편이 그 패킷 (혹은 연결된 여러 패킷) 을 받았다고 알리면, 상대로부터 마지막에 받았던 순서와
     같은 ACK 값을 담아 ACK 패킷을 보냅니다

* 연결을 끊을 때:
   * 닫는 쪽이 FIN 패킷을 보냅니다
   * 반대편이 FIN 패킷을 ACK하고 자신의 FIN을 보냅니다
   * 닫는 쪽이 반대편의 FIN을 ACK와 함께 확인하고 알립니다

TLS handshake
-------------

* 클라이언트 컴퓨터가 자신의 Transport Layer Security (TLS) 버전, 암호 알고리즘 목록 그리고
  사용 가능한 압축 방식을 ``ClientHello`` 메시지에 담아 서버로 보냅니다.

* 서버는 클라이언트에게 TLS 버전, 선택한 암호 알고리즘, 선택한 압축 방식 그리고
  CA (Certificate Authority) 가 사인한 서버의 공개 인증서를 ``ServerHello`` 메시지에 담아
  답장합니다. 이 인증서는 대칭키가 생성되기 전까지 클라이언트가 나머지 handshake 과정을 암호화하는
  데에 쓸 공개키를 담고 있죠.

* 클라이언트는 서버측 디지털 인증서가 유효한지를, 신뢰할 수 있는 CA 목록을 통해 확인합니다. 만약 CA를
  통해 신뢰성이 확보되면, 클라이언트는 의사 난수 (pseudo-random) 바이트를 생성해 서버의 공개키로
  암호화하구요. 이 난수 바이트는 대칭키를 정하는 데에 사용됩니다.

* 서버는 난수 바이트를 자기 개인키로 복호화해 대칭 마스터키 생성에 활용합니다.

* 클라이언트는 ``Finished`` 메시지를 서버에 보내면서, 지금까지의 교환 내역을 해시한 값을 대칭키로
  암호화하여 담습니다.

* 서버는 스스로도 해시를 생성해 클라이언트에서 도착한 값과 일치하는지 봅니다. 일치하면, 서버도 마찬가지로
  대칭키를 통해 암호화한 ``Finished`` 메시지를 클라이언트에 보내죠.

* 이제부터 TLS 세션이 대칭키로 암호화된 어플리케이션 (HTTP) 데이터를 전송합니다.

HTTP 프로토콜
-----------

구글이 만든 웹 브라우저라면, 페이지를 가져오기 위해 HTTP 요청을 보내는 대신, 서버에게 HTTP에서
SPDY로 "업그레이드"할 것을 협상해봅니다.

만약 클라이언트가 SPDY를 지원하지 않고 HTTP만 쓴다면, 서버에 다음과 같은 요청을 보내죠::

    GET / HTTP/1.1
    Host: google.com
    Connection: close
    [other headers]

``[other headers]`` 부분은 HTTP 사양에 따라 콜론으로 구분되고 각각 새 줄로 나뉘는 일련의 키-값
쌍을 나타냅니다. (이 부분은 사용된 브라우저가 HTTP 스펙을 벗어나는 어떠한 버그도 없을 때를 가정해요.
웹 브라우저가 ``HTTP/1.1`` 을 쓴다는 것도 마찬가지인데, 그렇지 않을 경우엔 ``Host`` 헤더가 요청에
포함되지 않고 ``GET`` 요청에 명시된 버전이 ``HTTP/1.0`` 혹은 ``HTTP/0.9`` 일 수도 있습니다. )

HTTP/1.1은 송신자측에서 응답을 받은 직후에 연결이 끊어질 것이라는 신호를 보내기 위해 "close"라는
연결 옵션을 정의합니다. 아래의 예처럼 말이죠.

    Connection: close

영구 접속을 허용하지 않는 HTTP/1.1 어플리케이션은 반드시 "close" 연결 옵션을 모든 메시지에 포함해야
합니다.

요청과 헤더를 보낸 후에, 웹 브라우저는 하나의 빈 줄을 서버에 보내 요청 내용이 모두 보내졌음을
알립니다.

서버는 요청의 상태를 나타내는 코드와 다음과 같은 형태의 답신으로 응답하죠::

    200 OK
    [response headers]

빈 줄을 하나 붙인 뒤, ``www.google.com`` 의 HTML 본문을 페이로드에 담아 보냅니다. 서버는 곧
연결을 끊거나, 클라이언트가 보낸 헤더에 요청이 있었을 시, 추가적인 요청을 위해 재사용될 수 있도록
연결을 유지해둡니다.

웹 브라우저에서 보낸 HTTP 헤더에, 마지막으로 보냈던 파일이 브라우저에 캐시되어 있고 그 뒤로 변하지
않았다는 판단을 내릴 만큼 충분한 정보 (예를 들어, 웹 브라우저가 ``ETag`` 헤더를 포함시켰다든지) 가
담겨 있었다면, 아래와 같이 응답할 수도 있어요::

    304 Not Modified
    [response headers]

페이로드 없이, 대신 브라우저가 자체 캐시에서 HTML 폼을 가져오게 말이죠.

HTML을 파싱한 후에는, 브라우저 (그리고 서버) 가 이 과정을 HTML 페이지에서 참조되는 모든 자원
(이미지, CSS, favicon.ico, 기타 등등) 에 대해 반복합니다. 요청이 ``GET / HTTP/1.1`` 대신
``GET /$(URL relative to www.google.com) HTTP/1.1`` 이 된다는 것만 빼고 말입니다.

HTML이 ``www.google.com`` 이 아닌 도메인의 자원을 참조할 땐, 브라우저가 다른 도메인을 확정하는
단계로 되돌아가 해당 도메인에 대해 여기까지의 과정들을 밟습니다. 요청에 들어있는 ``Host`` 헤더는
``google.com`` 대신 적당한 서버 이름으로 설정되겠죠.

HTTP Server Request Handle
--------------------------
The HTTPD (HTTP Daemon) server is the one handling the requests/responses on
the server side. The most common HTTPD servers are Apache or nginx for Linux
and IIS for Windows.

* The HTTPD (HTTP Daemon) receives the request.
* The server breaks down the request to the following parameters:
   * HTTP Request Method (either ``GET``, ``HEAD``, ``POST``, ``PUT``,
     ``DELETE``, ``CONNECT``, ``OPTIONS``, or ``TRACE``). In the case of a URL
     entered directly into the address bar, this will be ``GET``.
   * Domain, in this case - google.com.
   * Requested path/page, in this case - / (as no specific path/page was
     requested, / is the default path).
* The server verifies that there is a Virtual Host configured on the server
  that corresponds with google.com.
* The server verifies that google.com can accept GET requests.
* The server verifies that the client is allowed to use this method
  (by IP, authentication, etc.).
* If the server has a rewrite module installed (like mod_rewrite for Apache or
  URL Rewrite for IIS), it tries to match the request against one of the
  configured rules. If a matching rule is found, the server uses that rule to
  rewrite the request.
* The server goes to pull the content that corresponds with the request,
  in our case it will fall back to the index file, as "/" is the main file
  (some cases can override this, but this is the most common method).
* The server parses the file according to the handler. If Google
  is running on PHP, the server uses PHP to interpret the index file, and
  streams the output to the client.

Behind the scenes of the Browser
----------------------------------

Once the server supplies the resources (HTML, CSS, JS, images, etc.)
to the browser it undergoes the below process:

* Parsing - HTML, CSS, JS
* Rendering - Construct DOM Tree → Render Tree → Layout of Render Tree →
  Painting the render tree

Browser
-------

The browser's functionality is to present the web resource you choose, by
requesting it from the server and displaying it in the browser window.
The resource is usually an HTML document, but may also be a PDF,
image, or some other type of content. The location of the resource is
specified by the user using a URI (Uniform Resource Identifier).

The way the browser interprets and displays HTML files is specified
in the HTML and CSS specifications. These specifications are maintained
by the W3C (World Wide Web Consortium) organization, which is the
standards organization for the web.

Browser user interfaces have a lot in common with each other. Among the
common user interface elements are:

* An address bar for inserting a URI
* Back and forward buttons
* Bookmarking options
* Refresh and stop buttons for refreshing or stopping the loading of
  current documents
* Home button that takes you to your home page

**Browser High Level Structure**

The components of the browsers are:

* **User interface:** The user interface includes the address bar,
  back/forward button, bookmarking menu, etc. Every part of the browser
  display except the window where you see the requested page.
* **Browser engine:** The browser engine marshals actions between the UI
  and the rendering engine.
* **Rendering engine:** The rendering engine is responsible for displaying
  requested content. For example if the requested content is HTML, the
  rendering engine parses HTML and CSS, and displays the parsed content on
  the screen.
* **Networking:** The networking handles network calls such as HTTP requests,
  using different implementations for different platforms behind a
  platform-independent interface.
* **UI backend:** The UI backend is used for drawing basic widgets like combo
  boxes and windows. This backend exposes a generic interface that is not
  platform specific.
  Underneath it uses operating system user interface methods.
* **JavaScript engine:** The JavaScript engine is used to parse and
  execute JavaScript code.
* **Data storage:** The data storage is a persistence layer. The browser may
  need to save all sorts of data locally, such as cookies. Browsers also
  support storage mechanisms such as localStorage, IndexedDB, WebSQL and
  FileSystem.

HTML parsing
------------

The rendering engine starts getting the contents of the requested
document from the networking layer. This will usually be done in 8kB chunks.

The primary job of HTML parser to parse the HTML markup into a parse tree.

The output tree (the "parse tree") is a tree of DOM element and attribute
nodes. DOM is short for Document Object Model. It is the object presentation
of the HTML document and the interface of HTML elements to the outside world
like JavaScript. The root of the tree is the "Document" object. Prior of
any manipulation via scripting, the DOM has an almost one-to-one relation to
the markup.

**The parsing algorithm**

HTML cannot be parsed using the regular top-down or bottom-up parsers.

The reasons are:

* The forgiving nature of the language.
* The fact that browsers have traditional error tolerance to support well
  known cases of invalid HTML.
* The parsing process is reentrant. For other languages, the source doesn't
  change during parsing, but in HTML, dynamic code (such as script elements
  containing `document.write()` calls) can add extra tokens, so the parsing
  process actually modifies the input.

Unable to use the regular parsing techniques, the browser utilizes a custom
parser for parsing HTML. The parsing algorithm is described in
detail by the HTML5 specification.

The algorithm consists of two stages: tokenization and tree construction.

**Actions when the parsing is finished**

The browser begins fetching external resources linked to the page (CSS, images,
JavaScript files, etc.).

At this stage the browser marks the document as interactive and starts
parsing scripts that are in "deferred" mode: those that should be
executed after the document is parsed. The document state is
set to "complete" and a "load" event is fired.

Note there is never an "Invalid Syntax" error on an HTML page. Browsers fix
any invalid content and go on.

CSS interpretation
------------------

* Parse CSS files, ``<style>`` tag contents, and ``style`` attribute
  values using `"CSS lexical and syntax grammar"`_
* Each CSS file is parsed into a ``StyleSheet object``, where each object
  contains CSS rules with selectors and objects corresponding CSS grammar.
* A CSS parser can be top-down or bottom-up when a specific parser generator
  is used.

Page Rendering
--------------

* Create a 'Frame Tree' or 'Render Tree' by traversing the DOM nodes, and
  calculating the CSS style values for each node.
* Calculate the preferred width of each node in the 'Frame Tree' bottom up
  by summing the preferred width of the child nodes and the node's
  horizontal margins, borders, and padding.
* Calculate the actual width of each node top-down by allocating each node's
  available width to its children.
* Calculate the height of each node bottom-up by applying text wrapping and
  summing the child node heights and the node's margins, borders, and padding.
* Calculate the coordinates of each node using the information calculated
  above.
* More complicated steps are taken when elements are ``floated``,
  positioned ``absolutely`` or ``relatively``, or other complex features
  are used. See
  http://dev.w3.org/csswg/css2/ and http://www.w3.org/Style/CSS/current-work
  for more details.
* Create layers to describe which parts of the page can be animated as a group
  without being re-rasterized. Each frame/render object is assigned to a layer.
* Textures are allocated for each layer of the page.
* The frame/render objects for each layer are traversed and drawing commands
  are executed for their respective layer. This may be rasterized by the CPU
  or drawn on the GPU directly using D2D/SkiaGL.
* All of the above steps may reuse calculated values from the last time the
  webpage was rendered, so that incremental changes require less work.
* The page layers are sent to the compositing process where they are combined
  with layers for other visible content like the browser chrome, iframes
  and addon panels.
* Final layer positions are computed and the composite commands are issued
  via Direct3D/OpenGL. The GPU command buffer(s) are flushed to the GPU for
  asynchronous rendering and the frame is sent to the window server.

GPU Rendering
-------------

* During the rendering process the graphical computing layers can use general
  purpose ``CPU`` or the graphical processor ``GPU`` as well.

* When using ``GPU`` for graphical rendering computations the graphical
  software layers split the task into multiple pieces, so it can take advantage
  of ``GPU`` massive parallelism for float point calculations required for
  the rendering process.


Window Server
-------------

Post-rendering and user-induced execution
-----------------------------------------

After rendering has completed, the browser executes JavaScript code as a result
of some timing mechanism (such as a Google Doodle animation) or user
interaction (typing a query into the search box and receiving suggestions).
Plugins such as Flash or Java may execute as well, although not at this time on
the Google homepage. Scripts can cause additional network requests to be
performed, as well as modify the page or its layout, causing another round of
page rendering and painting.

.. _`Creative Commons Zero`: https://creativecommons.org/publicdomain/zero/1.0/
.. _`"CSS lexical and syntax grammar"`: http://www.w3.org/TR/CSS2/grammar.html
.. _`퓨니코드 (Punycode)`: https://en.wikipedia.org/wiki/Punycode
.. _`이더넷`: http://en.wikipedia.org/wiki/IEEE_802.3
.. _`와이파이`: https://en.wikipedia.org/wiki/IEEE_802.11
.. _`무선 통신 네트워크`: https://en.wikipedia.org/wiki/Cellular_data_communication_protocol
.. _`analog-to-digital converter`: https://en.wikipedia.org/wiki/Analog-to-digital_converter
.. _`network node`: https://en.wikipedia.org/wiki/Computer_network#Network_nodes
.. _`OS에 따라`: https://en.wikipedia.org/wiki/Hosts_%28file%29#Location_in_the_file_system
.. _`简体中文`: https://github.com/skyline75489/what-happens-when-zh_CN
.. _`다운그레이드 공격 (downgrade attack)`: http://en.wikipedia.org/wiki/SSL_stripping
.. _`OSI 모델`: https://en.wikipedia.org/wiki/OSI_model
.. _`이 곳`: https://github.com/alex/what-happens-when
