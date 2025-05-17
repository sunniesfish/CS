
| 사용 사례             | 7계층 (응용)           | 6계층 (표현)        | 5계층 (세션)   | 4계층 (전송)       | 3계층 (네트워크)  | 2계층 (데이터링크)         | 1계층 (물리)       |
| ----------------- | ------------------ | --------------- | ---------- | -------------- | ----------- | ------------------- | -------------- |
| **웹 브라우징**        | HTTP / HTTPS       | TLS             | (암묵적 세션)   | TCP            | IP          | Ethernet / Wi-Fi    | UTP / 무선 / 광섬유 |
| **API 통신 (REST)** | REST (HTTP 기반)     | TLS (HTTPS)     | -          | TCP            | IP          | Ethernet / Wi-Fi    | 물리매체           |
| **파일 전송 (SFTP)**  | SFTP               | SSH 암호화         | SSH 세션     | TCP            | IP          | Ethernet / Wi-Fi    | 물리매체           |
| **DNS 요청/응답**     | DNS                | -               | -          | UDP / TCP      | IP          | Ethernet / Wi-Fi    | 물리매체           |
| **이메일 송수신**       | SMTP, IMAP, POP3   | TLS / SSL       | -          | TCP            | IP          | Ethernet / Wi-Fi    | 물리매체           |
| **영상 스트리밍**       | HTTP, DASH, HLS    | TLS (HTTPS 기반)  | -          | TCP / UDP      | IP          | Ethernet / Wi-Fi    | 물리매체           |
| **온라인 게임**        | 자체 게임 프로토콜         | 선택적 암호화         | -          | UDP (+TCP 일부)  | IP          | Ethernet / Wi-Fi    | 물리매체           |
| **VoIP (인터넷전화)**  | SIP, RTP, SDP      | 코덱 포함 (G.711 등) | -          | UDP            | IP          | Ethernet / Wi-Fi    | 물리매체           |
| **IoT 통신**        | MQTT / CoAP        | TLS (MQTTS)     | -          | TCP / UDP      | IP          | ZigBee / Wi-Fi      | 무선 전파          |
| **VPN**           | -                  | IPsec / SSL     | L2TP, PPTP | UDP / TCP      | IP (터널링 사용) | Ethernet / PPP      | 물리매체           |
| **보안 DNS**        | DNS over HTTPS/TLS | TLS             | -          | TCP            | IP          | Ethernet / Wi-Fi    | 물리매체           |
| **클라우드 내부통신**     | gRPC, REST API     | TLS             | -          | HTTP/2 or QUIC | IP          | VXLAN / Geneve      | 물리계층 (데이터센터)   |
| **금융 (증권 거래)**    | FIX                | TLS             | -          | TCP            | IP          | 고속 이더넷              | 광케이블           |
| **산업제어**          | Modbus, OPC-UA     | -               | -          | TCP / UDP      | IP          | Profinet / EtherCAT | 전송선, 필드버스      |
