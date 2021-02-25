---
lab:
    title: '랩: 기능 테스트 설정 및 실행'
    module: '모듈 11: Azure Pipelines를 사용하여 연속 배포 구현'
---

# 랩: 기능 테스트 설정 및 실행
# 학생 랩 매뉴얼

## 랩 개요

웹 애플리케이션용으로 제공되는 이식 가능한 오픈 소스 소프트웨어 테스트 프레임워크인 [Selenium](http://www.seleniumhq.org/)은 거의 모든 운영 체제에서 작동되며 .NET(C#), Java를 포함한 모든 최신 브라우저와 다국어를 지원합니다.

이 랩에서는 C# 웹 애플리케이션에서 Selenium 테스트 케이스를 Azure DevOps 릴리스 파이프라인의 일부분으로 실행하는 방법을 알아봅니다. 

## 목표

이 랩을 완료하면 다음 작업을 수행할 수 있습니다.

- 자체 호스팅 Azure DevOps 에이전트 구성
- 릴리스 파이프라인 구성
- 빌드 및 릴리스 트리거
- Chrome과 Firefox에서 테스트 실행

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
-   Azure 구독의 소유자 역할, 그리고 Azure 구독과 연결된 Azure AD 테넌트의 전역 관리자 역할이 지정되어 있는 Microsoft 계정 또는 Azure AD 계정이 있는지 확인합니다. 자세한 내용은 [Azure Portal을 사용하여 Azure 역할 할당 목록 표시](https://docs.microsoft.com/ko-kr/azure/role-based-access-control/role-assignments-list-portal) 및 [Azure Active Directory에서 관리자 역할 확인 및 할당](https://docs.microsoft.com/ko-kr/azure/active-directory/roles/manage-roles-portal#view-my-roles)을 참조하세요.

### 연습 0: 랩 필수 구성 요소 구성

이 연습에서는 랩의 필수 구성 요소를 설정합니다. 구체적으로는 Azure DevOps 데모 생성기 템플릿과 Azure 리소스를 기반으로 하여 미리 구성된 Parts Unlimited 팀 프로젝트를 설정합니다. 

#### 작업 1: 팀 프로젝트 구성

이 작업에서는 Azure DevOps 데모 생성기를 사용하여 **Parts Unlimited** 템플릿을 기반으로 새 프로젝트를 생성합니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [Azure DevOps 데모 생성기](https://azuredevopsdemogenerator.azurewebsites.net)로 이동합니다. 이 유틸리티 사이트에서는 계정 내에서 새 Azure DevOps 프로젝트를 만드는 프로세스를 자동으로 진행할 수 있습니다. 이 프로젝트에는 랩에 필요한 작업 항목, 리포지토리 등의 콘텐츠가 미리 입력되어 있습니다. 

    > **참고**: 이 사이트에 대한 자세한 내용은 https://docs.microsoft.com/ko-kr/azure/devops/demo-gen을 참조하세요.

1.  **로그인**을 클릭하고 Azure DevOps 구독과 연결된 Microsoft 계정을 사용하여 로그인합니다.
1.  필요한 경우 **Azure DevOps 데모 생성기** 페이지에서 **수락**을 클릭하여 Azure DevOps 구독에 액세스하는 데 필요한 권한 요청을 수락합니다.
1.  **새 프로젝트 만들기** 페이지의 **새 프로젝트 이름** 텍스트 상자에 **기능 테스트 설정 및 실행**을 입력합니다. 그런 다음 **조직 선택** 드롭다운 목록에서 Azure DevOps 조직을 선택하고 **템플릿 선택**을 클릭합니다.
1.  템플릿 목록의 도구 모음에서 **DevOps 랩**을 클릭하고 **Selenium** 템플릿을 선택한 다음 **템플릿 선택**을 클릭합니다.
1.  **새 프로젝트 만들기** 페이지로 돌아와서 **프로젝트 만들기**를 클릭합니다.

    > **참고**: 프로세스가 완료될 때까지 기다립니다. 약 2분이 소요됩니다. 프로세스가 실패하면 DevOps 조직으로 이동하여 프로젝트를 삭제한 후에 다시 시도합니다.

1.  **새 프로젝트 만들기** 페이지에서 **프로젝트로 이동**을 클릭합니다.

#### 작업 2: Azure 리소스 만들기

이 작업에서는 Windows Server 2016과 SQL Express 2017, Chrome, Firefox를 실행하는 Azure VM을 프로비전합니다.

1.  아래의 Azure에 배포 단추를 클릭합니다. [![Azure에 배포](http://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2Falmvm%2Fmaster%2Flabs%2Fvstsextend%2Fselenium%2Farmtemplate%2Fazuredeploy.json). 그러면 Azure Portal의 **사용자 지정 배포** 블레이드로 자동 리디렉션됩니다.
1.  메시지가 표시되면 이 랩에서 사용할 Azure 구독의 소유자 역할, 그리고 해당 구독과 연결된 Azure AD 테넌트의 전역 관리자 역할이 지정된 사용자 계정으로 로그인합니다.
1.  **사용자 지정 배포** 블레이드에서 다음 설정을 지정합니다.

    | 설정 | 값 |
    | --- | --- |
    | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
    | 리소스 그룹 | 새 리소스 그룹의 이름 **az400m11l02-RG** |
    | 지역 | 이 랩에서 Azure 리소스를 배포할 Azure 지역의 이름 |
    | 가상 머신 이름 | **az40011bvm** |

1.  **검토 + 만들기**를 클릭한 다음, **만들기**를 클릭합니다. 

    > **참고**: 프로세스가 완료될 때까지 기다립니다. 약 15분이 소요됩니다. 

### 연습 1: 자체 호스팅 Azure DevOps 에이전트를 사용하여 Selenium 테스트 구현

이 연습에서는 자체 호스팅 Azure DevOps 에이전트를 사용하여 Selenium 테스트를 구현합니다.

#### 작업 1: 자체 호스팅 Azure DevOps 에이전트 구성

이 작업에서는 이전 연습에서 배포한 VM을 사용하여 자체 호스팅 에이전트를 구성합니다. Selenium에서 UI 테스트를 실행하려면 대화형 모드에서 에이전트를 실행해야 합니다.

1.  Azure Portal이 표시된 웹 브라우저 창에서 **가상 머신**을 검색하여 선택하고 **가상 머신** 블레이드에서 **az40011bvm**을 선택합니다.
1.  **az40011bvm** 블레이드에서 **연결**을 선택하고 드롭다운 메뉴에서 **RDP**를 선택합니다. 그런 다음 **az40011bvm \| 연결** 블레이드의 PDF  탭에서 **RDP 파일 다운로드**를 선택하고 다운로드한 파일을 엽니다.
1.  메시지가 표시되면 다음 자격 증명으로 로그인합니다.

    | 설정 | 값 |
    | --- | --- |
    | 사용자 이름 | **vmadmin** |
    | 암호 | **P2ssw0rd@123** |

1.  **az40011bvm**에 연결된 원격 데스크톱 세션 내에서 Chrome 웹 브라우저 창을 시작하고 **https://dev.azure.com** 으로 이동한 다음 Azure DevOps 조직에 로그인합니다. 
1.  **Azure DevOps** 포털 왼쪽 아래의 **조직 설정**을 클릭합니다.
1.  페이지 왼쪽의 세로 메뉴에 있는 **Pipelines** 섹션에서 **에이전트 풀**을 클릭합니다.
1.  **에이전트 풀** 창에서 **기본값**을 클릭합니다. 
1.  **기본값** 창에서 **새 에이전트**를 클릭합니다. 
1.  **에이전트 가져오기** 패널에서 **Windows** 탭과 **x64** 섹션이 선택되어 있는지 확인하고 **다운로드**를 클릭합니다.
1.  파일 탐색기를 시작하여 **C:\\AzAgent** 디렉터리를 만든 후 **Downloads** 폴더에 있는 다운로드한 에이전트 zip 파일의 압축을 풀어 해당 내용을 이 디렉터리에 저장합니다.
1.  **az40011bvm**에 연결된 원격 데스크톱 세션 내에서 **시작** 메뉴를 마우스 오른쪽 단추로 클릭하고 **명령 프롬프트(관리자)** 를 클릭합니다.
1.  **관리자: 명령 프롬프트** 창에서 다음 명령을 실행하여 에이전트 이진 파일 설치를 시작합니다.

    ```cmd
    cd C:\AzAgent
    Config.cmd
    ```
1.  **관리자: 명령 프롬프트** 창에서 **서버 URL 입력** 메시지가 표시되면 **https://dev.azure.com/<your-DevOpsYour-organization-name>** 을 입력합니다. 여기서e **<your-DevOpsYour-organization-name>** 은 Azure DevOps 조직의 이름을 나타냅니다. URL을 입력한 후에 **Enter** 키를 누릅니다.
1.  **관리자: 명령 프롬프트** 창에서 **인증 유형 입력(PAT의 경우 Enter 키 누르기)** 메시지가 표시되면 **Enter** 키를 누릅니다.
1.  **관리자: 명령 프롬프트** 창에서 **개인용 액세스 토큰 입력** 메시지가 표시되면 Azure DevOps 포털로 전환하여 **에이전트 가져오기** 패널을 닫고 Azure DevOps 페이지 오른쪽 위의 **사용자 설정** 아이콘을 클릭합니다. 그런 다음 드롭다운 메뉴에서 **개인용 액세스 토큰**을 클릭하고 **개인용 액세스 토큰** 창에서 **새 토큰**을 클릭합니다.
1.  **새 개인용 액세스 토큰 만들기** 창에서 다음 설정을 지정한 후에 **만들기**를 클릭합니다. 다른 값은 모두 기본값으로 유지합니다.

    | 설정 | 값 |
    | --- | --- |
    | 이름 | **기능 테스트 설정 및 실행 랩** |
    | 범위 | **모든 권한** |

1.  **성공** 창에서 개인용 액세스 토큰의 값을 클립보드에 붙여넣습니다.

    > **참고**: 지금 토큰을 복사해야 합니다. 이 창을 닫으면 토큰을 검색할 수 없습니다. 

1.  **성공** 창에서 **닫기**를 클릭합니다.
1.  **관리자: 명령 프롬프트** 창으로 다시 전환하여 클립보드의 내용을 붙여넣고 **Enter** 키를 누릅니다.
1.  **관리자: 명령 프롬프트** 창에서 **에이전트 풀 입력(기본값을 사용하려면 Enter 키 누르기)** 메시지가 표시되면 **Enter** 키를 누릅니다.
1.  **관리자: 명령 프롬프트** 창에서 **에이전트 이름 입력(az40011bvm을 사용하려면 Enter 키 누르기)** 메시지가 표시되면 **Enter** 키를 누릅니다.
1.  **관리자: 명령 프롬프트** 창에서 **작업 폴더 입력(_work를 사용하려면 Enter 키 누르기)** 메시지가 표시되면 **Enter** 키를 누릅니다.
1.  **관리자: 명령 프롬프트** 창에서 **각 단계의 작업에서 압축 풀기 수행 여부 입력(N을 입력하려면 Enter 키 누르기)** 메시지가 표시되면 **Enter** 키를 누릅니다.
1.  **관리자: 명령 프롬프트** 창에서 **에이전트를 서비스로 실행할지 여부 입력(Y/N)(N을 입력하려면 Enter 키 누르기)** 메시지가 표시되면 **Enter** 키를 누릅니다.
1.  **관리자: 명령 프롬프트** 창에서 **자동 로그온 구성 및 시작 시 에이전트 실행 여부 입력(Y/N)(N을 입력하려면 Enter 키 누르기)** 메시지가 표시되면 **Enter** 키를 누릅니다.
1.  에이전트가 등록되면 **관리자: 명령 프롬프트** 창에 **run.cmd**를 입력하고 **Enter** 키를 눌러 에이전트를 시작합니다.

    > **참고**: 랩 뒷부분에서 배포할 애플리케이션에 사용되는 Dac Framework도 설치해야 합니다.

1.  **az40011bvm**에 연결된 원격 데스크톱 세션 내에서 웹 브라우저를 시작하고 [Microsoft SQL Server Data-Tier Application Framework(18.2) 다운로드 페이지](https://www.microsoft.com/ko-kr/download/details.aspx?id=58207&WT.mc_id=rss_alldownloads_extensions)로 이동합니다. 그러면 다운로드가 자동으로 트리거됩니다.
1.  **DacFramework.msi** 파일 다운로드가 완료되면 이 파일을 사용해 기본 설정으로 Microsoft SQL Server Data-Tier Application Framework 설치를 실행합니다.

#### 작업 2: 릴리스 파이프라인 구성

이 작업에서는 릴리스 파이프라인을 구성합니다.

> **참고**: Azure VM에는 애플리케이션을 배포하고 Selenium 테스트 케이스를 실행하도록 구성된 에이전트가 있습니다. 릴리스 정의는 **[단계](https://docs.microsoft.com/ko-kr/vsts/build-release/concepts/process/phases)** 를 사용하여 대상 서버로의 배포를 진행합니다.

1.  **az40011bvm**에 연결된 원격 데스크톱 세션 내의 **Azure DevOps** 포털이 표시된 브라우저 창에서 왼쪽 위의 **Azure DevOps** 기호를 클릭합니다.
1.  조직 프로젝트가 표시된 창에서 **기능 테스트 설정 및 실행** 프로젝트를 나타내는 타일을 클릭합니다.
1.  **기능 테스트 설정 및 실행** 창의 세로 탐색 창에서 **Pipelines**를 선택하고 **파이프라인** 섹션 내에서 **릴리스**를 클릭한 다음 **Selenium** 창에서 **편집**을 클릭합니다.
1.  **모든 파이프라인 > Selenium** 창에서 **작업** 탭 머리글을 선택하고 드롭다운 메뉴에서 **개발**을 클릭합니다.
1.  **개발** 단계의 작업 목록에서 **IIS 배포**, **SQL 배포** 및 **Selenium 테스트 실행** 배포 단계를 검토합니다. 

- **IIS 배포 단계**: 이 단계에서는 다음 작업을 통해 VM에 애플리케이션을 배포합니다.

   - **IIS 웹앱 관리**: 에이전트를 등록한 대상 컴퓨터에서 실행되는 이 작업에서는 *웹 사이트*와 *애플리케이션 풀*이 로컬에 작성됩니다. 웹 사이트와 애플리케이션 풀의 이름은 **PartsUnlimited**이며 포트 **82**([**http://localhost:82**](http://localhost:82))에서 실행됩니다.
   - **IIS 웹앱 배포**: 이 작업에서는 **웹 배포**를 사용해 IIS 서버에 애플리케이션을 배포합니다.

- **데이터베이스 배포 단계**: 이 단계에서는 [**SQL Server 데이터베이스 배포**](https://github.com/Microsoft/vsts-tasks/blob/master/Tasks/SqlDacpacDeploymentOnMachineGroup/README.md) 작업을 사용하여 DB 서버에 [**dacpac**](https://docs.microsoft.com/ko-kr/sql/relational-databases/data-tier-applications/data-tier-applications) 파일을 배포합니다.

- **Selenium 테스트 실행**: 릴리스 프로세스의 일부분으로 **UI 테스트**를 실행하면 예기치 않은 변경을 검색할 수 있습니다. 자동화된 브라우저 기반 테스트를 설정하면 수동으로 테스트를 수행하지 않아도 애플리케이션의 품질을 개선할 수 있습니다. 이 단계에서는 배포된 웹 애플리케이션에서 Selenium 테스트를 실행합니다. 후속 작업에서는 Selenium을 사용하여 릴리스 파이프라인에서 웹 사이트를 테스트하는 과정을 설명합니다.

  - **Visual Studio 테스트 플랫폼 설치 관리자**: [Visual Studio 테스트 플랫폼 설치 관리자](https://docs.microsoft.com/ko-kr/azure/devops/pipelines/tasks/tool/vstest-platform-tool-installer?view=vsts) 작업에서는 nuget.org 또는 지정된 피드에서 Microsoft 테스트 플랫폼을 가져온 다음 도구 캐시에 추가합니다. 이 설치 관리자는 **vstest** 요구 사항을 충족하므로 에이전트 컴퓨터에 정품 Visual Studio를 설치하지 않아도 빌드 또는 릴리스 파이프라인 내의 모든 후속 Visual Studio 테스트 작업을 실행할 수 있습니다.
  - **Selenium UI 테스트 실행**: 이 [작업](https://github.com/Microsoft/azure-pipelines-tasks/blob/master/Tasks/VsTestV2/README.md)에서는 **vstest.console.exe**를 사용하여 에이전트 컴퓨터에서 Selenium 테스트 케이스를 실행합니다.

1.  **모든 파이프라인 > Selenium** 창에서 **IIS 배포** 단계를 클릭하고 **에이전트 작업** 창에서 **기본** 에이전트 풀이 선택되어 있는지 확인합니다. 
1.  **SQL 배포** 및 **Selenium 테스트 실행** 단계를 대상으로 이전 단계를 반복합니다. 필요한 경우 **저장**을 클릭하여 변경 내용을 저장합니다.

#### 작업 3: 빌드 및 릴리스 트리거

이 작업에서는 **빌드**를 트리거하여 Selenium C# 스크립트와 웹 애플리케이션을 컴파일합니다. 그러면 생성되는 이진 파일은 자체 호스팅 에이전트에 복사되며, Selenium 스크립트는 자동화된 **릴리스**의 일부분으로 실행됩니다.

1.  **az40011bvm**에 연결된 원격 데스크톱 내에서 **Azure DevOps** 포털이 표시된 브라우저 창의 세로 탐색 창에 있는 **Pipelines** 섹션에서 **파이프라인**을 클릭하고 **파이프라인** 창에서 **Selenium**을 클릭합니다.
1.  **Selenium** 창에서 **파이프라인 실행**을 클릭하고 **파이프라인 실행**에서 **실행**을 클릭합니다.

    > **참고**: 이 빌드에서는 Azure DevOps에 테스트 아티팩트를 게시합니다. 게시된 아티팩트는 릴리스에서 사용됩니다. 

    > **참고**: 빌드가 정상적으로 완료되면 릴리스가 트리거됩니다. 

1.  파이프라인 실행 창의 **작업** 섹션에서 **1단계**를 클릭하고 빌드가 완료될 때까지 진행률을 모니터링합니다.
1.  **Azure DevOps** 포털이 표시된 브라우저 창의 세로 탐색 창에 있는 **Pipelines** 섹션에서 **릴리스**를 클릭합니다. 그런 다음 릴리스를 나타내는 항목을 클릭하고 **Selenium > 릴리스-1** 창에서 **개발**을 클릭합니다.
1.  **Selenium > 릴리스-1 > 개발** 창에서 해당 배포를 모니터링합니다.
1.  **Selenium 테스트 실행** 단계가 시작되면 웹 브라우저 테스트를 모니터링합니다. 
1.  릴리스가 완료되면 **Selenium > 릴리스-1 > 개발** 창에서 **테스트** 탭을 클릭하여 테스트 결과를 분석합니다. **결과** 섹션의 드롭다운에서 필요한 필터를 선택하여 테스트 및 해당 상태를 확인합니다.

### 연습 2: Azure 랩 리소스 제거

이 연습에서는 예상치 못한 비용이 발생하지 않도록 이 랩에서 프로비전한 Azure 리소스를 제거합니다. 

>**참고**: 더 이상 사용하지 않는 새로 만든 Azure 리소스를 제거해야 합니다. 사용하지 않는 리소스를 제거하면 예기치 않은 비용이 발생하지 않습니다.

#### 작업 1: Azure 랩 리소스 제거

이 작업에서는 Azure Cloud Shell을 사용하여 불필요한 비용이 발생하지 않도록 이 랩에서 프로비전한 Azure 리소스를 제거합니다. 

1.  Azure Portal의 **Cloud Shell** 창에서 **Bash** 세션을 시작합니다.
1.  다음 명령을 실행하여 이 모듈의 전체 랩에서 생성된 모든 리소스 그룹을 나열합니다.

    ```sh
    az group list --query "[?starts_with(name,'az400m11l02-RG')].name" --output tsv
    ```

1.  다음 명령을 실행하여 이 모듈의 전체 랩에서 만든 모든 리소스 그룹을 삭제합니다.

    ```sh
    az group list --query "[?starts_with(name,'az400m11l02-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**참고**: 명령은 비동기적으로 실행되므로(--nowait 매개 변수에 의해 결정됨) 동일한 Bash 세션 내에서 즉시 다른 Azure CLI 명령을 실행할 수 있지만 리소스 그룹이 실제로 제거되기까지 몇 분 정도 걸립니다.


## 복습

이 랩에서는 C# 웹 애플리케이션에서 Selenium 테스트 케이스를 Azure DevOps 릴리스 파이프라인의 일부분으로 실행하는 방법을 알아보았습니다.
