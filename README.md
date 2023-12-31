# java-optimization

## 성능과 최적화
### 성능은 실험과학이다.
> 성능은 다음과 같은 활동을 하면서 원하는 결과를 얻기 위한, 일종의 실험과학이라고 볼 수 있다.  
> * 원하는 결과를 정의한다.
> * 기존 시스템을 측정한다.
> * 요건을 충족시키려면 무슨 일을 해야 할지 정한다.
> * 개선 활동을 추진한다.
> * 다시 테스트한다.
> * 목표가 달성됐는지 판단한다. 

### 성능 분류
> **처리율(throughput)**  
> 처리율은 시스템이 수행 가능한 작업 비율을 나타낸 지표이다. 보통 일정 시간 동안 완료한 작업 단위 수로 표시한다.(예: 초당 처리 가능한 트랜잭션 수 TPS)  
> 처리율이 실제 성능을 반영하는 의미 있는 지표가 되려면 수치를 얻은 기준 플랫폼에 대해서도 내용을 기술해야 한다. 
> * 하드웨어 스펙
> * OS
> * 소프트웨어 스택
> * 테스트한 시스템의 환경: 단일/클러스터
> * 워크로드(작업 할당량)
> 
> **지연(latency)**  
> 하나의 트랜잭션을 처리하고 그 결과를 응답받았을 때까지의 소요된 시간.
> 
> **용량(capacity)**  
> 용량은 시스템이 보유한 작업 병렬성의 총량, 즉 시스템이 동시 처리 가능한 작업 단위(즉, 트랜잭션) 개수를 말한다.  
> 
> **사용률(utilization)**  
> 성능 분석 업무 중 가장 흔한 태스크는 시스템 리소스를 효율적으로 활용하는 것이다. 
> 
> **효율(efficiency)**  
> 전체 시스템의 효율은 처리율을 리소스 사용률로 나눈 값으로 측정한다.
> 
> **확장성(scalability)**  
> 처리율이나 시스템 용량은, 처리하는 데 끌어 쓸 수 있는 리소스에 달려 있다. 리소스 추가에 따른 처리율 변화는 시스템/애플리케이션의 확장성을 가늠하는 척도이다.  
> 시스템 확장성은 궁극적으로는 정확히 리소스를 투입한 만큼 처리율이 변경되는 형태를 지향한다.
> 
> **저하(degradation)**  
> 시스템이 더 많은 부하를 받으면 지연 그리고 처리율 측정값에 변화가 생긴다. 시스템을 덜 사용하고 있으면 측정값이 느슨하게 변하지만, 
> 시스템이 풀 가동된 상태면 처리율이 더는 늘어나지 않는, 즉 지연이 증가하는 양상을 띤다. 이런 현상을 부하 증가에 따른 저하라고 한다.  

---

## JVM 이야기
### Hotspot VM 과 JIT(Just In Time) 컴파일러
> **C 언어의 동작 방식**
> C, C++, GoLang, Rust 등과 같은 컴파일 언어는 컴파일 과정에서 바로 기계어로 번역하고 실행 파일을 만들어낸다. 
> 그리고 컴파일 시에 코드 최적화까지 진행하여 처리 성능이 상당히 뛰어나다. 대신 생성된 기계어가 빌드 환경(CPU 아키텍처)에 종속적이라서, 플랫폼이 바뀐다면 다시 빌드해야 하는 문제가 있다.
> 
> **Java 언어의 동작 방식**  
> 자바는 이러한 플랫폼 종속적인 문제를 해결하고자 JVM을 도입하였고, 그래서 동작 과정이 조금 다르다.
> 만약 우리가 자바 애플리케이션을 실행한다고 하면, 자바 코드는 먼저 바이트 코드로 컴파일된다. 바이트 코드는 자바 코드보다 간결하고 간단하지만, 
> 0과 1만 이해할 수 있는 컴퓨터는 이를 읽어들일 수 없다. 바이트 코드는 JVM 을 위한 중간 언어인 것이다. 따라서 JVM 은 애플리케이션이 실행되면서 읽어들인 
> 바이트 코드를 실시간으로 기계어로 번역하고, CPU 가 번역된 기계어를 처리한다. 이러한 구조 덕분에 Java 는 플랫폼에 종속되지 않게 되었다.
> 하지만 실행 시에 바이트 코드를 기계어로 번역하는 작업 때문에 성능이 느려졌다. 그래서 이러한 문제를 해결하고자 바이트 코드를 기계어로 컴파일하는 
> JIT 컴파일러를 도입하여 사용하고 있다. JIT 컴파일러의 목표는 빠른 컴파일 및 특정 환경에 맞춤화된 최적화를 제공하는 것이며, 이를 위해 실행 프로파일 정보를 활용한다.
> 
> **JIT 컴파일러의 등장**
> Java 1.3부터는 Hotspot VM이 추가되었고, Hotspot VM 에는 2개의 JIT 컴파일러가 포함되어 있다.  
> c1
> * 클라이언트 컴파일러(Client Compiler)
> * 코드 최적화는 덜하지만 즉시 시작되는 속도는 빠름
> * 즉시 실행되는 데스크톱 애플리케이션 등에 적합함
> 
> c2  
> * 서버 컴파일러(Server Compiler)
> * 즉시 시작되는 속도는 느리지만 최적화는 많이 되어 warm-up 후에는 빠름
> * 장기 실행되는 서버 애플리케이션 등에 적합함
> 
> Hotspot VM은 초기 컴파일에 C1 컴파일러를 사용한다. 하지만 Hotspot VM은 각 메소드의 호출 여부를 계속해서 주시하고, 호출 횟수가 증가하면 해당 메소드를 재컴파일 하는데, 
> 이번에는 C2 컴파일러를 사용한다. 이를 계층형 컴파일(Tiered Compliation)이라고도 한다.
> HotSpot은 99년도에 나왔고, 오랜 연구 결과 끝에 많은 최적화가 구현되어 있다. C2 컴파일러는 극도로 최적화가 많이 되어 있어서 컴파일 언어를 능가하는 성능을 보여주기도 한다. 
> 최적화에는 프로파일링을 통해 얻은 정보들(디바이스 정보, 클럭 수 등)이 사용되며, 자바의 성능 향상은 이러한 프로파일링을 통해 얻어낸 정보를 바탕으로 하는 JIT 컴파일러의 최적화 역할이 크다.
> 하지만 반대로 이로 인해 애플리케이션이 초기에 느리게 실행되는 웜업 문제가 발생하기도 한다.
> 
> **JIT 컴파일러의 한계**
> 하지만 문제는 HotSpot VM의 C2 컴파일러가 한계에 직면했다는 것이다. C2 컴파일러는 일단 C++로 작성되어 개발자를 구하기도 어렵고, 상당히 오래된 만큼 복잡하다. 
> 이러한 이유로 최근 몇 년 동안 개발된 중요한 최적화가 없었을 뿐만 아니라 전문가들 역시 유지보수가 어렵다고 판단하였다. 즉, C2 컴파일러가 이제 End-of-life 에 직면한 것이다.
> * C++ 개발자를 구하기 어려움
> * 오래된 만큼 상당히 복잡함
> * 최근에 중요한 최적화가 거의 없었음
>
> 그래서 최적화가 더 이상 어려운 코드들은 HotSpotInstrinsicCandidate 어노테이션을 붙이도록 하는 작업들도 진행되었다. 
> 한계에 마주한 상황을 인정하고 애노테이션으로 명시해두는 것이다.
> JIT 컴파일러는 느리지 않으며, 오히려 엄청난 최적화로 인해 컴파일된 언어보다 뛰어난 성능을 보이기도 한다. 
> 대신 더 이상 유지보수하기 어려운 한계에 직면한 상황이다. 그래서 이에 대한 대안으로 등장한 것이 바로 GraalVM이다.

### GraalVM 의 등장과 아키텍처
>  GraalVM은 Oracle JDK가 아닌 OpenJDK8을 기반으로 만들어졌으며, 이는 새로운 단독 프로젝트가 아닌 기존의 HotSpot VM을 개선하고, 다양한 기능을 추가한 것임을 의미한다. 
> 그 중에서 GraalVM은 JIT 컴파일러들 중에서 기존의 c++로 작성된 C2 컴파일러인 Graal 컴파일러를 Java 기반으로 새롭게 작성하였다. 
> 즉, 자바로 만든 자바(Java on Java) 또는 자바로 만든 자바 컴파일러와 같은 수식어로 GraalVM을 설명할 수 있다.
> 
> GraalVM은 크게 아래의 3가지 특징을 갖고 있다고 볼 수 있으며, 이는 GraalVM의 아키텍처를 통해 그 이유를 알 수 있다.
> * 고성능 자바(High-Performance)
> * 다양한 언어의 통합(Polyglot)
> * Native 지원을 통한 빠른 start-up(Native Image)

### AOT 컴파일
> **정의**
> * AOT Compile 은 Ahead-Of-Time compile (사전 컴파일) 의 약자이다.
> * JIT(Just-In-Time) 컴파일과 주로 비교된다.
> * 프로그램 실행 중 즉시 변환을 수행하는 JIT(Just-In-Time) 컴파일과 대조적이다.
> * 고수준의 소스 코드를 네이티브 머신 코드로 미리 변환하는 프로세스를 의미한다.
> 
> **AOT 컴파일의 장점**
> * 시작 속도: 기계어로 사전 컴파일 되어 있어 시작이 빠르다. 런타임 환경에서 바이트 코드를 네이티브 코드로 변경할 필요가 없다.  
> * 예측 가능한 성능: AOT 컴파일된 앱은 보다 일관되고 예측 가능한 런타임 성능을 제공한다. JIT 컴파일은 런타임 중에 발생하는 컴파일 단계로 인해 가변 오버헤드가 발생할 수 있기 때문이다.  
> * 코드 난독화: 바이트 코드보다 리버스 엔지니어링이 더 어려워진다.
> * 리소스 효율: CPU 와 메모리 사용량 측면에서 더 효율적이다. JIT 과 비교해 런타임 컴파일 프로세스가 없으므로 앱의 오버헤드가 줄어든다.
> * 런타임 컴파일러 필요 없음: 런타임 컴파일러를 탑재할 필요가 없어 앱의 크기와 복잡성이 줄어든다.
> * 최적의 코드 경로: JIT 에서 수행하기에 비현실적인 공격적인 최적화를 수행한다.
> * 크로스 플랫폼 호환성: 다양한 대상 플랫폼용 실행 파일을 더 쉽게 생성할 수 있다.

### JIT vs AOT 
> 이제 바이트 코드 컴파일이 작동하는 방식과 두 가지 주요 전략(JIT 및 AOT)을 이해했으므로 어떤 접근 방식을 사용하는 것이 가장 좋은지 궁금할 것입니다. 
> 불행히도 대답은 예상대로 "그때 그때 다릅니다."
> 
> JIT 컴파일러는 프로그램을 크로스 플랫폼으로 만들어줍니다(즉, 플랫폼에 독립적이라는 의미). 실제로 “한번만 작성하면 어디에서나 실행 할 수 있다” 라는 슬로건은 
> 90년대 후반에 Java를 대중적인 언어로 만든 기능 중 하나였습니다. 
> JIT 컴파일러는 동시 가비지 컬렉터를 사용하여 최대 처리량 조건에서 메모리 회복력을 높여 STW(Stop the World)를 짧게 가져갑니다.
> 
> 반면에 AOT 컴파일러는 프로그램을 보다 효율적으로 실행하며, 이는 특히 클라우드 애플리케이션에 적합합니다. 
> 네이티브 이미지는 더 빠른 시작 속도를 제공하므로 애플리케이션의 부팅 시간이 단축되고, 이는 클라우드 서비스의 Scale-out이 더욱 간편해지게 만듭니다. 
> 또한, 클라우드에서 실행되는 Docker 컨테이너로 초기화된 마이크로서비스의 경우에 특히 유용합니다. 
> 사용되지 않는 코드의 완전한 제거(클래스, 필드, 메서드, 분기) 덕분에 파일의 크기가 작아지기 때문에 결과적으로 컨테이너의 이미지도 작아집니다. 
> 또한, 메모리 소비가 적기 때문에 동일한 메모리로 더 많은 컨테이너를 실행할 수 있으므로 클라우드 서비스의(AWS, GCP와 같은) 비용도 절감됩니다.
>
> 일반적으로 OpenJDK 에는 GraalVM 의 Native Image와 같은 AOT 컴파일러가 내장되어 있지 않습니다. 
> GraalVM은 특별히 AOT 컴파일을 위해 설계된 Oracle Labs의 프로젝트 중 하나입니다. GraalVM은 OpenJDK와 호환되면서도 몇 가지 추가 기능을 제공하는 특별한 배포입니다.
> 
> 만약 OpenJDK를 사용하고 있고, 특히 AOT 컴파일이 필요하다면 GraalVM을 설치하고 사용하는 것이 좋습니다. 
> GraalVM은 OpenJDK 기반으로 만들어져 있기 때문에 기존의 Java 어플리케이션을 호환성 있게 실행할 수 있으며, AOT 컴파일러를 통해 성능 향상을 얻을 수 있습니다.  

### 참조사이트
> [Java에서의 AOT vs JIT 컴파일](https://shirohoo.github.io/backend/java/2022-07-16-aot-vs-jit-in-java/)  
> [[Java] Hotspot VM의 한계와 이를 극복하기 위한 GraalVM의 등장](https://mangkyu.tistory.com/301)  
> [제이크서 위키 블로그:티스토리](https://jake-seo-dev.tistory.com/636)  

---

## 마이크로벤치마킹과 통계
### 자바 성능 측정 기초
> JIT 컴파일러는 코드를 조금이라도 효율적으로 작동시키려고 호출 계층을 최적화하므로 벤치마크 성능은 갭처 타이밍에 따라 달라진다.  
> 성능 측정 시 타이밍을 캡처하기 전에 JVM 이 가동 준비를 마칠 수 있게 웜업 기간을 두는 게 좋다. 보통 타이밍 세부를 캡처하지 않는 상태로 벤치마크
> 대상 코드를 여려 번 반복 실행하는 식으로 JVM 을 예열시킨다.
>
> 또 한 가지 고려할 외부 요인은 가비지 수집이다. 타이밍 캡처 도중에 GC 가 안 일어나게 설정한 다음 가동시키면 참 좋겠지만, 가비지 수집은 원래 불확정적
> 이어서 사람 마음대로 어찌할 도리가 없다.
> 가비지 수집이 작동되는 모습은 다음 VM 플래그를 추가하면 엿볼 수 있다.
> ```shell
> java -verbose:gc <Java Program>
> ```
>
> **Tip**
> * 시스템 전체를 벤치마크한다. 저수준 수치는 수집하지 않거나 그냥 무시한다.
> * 작은 자바 코드 조각보다 자바 애플리케이션 전체를 대상으로 성능 분석을 하는 편이 거의 항상 더 수월하다.
> * 여러분이 OpenJDK 나 범용 라이브러리 개발자라면 마땅히 마이크로벤치마킹이 필요하겠지만, 일반 개발자 입장에서는 시스템 성능 요건이 정말 마이크로벤치마크를 고려해야 할 정도인지 확실치 않을 것이다.

### JMH
> JMH 는 자바를 비롯해 JVM 을 타킷으로 하는 언어로 작성된 나노/마이크로/밀리/매크로 벤치마크를 제작, 실행, 분석하는 자바 도구이다.
>
> 정확한 벤치마킹 결과를 위해서 JMH 는 블랙홀 이라는 장치를 이용해서 벤치마크할 로직을 JVM 최적화로 부터 보호한다.
>
> 벤치마크가 통제된 환경을 나타낸다고 쉽게 가정하지만 실은 전혀 그렇지 못한 경우도 많다. 통제되지 않은 변수는 대개 찾아내기가 어려워서 JMH 같은 툴을 쓰더라도
> 각별히 잘 살펴야 한다.

### 정규 분포
> 정규 분포는 종 모양의 곡선으로, 평균 주위에 대부분의 데이터가 모여 있는 형태를 나타냅니다. 이 분포는 평균과 표준 편차라는 두 매개변수에 의해 완전히 정의됩니다.
> `평균 (Mean)`: 데이터의 평균값을 나타냅니다. 정규 분포의 중심이 되는 지점이며, 곡선의 정점이기도 합니다.  
> `표준 편차 (Standard Deviation)`: 데이터가 얼마나 평균 주위에 모여있는지를 나타내는 측도입니다. 표준 편차가 작을수록 데이터가 평균 주위에 모여있는 것이고, 표준 편차가 클수록 데이터가 흩어져 있는 것입니다.  
> 
> 정규 분포의 특징은 다음과 같습니다:
> * 대부분의 데이터는 평균 주위에 모여 있습니다.
> * 평균에서 멀어질수록 데이터의 빈도가 감소합니다.
> * 평균에서 표준 편차의 배수만큼 떨어진 곳에서의 빈도가 정해진 법칙에 따라 나타납니다.

### 비정규 분포
> 비정규 분포는 정규 분포의 특징을 따르지 않는 확률 분포를 말합니다. 정규 분포와는 달리 비정규 분포는 종 모양의 대칭적인 형태가 아닐 수 있으며, 데이터의 분포가 왜곡되거나 비대칭일 수 있습니다. 
> 다양한 비정규 분포가 있지만, 몇 가지 대표적인 예시를 살펴보겠습니다.
> * 균등 분포 (Uniform Distribution): 모든 값이 동일한 확률로 나타나는 분포를 말합니다. 이는 주사위를 굴렸을 때 나오는 눈금의 분포와 비슷한 개념입니다.
> * 지수 분포 (Exponential Distribution): 주어진 시간 내에 어떤 사건이 발생할 때까지 걸리는 시간을 나타내는 분포로, 지수 함수의 형태를 가지고 있습니다. 주로 사건 간격이나 수명과 관련된 분포로 사용됩니다.
> * 파레토 분포 (Pareto Distribution): 상대적으로 적은 수의 사건이 매우 빈번하게 발생하는 분포를 나타냅니다. 상위 높은 빈도를 가진 소수의 원소들이 전체 현상의 대다수를 차지하는 경우에 많이 나타납니다.
> * 슈퍼노바 분포 (Power Law Distribution): 일부 사건이 다른 사건보다 훨씬 자주 발생하는 분포를 말합니다. 인터넷에서의 웹페이지 방문 횟수나 도시의 인구 분포 등에서 발견될 수 있습니다.
> * 긴 꼬리형 비정규 분포는 어떤 사건이 매우 드물게 발생하지만 극단적인 값이 발생할 가능성이 있는 확률 분포를 나타냅니다.   
> 이는 주로 슈퍼노바 분포나 파레토 분포와 같이 일부 사건이 다른 사건보다 훨씬 높은 빈도로 발생하는 상황에서 나타납니다.  
> 긴 꼬리형 분포는 두 가지 주요 특징을 가지고 있습니다:  
> 꼬리가 길다 (Long Tail): 분포의 꼬리 부분이 정규 분포보다 더 늘어져 있습니다. 이는 극단적인 값이 나타날 가능성이 높음을 의미합니다.  
> 절단되지 않은 분포 (Uncut Distribution): 일반적으로 분포의 꼬리 부분이 잘린 것이 아니라, 극단적인 값이 발생할 수 있는 확률이 존재합니다.  
> 긴 꼬리형 분포의 예시로는 온라인 서비스에서의 이용자 행동 데이터, 출판물의 판매량, 인터넷 검색어 사용 빈도 등이 있습니다.   
> 이러한 분포에서는 소수의 사건이 다수의 사건을 훨씬 능가하는 경우가 흔하며, 상대적으로 드문 사건이 큰 영향을 미칠 수 있습니다.  

### JVM 성능 통계
> 모든 측정은 어느 정도의 오차를 수반한다. 자바 개발자가 성능 분석 시 흔히 맞닥뜨리는 두 가지 주요 오차 유형은 다음과 같다.   
> * 랜덤 오차(Random error): 측정 오차 또는 무관계 요인이 어떤 상관관계 없이 결과에 영향을 미친다.  
> * 계통 오차(Systematic error): 원인을 알 수 없는 요인이 상관관계 있는 형태로 측정에 영향을 미친다.  
> 
> 정확도(Accuracy)는 계통 오차(Systematic error)의 수준을 나태내는 용어로, 정확도가 높으면 계통 오차가 낮은 것이다.  
> 정밀도(Precision)는 랜덤 오차를 나타내는 용어로서, 정밀도가 높으면 랜덤 오차가 낮은 것이다.  
> 
> 랜덤 오차는 원인을 알 수 없는, 또는 예기치 못한 환경상의 변화 때문에 일어난다. 기초과학 실험에서는 그런 변화가 측정 장비나 환경 자체에서 일어나지만, 소프트웨어에서는
> 측정 툴을 못 믿을 이유가 없으므로 랜덤 오차의 근원은 오직 운영 환경뿐이다.  
> 랜덤 오차는 대부분 정규 분포(가우시안 분포)를 따른다. 정규 분포는 오차가 측정값에 미치는 긍정적/부정적 영향도가 얼추 비슷한 경우에는 적합하지만 JVM 에는
> 이 모델이 잘 맞지 않는다.
> 
> `메서드 타이밍`: 메서드 실행 시간을 재는(기록하는) 행위.  

---

## 가비지 수집 기초
### 마크 앤 스위프
> 자바의 가비지 수집은 마크 앤 스위프(표시하고 쓸어 담기) 알고리즘이 기초이다.  

### 힙 공간 용어
> * 힙 공간 (Heap Space): 힙 메모리는 크게 "Young Generation"과 "Old Generation"으로 나뉩니다. 
> Young Generation은 새로 생성된 객체들이 위치하는 영역이며, Old Generation은 오래된 객체들이 유지되는 영역입니다.
> * Young Generation: 새로 생성된 객체들이 할당되는 영역으로, 대부분의 객체는 여기에 위치합니다. 이 영역에서 가비지 컬렉션은 주로 "Minor GC"라고 불립니다.
> * Old Generation (Tenured Generation): Young Generation에서 살아남은 객체들이 이동하는 영역입니다. 
> 여기에 있는 객체들은 일정 기간 동안 살아남은 객체로 간주되며, 가비지 컬렉션은 여기에서 더 오랜 기간 동안 사용되지 않는 객체들을 찾아 "Major GC" 또는 "Full GC"를 수행하여 회수합니다.
> * 에덴(Eden) 공간: 에덴 공간은 Java 가상 머신(Java Virtual Machine, JVM)의 힙 메모리 영역 중 하나로, 새로 생성된 객체가 최초로 할당되는 영역입니다.
> 주로 Young Generation이라고 불리는 힙의 부분 중 하나로, 새로 생성된 객체들은 처음에 Eden 영역에 할당됩니다. Eden 영역에서는 대다수의 객체가 생성되지만, 
> 이 중에서 오래 살아남은 객체들은 다른 영역으로 이동하게 됩니다. Young Generation은 보통 세 부분으로 나누어집니다: Eden, S0(Survivor Space 0), S1(Survivor Space 1). 
> 새로 생성된 객체는 Eden에 할당되고, 일정 기간 동안 살아남은 객체는 S0 또는 S1 중 하나의 Survivor 영역으로 이동합니다. 
> 가비지 컬렉션의 일환으로, 살아남은 객체는 다른 Survivor 영역으로 이동하며, 이 프로세스가 반복됩니다.
> 만약 객체가 계속해서 오랫동안 살아남아 Old Generation으로 이동할 수 없을 정도로 계속해서 살아남는다면, 이를 "프로모션(promotion)"이라고 부릅니다. 
> 에덴 공간은 새로 생성된 객체들이 일시적으로 머무르는 장소로, 많은 객체가 빠르게 생성되고 소멸되는 경우가 많습니다.
> * 조기 승격(Early Promotion)은 Java 가상 머신(JVM)의 가비지 컬렉션 과정 중에서 Young Generation 의 객체가 일정 기간 동안 살아남아 Old Generation 으로 이동하는 현상을 나타냅니다.
> Young Generation 은 주로 Eden 영역과 두 개의 Survivor 영역(S0, S1)으로 나뉘어져 있습니다. 
> 새로 생성된 객체는 Eden 영역에 할당되며, 일정 기간 동안 살아남은 객체는 Survivor 영역으로 이동합니다. 
> 이러한 과정을 반복하다가 살아남은 객체가 일정 횟수 이상 차면, 해당 객체는 Old Generation으로 이동합니다. 
> 이렇게 Young Generation의 일부 객체가 Old Generation으로 이동하는 것을 "조기 승격"이라고 합니다.

### 가비지 수집 용어
> * STW(Stop The World): GC 사이클이 발생하여 가비지를 수집하는 동안에는 모든 애플리케이션 스레드가 중단된다. 따라서 애플리케이션 코드는 GC 스레드가 바라보는 힙 상태를 무효화할 수 없다.
> 단순 GC 알고리즘에서는 대부분 이럴 때 STW(Stop The World) 가 일어난다.
> * 동시: GC 스레드는 애플리케이션 스레드와 동시(병행) 실행될 수 있다. 이는 계산 비용 면에서 아주 아주 어렵고 비싼 작업인 데다, 실상 100% 동시 실행을 보장하는 알고리즘은 없다.
> * 병렬: 여러 스레드를 동원해서 가비지 수집을 한다.
> * 정확: 정확한 GC 스킴은 전체 가비지를 한방에 수집할 수 있게 힙 상태에 관한 충분한 타입 정보를 지니고 있다. 대략 int 와 포인터의 차이점을 언제나 분간할 수 있는
> 속성을 지닌 스킴이 정확한 것이다.
> * 보수: 보수적인 스킴은 정확한 스킴의 정보가 없다. 그래서 리소스를 낭비하는 일이 잦고 근본적으로 타입 체계를 무시하기 때문에 훨씬 비효율적이다.
> * 이동: 이동 수집기에서 객체는 메모리를 여기저기 오갈 수 있다. 즉, 객체 주소가 고정된 게 아니다. 
> * 압착: 할당된 메모리(즉, 살아남은 객체들)는 GC 사이클 마지막에 연속된 단일 영역으로 배열되며, 객체 쓰기가 가능한 여백의 시작점을 가리키는 포인터가 있다. 
> 압착 수집기는 메모리 단편화(memory fragmentation)를 방지한다.
> * 방출: 수집 사이클 마지막에 할당된 영역을 완전히 비우고 살아남은 객체는 모두 다른 메모리 영역으로 이동(방출)한다.

### 할당과 수명
> 자바 애플리케이션에서 가비지 수집이 일어나는 주된 원인은 다음 두 가지이다.
> * 할당률
> * 객체 수명
> 
> 할당률은 일정 기간 새로 생성된 객체가 사용한 메모리량이다.(단위가 보통 MB/s) JVM 은 할당률을 직접 기록하지 않지만, 이 값은 비교적 쉽게 측정할 수 있고 
> 센섬 같은 툴을 쓰면 정확하게 구할 수 있다.
> 
> 반면, 객체 수명은 대부분 측정하기 어렵다. 사실, 수동 메모리 관리 시스템에서 가장 논란이 됐던 부분 중 하나가, 실제 애플리케이션에서 객체 수명을 제대로 파악하기가
> 너무 복잡하다는 것이다. 그 결과, 객체 수명이 할당률보다 더 핵심적인 요인이다.  
> 
> tip. 가비지 수집은 '메모리를 회수해 재사용'하는 일이다. 객체는 대부분 단명(short-lived) 하므로 가비지 수집에서 핵심 전제는 동일한 물리 메모리 조각을
> 몇 번이고 계속 다시 쓸 수 있는가 하는 점이다.
> 
> **약한 세대별 가설(Weak Generational Hypothesis)**  
> JVM 및 유사 소프트웨어 시스템에서 객체 수명은 이원적 분포 양상을 보인다. 거의 대부분의 객체는 아주 짧은 시간만 살아 있지만, 나머지 객체는 기대 수명이 훨씬 길다.
> 
> 약한 세대별 가설은 소프트웨어 시스템의 런타임 작용을 관찰한 결과 알게 된 경험 지식으로, JVM 메모리 관리의 이론적 근간을 형성한다.
> 
> 가비지를 수집하는 힙은, 단명 객체를 쉽고 빠르게 수집할 수 있게 설계해야 하며, 장수 객체와 단명 객체를 완전히 떼어놓는 게 가장 좋다.  
> 
> 핫스팟은 몇 가지 메커니즘을 응용하여 약한 세대별 가설을 십분 활용한다.  
> * 객체마다 '세대 카운트(객체가 지금까지 무사 통과한 가비지 수집 횟수'를 센다.
> * 큰 객체를 제외한 나머지 객체는 에덴(Eden) 공간에 생성한다. 여기서 살아남은 객체는 다른 곳으로 옮긴다.
> * 장수했다고 할 정도로 충분히 오래 살아남은 객체들은 별도의 메모리 영역(올드 또는 테뉴어드 세대)에 보관한다.

### 병렬 수집기
> 자바 8 이전까지 JVM 디폴트 가비지 수집기는 병렬 수집기이다. 병렬 수집기도 여러 가지가 존재한다.  
> * Parallel GC: 가장 단순한 영 세대용 병렬 수집기이다.
> * ParNew GC: CMS 수집기와 함께 사용할 수 있게 Parallel GC 를 조금 변형한 것이다.
> * ParallelOld GC: 올드(테뉴어드) 세대용 병렬 수집기이다.
>
> **한계**  
> 병렬 수집기는 세대 전체 콘텐츠를 대상으로 한번에, 가능한 한 효율적으로 가비지를 수집한다. 그런데 이런 설계 방식에도 단점이 있다.
> 우선, 풀 STW 를 유발한다.  
> 전체 힙에서 영 영역을 작게 구성한 설계는 기본적으로 대부분의 워크로드에서 영 수집에 따른 중단 시간이 매우 짧다고 가정한 것이다. 실제로, 최신 2GB 짜리 JVM
> 에서 영 수집 중단 시간은 거의 대부분 10밀리초 이하이다.  
> 그러나 올드 수집은 사정이 전혀 다르다. 올드 세대는 디폴트 크기 자체가 영 세대의 7배나 된다. 이 사실 하나만으로도 풀 수집 시 STW 시간이 영 수집보다 훨씬 더
> 길어지리라 예상할 수 있다.

### 할당의 역할
> 자바의 가비지 수집 프로세스는 보통 유입된 메모리 할당 요청을 수용하기에 메모리가 부족할 때 작동하여 필요한 만큼 메모리를 공급한다. 즉, GC 사이클은 어떤 고정된,
> 예측 가능한 일정에 맞춰 발생하는 게 아니라, 순전히 그때그때 필요에 의해 발생한다.  
> 이렇게 불확정적으로, 불규칙하게 발생한다는 점이 가비지 수집의 가장 중요한 특징이다. GC 사이클은 하나 이상의 힙 메모리 공간이 꽉 채워져 더 이상 객체를 생성할 공간이
> 없을 때 일어난다.
> 
> GC 가 발생하면 모든 애플리케이션 스레드가 멈춘다.(더 이상 객체를 생성할 수 없으니 오래 실행될 자바 코드는 사실상 없는 셈이다.). JVM 은 모든 코어를 총동원해
> 가비지를 수집하고 메모리를 회수한 후, 애플리케이션 스레드를 재개한다.  

---

## 가비지 수집 고급
### 트레이드오프와 탈착형 수집기
> 썬 마이크로시스템즈 환경 내부에서 GC 는 탈착형 서브시스템(Pluggable subsystem)으로 취급된다. 탈착형으로 수집기를 쓰는 건, GC 가 아주 일반적인 컴퓨팅
> 기법인 데다 같은 알고리즘이라도 모든 워크로드 유형에 다 적합한 건 아니기 때문이다. 모든 GC 관심사를 일제히 최적화하는, 유일무이한 범용 GC 알고리즘은 없다.  
> 
> 개발자는 가비지 수집기 선정 시 다음 항목을 충분히 고민해야 한다.
> * 중단 시간: 중단 길이 또는 기간
> * 처리율: 애플리케이션 런타임 대비 GC 시간 %
> * 중단 빈도: 수집기 때문에 애플리케이션이 얼마나 자주 멈추는가?
> * 회수 효울: GC 사이클 당 얼마나 많은 가비지가 수집되는가?
> * 중단 일관성: 중단 시간이 고른 편이가?
> 
> 이 중에서 단연 최고의 관심사는 중단 시간이지만, 대부분 애플리케이션에서 이것 하나만 따로 떨어뜨려 놓고 생각해선 안된다.
> 예를 들어, 고도 병렬 배치 시스템이나 빅 데이터 애플리케이션에서는 중단 기간보다 처리율이 더 큰 영향을 미친다. 평범한 배치 잡을 실행하면서 수십 초간
> 중단이 발생했다고 큰일 나는 건 아니므로 무조건 CPU 효율 및 처리율이 우수한 GC 알고리즘이 중단 시간이 짧은 알고리즘보다 우선이다.  

### CMS 
> CMS 수집기는 중단 시간을 아주 짧게 하려고 설계된, 테뉴어드(올드) 공간 전용 수집기이다. 보통 영 세대 수집용 병렬 수집기(Parallel GC)를 조금 변형한 수집기와 함께 쓴다.  
> 전체적으로 한 차례 긴 STW 중단을 일반적으로 매우 짧은 두 차례 STW 중단으로 대체해서 중단 시간을 줄였다.  
> 대부분 워크로드에 CMS 를 적용하면 다음과 같은 효험이 있다. 
> * 애플리케이션 스레드가 오랫동안 멈추지 않는다.
> * 단일 풀 GC 사이클 시간(벽시계 시간)이 더 길다.
> * CMS GC 사이클이 실행되는 동안, 애플리케이션 처리율은 감소한다.
> * GC 가 객체를 추적해야 하므로 메모리를 더 많이 쓴다.
> * GC 수행에 훨씬 더 많은 CPU 시간이 필요하다.
> * CMS 는 힙을 압착하지 않으므로 테뉴어드 영역은 단편화될 수 있다: 즉, 메모리의 일부는 사용 중이지만 빈틈없이 연속된 상태가 아닌 상태로 남게 되어, 효율적인 메모리 사용을 제한할 수 있다.
>
> 영 수집이 CMS 보다 더 빠르거나 힙 단편화가 발생하면 풀 STW ParallelOld GC 수집으로 회귀할 수 밖에 없는데, 애플리케이션 입장에서는 중대한 사건이다.
> CMS 를 적용한 저지연 애플리케이션에서는 사실상 튜닝 자체가 주요 이슈이다.  
> 
> **CMS 기본 JVM 플래그**  
> ```shell
> -XX:+UseConcMarkSweepGC
> ```
> 최신 핫스팟 버전에서 이 플래그를 쓰면 ParNew GC(병렬 영 수집기를 조금 변형한 수집기)도 함께 작동한다.  
> CMS 는 일반적으로 플래그 가지수가 엄청나게 많다. 하지만 민간 튜닝 안티패던에 빠질 위험이 있으니 다양한 옵션을 적용해보는 것은 삼가할 일이다.  

### G1
> Garbage First(가비지 우선)은 병렬 수집기, CMS 와는 전혀 스타일이 다른 수집이다. 처음부터 중단 시간이 짧은 새로운 수집기로 설계된 G1 은 다음과 같은 특성이 있다.
> * CMS 보다 훨씬 튜닝하기 쉽다.
> * 조기 승격에 덜 취약하다.
> * 대용량 힙에서 확장성(특히, 중단 시간)이 우수하다.
> * 풀 STW 수집을 없앨 수 있다.(또는 풀 STW 수집으로 되돌아갈 일을 확 줄일 수 있다.)
> 
> 시간이 지나면서 G1 은 대용량 힙에서 중단 시간이 짧은 범용 수집기로 점점 더 굳혀졌다.
> 
> G1 힙은 영역(region)으로 구성된다. 영역은 디폴트 크기가 1MB(힙이 클수록 크기가 커짐)인 메모리 공간이다. 영역을 이용하면 세대를 불연속적으로 배치할 수 있고,
> 수집기가 매번 실행될 때마다 전체 가비지를 수집할 필요가 없다.
> 
> **G1 기본 JVM 플래그**  
> 자바 8 이전까지는 다음 스위치로 G1 을 작동시킨다.
> ```shell
> +XX:UseG1GC
> ```
> 
> 다음은 디폴트 중단 시간 목표를 200밀리초로 설정하는 스위치이다.
> ```shell
> -XX:MaxGCPauseMillis=200
> ```
> 실제로 중단 시간을 100 밀리초 이하로 설정하면 현실성이 너무 떨어져 수집기가 지키지 못할 공산이 크다. 디폴트 영역 크기값을 변경하는 방법도 있다.
> ```shell
> -XX:G1HeapRegionSize=<n>
> ```
> <n>은 1부터 64까지의 2의 제곱수이다(단위: MB). 
> 
> 전반적으로 G1 은 안정된 알고리즘으로서 오라클의 전폭적인 지원을 받고 있다. 

### 엡십론
> 엡실론(Epsilon collector)는 레거시 수집기는 아니지만, 어느 운영계 환경에서건 절대 사용 금물인 수집기다.  
> 엡실론은 테스트 전용으로 설계된, 아무 일도 안 하는 시험 수집기이다. 실제로 가비지 수집 활동을 일체 하지 않는다. 다음과 같은 작업에 유용하다.
> * 테스트 및 마이크로벤치마크 수행
> * 회귀 테스트
> * 할당률이 낮거나 0인 자바 애플리케이션 또는 라이브러리 코드의 테스트

---

## GC 로깅, 모니터링, 튜닝, 툴
### GC 로깅
> GC 로깅은 시스템이 내려간 원인의 단서를 찾는 콜드 케이스(Cold case: 원일을 알 수 없는 현상) 분석을 할 때 매우 유용하다.  
> 모든 중요한 애플리케이션에서는 다음 두 가지를 설정해야 한다.
> * GC 로그를 생성한다.
> * 애플리케이션 출력과는 별도로 특정 파일에 GC 로그를 보관한다.
> 
> GC 로깅은 사실 오버헤드가 거의 없는 것이나 다음없으니 주요 JVM 프로세스는 항상 로깅을 켜놓아야 한다.
> 
> GC 로깅 플래그
> ```shell
> -Xloggc:gc.log -XX:+PrintGCDetails -XX:+PrintTenuringDistribution -XX:+PrintGCTimeStamps -XX+PrintGCDateStamps
> ```
> * -Xloggc:gc.log: GC 이벤트에 로깅할 파일을 지정한다.
> * -XX:+PrintGCDetails: GC 이벤트 세부 정보를 로깅한다. 
> * -XX:+PrintTenuringDistribution: 툴링에 꼭 필요한, 부가적인 GC 이벤트 세부 정보를 추가한다.
> * -XX:+PrintGCTimeStamps: GC 이벤트 발생 시간을 (VM 시작 이후 경과한 시간을 초 단위로) 출력한다.
> * -XX+PrintGCDateStamps: GC 이벤트 발생 시간을(벽시계 시간 기준으로) 출력한다.
> 
> GC 로그 순환 플래그
> ```shell
> -XX:+UseGCLogFileRotation -XX:+NumberOfGCLogFiles=<n> -XX:+GCLogFileSize=<size>
> ```
> * -XX:+UseGCLogFileRotation: 로그 순환 기능을 켠다.
> * -XX:+NumberOfGCLogFiles=<n>: 보관 가능한 최대 로그파일 개수를 설정한다.
> * -XX:+GCLogFileSize=<size>: 순환 직전 각 파일의 최대 크기를 설정한다.

### GC 기본 튜닝
> GC 힙 크기 조정 플래그
> ```shell
> -Xms<size> -Xmx<size> -XX:MaxPermSize=<size> -XX:MaxMetaspaceSize=<size>
> ```
> * -Xms<size>: 힙 메모리의 최소 크기를 설정한다.
> * -Xmx<size>: 힙 메모리의 최대 크기를 설정한다.
> * -XX:MaxPermSize=<size>: 펌젠 메모리의 최대 크기를 설정한다.(자바 7 이전)
> * -XX:MaxMetaspaceSize=<size>: 메타스페이스 메모리의 최대 크기를 설정한다. (자바 8 이후)
> 
> MaxPermSize 는 자바 7 이전에만 적용되는 레거시 플래그이다. 자바 8부터는 펌젠이 사라지고 메타스페이스로 교체됐다.  
> 자바 8 애플리케이션에 MaxPermSize 설정값이 있으면 지우는 것을 권장한다. JVM 이 그냥 무시하기 때문에 애플리케이션에 아무 상관이 없다.  
> 
> 튜닝 시 GC 플래그는 다음과 같이 추가한다.
> * 한번에 한 플래그씩 추가한다.
> * 각 플래그가 무슨 작용을 하는지 숙지해야 한다.
> * 부수 효과를 일으키는 플래그 조합도 있음을 명심한다.
> 
> 현재 이벤트가 발생 중이라면, 성능 문제를 일으키는 원인이 GC 인지 아닌지 판단하는 건 어렵지 않다. 먼저, `vmstat` 같은 툴로 고수준의 머신 지표를
> 체크하고 성능이 떨어진 시스템에 로그인해서 다음을 확인한다.
> * CPU 사용률이 100%에 가까운가?
> * 대부분의 시간(90% 이상)이 유저 공간에서 소비되는가?
> * GC 로그가 쌓이고 있다면 현재 GC 가 실행 중이라는 증거다.
> 
> GC 가 성능 문제의 출처라고 밝힌 다음에는 할당과 중단 시간 양상을 파악한 다음, GC 를 튜닝하고 필요 시 메모리 프로파일러를 활용한다.

### 할당률 분석
> 할당률 분석은 튜닝 방법뿐만 아니라, 실제로 가비지 수집기를 튜닝하면 성능이 개선될지 여부를 판단하는데 꼭 필요한 과정이다.
> 할당률 수치가 1GB/s 이상으로 일정 시간 지속한다면 십중팔구 가비지 수집기 튜닝만으로는 해결할 수 없는 성능 문제가 터진 것이다.  
> 이 경우 성능을 향상시키려면 애플리케이션 핵심부의 할당 로직을 제거하는 리팩터링을 수행하여 메모리 효율을 개선하는 방법 밖에 없다.

### Parallel GC 튜닝
> Parallel GC 는 가장 단순한 수집기라서 튜닝 역시 제일 쉽다. 사실, 최소한의 튜닝만으로도 충분하다. 이 수집기의 목표와 트레이드오프는 뚜렷하다.
> * 풀 STW
> * GC 처리율이 높고 계산 비용이 싸다.(가비지 컬렉션(Garbage Collection)이 효율적으로 이루어져서 애플리케이션의 일시 중단 시간이 짧고, GC 작업이 빠르게 수행된다는 것을 의미)
> * 부분 수집이 일어날 가능성은 없다.(풀 가비지 컬렉션만 일어난다.)
> * 중단 시간은 힙 크기에 비례하여 늘어난다.
> 
> 이와 같은 특성들이 별문제가 안 되는 애플리케이션에서는 (특히, 힙이 4기가바이트 이하로 작을 경우) Parallel GC 가 아주 효과적인 선택이다.
> 대부분의 최신 애플리케이션은 사람보다 프로그램이 크기를 알아서 잘 결정하기 때문에 아래의 설정값들을 명시적으로 설정하는 일은 삼가는 게 좋다.
> ```shell
> -XX:NewRatio
> -XX:SurvivorRatio
> -XX:NeewSize
> -XX:MaxNewSize
> -XX:MinHeapFreeRatio
> -XX:MaxHeapFreeRatio
> ```

### CMS 튜닝
> CMS 는 튜닝이 까다롭기로 소문난 수집기이다. CMS 로 최상의 성능을 얻는 과정에는 여러 가지 복잡성과 트레이드오프가 있다.
> CMS 처럼 중단 시간이 짧은 수집기는 정말로 STW 중단 시간을 단축시켜야 하는 유스케이스에 한해 어쩔 수 없을 때만 사용해야 한다.
> 안 그러면 딱히 이렇다 할 애플리케이션 성능 향상도 못 본 채, 팀원 모두 튜닝하기 힘든 수집기를 붙들고 고생하게 된다.  
> 
> CMS 수집이 일어나면 기본적으로 코어 절반은 GC 에 할당되므로 애플리케이션 처리율은 그 만큼 반토막 난다. 
> 성능 엔지니어는 튜닝할 때 이런 상황이 발생해도 괜찮은지 일단 고민해보고, 괜찮지 안핟면 호스트에 코어 수를 늘리는 해결 방안을 모색해야 한다.  
> CMS 수집 중 GC 에 할당된 코어 수를 줄이는 방법도 있다. 물론 그만큼 수집 수행 CPU 시간이 줄어들고 부하 급증 시 애플리케이션의 회복력이 떨어지는 위험은 감수해야 한다.
> 다음 스위치로 조절한다.
> ```shell
> -XX:ConcGCThreads=<n>
> ```

### G1 튜닝
> 엔드 유저가 최대 힙 크기와 최대 GC 중단 시간을 간단히 설정하면 나머지는 수집기가 알아서 처리하게 하는 것이 G1 튜닝의 최종 목표이다. 

---
