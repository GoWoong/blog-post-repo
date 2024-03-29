# [AWS] AWS Elastic Beanstalk를 이용해 FastAPI 서버 실행하기 1탄

안녕하세요 고웅입니다. 오늘 부터 시리즈로 구성한 AWS Elastic Beansk를 이용해 FastAPI 서버 실행하기를 진행하려고 합니다. 이번 시리즈를 통해 제가 회사에서 초기 스타트업에서 어떤 서비스를 사용해서 인프라 구성을 했고 왜 그런 선택을 했는지 작성해보려고 합니다.

# 목차

- [Elastic Beanstalk란?](#elastic-beanstalk란)
- [왜 Beanstalk이었나?](#왜-beanstalk-이었나)
  - [개선해야 했던 사안](#개선해야-했던-사안)
  - [Beanstalk로 어떻게 개선할 것인가?](#beanstalk로-어떻게-개선할-것인가)
  - [그래서 Beanstalk로 어떻게 변화했는가?](#그래서-beanstalk로-어떻게-변화했는가)
- [장단점](#장단점)
  - [장점](#장점)
  - [단점](#단점)
- [그럼에도 Beanstalk를 선택한 이유](#그럼에도-beanstalk를-선택한-이유)
- [Reference](#reference)

---

# [Elastic Beanstalk](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/Welcome.html)란?

AWS는 100개 이상의 서비스를 제공하고 있습니다. 즉 사용자는 다양한 서비스를 이용하여 AWS 인프라 관리 방법의 유연성을 가질 수 있습니다. 하지만 이런 다양한 서비스로 인하여 사용자는 어떤 서비스를 사용해야 하며 이러한 서비스를 프로비저닝하는 방법을 파악하는 것이 어려울 수 있습니다.

Elastic Beanstalk를 사용하면 애플리케이션을 실행하는 인프라에 대해 자세히 알지 못해도 AWS에서 애플리케이션을 신속하게 배포하고 관리할 수 있습니다. 애플리케이션을 업로드하기만 하면 Elastic Beanstalk에서 용량 프로비저닝, 로드 밸런싱, 조정, 애플리케이션 상태 모니터링에 대한 세부 정보를 자동으로 처리합니다.

Elastic Beanstalk는 Go, Java, .NET, Node.js, PHP, Python 및 Ruby에서 개발된 애플리케이션을 지원합니다.

# 왜 Beanstalk 이었나?

제가 현재 다니고 있는 회사에서는 제품 출시와 함께 앱 서비스를 사용자들에게 제공하기 위해 개발을 진행하고 있었습니다. 회사는 의료기기 회사이기 때문에 기기가 허가가 나지 않는 이상 판매가 불가능 합니다. 즉 허가가 나기 전에는 제가 만든 서비스도 출시가 되지 않는 상황이기에 AWS의 프리티어 서비스를 넘기지 않는 정도의 개발용 서버만 가지고 있었습니다.

<center><img src="https://github.com/GoWoong/blog-post-repo/blob/main/image/Serise-Elastic-Beanstalk/%EC%B4%88%EA%B8%B0%EC%84%9C%EB%B2%84%EA%B5%AC%EC%84%B1.png?raw=true" width="80%" height="80%"/></center>
<center><p>초기서버구성</p></center>

이런 상황에서 24년 제품 출시 계획이 구체화 되기 시작했고 그로 인한 상용 서비스 준비가 필요하게 되었습니다.

## 개선해야 했던 사안

먼저 현재의 아키텍처를 살펴보고 무엇을 개선해 나가야 하는지 확인했어야 했습니다.

1. 가용성
2. 인스턴스 보안
3. 자동 배포

크게 본다면 3가지의 개선 사안이 존재했습니다. 상용 서비스를 기획하기 전 까지는 AWS에서 기본적인 VPC 즉 퍼블릭 서브넷을 이용하며 DB와 EC2까지 모두 퍼블릭 서브넷에 위치하고 그나마 로드밸런서와 Route 53을 이용해 DNS를 제공하는 정도가 서버 인프라 구성이 었습니다. 마치 기초적인 사이드 프로젝트하는 것 같은 서버 인프라를 가지고 있었습니다.

## Beanstalk로 어떻게 개선할 것인가?

### 가용성

가용성이란 서버와 네트워크, 프로그램 등의 정보 시스템이 정상적으로 사용 가능한 정도를 말한다라고 Wikipedia에서 소개합니다.
즉 우리 서비스가 전체 사용시간 중 정상적인 사용시간이 높아야 한다는 것인데 이는 장애가 있어도 사용자는 정상적인 사용을 할 수 있어야 한다는 것과 같습니다.

그러기 위해서는 장애가 발생하여 사용자의 사용에 문제를 일으키지 않는 것과 장애가 있어도 사용자가 모르게 하는 것이 있다고 생각했습니다. 제가 AWS 자격증을 공부하면서 접했던 가용성에 대한 사례 중에는 단일 AZ를 사용하는 것이 아닌 멀티 AZ를 사용하라는 안내가 있었습니다. 물론 DR을 구성하는 것도 있지만 이 경우는 오버 스펙이라 제외 했습니다.

또한 Auto Scailing Group을 설정해 지정한 모니터링 수치에 도달하면 EC2인스턴스를 스케일 아웃하여 가용성을 증가 시킬 수 있습니다.

### 인스턴스 보안

Public 서브넷을 사용했던 환경에서 벗어나기 위해서는 당연히 Private 서브넷을 사용해야 합니다. Private 서브넷에 인스턴스를 배치해서 외부의 접속을 막아야 하는게 인스턴스 보안의 시작이라 생각했습니다. 물론 더 깊게 들어가면 부족한 부분이 있겠지만 일단 넘어가겠습니다.

### 자동 배포

그 동안 서버에 변경사항을 적용하기 위해서는 SSH로 연결해서 직접 가상환경을 실행하고 코드를 실행하는 과정을 반복했어야 합니다. 중간에 실수하면 서버를 다시 적용해야 하는 일도 허다했습니다. 개발만 하기에도 시간이 부족한데 배포하느라 소비되는 시간도 만만치 않았습니다.

## 그래서 Beanstalk로 어떻게 변화했는가?

Beanstalk를 통해서 고가용성 설정을 진행했습니다. 먼저 VPC 콘솔에서 VPC 마법사를 이용해 손 쉽게 퍼블릭, 프라이빗 서브넷을 적용하고 NAT GateWay를 구성하였습니다. 이 후 Beanstalk 애플리케이션을 만들고 환경을 만들었습니다.

환경이란 애플리케이션 버전을 실행 중인 AWS 리소스 모음입니다. 이 환경을 생성하는 과정을 통해 EC2 환경티어나 VPC 설정, 서브넷 설정, 로드밸런서 설정, DB 설정, 배포 설정 등 다양한 설정을 진행하여 실행가능한 환경을 만들 수 있습니다.

설정을 완료하고 생성을 하면 Beanstalk에서 CloudFormation 스텍을 자동으로 만들어 주고 배포를 진행하게 됩니다.

배포에는 시간이 걸리긴 하지만 성공표시를 확인하고 배포된 주소로 접속해보면서 배포가 제대로 되었는지 확인 할 수 있습니다.

추가로 eb cli를 설치하여 터미널 환경에서 Beanstalk를 배포할 수 있습니다. 저는 이 eb cli를 이용해 로컬 개발 환경에서 Beanstalk 배포 명령을 내리고 자동으로 배포를 진행 할 수 있었습니다.

# 장단점

물론 Beanstlk의 장점도 있지만 단점도 당연히 존재합니다.

## 장점

### 1. 사용이 간편합니다.

단지 코드를 제공하는 것만으로도 쉽게 새로운 버전의 코드를 배포할 수 있습니다. 제가 다니는 회사에는 제대로 된 팀이 없습니다. 스타트업의 고질적인 문제인 인력난입니다. 즉 DevOps 전문가가 있다는 것은 꿈만 같은 얘기입니다. 결국 제가 백엔드 코드를 작성하면서 인프라 문제도 해결해야 한다는 것인데 많은 경험이 없다보니 쉬운일이 아니었습니다. 그렇기 때문에 인프라를 자동으로 생성해주고 관리할 수 있는 것은 큰 매리트였습니다.

### 2. 무중단 배포와 롤백이 쉽습니다.

beanstalk를 이용하면 무중단 배포가 가능합니다. 저는 Beanstalk 환경을 구성하며 배포시 추가적인 EC2를 생성하고 코드를 배포하고 상태 확인 결과 정상이면 새로운 환경으로 트래픽을 전환하도록 설정을 했습니다. 그리고 문제가 있다면 제가 배포했던 버전 이전 버전을 복구하면 다시 롤백이 가능했습니다.

### 3. Beanstalk 자체는 비용이 없습니다.

이런 많은 작업을 대신해 주는데 Beanstalk 자체적으로는 비용이 부과되지 않습니다. 대신 Beanstalk가 자동으로 생성해준 서비스들에 대한 사용 비용만 내면 됩니다. 이 것은 직접 수동으로 같은 서비스를 만든 것과 같은 비용이 든다는 것 입니다.

## 단점

### 1. 배포 시간이 걸립니다.

제가 다른 CD툴을 이용한 자동배포를 경험하지는 못 해봤지만 Beanstalk의 단점으로 배포적용되는 속도가 느리다는 의견이 많았습니다. CloudFromation을 이용해 서비스들이 생성되고 실행되는 속도가 느리다는 점이 있었습니다.

### 2. 모니터링과 디버깅이 힘듭니다.

제가 직접해보니 환경을 설정하고 실행하는데 있어서 502 에러가 발생하거나 생성에 실패하는 사례가 있었는데 이 때 로그를 확인하기 힘들었습니다. 이미 환경이 배포 된 상태에서 새로운 환경이 배포되다 문제가 있으면 그래도 배포 실행이 실패한 인스턴스에 남아있던 로그를 확인 할 수 있었는데 특히 처음 환경을 구성하여 실행하던 중에 실패하면 로그를 확인하고 싶어도 확인할 수 없는 상황이 많아서 환경을 처음 부터 다시 구성하는 경우를 겪기도 했습니다.

### 3. 하드 제한이 있다고 합니다.

많은 환경변수가 필요한 경우가 있는데 Beanstalk는 키-값을 저장하는데 4KB의 하드 제한이 있다고 합니다.

### 4. 복잡한 아키텍처의 구성, 확장이 어렵다.

만약 요구되는 아키텍처가 복잡한 경우 Beanstalk를 이용해 구성하기 힘들다는 것이었습니다. Beanstalk에서는 .ebextension 이라는 폴더안에 config 파일을 생성해야 하거나 Nginx 설정등을 위해 추가적인 파일을 만들어야한다는데 이런 설정을 하는 것이 어려웠습니다.

# 그럼에도 Beanstalk를 선택한 이유

제가 다니는 회사는 말 그대로 스타트업입니다. 개발자라고는 백엔드 개발자 1명(본인), 안드로이드 개발자 1명, Ios 개발자 1명이 전부입니다. 제대로된 시니어 개발자또한 없이 개발을 하고 있습니다.

거기다 의료기기 회사인데 의료기기 판매 허가를 위해 많은 시간이 걸리다보니 회사 입장에서는 하나의 기기가 망하더라도 살기 위해서 수 많은 프로젝트를 진행하고 판매 허가를 위해 준비하고 있습니다. 즉 아직 판매될지 모르는데 개발해야 하는 앱들이 상당하고 기기마다 특성이 다르고 필요한 컨텐츠나 기능이 다르다는 것 입니다.

한마디로 개발하기도 바쁘다는 것입니다. 거기에 앱 기획이나 설계 의료기기이기 때문에 사이버 보안 문서도 작성합니다.

사실 여유 시간이 많다면 수동을 설정하고 CI/CD 도 제대로 갖추어서 개발하는 것이 더 좋은 경험이 될 수 있습니다. 하지만 일단 급하니 Beanstalk를 사용하기로 결정했습니다.

바람이지만 서비스가 잘 되어서 트래픽 때문에 더 진화한 아키텍처를 구성하고 문제를 해결하는 경험이 더 갚지게 느껴질 것 같습니다.

---

첫 편으로는 말로 장황하게 설명했는데 다음 화 부터는 간단한? 실습으로 돌아오도록 하겠습니다. 감사합니다!!

---

# Reference

- [Elastic Beanstalk](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/Welcome.html "AWS Elastic Beanstalk
개발자 안내서")
- [[소개] AWS Elastic Beanstalk란?](https://tech.cloud.nongshim.co.kr/2021/11/01/%EC%86%8C%EA%B0%9C-aws-elastic-beanstalk%EB%9E%80/ "NDS 클라우드 기술 블로그")
- [AWS Elastic Beanstalk으로 배포하기](https://velog.io/@bcl0206/AWS-Elastic-Beanstalk-%EB%B0%B0%ED%8F%AC-%EB%B0%A9%EB%B2%95-DB-%EC%97%B0%EA%B2%B0 "HHB.log")
