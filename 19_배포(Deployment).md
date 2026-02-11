# 5단계: 배포와 심화 학습 (Deployment & Beyond)

## 4. 배포 (Deployment)

**배포**는 내가 개발한 애플리케이션을 서버 컴퓨터에 올려서, 다른 사용자들이 인터넷을 통해 접근하고 사용할 수 있도록 공개하는 과정을 말합니다. 이전 단계에서 만든 Docker 컨테이너를 클라우드 서비스에 올리는 것이 현대적인 배포 방식의 핵심입니다. 여기서는 가장 대표적인 클라우드 서비스인 **AWS(Amazon Web Services)**를 예시로 배포의 전체적인 흐름과 개념을 알아보겠습니다.

---

### **배포의 전체 과정 (개념)**

로컬 컴퓨터에서 만든 Docker 이미지를 클라우드에 배포하는 과정은 크게 3단계로 나눌 수 있습니다.

1.  **이미지 공유 (Push to Registry):** 로컬에 있는 Docker 이미지를 클라우드에 있는 원격 이미지 저장소에 업로드합니다.
2.  **컨테이너 실행 (Run Container):** 원격 저장소의 이미지를 가져와 클라우드 서버에서 컨테이너로 실행합니다.
3.  **외부 연결 (Expose to Web):** 실행된 컨테이너에 외부 사용자가 접속할 수 있도록 네트워크와 도메인을 설정합니다.

---

### **1단계: 컨테이너 레지스트리에 이미지 올리기**

내 컴퓨터에 있는 Docker 이미지는 나만 쓸 수 있습니다. 클라우드 서버가 이 이미지를 사용하게 하려면, 모두가 접근할 수 있는 원격 저장소에 이미지를 올려야 합니다. 이를 **컨테이너 레지스트리(Container Registry)**라고 합니다.

-   **대표적인 레지스트리:**
    -   **Docker Hub:** Docker 사에서 운영하는 기본 레지스트리입니다.
    -   **Amazon ECR (Elastic Container Registry):** AWS에서 제공하는 프라이빗 레지스트리로, AWS의 다른 서비스와 연동이 매우 편리합니다.

**ECR에 이미지를 올리는 과정 (개념):**

1.  `docker login`: AWS ECR에 로그인합니다.
2.  `docker tag [로컬 이미지]:[태그] [ECR 레지스트리 주소]/[이미지 이름]:[태그]`: 로컬 이미지에 ECR 저장소 주소를 포함하는 새 태그를 부여합니다.
3.  `docker push [ECR 레지스트리 주소]/[이미지 이름]:[태그]`: 태그가 부여된 이미지를 ECR로 업로드합니다.

---

### **2단계: 클라우드에서 컨테이너 실행하기**

ECR에 이미지가 준비되었다면, 이제 AWS의 여러 서비스 중 하나를 선택하여 컨테이너를 실행할 수 있습니다.

-   **IaaS (Infrastructure as a Service) - 예: Amazon EC2**
    -   가상의 서버 컴퓨터 한 대를 빌려서, 그 안에 직접 Docker를 설치하고 `docker run` 명령어를 실행하는 방식입니다.
    -   **장점:** 모든 것을 직접 제어할 수 있어 자유도가 높습니다.
    -   **단점:** 서버 관리, Docker 설치, 업데이트, 보안 설정 등을 모두 직접 해야 해서 초보자에게는 복잡하고 어렵습니다.

-   **PaaS / CaaS (Platform/Container as a Service) - 예: AWS App Runner, Elastic Beanstalk**
    -   서버나 Docker 설치 같은 인프라 관리는 AWS에 맡기고, 개발자는 **어떤 Docker 이미지를 실행할지만 알려주는** 방식입니다.
    -   **장점:** 사용법이 매우 간단하고, 버튼 몇 번 클릭하거나 간단한 설정만으로 배포, 확장, 로드 밸런싱이 자동으로 처리됩니다. **초보자에게 가장 추천되는 방식입니다.**
    -   **단점:** 정해진 틀 안에서 사용해야 하므로 자유도는 상대적으로 낮습니다.

#### **가장 간단한 배포 예시: AWS App Runner**

AWS App Runner는 소스 코드나 컨테이너 이미지만 제공하면 빌드부터 배포, 운영까지 모든 것을 자동화해주는 완전 관리형 서비스입니다. 개념적인 사용 흐름은 다음과 같습니다.

1.  AWS 콘솔에서 App Runner 서비스 생성을 시작합니다.
2.  컨테이너 소스로 **Amazon ECR**을 선택하고, 이전에 푸시한 이미지의 주소를 지정합니다.
3.  App Runner가 컨테이너의 **8080 포트**를 감지하도록 포트 설정을 합니다.
4.  서비스 생성을 완료하면, App Runner가 자동으로 이미지를 가져와 컨테이너를 실행하고, `https://[고유ID].awsapprunner.com` 형태의 **공개 도메인 주소**를 제공합니다.

이 주소로 접속하면 전 세계 어디서든 내가 만든 애플리케이션을 사용할 수 있게 됩니다.

---

### **3단계 (심화): CI/CD - 배포 자동화**

지금까지의 과정은 모두 수동입니다. 코드를 수정한 후에는 매번 `build -> tag -> push -> run` 과정을 반복해야 합니다. **CI/CD (Continuous Integration/Continuous Deployment)**는 이 모든 과정을 자동화하는 것을 의미합니다.

-   **CI (지속적 통합):** 개발자가 코드를 GitHub 같은 곳에 푸시하면, 자동으로 테스트와 빌드가 실행되는 단계입니다.
-   **CD (지속적 배포):** CI가 성공적으로 완료되면, 자동으로 Docker 이미지를 만들고 클라우드에 배포까지 하는 단계입니다.

**GitHub Actions**와 같은 도구를 사용하면, "내 노트북에서 코드를 `git push` 하기만 하면, 몇 분 뒤에 자동으로 서버에 배포가 완료되는" 파이프라인을 구축할 수 있습니다.

---

### **요약**

-   **배포**는 개발이 끝난 애플리케이션을 서버에 올려 세상에 공개하는 과정입니다.
-   현대적인 배포는 **Docker 컨테이너**를 **클라우드 서비스**에 올리는 방식을 주로 사용합니다.
-   배포 과정: 로컬의 Docker 이미지를 **ECR** 같은 원격 저장소에 `push`하고, **App Runner** 같은 관리형 서비스에서 해당 이미지를 `run`합니다.
-   **AWS App Runner** 같은 PaaS/CaaS를 사용하면 인프라 관리 부담 없이 매우 쉽게 컨테이너를 배포할 수 있어 초보자에게 적합합니다.
-   **CI/CD**는 이 모든 배포 과정을 자동화하여 개발 효율을 극대화하는 다음 단계의 목표입니다.

---

### **출처 및 참고 자료**

-   **공식 문서:**
    -   Deploy Docker Containers on Amazon ECS: [https://aws.amazon.com/getting-started/hands-on/deploy-docker-containers/](https://aws.amazon.com/getting-started/hands-on/deploy-docker-containers/)
    -   What is AWS App Runner?: [https://docs.aws.amazon.com/apprunner/latest/dg/what-is-apprunner.html](https://docs.aws.amazon.com/apprunner/latest/dg/what-is-apprunner.html)
-   **개념 이해를 위한 자료:**
    -   AWS에 Docker 컨테이너를 배포하는 3가지 방법 (nOps 블로그): [https://www.nops.io/blog/top-3-ways-to-run-docker-container-on-aws/](https://www.nops.io/blog/top-3-ways-to-run-docker-container-on-aws/)
    -   CI/CD란 무엇인가? (Red Hat): [https://www.redhat.com/ko/topics/devops/what-is-ci-cd](https://www.redhat.com/ko/topics/devops/what-is-ci-cd)
