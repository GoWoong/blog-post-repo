# [AWS] AWS Elastic Beanstalk를 이용해 FastAPI 서버 실행하기 2탄

안녕하세요 고웅입니다. 지난 번에 Beanstalk를 이용하게 된 배경에 대해 소개했는데요 이번 편 부터는 어떻게 Beanstalk를 이용한 운영 서버를 구성했는지를 차례 차례 소개해 보려고 합니다. 물론 운영서버를 제가 이렇게 구성했다고 맞는 것은 아니며 요즘 기업들이 클라우드를 이용해서 구축하는 서비스 인프라와는 맞지 않거나 과거의 기술일 수 있습니다. 하지만 현재 제가 다니는 회사의 특성상 현재 방법이 최선일 수 있어 선택했단는 점 참고 바랍니다. 그럼 시작해 보겠습니다.

# 목차

### [VPC 생성하기](#vpc-생성하기)

- [AWS 콘솔에서 VPC 이동](#aws-콘솔에서-vpc-이동)
- [VPC 생성 마법사 실행](#vpc-생성-마법사-실행)
- [VPC 생성 확인](#vpc-생성-확인)
  - [서브넷 생성 확인](#서브넷-생성-확인)
  - [라우팅 테이블 생성 확인](#라우팅-테이블-생성-확인)
  - [인터넷 게이트웨이 생성 확인](#인터넷-게이트웨이-생성-확인)
  - [NAT 게이트웨이 생성 확인](#nat-게이트웨이-생성-확인)
- [마무리](#마무리)

[Reference](#reference)

---

# VPC 생성하기

먼저 VPC를 생성합니다. Amazon Virtual Private Cloud(Amazon VPC)는 논리적으로 격리된 가상 네트워크를 만들 수 있으며 사용자의 선택에 따라 서브넷 지정하거나 추가적인 리소스를 생성할 수 있습니다.

## AWS 콘솔에서 VPC 이동

AWS 콘솔에서 VPC 서비스를 검색해서 이동하겠습니다.

<center><img src="https://github.com/GoWoong/blog-post-repo/blob/main/image/Serise-Elastic-Beanstalk/VPC%20%EA%B2%80%EC%83%89.png?raw=true" width="80%" height="80%"/></center>
<center><p>VPC 이동</p></center>

그 다음 VPC 홈에서 VPC 생성이라고 되어있는 주황색 버튼을 클릭합니다.

<center><img src="https://github.com/GoWoong/blog-post-repo/blob/main/image/Serise-Elastic-Beanstalk/VPC%20%ED%99%88.png?raw=true" width="80%" height="80%"/></center>
<center><p>VPC 생성</p></center>

## VPC 생성 마법사 실행

제가 처음 AWS를 접했던 22년도에는 VPC 만들 때 직접 모든 VPC를 이루는 요소들을 생성하고 설정해야 했거나 마법사가 있어도 무슨 말인지 모르겠어서 VPC는 Default VPC 를 사용했던 기억이 있는데 보시면 요즘에는 AWS VPC 마법사가 잘되어 있어 이해도 잘되고 오른쪽에 보시면 생성되는 서비스들이 어떻게 연결되어 있는지도 보기 좋아서 쉽게 VPC를 생성하실 수 있습니다.

<center><img src="https://github.com/GoWoong/blog-post-repo/blob/main/image/Serise-Elastic-Beanstalk/VPC%20%EB%A7%88%EB%B2%95%EC%82%AC.png?raw=true" width="80%" height="80%"/></center>
<center><p>VPC 마법사</p></center>

그럼 본격적으로 VPC를 생성해 봐야겠죠?

<center><img src="https://github.com/GoWoong/blog-post-repo/blob/main/image/Serise-Elastic-Beanstalk/VPC%20%EC%83%9D%EC%84%B11.png?raw=true" width="80%" height="80%"/></center>
먼저 위에서 부터 설정을 보시면 생성할 리소스라고 해서 VPC 만 VPC 등 이라고 선택할 수 있습니다. VPC만은 진짜 VPC 만 하나 만들고 그안에 서브넷이나 igw 등은 수동으로 만들고 싶다고 할 때 사용합니다. 하지만 그러면 마법사를 사용하는 이유가 없죠? 저희는 VPC 등을 선택하겠습니다.

</br>
<center><img src="https://github.com/GoWoong/blog-post-repo/blob/main/image/Serise-Elastic-Beanstalk/VPC%20%EC%83%9D%EC%84%B11-1.png?raw=true" width="80%" height="80%"/></center>
그 다음으로 VPC 이름을 입력합니다. 이곳에 입력하는 이름으로 서브넷 등 이 VPC 마법사로 생성되는 리소스들의 이름에 전부 같은 값이 사용됩니다.

<br>
<center><img src="https://github.com/GoWoong/blog-post-repo/blob/main/image/Serise-Elastic-Beanstalk/VPC%20%EC%83%9D%EC%84%B11-2.png?raw=true" width="80%" height="80%"/></center>

밑에 IPv4 CIDR 블록 설정 등이 있는데 이 값은 다루기 위해서는 오랜 시간이 걸릴 것 같으니 넘어가겠습니다. 디폴트로 두어도 실행에 문제 없으니 말입니다.
<br>

<center><img src="https://github.com/GoWoong/blog-post-repo/blob/main/image/Serise-Elastic-Beanstalk/VPC%20%EC%83%9D%EC%84%B11-3.png?raw=true" width="80%" height="80%"/></center>

다음 설정을 살펴보겠습니다.
</br>

<center><img src="https://github.com/GoWoong/blog-post-repo/blob/main/image/Serise-Elastic-Beanstalk/VPC%20%EC%83%9D%EC%84%B12.png?raw=true" width="80%" height="80%"/></center>

이 부분이 중요합니다. 가용영역(AZ) 수를 설정할 수 있는데 이 설정은 리전의 가용영역 중 어느 것을 사용할지 선택하는 곳입니다. 가용영역을 일종의 데이터 센터라고 생각하시면 될 것 같은데 가용영역끼리는 물리적으로 분리되어 있어서 만약 한 곳의 가용영역에 문제가 있어도 재난 상황이 아닌 한 다른 가용영역은 동작하고 있을 수 있습니다. 그래서 운영서버의 가용성을 위해서는 단일 가용영역이 아닌 다중 가용영역을 설정하셔야 합니다. 저희는 2개로 선택하겠습니다. 추가로 사용할 가용영역을 선택하실 수 도 있습니다.
</br>

<center><img src="https://github.com/GoWoong/blog-post-repo/blob/main/image/Serise-Elastic-Beanstalk/VPC%20%EC%83%9D%EC%84%B12-1.png?raw=true" width="80%" height="80%"/></center>

다음 설정은 퍼블릭 서브넷과 프라이빗 서브넷 수를 설정할 수 있습니다. 저희는 운영서버를 만들 예정이기 때문에 2개의 퍼블릭 서브넷과 2개의 프라이빗 서브넷을 설정하겠습니다.
</br>

<center><img src="https://github.com/GoWoong/blog-post-repo/blob/main/image/Serise-Elastic-Beanstalk/VPC%20%EC%83%9D%EC%84%B12-2.png?raw=true" width="80%" height="80%"/></center>

퍼블릭 서브넷을 생성하는 이유는 기존의 서버에서는 EC2를 Public 서브넷에 위치시켰습니다. 즉 Public 서브넷에 모든 인스턴스를 위치시켜서 사용했는데 이런 경우 보안부분에서 문제가 생길 가능성이 높습니다. 그래서 프라이빗 서브넷 이름에서 부터 뭔가 비밀스러운 느낌이 나는 서브넷을 생성해서 일반적인 방법으로는 접근할 수 없도록 만들어진 서브넷을 통해 보안을 강화할 것입니다.
</br>

<center><img src="https://github.com/GoWoong/blog-post-repo/blob/main/image/Serise-Elastic-Beanstalk/VPC%20%EC%83%9D%EC%84%B13-1.png?raw=true" width="80%" height="80%"/></center>

다음 설정으로는 NAT 게이트웨이가 있습니다. NAT 게이트웨이는 프라이빗 서브넷이 외부와의 인터넷 통신을 위해 필요합니다. 자세한 것은 NAT 게이트웨이에 대해 다루신 다른 훌륭하신 선생님들의 포스트를 참조해주세요

그래서 저희는 일단 필요하니 1개만 만들겠습니다. 물론 운영환경에서는 AZ당 1개 씩 만드는 것이 가용성 측면에서는 더 좋지만 찾아보시면 알겠지만 NAT 게이트웨이 생각보다 만만치 않게 비싼 놈입니다. 특히 스타트업 혹은 개인이 사용하는 프로젝트에서 멋모르고 만들었다가는 한 달에 기본 5만원은 나오지 않을까 싶은 무시무시한 서비스입니다. 그러니 일단 1개 만들고 저는 다음에 개인 프로젝트에 맞도록 NAT 인스턴스로 변경하도록 하겠습니다.

VPC 엔드포인트의 경우는 생성해두겠습니다. 이 설정은 필요하게 된다면 추 후 설명 드리도록 하겠습니다.

## VPC 생성 확인

마지막으로 VPC 생성을 누르면 됩니다.
</br>

<center><img src="https://github.com/GoWoong/blog-post-repo/blob/main/image/Serise-Elastic-Beanstalk/VPC%20%EC%83%9D%EC%84%B1%EB%90%A8.png?raw=true" width="80%" height="80%"/></center>

그럼 마법사가 알아서 리소스들을 생성합니다. NAT 게이트웨이 생성에 대부분 시간이 걸리실 겁니다.
</br>

<center><img src="https://github.com/GoWoong/blog-post-repo/blob/main/image/Serise-Elastic-Beanstalk/VPC%20%EC%83%9D%EC%84%B1%EC%99%84%EB%A3%8C.png?raw=true" width="80%" height="80%"/></center>

생성이 왼료되면 아래쪽에 VPC 보기가 생겨납니다. 클릭하시면 생성된 VPC를 보실 수 있습니다.
</br>

<center><img src="https://github.com/GoWoong/blog-post-repo/blob/main/image/Serise-Elastic-Beanstalk/%EC%83%9D%EC%84%B1%EB%90%9C%20VPC.png?raw=true" width="80%" height="80%"/></center>

### 서브넷 생성 확인

</br>
<center><img src="https://github.com/GoWoong/blog-post-repo/blob/main/image/Serise-Elastic-Beanstalk/%EC%83%9D%EC%84%B1%EB%90%9C%20%EC%84%9C%EB%B8%8C%EB%84%B7.png?raw=true" width="80%" height="80%"/></center>
<center><p>생성된 서브넷</p></center>

### 라우팅 테이블 생성 확인

</br>
<center><img src="https://github.com/GoWoong/blog-post-repo/blob/main/image/Serise-Elastic-Beanstalk/%EC%83%9D%EC%84%B1%EB%90%9C%20%EB%9D%BC%EC%9A%B0%ED%8C%85%ED%85%8C%EC%9D%B4%EB%B8%94.png?raw=true" width="80%" height="80%"/></center>
<center><p>생성된 라우팅테이블</p></center>

### 인터넷 게이트웨이 생성 확인

</br>
<center><img src="https://github.com/GoWoong/blog-post-repo/blob/main/image/Serise-Elastic-Beanstalk/%EC%83%9D%EC%84%B1%EB%90%9C%20igw.png?raw=true" width="80%" height="80%"/></center>
<center><p>생성된 인터넷 게이트웨이</p></center>

### NAT 게이트웨이 생성 확인

</br>
<center><img src="https://github.com/GoWoong/blog-post-repo/blob/main/image/Serise-Elastic-Beanstalk/%EC%83%9D%EC%84%B1%EB%90%9C%20NAT.png?raw=true" width="80%" height="80%"/></center>
<center><p>생성된 NAT 게이트웨이</p></center>
<br>

## 마무리

지금까지 VPC를 생성했습니다. 이제 저희는 운영환경에서 사용할 수 있는 VPC를 생성해냈습니다. 다음편에서는 실제 Beanstalk 실행하는 것을 보여드리도록 하겠습니다. 감사합니다.

# Reference

- [Amazon VPC란 무엇인가?](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/what-is-amazon-vpc.html "AWS 사용 설명서")
- [VPC CIDR 블록](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/vpc-cidr-blocks.html "AWS 사용 설명서")
- [NAT 게이트웨이](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/vpc-nat-gateway.html "AWS 사용 설명서")
