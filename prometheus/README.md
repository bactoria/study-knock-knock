# Prometheus 기초 공사  🔨뚝딱뚝딱

<img src="https://user-images.githubusercontent.com/25674959/115962989-c83bcf00-a558-11eb-8d9a-ce8cb6bdb7b1.png" width=400>

(증오하지 않아!!)

&nbsp;

## 어디다 쓰는건가 ? 왜 알아야하지? 🤔

모니터링 도구

&nbsp;

## 모니터링 기본 지식

<details markdown="1">
<summary>수집하는 지표</summary>

1. 메트릭
2. 로그
3. 머더라 - tcpdump
4. 또하나

</details>


<details markdown="1">
<summary>지표 수집 방식 pull / push</summary>

ㅇ

</details>



## 프로메테우스 아키텍처

<img src="https://user-images.githubusercontent.com/25674959/115963748-b0b21580-a55b-11eb-90a5-ab2e9f935923.png" width=500>

컴포넌트
1. 프로메테우스 서버 : 시계열 데이터를 수집하여 저장. 쿼리 가능
2. 푸시 게이트웨이 : 클라이언트에서 직접 프로메테우스로 데이터를 보낼 때 받는 역할을 함  
  (클라이언트 라이브러리) : 애플리케이션을 개발할 때 프로메테우스에서 데이터를 수집하도록 만드는 라이브러리  
  이를 통해 애플리케이션 인스턴스의 메트릭을 정의하고, HTTP 엔드 포인트를 통하여 노출 할 수 있다.
3. 익스포터 : 프로메테우스 클라이언트 라이브러리를 내장해 만들지 않은 애플리케이션에서 데이터를 수집. (?) 100개가 넘는 다양한 익스포터가 있으므로 거의 모든 애플리케이션에서 데이터를 수집할 수 있다.
4. 알람 관리자 : 알람을 보낼 때 중복 처리, 그룹화 등을 하며 알람을 어디로 보낼지 관리  
  eg. 슬랙, 이메일


### 메트릭 수집 방식

메트릭: 타임스탬프와 보통 한두 가지 숫자 값을 포함하는 이벤트
(특정 상황에 발생하는 로그와는 달리, 주기적으로 발생함.)

프로메테우스는 pull 방식을 사용한다.

push 방식 : eg.
장점: 서비스가 오토 스케일링 등으로 가변적일 경우 유리. 

pull 방식 : eg.
단점: 풀링의 경우에는 모니터링 대상이 가변적으로 변경될 때 모니터링 대상의 IP 주소들을 알 수가 없다. eg. 웹서버 VM 2개의 주소를 설정 파일에 넣고 모니터링을 하고 있었는데, 오토 스케일링으로 인해서 VM이 3개가 더 추가되면, 추가된 VM들은 설정 파일에 IP가 들어 있지 않기 때문에 모니터링 대상에서 제외 된다.  <- 조대협님 블로그

위 그림에서 볼 수 있듯이, 프로메테우스는 ~ 로 ~한다. (pull 방식)

서비스 디스커버리
특정 시스템이 현재 기동중인 서비스들의 목록과 IP 주소를 가지고 있으면 된다. 예를 들어 앞에서 VM들을 내부 DNS에 등록해놓고 새로운 VM이 생성될때에도 DNS에 등록을 하도록 하면, DNS에서 현재 기동중인 VM 목록을 얻어와서 그 목록의 IP들로 풀링을 하면 되는 구조이다. <- 조대협님.
쿠버네티스를 사용하는 경우에는 ~

**익스포터**
익스포터를 다시 보자면, 프로메테우스는 익스포터에 HTTP GET으로 요청을 하고, 익스포터는 타겟 시스템에서 메트릭을 읽어서 텍스트 형태로 프로메테우스에 리턴한다. (요청 당시의 데이타를 리턴하는 것) Exporter 는 히스토리를 저장하고있지 않다. 라고함.. 대협님이..

장점: 구현이 쉽다, 별도의 포트개방 필요없다. 대협님.

**Retrieval**
서비스 디스커버리 시스템으로 부터 모니터링 대상 목록을 받아오고, Exporter로 부터 주기적으로 그 대상으로 부터 메트릭을 수집하는 모듈이 프로메테우스내의 Retrieval 이라는 컴포넌트이다. [조대협의 블로그]

프로메테우스는 구조상 HA를 위한 이중화나 클러스터링등이 불가능하다. (클러스터링 대신 샤딩을 사용한다. HA는 복제가 아니라 프로메테우스를 두개를 띄워서 같은 타겟을 동시에 같이 저장 하는 방법을 사용한다. 이 문제에 대한 해결 방법은 Thanos 라는 오픈 소스를 사용하면 되는데, 이는 다음에 다시 설명하도록 한다. )
홀리싵!!!!!!!!!!!!!!!! 이거 엄청 중요한거같은데

- https://gompangs.github.io/2018/12/14/prometheus/
- https://www.youtube.com/watch?v=qHIgk547SVA
- https://bcho.tistory.com/1372
- https://medium.com/finda-tech/prometheus%EB%9E%80-cf52c9a8785f
- https://teamsmiley.github.io/2020/01/17/prometheus/

### 시각화

프로메테우스 서버의 9090 포트로 접속하면 대쉬보드가 나온다. 
여기서 쿼리 입력 시 테이블/그래프를 볼 수 있다.


시각화는 Grafana를 이용한다.
Grafana 는 프로메테우스가 수집한 메트릭을 시각화해주는 오픈소수 도구이다. 


## Hello World

로컬에서 스프링부트의 메트릭 수집 & 그라파나 확인 실습. 
쿠버네티스 파드로부터 수집 실습. <- 프로메테우스도 파드로 실행 시켜야하려나 ? 

