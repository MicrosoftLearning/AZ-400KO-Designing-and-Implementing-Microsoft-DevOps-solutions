---
lab:
    title: '랩: Terraform 및 Azure Pipelines를 통해 클라우드에서 인프라 배포 자동화'
    az400Module: '모듈 14: Azure에서 사용 가능한 타사 IaC(코드형 인프라) 도구'
---

# 랩: Terraform 및 Azure Pipelines를 통해 클라우드에서 인프라 배포 자동화
# 학생 랩 매뉴얼

## 랩 개요

[Terraform](https://www.terraform.io/intro/index.html)은 인프라 빌드, 변경 및 버전 관리용 도구입니다. Terraform은 기존의 인기 있는 클라우드 서비스 공급자 뿐 아니라 맞춤형 사내 솔루션을 관리할 수 있습니다.

Terraform 구성 파일에는 단일 애플리케이션 또는 전체 데이터 센터를 실행하는 데 필요한 구성 요소의 설명이 포함되어 있습니다. Terraform은 원하는 상태에 도달하기 위해 무엇을 실행할지 나타내는 실행 계획을 생성하고, 이를 실행하여 설명된 인프라를 빌드합니다. 구성이 변경되면 Terraform은 변경된 내용을 확인한 후 실행할 증분 실행 계획을 작성할 수 있습니다. 

이 랩에서는 IaC(코드 제공 인프라) 구현을 위해 Azure Pipelines에 Terraform을 통합하는 방법을 배웁니다. 

## 목표

이 랩을 완료하면 다음 작업을 수행할 수 있습니다.

- Terraform을 사용하여 IaC(코드 제공 인프라) 구현
- Terraform 및 Azure Pipelines를 통해 Azure에서 인프라 배포 자동화

## 랩 소요 시간

-   예상 시간: **60분**

## 지침

### 시작하기 전

#### 랩 가상 머신에 로그인

다음 자격 증명을 사용하여 Windows 10 가상 머신에 로그인해야 합니다.
    
-   사용자 이름: **Student**
-   암호: **Pa55w.rd**

#### 이 랩에 필요한 애플리케이션 검토

이 랩에서 사용할 애플리케이션을 확인합니다.
    
-   Microsoft Edge

#### Azure DevOps 조직 설정 

이 랩에 사용할 수 있는 Azure DevOps 조직이 아직 없으면 [조직 또는 프로젝트 컬렉션 만들기](https://docs.microsoft.com/ko-kr/azure/devops/organizations/accounts/create-organization?view=azure-devops)에서 제공되는 지침에 따라 조직을 만듭니다.

#### Azure 구독 준비

-   기존 Azure 구독을 확인하거나 새 구독을 만듭니다.
-   Azure 구독의 소유자 역할, 그리고 Azure 구독과 연결된 Azure AD 테넌트의 전역 관리자 역할이 지정되어 있는 Microsoft 계정 또는 Azure AD 계정이 있는지 확인합니다.

### 연습 0: 랩 필수 구성 요소 구성

이 연습에서는 랩의 필수 구성 요소를 설정합니다. 구체적으로는 Azure DevOps 데모 생성기 템플릿을 기반으로 하여 미리 구성된 Parts Unlimited 팀 프로젝트를 설정합니다. 

#### 작업 1: 팀 프로젝트 구성

이 작업에서는 Azure DevOps 데모 생성기를 사용하여 **Terraform** 템플릿을 기반으로 새 프로젝트를 생성합니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [Azure DevOps 데모 생성기](https://azuredevopsdemogenerator.azurewebsites.net)로 이동합니다. 이 유틸리티 사이트에서는 계정 내에서 새 Azure DevOps 프로젝트를 만드는 프로세스를 자동으로 진행할 수 있습니다. 이 프로젝트에는 랩에 필요한 작업 항목, 리포지토리 등의 콘텐츠가 미리 입력되어 있습니다. 

    > **참고**: 이 사이트에 대한 자세한 내용은 https://docs.microsoft.com/ko-kr/azure/devops/demo-gen을 참조하세요.

1.  **로그인**을 클릭하고 Azure DevOps 구독과 연결된 Microsoft 계정을 사용하여 로그인합니다.
1.  필요한 경우 **Azure DevOps 데모 생성기** 페이지에서 **수락**을 클릭하여 Azure DevOps 구독에 액세스하는 데 필요한 권한 요청을 수락합니다.
1.  **새 프로젝트 만들기** 페이지의 **새 프로젝트 이름** 텍스트 상자에 *Terraform을 통해 인프라 배포 자동화**를 입력합니다. 그런 다음 **조직 선택** 드롭다운 목록에서 Azure DevOps 조직을 선택하고 **템플릿 선택**을 클릭합니다.
1.  템플릿 목록의 도구 모음에서 **DevOps 랩**을 클릭하고 **Terraform** 템플릿을 선택한 다음 **템플릿 선택**을 클릭합니다.
1.  **새 프로젝트 만들기** 페이지로 돌아와서 누락된 확장을 설치하라는 메시지가 표시되면 **토큰 바꾸기** 및 **Terraform** 레이블 아래의 체크박스를 선택하고 **프로젝트 만들기**를 클릭합니다.

    > **참고**: 프로세스가 완료될 때까지 기다립니다. 약 2분이 소요됩니다. 프로세스가 실패하면 DevOps 조직으로 이동하여 프로젝트를 삭제한 후에 다시 시도합니다.

1.  **새 프로젝트 만들기** 페이지에서 **프로젝트로 이동**을 클릭합니다.

### 연습 1: Terraform 및 Azure Pipelines를 통해 클라우드에서 인프라 배포 자동화

이 연습에서는 Terraform과 Azure Pipelines를 사용하여 Azure에 IaC(코드 제공 인프라)를 배포합니다.

#### 작업 1: Terraform 구성 파일 검사

이 작업에서는 Terraform을 사용하여 PartsUnlimited 웹 사이트를 배포하는 데 필요한 Azure 리소스를 프로비전하는 방법을 살펴봅니다.

1.  랩 컴퓨터에서 **Terraform을 통해 인프라 배포 자동화** 프로젝트가 열려 있는 Azure DevOps 포털이 표시된 웹 브라우저 창 내의 Azure DevOps 포털 맨 왼쪽 세로 메뉴 모음에서 **Repos**를 클릭합니다.
1.  **파일** 창에서 맨 위의 **master** 항목 옆에 있는 아래쪽 캐럿을 클릭하고 분기 드롭다운 목록에서 **terraform** 분기를 나타내는 항목을 클릭합니다.

    > **참고**: 현재 위치가 **terraform** 분기이며, 리포지토리 콘텐츠 내에 **Terraform** 폴더가 표시되는지 확인합니다. 

1.  **Terraform** 리포지토리의 폴더 계층 구조에서 **Terraform** 폴더를을 확장하고 **webapp.tf**를 클릭합니다.
1.  **webapp.tf** 파일의 내용을 검토합니다. 

    > **참고**: **webapp.tf**는 terraform 구성 파일입니다. Terraform은 YAML과 비슷한 자체 파일 형식인 HCL(Hashicorp configuration Language)을 사용합니다.

    > **참고**: 이 예제에서는 웹 사이트를 배포하는 데 필요한 Azure 리소스 그룹, App Service 계획 및 App Service를 만듭니다. Terraform 파일은 Azure DevOps 프로젝트의 소스 제어 리포지토리에 추가되어 있으므로 필요한 Azure 리소스를 배포하는 데 사용할 수 있습니다. 
    
#### 작업 2: Azure CI 파이프라인을 사용하여 애플리케이션 빌드

이 작업에서는 애플리케이션을 빌드하고 필수 파일을 drop 아티팩트로 게시합니다.

1.  Azure DevOps 포털 왼쪽의 세로 메뉴 모음에서 **Pipelines**를 클릭하고 그 아래에서 **파이프라인**을 선택합니다.
1.  **파이프라인** 창에서 **Terraform-CI**를 클릭하여 선택하고 **Terraform-CI** 창에서 **편집**을 클릭합니다.

    > **참고**: 이 CI 파이프라인에는 .NET Core 프로젝트를 컴파일하는 작업이 포함되어 있습니다. 파이프라인의 DotNet 작업은 종속성을 복원하고 빌드와 테스트를 진행한 다음 빌드 출력을 zip 파일(패키지)로 게시합니다. 이 파일을 웹 애플리케이션으로 배포할 수 있습니다. 여기서는 애플리케이션 빌드 작업 외에 CD 파이프라인에서 사용할 수 있도록 빌드 아티팩트에 terraform 파일도 게시해야 합니다. 따라서 **파일 복사** 작업을 통해 Terraform 파일을 Artifacts 디렉터리에 복사합니다.

1.  **Terraform-CI** 창의 **작업** 탭을 검토한 후 오른쪽 위의 줄임표 기호를 클릭하고 드롭다운 메뉴에서 **큐**를 클릭합니다.
1.  **파이프라인 실행** 창에서 **실행**을 클릭하여 빌드를 시작합니다. 
1.  빌드 실행 창의 **요약** 탭 **작업** 섹션에서 **에이전트 작업 1**을 클릭하고 빌드 프로세스 진행률을 모니터링합니다.
1.  빌드가 정상적으로 완료되면 빌드 실행 창의 **요약** 탭으로 다시 전환하여 **관련 항목** 섹션에서 **1 게시됨, 1 사용됨** 링크를 클릭합니다. 그러면 **아티팩트** 창이 표시됩니다.
1.  **아티팩트** 창의 **게시됨** 탭에서 **drop** 폴더를 확장하여 **PartsUnlimitedwebsite.zip** 파일이 포함되어 있는지 확인합니다. 그런 다음 **Terraform** 하위 폴더를 확장하여 **webapp.tf** 파일이 포함되어 있는지 확인합니다. 

#### 작업 3: Azure CD 파이프라인에서 Terraform(IaC)을 사용하여 리소스 배포

이 작업에서는 배포 파이프라인의 일부분으로 Terraform을 사용하여 Azure 리소스를 만든 다음, Terraform에서 프로비전한 Azure App Service 웹앱에 PartsUnlimited 애플리케이션을 배포합니다.

1.  Azure DevOps 포털 왼쪽 세로 메뉴 모음의 **Pipelines** 섹션에서 **릴리스**를 클릭하고 **Terraform-CD** 항목이 선택되어 있는지 확인한 후에 **편집**을 클릭합니다.
1.  **모든 파이프라인 > Terraform-CD** 창의 **개발** 단계를 나타내는 사각형에서 **작업 1개, 태스크 8개** 링크를 클릭합니다.
1.  **개발** 단계의 태스크 목록에서 **Azure CLI를 사용하여 필요한 Azure 리소스 배포** 태스크를 선택합니다. 
1.  **Azure CLI** 창의 **Azure 구독** 드롭다운 목록에서 Azure 구독을 나타내는 항목을 선택하고 **권한 부여**를 클릭하여 해당 서비스 연결을 구성합니다. 메시지가 표시되면 Azure 구독의 소유자 역할, 그리고 Azure 구독과 연결된 Azure AD 테넌트의 전역 관리자 역할이 지정되어 있는 계정을 사용하여 로그인합니다.

    > **참고**: 기본적으로 Terraform은 로컬의 terraform.tfstate 파일에 상태를 저장합니다. 그런데 팀에서 Terraform으로 작업할 때 로컬 파일을 사용하면 Terraform 사용 과정이 복잡해집니다. Terraform은 원격 데이터 저장소를 지원하므로 상태를 쉽게 공유할 수 있습니다. 여기서는 **Azure CLI** 작업을 사용해 Terraform 상태를 저장할 Azure 스토리지 계정과 Blob 컨테이너를 만듭니다. Terraform 원격 상태에 대한 자세한 내용은 [Terraform 설명서](https://www.terraform.io/docs/state/remote.html)를 참조하세요.

1.  **개발** 단계의 태스크 목록에서 **Azure PowerShell** 태스크를 선택합니다. 
1.  **Azure PowerShell** 창의 **Azure 연결 형식** 드롭다운 목록에서 **Azure Resource Manager**를 선택하고 **Azure 구독** 드롭다운 목록에서 새로 만든 Azure 서비스 연결을 선택합니다.

    > **참고**: Terraform [백 엔드](https://www.terraform.io/docs/backends/)를 구성하려면 Terraform 상태를 호스트하는 Azure 스토리지 계정의 액세스 키가 필요합니다. 여기서는 Azure PowerShell 태스크를 사용하여 이전 작업에서 프로비전한 Azure 스토리지 계정의 액세스 키를 검색합니다.

1.  **개발** 단계의 태스크 목록에서 **Terraform 파일의 토큰 바꾸기** 태스크를 선택합니다.

    > **참고**: **webapp.tf** 파일을 유심히 검토했다면 `__terraformstorageaccount__`와 같이 **__**가 접두사와 접미사로 사용된 값을 몇 개 확인했을 것입니다. **토큰 바꾸기** 태스크에서는 이러한 값을 릴리스 파이프라인에 정의된 변수 값으로 바꿉니다.
     
1.  Terraform 도구 설치 관리자 작업을 사용하여 인터넷이나 도구 캐시에서 지정된 Terraform 버전을 설치합니다. 그런 다음 Azure Pipelines 에이전트(호스트형 또는 프라이빗)의 PATH 앞에 이 버전을 추가합니다.
1.  **개발** 단계의 태스크 목록에서 **Terraform 설치** 태스크를 선택하여 검토합니다. 이 태스크에서는 지정한 Terraform 버전을 설치합니다.

    > **참고**: Terraform을 자동으로 실행할 때는 대개 핵심 계획/적용 주기를 실행합니다. 이 주기에는 다음의 3개 단계가 포함됩니다.

    1.  Terraform 작업 디렉터리 초기화
    1.  원하는 구성과 일치하도록 현재 구성을 추가하는 계획 생성
    1.  계획 단계에서 확인한 변경 내용 적용
    
    > **참고**: 릴리스 파이프라인의 나머지 Terraform 태스크에서 이 워크플로를 구현합니다.

1.  **개발** 단계의 태스크 목록에서 **Terraform: init** 태스크를 선택합니다. 
1.  **Terraform** 창의 **Azure 구독** 드롭다운 목록에서 이전에 사용한 것과 같은 Azure 서비스 연결을 선택합니다. 
1.  **Terraform** 창의 **컨테이너** 드롭다운 목록에서 **terraform**을 입력하고 **Key** 매개 변수가 **terraform.tfstate**로 설정되어 있는지 확인합니다. 

    > **참고**: `terraform init` 명령은 현재 작업 디렉터리의 모든 *.tf 파일을 구문 분석하고 이러한 파일을 처리하는 데 필요한 모든 공급자를 자동으로 다운로드합니다. 이 예에서는 Azure 리소스를 배포하므로 [Azure 공급자](https://www.terraform.io/docs/providers/azurerm/)를 다운로드합니다. `terraform init` 명령에 대한 자세한 내용은 [Terraform 설명서](https://www.terraform.io/docs/commands/init.html)를 참조하세요.

1.  **개발** 단계의 태스크 목록에서 **Terraform: plan** 태스크를 선택합니다.
1.  Terraform 창의 **Azure 구독** 드롭다운 목록에서 이전에 사용한 것과 같은 Azure 서비스 연결을 선택합니다. 

    > **참고**: `terraform plan` 명령은 실행 계획을 만드는 데 사용됩니다. Terraform은 구성 파일에 지정된 필요한 상태를 구현하는 데 필요한 작업을 확인합니다. 따라서 변경 내용을 실제로 적용하지 않고도 범위 내에 포함된 변경 내용을 검토할 수 있습니다. `terraform plan` 명령에 대한 자세한 내용은 [Terraform 설명서](https://www.terraform.io/docs/commands/plan.html)를 참조하세요.

1.  **개발** 단계의 태스크 목록에서 **Terraform: apply -auto-approve** 태스크를 선택합니다. 
1.  Terraform 창의 **Azure 구독** 드롭다운 목록에서 이전에 사용한 것과 같은 Azure 서비스 연결을 선택합니다. 

    > **참고**: 이 태스크에서는 `terraform apply` 명령을 실행하여 리소스를 배포합니다. 그리고 계속 진행할지를 확인하라는 메시지도 기본적으로 표시됩니다. 여기서는 배포를 자동화할 것이므로 이 태스크에는 사용자가 직접 확인을 할 필요가 없도록 `auto-approve` 매개 변수가 포함됩니다. 

1.  **개발** 단계의 태스크 목록에서 **Azure App Service 배포** 태스크를 선택합니다. 그런 후에 드롭다운 목록에서 Azure 서비스 연결을 선택합니다.

    > **참고**: 이 태스크에서는 이전 단계의 **Terraform: apply -auto-approve** 태스크에서 프로비전한 Azure App Service에 PartsUnlimited 패키지를 배포합니다.

1.  **모든 파이프라인 > Terraform-CD** 창에서 **저장**을 클릭하고 **저장** 대화 상자에서 **확인**을 클릭한 후에 오른쪽 위의 **릴리스 만들기**를 클릭합니다. 
1.  **새 릴리스 만들기** 창의 **트리거가 자동에서 수동으로 변경되는 단계** 드롭다운 목록에서 **개발**을 클릭합니다. 그런 다음 **아티팩트** 섹션의 **버전** 드롭다운 목록에서 이 릴리스용 아티팩트의 버전을 나타내는 항목을 선택하고 **만들기**를 클릭합니다.
1.  Azure DevOps 포털에서 **Terraform-CD** 창으로 다시 이동하여 새로 만든 릴리스를 나타내는 **릴리스-1** 항목을 클릭합니다. 
1.  **Terraform-CD > 릴리스-1** 블레이드에서 **개발** 단계를 나타내는 사각형을 클릭하고 **개발** 창에서 **배포**, **배포**를 차례로 클릭합니다.
1.  **Terraform-CD > 릴리스-1** 블레이드로 돌아와 **개발** 단계를 나타내는 사각형을 클릭하고 배포 프로세스를 모니터링합니다. 
1.  릴리스가 정상적으로 완료되면 랩 컴퓨터에서 다른 웹 브라우저 창을 시작하고 [**Azure Portal**](https://portal.azure.com)로 이동합니다. 그런 다음 이 랩에서 사용할 Azure 구독에서 Contributor 이상의 역할이 지정된 사용자 계정으로 로그인합니다. 
1.  Azure Portal에서 **App Services** 리소스를 검색하여 선택하고 **App Services** 블레이드에서 이름이 **pulterraformweb**으로 시작하는 웹앱으로 이동합니다. 
1.  웹앱 블레이드에서 **찾아보기**를 클릭합니다. 그러면 다른 웹 브라우저 탭이 열리고 새로 배포한 웹 애플리케이션이 표시됩니다.

## 복습

이 랩에서는 Azure Pipelines를 사용하여 Azure에서 Terraform을 통해 반복 가능 배포를 자동화하는 방법을 배웠습니다.
