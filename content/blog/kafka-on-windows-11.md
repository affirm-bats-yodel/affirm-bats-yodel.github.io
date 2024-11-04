+++
title = "Windows 11 환경에서 Apache Kafka 설치하기 (WMIC 에러 해결)"
date = "2024-11-04T22:34:37Z"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."
description = "Windows 11 환경에서 Apache Kafka 돌려보는 가이드, WMIC 문제 해결 방법 포함."

tags = ["kafka", "apache", "windows", "11", "카프카", "아파치", "윈도우"]
+++

최근에 Stream Processing 을 연습해볼 겸 [Apache Kafka](https://kafka.apache.org/)
를 노트북에 설치하여 사용을 시도해보게 되면서, 노트북 운영체제는 Windows 11 을
사용하고 있는데 해당 환경에서 설정 방법이 제대로 작성되어 있지 않은 것 같아 가이드
를 작성해 본다.

Windows 환경은 `24H2` 환경임.

요약: `wmic` 문제라, 본 글의 [1.2. WMIC 설치](#12-wmic-설치) 참고하여 설치할 것.

## 1. 설치하기 전 해야 할 것

### 1.1. Java 설치

Kafka 는 Java 환경에서 돌아가기 때문에, Java Runtime (혹은 Java Development Kit) 을 필요로 함. 
각자 필요한 용도에 맞게 Java Runtime (혹은 Java Development Kit) 을 윈도우 환경에 설치할 것.

필자 같은 경우에는, [Chocolatey](https://chocolatey.org/) 를 사용하여 Temurin JDK
21 버전을 설치하였음. (패키지 정보: https://community.chocolatey.org/packages/Temurin)

```powershell
choco install temurin
```

이후 `java.exe` 를 호출하여 Java 가 정상적으로 설치되었는지 확인할 것.

```powershell
java.exe --version
```

```
openjdk 21.0.4 2024-07-16 LTS
OpenJDK Runtime Environment Temurin-21.0.4+7 (build 21.0.4+7-LTS)
OpenJDK 64-Bit Server VM Temurin-21.0.4+7 (build 21.0.4+7-LTS, mixed mode, sharing)
```

### 1.2. WMIC 설치

> 본 글을 작성한 이유가 이 부분 때문이다.
>
> 아무도 해당 부분에 대해서 설명하지 않고 있다.

Windows 11 환경에서 WMIC 에 대한 기능을 몇 버전부터 Deprecated 하였는지는 모르겠지만
WMIC 기능이 꺼져있다면, `kafka-server-start.bat` 파일을 실행할 때 WMIC 가 없다는
에러와 함께 실행이 되지 않는다.

WMIC 을 설치하는 방법은 Microsoft Tech Community 의
[How to install WMIC Feature on Demand on Windows 11](https://techcommunity.microsoft.com/t5/windows-it-pro-blog/how-to-install-wmic-feature-on-demand-on-windows-11/ba-p/4189530)
글을 참고하였으며, 간략한 설치 단계는 다음과 같음.

1. Windows 검색 창에서 "선택적 기능" 을 검색하여 클릭.
2. "선택적 기능 추가" 항목에서 "기능 보기" 를 클릭.
3. "사용 가능한 선택적 기능 찾기" 검색 창에 `WMIC` 을 입력 후 WMIC
를 선택하고 "다음" 버튼 클릭.
4. WMIC 가 설치될 때 까지 기다리기.

위의 원문 글에 사진과 함께 잘 설명되어 있으니, 해당 사진을 참고하여
WMIC 을 설치하는 것을 권장한다.

이후 설치가 완료되었다면, `wmic os` 를 윈도우의 터미널 창에서 입력하여
이것저것 오류 없이 출력되는 것을 확인할 수 있다.

```powershell
wmic os
```

## 2. Kafka 설치

### 2.1. Kafka 다운로드

[Kafka Quickstart](https://kafka.apache.org/quickstart) 으로 진입하여, "STEP 1: GET KAFKA"
항목의 "Download" 링크를 클릭하여 다운로드 주소로 이동한 후, `https://dlcdn...` 으로 시작하는
링크를 클릭하여 `tgz` 파일을 다운로드

### 2.2. Kafka Tgz 파일 압축 해제

압축을 아무 곳이나 풀면 `*.bat` 파일을 실행하였을 때, FilePath 의 길이 관련 문제로 인해서
정상적으로 실행되지 않는 문제가 있어서, 최대한 `C:\` 에 가깝게 압축을 푼다.

Kafka 디렉토리로 이동하여 `.\bin\windows\kafka-storage.bat` 을 아래와 같이
실행하였을 때, Usage Help 가 나오는 것을 확인할 것.

```powershell
.\bin\windows\kafka-storage.bat --help
```

```
usage: kafka-storage [-h] {info,format,random-uuid} ...

The Kafka storage tool.

positional arguments:
  {info,format,random-uuid}
    info                 Get information about  the  Kafka  log directories
                         on this node.
    format               Format the Kafka log directories on this node.
    random-uuid          Print a random UUID.

optional arguments:
  -h, --help             show this help message and exit
```

## 3. Kafka 실행

* https://kafka.apache.org/quickstart 항목을 윈도우에 맞게 명령어를 수정함.

### 3.1. KAFKA_CLUSTER_ID 생성

Kafka 가 Zookeeper 을 버리고, KRaft 으로 이동하게 되면서, 각 클러스터는 고유한
UUID 를 갖고 있어야 함.

아래의 명령어를 통해서 `KAFKA_CLUSTER_ID` 변수를 생성할 것.

```powershell
$KAFKA_CLUSTER_ID = .\bin\windows\kafka-storage.bat random-uuid
```

이후 `echo` 를 통해서 `$KAFKA_CLUSTER_ID` 의 UUID 가 정상적으로 생성되었는지
확인할 것. (ID 는 랜덤하게 생성되는 ID 이기 때문에 아래와 비슷하게만 출려되면
된다.)

```powershell
echo $KAFKA_CLUSTER_ID
```

```
vattk4pUQlW9OEQ_xT6Peg
```

### 3.2. Log 디렉토리 초기화

아래의 명령어를 통하여 이전 단계에서 생성한 `$KAFKA_CLUSTER_ID` 을 갖고
Kafka Log 디렉토리를 초기화한다.

```powershell
.\bin\windows\kafka-storage.bat format -t $KAFKA_CLUSTER_ID -c .\config\kraft\server.properties
```

```
Formatting /tmp/kraft-combined-logs with metadata.version 3.8-IV0.
```

설정을 수정하지 않았다면, `C:\tmp` 디렉토리에 `kafka-combined-logs` 디렉토리로 
초기화가 되어 있을 것이다. `ls` 를 통하여 확인해보면 다음과 같이 출력될 것이다.

```powershell
ls C:\tmp\
```

```

    디렉터리: C:\tmp


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----      2024-11-05   오전 8:07                kraft-combined-logs

```

### 3.3. Kafka 서버 시작

아래의 명령어를 통하여 서버를 시작한다.

```powershell
.\bin\windows\kafka-server-start.bat .\config\kraft\server.properties
```

실행을 하게 되면 아래와 같이 어쩌고 저쩌고 로그가 뜨면서 실행이 되는 것을
확인할 수 있다.

```
[2024-11-05 08:11:35,858] INFO Registered kafka:type=kafka.Log4jController MBean (kafka.utils.Log4jControllerRegistration$)
[2024-11-05 08:11:35,985] INFO Setting -D jdk.tls.rejectClientInitiatedRenegotiation=true to disable client-initiated TLS renegotiation (org.apache.zookeeper.common.X509Util)
...
[2024-11-05 08:11:37,037] INFO [BrokerServer id=1] Waiting for the broker to be unfenced (kafka.server.BrokerServer)
[2024-11-05 08:11:37,081] INFO [BrokerLifecycleManager id=1] The broker has been unfenced. Transitioning from RECOVERY to RUNNING. (kafka.server.BrokerLifecycleManager)
[2024-11-05 08:11:37,081] INFO [BrokerServer id=1] Finished waiting for the broker to be unfenced (kafka.server.BrokerServer)
[2024-11-05 08:11:37,081] INFO authorizerStart completed for endpoint PLAINTEXT. Endpoint is now READY. (org.apache.kafka.server.network.EndpointReadyFutures)
[2024-11-05 08:11:37,081] INFO [SocketServer listenerType=BROKER, nodeId=1] Enabling request processing. (kafka.network.SocketServer)
[2024-11-05 08:11:37,081] INFO Awaiting socket connections on 0.0.0.0:9092. (kafka.network.DataPlaneAcceptor)
[2024-11-05 08:11:37,081] INFO [BrokerServer id=1] Waiting for all of the authorizer futures to be completed (kafka.server.BrokerServer)
[2024-11-05 08:11:37,081] INFO [BrokerServer id=1] Finished waiting for all of the authorizer futures to be completed (kafka.server.BrokerServer)
[2024-11-05 08:11:37,081] INFO [BrokerServer id=1] Waiting for all of the SocketServer Acceptors to be started (kafka.server.BrokerServer)
[2024-11-05 08:11:37,081] INFO [BrokerServer id=1] Finished waiting for all of the SocketServer Acceptors to be started (kafka.server.BrokerServer)
[2024-11-05 08:11:37,081] INFO [BrokerServer id=1] Transition from STARTING to STARTED (kafka.server.BrokerServer)
[2024-11-05 08:11:37,081] INFO Kafka version: 3.8.1 (org.apache.kafka.common.utils.AppInfoParser)
[2024-11-05 08:11:37,081] INFO Kafka commitId: 70d6ff42debf7e17 (org.apache.kafka.common.utils.AppInfoParser)
[2024-11-05 08:11:37,081] INFO Kafka startTimeMs: 1730761897081 (org.apache.kafka.common.utils.AppInfoParser)
[2024-11-05 08:11:37,081] INFO [KafkaRaftServer nodeId=1] Kafka Server started (kafka.server.KafkaRaftServer)
```

Kafka 서버는 `0.0.0.0:9092` 으로 바인딩되어 동작하고 있고, 아래와 같은 명령어를 통하여
서버에 대한 정보를 확인할 수 있다.

```powershell
.\bin\windows\kafka-broker-api-versions.bat --bootstrap-server localhost:9092
```

실행하면 아래와 같이 서버 정보에 대하여 어쩌고 저쩌고 값이 나오는 것을 확인할 수 있다.

```
localhost:9092 (id: 1 rack: null) -> (
        Produce(0): 0 to 11 [usable: 11],
        Fetch(1): 0 to 16 [usable: 16],
        ListOffsets(2): 0 to 8 [usable: 8],
        Metadata(3): 0 to 12 [usable: 12],
        LeaderAndIsr(4): UNSUPPORTED,
        StopReplica(5): UNSUPPORTED,
        UpdateMetadata(6): UNSUPPORTED,
        ControlledShutdown(7): UNSUPPORTED,
        OffsetCommit(8): 0 to 9 [usable: 9],
        OffsetFetch(9): 0 to 9 [usable: 9],
        ...
```

끝.
