# CORS
- **SOP** (Same Origin Policy) : 다른 출처에서 리소스 접근 제어하는것을 제어하는 보안 방식
  - 기존에는 Resource와 View를 같은 Origin에서 제공하는 경우가 많았지만, 이제는 서로 다른 서버에서 자원을 주고 받는 일이 많아 설정이 필요
  - 같은 Origin이 되려면 아래의 세 가지가 모두 동일해야함
    1. 프로토콜 (Protocol): 예를 들어, http 또는 https와 같은 프로토콜이 동일해야 함.
    2. 호스트 (Host): 도메인 이름이나 IP 주소.
    3. 포트 (Port)
- **Cross-Origin Resource Sharing** : 다른 출처(Origin)에서 리소스에 접근할 수 있는 권한을 부여하는 웹 브라우저 보안 메커니즘
- **Preflight Request** :  클라이언트가 실제 요청을 보내기 전에 서버로 보내는 사전 요청
  - 서버가 클라이언트의 요청을 받아들일지 여부를 결정하는 데 사용
  - 실제 요청의 메타데이터와 함께 OPTIONS 메서드를 사용하여 CORS(Cross-Origin Resource Sharing) 정책을 확인하는 데 필요한 정보가 포함
  - 서버는 이 Preflight Request를 받아들이고 CORS 정책을 확인한 후, 실제 요청을 허용하거나 거부할 수 있음
  - **HTTP OPTIONS 메소드를 사용**
  - **Preflight Request**가 생략되는 경우 : 
    1. Simple Request(단순 요청)인 경우 (아래 조건을 만족해야함)
      - 요청 : GET, HEAD, POST
      - Custom 헤더가 "Accept", "Accept-Language", "Content-Language"일 때
      - 헤더가 Content Type이고 미디어 타입이 text/plain, application/x-www-form-urlencoded, multipart/form-data 중 하나일때
    2. 브라우저가 CORS Preflight Caching을 사용하는 경우: 브라우저는 이전에 서버로부터 받은 CORS 정책을 캐싱하여, 동일한 요청이 다시 발생할 때 Preflight Request를 생략함, (네트워크 부하 감소)
      - 서버에서 Cors설정 중 캐싱 시간을 정할 수 있음 : Access-Control-Max-Age
    

   
----![image](https://github.com/IMHYEWON/java-spring-notes/assets/37797830/105bc8ef-6637-4e69-add2-582cfb0fe114)

![image](https://github.com/IMHYEWON/java-spring-notes/assets/37797830/bcb1aa0d-1040-4674-9020-d7935be1ffb3)
