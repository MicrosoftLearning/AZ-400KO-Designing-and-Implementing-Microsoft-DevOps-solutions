---
lab:
    title: '랩: Azure Kubernetes Service에 다중 컨테이너 애플리케이션 배포'
    module: '모듈 16: Kubernetes Service 인프라 만들기 및 관리'
---

# 랩: Azure Kubernetes Service에 다중 컨테이너 애플리케이션 배포
# 학생 랩 매뉴얼

## 랩 개요

[**AKS(Azure Kubernetes Service)**](https://azure.microsoft.com/ko-kr/services/kubernetes-service/)는 Azure에서 Kubernetes를 사용하는 가장 빠른 방법입니다. **AKS(Azure Kubernetes Service)** 는 호스팅된 Kubernetes 환경을 관리하므로 컨테이너 오케스트레이션에 대한 전문 지식이 없어도 컨테이너화된 애플리케이션을 간편하게 배포하고 관리할 수 있습니다. 그리고 컨테이너화된 워크로드의 가용성, 확장성, 유연성도 개선할 수 있습니다. 연속 빌드 및 배포 기능이 제공되는 Azure DevOps에서는 AKS 작업을 더욱 원활하게 진행할 수 있습니다. 

이 랩에서는 Azure DevOps를 사용해 컨테이너화된 ASP.NET Core 웹 애플리케이션 **MyHealthClinic**(MHC)을 AKS 클러스터에 배포합니다. 

## 목표

이 랩을 완료하면 다음 작업을 수행할 수 있습니다.

- Azure DevOps 데모 생성기 도구를 사용하여 .NET Core 애플리케이션이 포함된 Azure DevOps 팀 프로젝트 만들기
- Azure CLI를 사용하여 ACR(Azure Container Registry), AKS 클러스터 및 Azure SQL 데이터베이스 만들기
- Azure DevOps를 사용하여 컨테이너화된 애플리케이션 및 데이터베이스 배포 구성
- Azure DevOps 파이프라인을 사용하여 컨테이너화된 애플리케이션 빌드 및 자동 배포

## 랩 소요 시간

-   예상 시간: **60분**

## 지침

### 시작하기 전

#### 랩 가상 머신에 로그인

다음 자격 증명을 사용하여 Windows 10 컴퓨터에 로그인했는지 확인합니다.
    
-   사용자 이름: **Student**
-   암호: **Pa55w.rd**

#### 이 랩에 필요한 애플리케이션 검토

이 랩에서 사용할 애플리케이션을 확인합니다.
    
-   Microsoft Edge

#### Azure DevOps 조직 설정 

이 랩에 사용할 수 있는 Azure DevOps 조직이 아직 없으면 [조직 또는 프로젝트 컬렉션 만들기](https://docs.microsoft.com/ko-kr/azure/devops/organizations/accounts/create-organization?view=azure-devops)에서 제공되는 지침에 따라 조직을 만듭니다.

#### Azure 구독 준비

-   기존 Azure 구독을 확인하거나 새 구독을 만듭니다.
-   Azure 구독의 소유자 역할, 그리고 Azure 구독과 연결된 Azure AD 테넌트의 전역 관리자 역할이 지정되어 있는 Microsoft 계정 또는 Azure AD 계정이 있는지 확인합니다. 자세한 내용은 [Azure Portal을 사용하여 Azure 역할 할당 목록 표시](https://docs.microsoft.com/ko-kr/azure/role-based-access-control/role-assignments-list-portal) 및 [Azure Active Directory에서 관리자 역할 확인 및 할당](https://docs.microsoft.com/ko-kr/azure/active-directory/roles/manage-roles-portal#view-my-roles)을 참조하세요.

### 연습 0: 랩 필수 구성 요소 구성

이 연습에서는 랩의 필수 구성 요소를 설정합니다. 구체적으로는 Azure DevOps 데모 생성기 템플릿을 기반으로 하여 미리 구성된 팀 프로젝트를 설정합니다. 

#### 작업 1: 팀 프로젝트 구성

이 작업에서는 Azure DevOps 데모 생성기를 사용하여 **Azure Kubernetes Service** 템플릿을 기반으로 새 프로젝트를 생성합니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [Azure DevOps 데모 생성기](https://azuredevopsdemogenerator.azurewebsites.net)로 이동합니다. 이 유틸리티 사이트에서는 계정 내에서 새 Azure DevOps 프로젝트를 만드는 프로세스를 자동으로 진행할 수 있습니다. 이 프로젝트에는 랩에 필요한 작업 항목, 리포지토리 등의 콘텐츠가 미리 입력되어 있습니다. 

    > **참고**: 이 사이트에 대한 자세한 내용은 https://docs.microsoft.com/ko-kr/azure/devops/demo-gen을 참조하세요.

1.  **로그인**을 클릭하고 Azure DevOps 구독과 연결된 Microsoft 계정을 사용하여 로그인합니다.
1.  필요한 경우 **Azure DevOps 데모 생성기** 페이지에서 **수락**을 클릭하여 Azure DevOps 구독에 액세스하는 데 필요한 권한 요청을 수락합니다.
1.  **새 프로젝트 만들기** 페이지의 **새 프로젝트 이름** 텍스트 상자에 ***를 입력합니다. 그런 다음 **조직 선택** 드롭다운 목록에서 Azure DevOps 조직을 선택하고 **템플릿 선택**을 클릭합니다.
1.  템플릿 목록의 도구 모음에서 **DevOps 랩**을 클릭하고 **Azure Kubernetes Service** 템플릿을 선택한 다음 **템플릿 선택**을 클릭합니다.
1.  **새 프로젝트 만들기** 페이지로 돌아와서 누락된 확장을 설치하라는 메시지가 표시되면 **토큰 바꾸기** 및 **Kubernetes 확장** 레이블 아래의 체크박스를 선택하고 **프로젝트 만들기**를 클릭합니다.

    > **참고**: 프로세스가 완료될 때까지 기다립니다. 약 2분이 소요됩니다. 프로세스가 실패하면 DevOps 조직으로 이동하여 프로젝트를 삭제한 후에 다시 시도합니다.

1.  **새 프로젝트 만들기** 페이지에서 **프로젝트로 이동**을 클릭합니다.

### 연습 1: Azure DevOps를 사용하여 AKS 클러스터에 컨테이너화된 ASP.NET Core 웹 애플리케이션 배포

이 연습에서는 Azure DevOps를 사용하여 AKS 클러스터에 컨테이너화된 ASP.NET Core 웹 애플리케이션을 배포합니다.

#### 작업 1: 랩용 Azure 리소스 배포

이 작업에서는 Azure CLI를 사용하여 이 랩에 필요한 다음의 Azure 리소스 배포를 수행합니다.

| Azure 리소스 | 설명 |
| --------------- | ----------- |
| Azure Container Registry | Docker 이미지의 프라이빗 저장소로 사용됨 |
| AKS | Docker 이미지를 실행하는 컨테이너의 오케스트레이터로 사용됨 |
| Azure SQL Database | AKS에서 실행되는 컨테이너화된 워크로드용 영구 저장소 제공 |

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [**Azure Portal**](https://portal.azure.com)로 이동한 다음 이 랩에서 사용하는 Azure 구독에서 Contributor 이상의 역할이 지정된 사용자 계정으로 로그인합니다.
1.  Azure Portal의 도구 모음에서 검색 텍스트 상자 바로 오른쪽에 있는 **Cloud Shell** 아이콘을 클릭합니다. 
1.  **Bash** 또는 **PowerShell**을 선택하라는 메시지가 표시되면 **Bash**를 선택합니다. 

    >**참고**: **Cloud Shell**을 처음 시작하고 **탑재된 스토리지가 없음** 메시지를 받으면, 이 랩에서 사용하는 구독을 선택하고 **스토리지 만들기**를 선택합니다. 

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 이 랩에서 사용할 Azure 지역에서 제공되는 Kubernetes의 최신 버전을 확인합니다. 여기서 `<Azure_region>` 자리 표시자는 이 랩에서 리소스를 배포할 Azure 지역 이름으로 바꿉니다.

    ```bash
    LOCATION=<Azure_region>
    VERSION=$(az aks get-versions --location $LOCATION --query 'orchestrators[-1].orchestratorVersion' --output tsv)
    ```
   
1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 AKS 배포를 호스트할 리소스 그룹을 만듭니다.

    ```bash
    RGNAME=az400m16l01a-RG
    az group create --name $RGNAME --location $LOCATION
    ```

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 사용 가능한 최신 버전을 사용해 AKS 클러스터를 만듭니다.
    
    ```bash
    AKSNAME='az400m16aks'$RANDOM$RANDOM
    az aks create --location $LOCATION --resource-group $RGNAME --name $AKSNAME --enable-addons monitoring --kubernetes-version $VERSION --generate-ssh-keys
    ```
    
    >**참고**: 배포가 완료될 때까지 기다렸다가 다음 작업을 진행합니다. AKS 배포는 5분 정도 걸릴 수 있습니다. 

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 이 랩에서 사용할 Azure SQL 데이터베이스를 호스트할 논리 서버를 만듭니다.
  
    ```bash
    SQLNAME='az400m16sql'$RANDOM$RANDOM
    az sql server create --location $LOCATION --resource-group $RGNAME --name $SQLNAME --admin-user sqladmin --admin-password P2ssw0rd1234
    ```

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 Azure에서 새로 프로비전한 논리 서버에 액세스할 수 있도록 허용합니다.

    ```bash
    az sql server firewall-rule create --resource-group $RGNAME --server $SQLNAME --name allowAzure --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
    ```

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 이 랩에서 사용할 Azure SQL 데이터베이스를 만듭니다.


    ```bash
    az sql db create --resource-group $RGNAME --server $SQLNAME --name mhcdb --service-objective S0 --no-wait
    ```

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 이 랩에서 사용할 AAzure Container Registry를 만듭니다.

    ```bash
    ACRNAME='az400m16acr'$RANDOM$RANDOM
    az acr create --location $LOCATION --resource-group $RGNAME --name $ACRNAME --sku Standard
    ```

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 AKS에서 생성된 관리 ID에 새로 만든 ACR 액세스 권한을 부여합니다.

    ```bash
    # Retrieve the id of the service principal configured for AKS
    CLIENT_ID=$(az aks show --resource-group $RGNAME --name $AKSNAME --query "identityProfile.kubeletidentity.clientId" --output tsv)

    # Retrieve the ACR registry resource id
    ACR_ID=$(az acr show --name $ACRNAME --resource-group $RGNAME --query "id" --output tsv)

    # Create role assignment
    az role assignment create --assignee $CLIENT_ID --role acrpull --scope $ACR_ID
    ```

    >**참고**: 이 할당에 대한 자세한 내용은 [Azure Kubernetes Service에서 Azure Container Registry를 사용하여 인증](https://docs.microsoft.com/ko-kr/azure/container-registry/container-registry-auth-aks)을 참조하세요.

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 이 작업의 앞부분에서 만든 Azure SQL 데이터베이스를 호스트하는 논리 서버의 이름을 표시합니다.

    ```bash
    echo $(az sql server list --resource-group $RGNAME --query '[].name' --output tsv)'.database.windows.net'
    ```

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 이 작업의 앞부분에서 만든 Azure Container Registry의 로그인 서버 이름을 표시합니다.

    ```bash
    az acr show --name $ACRNAME --resource-group $RGNAME --query "loginServer" --output tsv
    ```

    >**참고**: 논리 서버의 이름과 로그인 서버의 이름을 모두 적어 두세요. 다음 작업에서 이 두 값이 필요합니다.

1. Cloud Shell 창을 닫습니다.

#### 작업 2: 빌드 및 릴리스 파이프라인 구성

이 작업에서는 이 랩 앞부분에서 생성한 Azure DevOps 프로젝트에서 빌드 및 릴리스 파이프라인을 구성합니다. 이 구성 과정에서는 AKS 클러스터 및 Azure Container Registry를 비롯한 Azure 리소스를 빌드 및 릴리스 정의에 매핑합니다.

1.  랩 컴퓨터에서 **AKS에 다중 컨테이너 애플리케이션 배포** 프로젝트가 열려 있는 Azure DevOps 포털이 표시된 웹 브라우저 창으로 전환하여 Azure DevOps 포털 맨 왼쪽의 세로 메뉴 모음에서 **Pipelines**를 클릭합니다.
1.  **파이프라인** 창에서 **MyHealth.AKS.build** 파이프라인을 나타내는 항목을 클릭하고 **MyHealth.AKS.build** 창에서 **편집**을 클릭합니다.
1.  파이프라인의 작업 목록에서 **서비스 실행** 작업을 클릭하고 오른쪽의 **Docker Compose** 창에 있는 **Azure 구독** 드롭다운 목록에서 
이 랩에서 사용 중인 Azure 구독을 나타내는 항목을 선택하고 **권한 부여**를 클릭하여 해당 서비스 연결을 만듭니다. 메시지가 표시되면 Azure 구독의 소유자 역할, 그리고 Azure 구독과 연결된 Azure AD 테넌트의 전역 관리자 역할이 지정되어 있는 계정을 사용하여 로그인합니다.

    >**참고**: 이 단계에서는 Azure 서비스 연결을 만듭니다. 구체적으로는 SPA(서비스 주체 인증)를 사용하여 대상 Azure 구독에 대한 연결을 정의하고 보호합니다. 

1.  파이프라인의 작업 목록에서 **서비스 실행** 작업을 선택한 상태로 오른쪽의 **Docker Compose** 창에 있는 **Azure Container Registry** 드롭다운 목록에서 이 랩의 앞부분에서 만든 ACR 인스턴스를 나타내는 항목을 선택합니다.
1.  위의 두 단계를 반복하여 **서비스 빌드**, **서비스 푸시**, **서비스 잠금** 작업에서 **Azure 구독** 및 **Azure Container Registry** 설정을 구성합니다. 단, 이번에는 Azure 구독을 선택하는 대신 새로 만든 서비스 연결을 선택합니다. 

    >**참고**: 파이프라인은 다음과 같은 작업으로 구성되어 있습니다.

    | 작업 | 사용법 |
    | ------ | ------ |
    | **토큰 바꾸기** | 자리 표시자를 **appsettings.json** 파일과 **mhc-aks.yaml** 매니페스트 파일의 데이터베이스 연결 문자열에 포함된 ACR의 이름으로 바꿉니다. |
    | **서비스 실행** |  aspnetcore-build:1.0-2.0 등의 필수 이미지를 끌어온 다음 **csproj**에서 참조되는 패키지를 복원하여 환경을 준비합니다. |
    | **서비스 빌드** |  **docker-compose.yml** 파일에 지정된 Docker 이미지를 빌드하고 이미지에 **$(Build.BuildId)** 및 **latest** 태그를 지정합니다. |
    | **서비스 푸시** |  Docker 이미지 **myhealth.web**을 Azure Container Registry로 푸시합니다. |
    | **빌드 아티팩트 게시** |  **mhc-aks.yaml** 및 **myhealth.dacpac** 파일을 후속 릴리스에서 사용할 수 있도록 Azure DevOps의 아티팩트 저장 위치에 게시합니다. |

    >**참고**: **appsettings.json** 파일에는 이 랩의 앞부분에서 만든 Azure SQL 데이터베이스에 연결하는 데 사용되는 데이터베이스 연결 문자열의 세부 정보가 들어 있습니다.  **mhc-aks.yaml** 매니페스트 파일에는 Azure Kubernetes Service에서 배포할 **deployments**, **services** 및 **pods**의 구성 세부 정보가 들어 있습니다. 배포 매니페스트에 대한 자세한 내용은 [AKS 배포 및 YAML 매니페스트](https://docs.microsoft.com/ko-kr/azure/aks/concepts-clusters-workloads#deployments-and-yaml-manifests)를 참조하세요.
1.  **파이프라인 변수** 목록에서 **ACR** 및 **SQLserver** 변수의 값을 이전 작업의 끝부분에서 적어 둔 값으로 업데이트하고 **저장 및 큐에 넣기** 단추 옆의 아래쪽 캐럿을 클릭합니다. 그런 다음 **저장**을 클릭하여 변경 내용을 저장하고 다시 메시지가 표시되면 **저장**을 클릭합니다.
1.  Azure DevOps 포털이 표시된 웹 브라우저 창에서 Azure DevOps 포털 맨 왼쪽 세로 메뉴 모음의 **Pipelines** 섹션에 있는 **릴리스**를 클릭합니다. 
1.  **파이프라인/릴리스** 창에서 **MyHealth.AKS.Release** 항목을 선택하고 **편집**을 클릭합니다.
1.  **모든 파이프라인 > MyHealth.AKS.Release** 창에 있는 배포의 **개발** 단계를 나타내는 사각형에서 **2개 작업, 3개 태스크** 링크를 클릭합니다.
1.  **개발** 단계의 태스크 목록에 있는 **DB 배포** 작업 섹션 내에서 **Azure SQL 실행: DacpacTask** 태스크를 선택합니다. 그런 다음 오른쪽의 **Azure SQL 데이터베이스 배포** 창에 있는 **Azure 구독** 드롭다운 목록에서 이 작업 앞부분에서 만든 Azure 서비스 연결을 나타내는 항목을 선택합니다.
1.  **개발** 단계의 태스크 목록에 있는 **AKS 배포** 작업 섹션 내에서 **AKS에서 배포 및 서비스 만들기** 태스크를 선택합니다. 
1.  오른쪽의 **Kubectl** 창에 있는 **Azure 구독** 드롭다운 목록에서 같은 Azure 서비스 연결을 나타내는 항목을 선택하고 **리소스 그룹** 드롭다운 목록에서 **az400m16l01a-RG** 항목을 선택합니다. 그런 다음 **Kubernetes 클러스터** 드롭다운 목록에서 이 랩의 앞부분에서 배포한 AKS 클러스터를 나타내는 항목을 선택합니다.
1.  **개발** 단계의 태스크 목록에 있는 **AKS 배포** 작업 섹션에서 **AKS에서 배포 및 서비스 만들기** 태스크를 선택한 상태로 오른쪽의 **Kubectl** 창에서 아래쪽으로 스크롤하여 **비밀** 섹션을 확장합니다. 그런 다음 **Azure 구독** 드롭다운 목록에서 같은 Azure 서비스 연결을 나타내는 항목을 선택하고 **Azure Container Registry** 드롭다운 목록에서는 이 랩의 앞부분에서 만든 Azure Container Registry를 나타내는 항목을 선택합니다.
1.  **AKS에서 이미지 업데이트** 태스크에서도 위의 두 단계를 반복합니다.

    >**참고**: **AKS에서 배포 및 서비스 만들기** 태스크에서는 **mhc-aks.yaml** 파일에 지정된 구성에 따라 AKS에서 필요한 배포 및 서비스를 만듭니다. 그러면 Pod가 최신 Docker 이미지를 끌어옵니다.

    >**참고**: **AKS에서 이미지 업데이트** 태스크에서는 지정된 리포지토리에서 BuildID에 해당하는 필수 이미지를 끌어온 다음 AKS에서 실행 중인 **mhc-front pod**에 해당 이미지를 배포합니다.

    >**참고**: 백그라운드에서는 `kubectl create secret` 명령을 사용하여 Azure DevOps를 통해 AKS 클러스터에 비밀(**mysecretkey**)이 생성됩니다. 이 비밀은 myhealth.web 이미지를 끌어오기 위해 Azure Container Registry 액세스 권한을 부여하는 데 사용됩니다.

1.  **MyHealth.AKS.Release** 릴리스 파이프라인 **개발** 단계의 **태스크** 창에서 **변수** 탭을 클릭합니다. 
1.  **파이프라인 변수** 목록에서 **ACR** 변수의 값을 이전 작업 끝부분에서 적어 둔 Azure Container Registry 이름으로 업데이트합니다. 
1.  **파이프라인 변수** 목록에서 **SQLserver** 변수의 값을 이전 작업 끝부분에서 적어 둔 논리 서버 이름으로 업데이트합니다. 
1.  **모든 파이프라인/MyHealth.AKS.Release** 창 오른쪽 위의 **저장**을 클릭하고 메시지가 표시되면 **저장**을 다시 클릭하여 변경 내용을 저장합니다.

    >**참고**: 파이프라인 변수 목록에서 **DatabaseName**은 **mhcdb**로, **SQLuser**는 **sqladmin**으로, **SQLpassword**는 **P2ssw0rd1234**로 설정되어 있습니다. 이 랩의 앞부분에서 Azure SQL 데이터베이스를 만들 때 다른 값을 입력했다면 변수 값을 적절하게 업데이트하세요.

1. **DB 배포** 작업 및 **AKS 배포**에서는 "에이전트 풀" **Azure Pipelines --> windows-2019**를 선택합니다.


#### 작업 3: 빌드 및 릴리스 파이프라인 트리거

이 작업에서는 빌드 및 릴리스 파이프라인을 트리거하고 해당 파이프라인이 완료되는지 유효성을 검사합니다.

1.  Azure DevOps 포털이 표시된 웹 브라우저 창에서 Azure DevOps 포털 맨 왼쪽 세로 메뉴 모음의 **Pipelines** 섹션에 있는 **파이프라인**을 클릭합니다. 
1.  **파이프라인** 창에서 **MyHealth.AKS.build** 파이프라인을 선택하고 **MyHealth.AKS.build** 창에서 **파이프라인 실행**을 클릭합니다. 그런 다음 **파이프라인 실행** 창에서 **실행**을 클릭합니다.
1.  빌드 파이프라인 실행 창의 **작업** 섹션에서 **1단계**를 클릭하고 빌드 프로세스 진행률을 모니터링합니다.

    >**참고**: 빌드에서는 Docker 이미지가 생성되어 ACR로 푸시됩니다. 빌드가 완료되면 빌드 요약을 검토할 수 있습니다. 

1.  생성된 이미지를 검토하려면 Azure Portal이 표시된 웹 브라우저 창으로 전환합니다.
1.  Azure Portal에서 **컨테이너 레지스트리** 리소스 종류를 검색하여 선택하고 **컨테이너 레지스트리** 블레이드에서 이 랩의 앞부분에서 만든 Azure Container Registry를 선택합니다.
1.  Azure Container Registry 블레이드의 **서비스** 섹션에서 **리포지토리**를 클릭하고 리포지토리 목록에 **myhealth.web** 항목이 포함되어 있는지 확인합니다.
1.  Azure DevOps 포털이 표시된 웹 브라우저 창으로 다시 전환합니다.
1.  Azure DevOps 포털 맨 왼쪽의 세로 메뉴 모음에 있는 **Pipelines** 섹션에서 **릴리스**를 클릭하고 **MyHealth.AKS.Release** 블레이드에서 최신 릴리스를 클릭합니다. 그런 다음 **진행 중** 링크를 선택하여 릴리스 진행률을 모니터링합니다.
1.  릴리스가 완료되면 Azure Portal이 표시된 웹 브라우저 창으로 전환합니다.
1.  Azure Portal의 도구 모음에서 검색 텍스트 상자 바로 오른쪽에 있는 **Cloud Shell** 아이콘을 클릭합니다. 
1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 이 랩의 앞부분에서 배포한 AKS 클러스터에 액세스합니다.

    ```bash
    RGNAME=az400m16l01a-RG
    AKSNAME=$(az aks list --resource-group $RGNAME --query '[].name' --output tsv)
    az aks get-credentials --resource-group $RGNAME --name $AKSNAME
    ```

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 릴리스 파이프라인을 사용해 배포한 AKS에서 실행 중인 Pod의 목록을 표시합니다.

    ```bash
    kubectl get pods
    ```

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 컨테이너화된 애플리케이션에 액세스하는 데 사용할 수 있는 외부 IP 주소를 제공하는 부하 분산 장치 서비스의 목록을 표시합니다.

    ```bash
    kubectl get service mhc-front --watch
    ```

    >**참고**: 이 애플리케이션은 외부 연결 기능을 제공하는 부하 분산 장치 서비스가 포함된 Pod에 배포할 수 있습니다. 

1.  명령 출력에서 **External-IP** 열의 IP 주소 값을 적어 두고 새 브라우저 탭을 열어 해당 IP 주소로 이동한 다음 **MyHealthClinic** 애플리케이션이 실행되고 있는지 확인합니다.

    >**참고**: Kubernetes에는 기본 관리 작업에 사용할 수 있는 웹 대시보드가 포함됩니다. 이 대시보드를 사용하면 애플리케이션의 기본 상태와 메트릭을 보고 서비스를 작성 및 배포하며 기존 애플리케이션을 편집할 수 있습니다. [Microsoft Docs](https://docs.microsoft.com/ko-kr/azure/aks/kubernetes-dashboard)의 설명에 따라 AKS(Azure Kubernetes Service)의 Kubernetes 웹 대시보드에 액세스합니다.

### 연습 2: Azure 랩 리소스 제거

이 연습에서는 예상치 못한 비용이 발생하지 않도록 이 랩에서 프로비전한 Azure 리소스를 제거합니다. 

>**참고**: 더 이상 사용하지 않는 새로 만든 Azure 리소스를 제거해야 합니다. 사용하지 않는 리소스를 제거하면 예기치 않은 비용이 발생하지 않습니다.

#### 작업 1: Azure 랩 리소스 제거

이 작업에서는 Azure Cloud Shell을 사용하여 불필요한 비용이 발생하지 않도록 이 랩에서 프로비전한 Azure 리소스를 제거합니다. 

1.  Azure Portal의 **Cloud Shell** 창에서 **Bash** 세션을 시작합니다.
1.  다음 명령을 실행하여 이 모듈의 전체 랩에서 생성된 모든 리소스 그룹을 나열합니다.

    ```sh
    az group list --query "[?starts_with(name,'az400m16l01a-RG')].name" --output tsv
    ```

1.  다음 명령을 실행하여 이 모듈의 전체 랩에서 만든 모든 리소스 그룹을 삭제합니다.

    ```sh
    az group list --query "[?starts_with(name,'az400m16l01a-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**참고**: 명령은 비동기적으로 실행되므로(--nowait 매개 변수에 의해 결정됨) 동일한 Bash 세션 내에서 즉시 다른 Azure CLI 명령을 실행할 수 있지만 리소스 그룹이 실제로 제거되기까지 몇 분 정도 걸립니다.

## 복습

이 랩에서는 Azure DevOps를 사용해 컨테이너화된 ASP.NET Core 웹 애플리케이션 **MyHealthClinic**(MHC)을 AKS 클러스터에 배포하는 방법을 알아보았습니다. 