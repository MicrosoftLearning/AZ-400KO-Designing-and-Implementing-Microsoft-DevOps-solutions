---
lab:
    title: '랩: Azure DevOps와 Teams 통합'
    module: '모듈 18: 시스템 피드백 메커니즘 구현'
---

# 랩: Azure DevOps와 팀 간의 통합
# 학생 랩 매뉴얼

## 랩 개요

**[Microsoft Teams](https://teams.microsoft.com/start)** 는 Office 365의 팀워크를 위한 허브입니다. Microsoft Teams에서는 팀의 모든 채팅, 모임, 파일, 앱을 한 곳에서 관리하고 사용할 수 있습니다. 또한 Office 365 및 Azure DevOps 전반에서 다양한 팀, 대화, 콘텐츠 및 도구를 활용할 수 있는 허브를 소프트웨어 개발 팀에 제공합니다.

이 랩에서는 Azure DevOps Services와 Microsoft Teams 간의 통합 시나리오를 구현합니다.

> **참고**: **Azure DevOps Services**와 Microsoft Teams를 통합하면 개발 주기 전반에서 활용할 수 있는 포괄적인 채팅 및 공동 작업 환경이 제공됩니다. 그러므로 개발 팀은 작업 항목, 끌어오기 요청, 코드 커밋, 빌드 및 릴리스 이벤트 등과 관련한 알림과 경고를 통해 Azure DevOps 팀 프로젝트에서 진행되는 중요한 활동 관련 정보를 지속적으로 파악할 수 있습니다.

## 목표

이 랩을 완료하면 다음 작업을 수행할 수 있습니다.

- Azure DevOps와 Microsoft Teams 통합
- Teams의 대시보드와 Azure DevOps Kanban 보드 통합
- Azure Pipelines와 Microsoft Teams 통합
- Microsoft Teams에서 Azure Pipelines 앱 설치
- Azure Pipelines 알림 구독

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
-   Microsoft Teams. 이 랩의 필수 구성 요소로 설치됩니다.

#### Office 365 구독 설정 

[Microsoft Teams 등록 페이지](https://teams.microsoft.com/start)에서 무료 평가판 구독을 만듭니다. 

#### Azure DevOps 조직 설정 

[조직 또는 프로젝트 컬렉션 만들기](https://docs.microsoft.com/ko-kr/azure/devops/organizations/accounts/create-organization?view=azure-devops)에서 제공되는 지침을 따르세요. Azure DevOps 조직을 만들 때는 Office 365 구독을 설정할 때 사용한 것과 같은 사용자 계정으로 로그인합니다.

> **참고**: Office 365 구독과 Azure DevOps 조직은 같은 Azure Active Directory(Azure AD) 테넌트를 공유해야 합니다.

### 연습 0: 랩 필수 구성 요소 구성

이 연습에서는 랩의 필수 구성 요소를 설정합니다. 구체적으로는 Azure DevOps 데모 생성기 템플릿과 Microsoft Teams에서 만든 팀을 기반으로 하여 미리 구성된 **Tailwind Traders** 팀 프로젝트를 설정합니다.

#### 작업 1: 팀 프로젝트 구성

이 작업에서는 Azure DevOps 데모 생성기를 사용하여 **Tailwind Traders** 템플릿을 기반으로 새 프로젝트를 생성합니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [Azure DevOps 데모 생성기](https://azuredevopsdemogenerator.azurewebsites.net)로 이동합니다. 이 유틸리티 사이트에서는 계정 내에서 새 Azure DevOps 프로젝트를 만드는 프로세스를 자동으로 진행할 수 있습니다. 이 프로젝트에는 랩에 필요한 작업 항목, 리포지토리 등의 콘텐츠가 미리 입력되어 있습니다. 

    > **참고**: 이 사이트에 대한 자세한 내용은 https://docs.microsoft.com/ko-kr/azure/devops/demo-gen을 참조하세요.

1.  **로그인**을 클릭하고 Azure DevOps 구독과 연결된 Microsoft 계정을 사용하여 로그인합니다.
1.  필요한 경우 **Azure DevOps 데모 생성기** 페이지에서 **수락**을 클릭하여 Azure DevOps 구독에 액세스하는 데 필요한 권한 요청을 수락합니다.
1.  **새 프로젝트 만들기** 페이지의 **새 프로젝트 이름** 텍스트 상자에 **Tailwind Traders**를 입력합니다. 그런 다음 **조직 선택** 드롭다운 목록에서 Azure DevOps 조직을 선택하고 **템플릿 선택**을 클릭합니다.
1.  템플릿 목록에서 **Tailwind Traders** 템플릿을 선택한 다음 **템플릿 선택**을 클릭합니다.
1.  **새 프로젝트 만들기** 페이지로 돌아와서 누락된 확장을 설치하라는 메시지가 표시되면 **ARM 출력** 레이블 아래의 체크박스를 선택하고 **프로젝트 만들기**를 클릭합니다.

    > **참고**: 프로세스가 완료될 때까지 기다립니다. 약 2분이 소요됩니다. 프로세스가 실패하면 DevOps 조직으로 이동하여 프로젝트를 삭제한 후에 다시 시도합니다.

1.  **새 프로젝트 만들기** 페이지에서 **프로젝트로 이동**을 클릭합니다.

#### 작업 2: Microsoft Teams에서 팀 만들기

이 작업에서는 Microsoft Teams에서 팀을 만듭니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [Microsoft Teams 다운로드 페이지](https://www.microsoft.com/ko-kr/microsoft-365/microsoft-teams/download-app)로 이동하여 Microsoft Teams 다운로드한 다음 기본 설정으로 설치합니다. 
1.  랩 컴퓨터에서 데스크톱 앱을 사용해 **Microsoft Teams**를 시작합니다.

    > **참고**: 웹 브라우저를 통해 [Microsoft Teams 시작 페이지](https://teams.microsoft.com/dl/launcher/launcher.html?url=/_%23/l/home/0/0&type=home)로 이동할 수도 있습니다.

1.  로그인하라는 메시지가 표시되면 Azure DevOps 조직 액세스 권한이 있으며 Office 365 구독에 포함된 사용자 계정으로 로그인합니다.
1.  Microsoft Teams 페이지 왼쪽의 도구 모음에서 **팀**을 클릭하고 팀 목록 아래쪽에서 **팀 가입 또는 만들기**를 클릭합니다.

    >**참고**: 팀은 공통의 목표를 달성하기 위해 함께 작업을 하는 사용자 집합입니다. 

1.  **팀 가입 또는 만들기** 창에서 **팀 만들기**를 클릭합니다.
1.  **팀 만들기** 패널에서 **처음부터**를 클릭하고 **팀 종류** 패널에서 **비공개**를 클릭합니다.
1.  **비공개 팀에 관한 일부 간단한 세부 정보** 패널에서 **팀 이름 지정**을 **Tailwind Traders**로 바꾸고 **만들기**를 클릭합니다.
1.  **Tailwind Traders에 구성원 추가** 패널에서 **건너뛰기**를 클릭합니다.

### 연습 1: Azure Boards와 Microsoft Teams 통합

이 연습에서는 Azure Boards와 Microsoft Teams 간의 통합을 구현합니다.

#### 작업 1: Microsoft Teams에서 Azure Boards 앱 설치 및 구성

이 작업에서는 Microsoft Teams에서 새로 만든 팀에서 Azure Boards 앱을 설치하고 구성합니다.

1.  Microsoft Teams 창 왼쪽 아래에서 **앱** 아이콘을 클릭합니다. 그러면 **앱** 창이 열립니다.
1.  **앱** 창의 **모든 앱 검색** 텍스트 상자에 **Azure Boards**를 입력하고 앱 목록에서 **Azure Boards**를 클릭합니다.
1.  **Azure Boards** 패널에서 **추가**를 클릭합니다.
1.  Microsoft Teams 창 위쪽의 **Azure Boards** 드롭다운 목록에서 **작업 항목 검색**을 클릭합니다. 그러면 **Azure Boards** 창이 표시됩니다.
1.  **Azure Boards** 패널에서 **로그인** 링크를 클릭합니다.
1.  **작업 항목(전체)**, **프로젝트 및 팀(읽기)**, **Teams 통합** 권한을 부여하라는 메시지가 표시되면 **Azure DevOps를 통한 Azure Boards Microsoft Teams 통합** 대화 상자에서 **수락**을 클릭합니다.
1.  **Azure Boards** 패널로 돌아와서 **설정** 링크를 클릭합니다. 그러면 **Microsoft Azure DevOps Services - 프로필 1** 창이 표시됩니다.
1.  **Microsoft Azure DevOps Services - 프로필 1** 창의 **조직** 드롭다운 목록에서 Azure DevOps 조직을 선택하고 **계속**을 클릭합니다.
1.  **Microsoft Azure DevOps Services - 프로필 1** 창의 **프로젝트** 드롭다운 목록에서 **Tailwind Traders**를 선택하고 **계속**을 클릭합니다.
 
    >**참고**: 그러면 **Tailwind Traders** Azure DevOps 프로젝트의 기존 작업 항목 목록이 표시됩니다.

1.  작업 항목 목록을 스크롤하여 아무 항목이나 선택한 다음 작업 항목 이름을 클릭합니다. 그러면 해당 작업 항목을 열 앱을 선택하라는 메시지가 표시됩니다. 앱 목록에서 Microsoft Edge를 선택하고 **확인**을 클릭합니다. 그러면 새 웹 브라우저 창이 자동으로 열리고 Azure DevOps 포털에 작업 항목 세부 정보가 표시됩니다.
1.  새 브라우저 창을 닫고 Microsoft Teams로 돌아옵니다.

    >**참고**: **+ 작업 항목 만들기** 옵션을 선택하여 Microsoft Teams 인터페이스에서 바로 작업 항목을 만들 수도 있습니다.


1.  **Azure Boards** 패널로 돌아와 **열기** 단추 바로 오른쪽의 아래쪽 화살표를 클릭하고 드롭다운 목록에서 **팀에 추가** 항목을 선택합니다.
1.  **팀에 대해 Azure Boards 설정** 패널의 **검색** 텍스트 상자에 **Tailwind Traders**를 입력하고 결과 목록에서 **Tailwind Traders > 일반** 항목을 선택한 후에 **봇 설정**을 클릭합니다.
1.  **Tailwind Traders** 팀의 **일반** 채널 게시물 목록에서 봇이 게시한 다음과 같은 메시지를 검토합니다.

    ```
    Sign in to your Azure Boards account with: @Azure Boards signin
    To see what else you can do, type @Azure Boards help
    ```
1.  **Tailwind Traders** 팀의 **일반** 채널 게시물 목록에서 제목이 **Azure Boards**인 게시물을 선택하고 **Enter** 키를 누른 다음 봇이 게시한 추가 메시지를 검토합니다.

    ```
    link [project url] - Link to a project to create work items and receive notifications
    subscriptions - Add or remove subscriptions for this channel
    addAreapath [area path] - Add an area path from your project to this channel
    signin - Sign in to your Azure Boards account
    signout - Sign out from your Azure Boards account
    unlink - Unlink a project from this channel
    feedback - Report a problem or suggest a feature
    ```

#### 작업 2: Microsoft Teams에 Azure Boards Kanban 보드 추가

이 작업에서는 Microsoft Teams의 탭에 Azure Boards Kanban 보드를 추가합니다.

> **참고**: Kanban 보드는 백로그를 대화형 간판으로 변환하여 작업 흐름을 시각적으로 제공합니다. 아이디어 구상에서 제품 완성까지의 작업을 진행하는 과정에서 보드의 항목을 업데이트합니다. 각 열은 작업 단계를 나타내며 각 카드는 해당 작업 단계의 사용자 스토리(파란색 카드) 또는 버그(빨간색 카드)를 나타냅니다. 탭을 사용하면 Teams Kanban 보드 또는 즐겨 사용하는 대시보드를 Microsoft Teams에 바로 추가할 수 있습니다. 팀 구성원은 **탭**을 통해 사용자의 개인 앱 공간이나 채널 내의 전용 캔버스에서 서비스에 액세스할 수 있습니다. 기존 웹앱을 활용하여 Teams 내에서 유용한 탭 환경을 만들 수 있습니다.

1.  랩 컴퓨터에서 Azure DevOps 포털의 **Tailwind Traders** 프로젝트가 표시된 웹 브라우저로 전환하여 Azure DevOps 포털 맨 왼쪽의 세로 메뉴 모음에 있는 **Boards**를 클릭하고 **보드** 섹션에서 **Boards**를 클릭합니다.
1.  **Boards** 창의 **내 팀 보드(1)** 섹션에서 **Tailwind Traders 팀 보드** 항목을 클릭합니다. 
1.  **Tailwind Traders 팀** 창을 표시한 상태로 웹 브라우저 창에서 이 보드의 URL을 클립보드에 복사합니다.
1.  Microsoft Teams 창으로 전환하여 새로 만든 팀인 **Tailwind Traders**의 **일반** 채널이 선택되어 있는지 확인하고 **일반** 창 위쪽 섹션에서 더하기 기호를 클릭합니다. 그러면 **탭 추가** 패널이 표시됩니다.
1.  **탭 추가** 패널에서 **웹 사이트**를 클릭하고 **웹 사이트** 패널에서 **탭 이름**은 **Tailwind Traders 팀 보드**로, **URL**은 방금 클립보드에 복사한 URL로 설정한 후에 **저장**을 클릭합니다.
1.  Microsoft Teams 창에서 **Tailwind Traders** 팀의 **일반** 채널을 선택한 상태로 위쪽 메뉴의 탭 목록에서 새로 추가한 **Tailwind Traders 팀 보드** 탭을 클릭합니다. 그리고 Azure DevOps 포털에서 사용 가능한 **Tailwind Traders 팀** 보드와 일치하는 콘텐츠가 이 탭에 포함되어 있는지 확인합니다.

> **참고**: 일일 스탠드업 회의에서 모든 작업을 모니터링할 수 있으며 해당하는 작업 항목 상태가 변경될 때마다 업데이트는 실시간으로 반영됩니다. Microsoft Teams에서 Kanban 보드를 수정할 수도 있습니다.

### 연습 2: Azure Pipelines와 Microsoft Teams 통합

이 연습에서는 Azure Pipelines와 Microsoft Teams 간의 통합을 구현합니다.

#### 작업 1: Microsoft Teams에서 Azure Pipelines 앱 설치 및 구성

이 작업에서는 Microsoft Teams에서 지정한 팀에서 Azure Pipelines 앱을 설치하고 구성합니다.

> **참고**: Microsoft Teams에서 Azure Pipelines 앱을 설치하면 파이프라인의 이벤트를 모니터링할 수 있습니다. 릴리스, 보류 중인 승인, 완료된 빌드와 같은 이벤트용으로 구독을 설정하여 관리할 수 있습니다. 그러면 Teams 채널에 알림이 바로 게시됩니다. Teams 채널 내에서 릴리스를 승인할 수도 있습니다.

1.  Microsoft Teams 창 왼쪽 아래에서 **앱** 아이콘을 클릭합니다. 그러면 **앱** 창이 열립니다.
1.  **앱** 창의 **모든 앱 검색** 텍스트 상자에 **Azure Pipelines**를 입력하고 앱 목록에서 **Azure Pipelines**를 클릭합니다.
1.  **Azure Pipelines** 패널에서 **열기** 단추 바로 오른쪽의 아래쪽 화살표를 클릭하고 드롭다운 목록에서 **팀에 추가** 항목을 선택합니다.
1.  **팀에 대해 Azure Pipelines 설정** 패널의 **검색** 텍스트 상자에 **Tailwind Traders**를 입력하고 결과 목록에서 **Tailwind Traders > 일반** 항목을 선택한 후에 **봇 설정**을 클릭합니다. 그러면 **Tailwind Traders** 팀 **일반** 채널의 게시물 보기로 자동 리디렉션됩니다.
1.  **Tailwind Traders** 팀의 **일반** 채널 게시물 목록에서 봇이 게시한 다음과 같은 메시지를 검토합니다.

    ```
    Subscribe to one or more pipelines or all pipelines in a project with: @Azure Pipelines subscribe [pipeline url/ project url]
    To see what else you can do, type @Azure Pipelines help
    ```

1.  **Tailwind Traders** 팀의 **일반** 채널 게시물 목록에서 제목이 **Azure Pipelines**인 게시물을 선택하고 **Enter** 키를 누른 다음 봇이 게시한 추가 메시지를 검토합니다.

    ```
    subscribe [pipeline url/ project url] - Subscribe to a pipeline or all pipelines in a project to receive notifications
    subscriptions - Add or remove subscriptions for this channel
    feedback - Report a problem or suggest a feature
    signin - Sign in to your Azure Pipelines account
    signout - Sign out from your Azure Pipelines account
    ```
   
#### 작업 2: Microsoft Teams에서 Azure Pipelines 알림 구독

이 작업에서는 Microsoft Teams에서 Azure Pipelines 알림을 구독합니다.

> **참고**: `@Azure Pipelines` 핸들을 사용하여 앱과의 상호 작용을 시작할 수 있습니다.

1.  **게시물** 탭을 선택한 상태로 **Tailwind Traders** 팀의 **일반** 채널에 `@Azure Pipelines signin`을 게시해 인증을 진행합니다. 메시지가 표시되면 **로그인**을 클릭합니다.
1.  **Azure Pipelines 로그인** 창에서 **로그인**을 클릭합니다.
1.  **서비스 후크(읽기 및 쓰기)**, **빌드(빌드 및 실행)**, **릴리스(읽기, 쓰기, 실행 및 관리)**, **프로젝트 및 팀(읽기)**, **ID 선택(읽기)** 및 **Teams 통합** 권한을 부여하라는 메시지가 표시되면 **수락**을 클릭하고 **닫기**를 클릭합니다.

    >**참고**: 이제 `@azure pipelines subscribe [pipeline url]` 명령을 사용하여 Azure DevOps 파이프라인을 구독할 수 있습니다.

1.  랩 컴퓨터에서 Azure DevOps 포털의 **Tailwind Traders** 프로젝트가 표시된 웹 브라우저로 전환하여 Azure DevOps 포털 맨 왼쪽의 세로 메뉴 모음에 있는 **Pipelines**를 클릭합니다. 그런 다음 **파이프라인** 창에서 **Website-CI** 항목을 클릭하고 **Website-CI** 창을 선택한 상태로 웹 브라우저 창에서 이 파이프라인의 URL을 클립보드에 복사합니다.

    >**참고**: 이 URL은 `https://dev.azure.com/<organization_name>liveid2530565/Tailwind%20Traders/_build?definitionId=6` 형식입니다. 여기서 `<organization_name>`은 Azure DevOps 조직의 이름을 나타내는 자리 표시자입니다.

    >**참고**: URL에 *definitionId* 또는 *buildId/releaseId*가 있는 파이프라인 내의 어떤 페이지 URL이나 파이프라인 URL로 사용할 수 있습니다.

1.  **게시물** 탭을 선택한 상태로 **Tailwind Traders** 팀의 **일반** 채널에 `@Azure Pipelines subscribe https://dev.azure.com/<organization_name>/Tailwind%20Traders/_build?definitionId=6`을 게시하여 빌드 파이프라인을 구독합니다. 이때 `<organization_name>` 자리 표시자는 DevOps 조직 이름으로 바꿔야 합니다.
1.  구독이 정상적으로 생성되었다는 확인 메시지가 표시될 때까지 기다립니다.
 
    >**참고**: 빌드 파이프라인의 경우 채널이 **실행 스테이지 단계가 변경됨** 및 **실행 스테이지 승인 대기 중** 알림을 구독하게 됩니다.

1.  랩 컴퓨터에서 Azure DevOps 포털의 **Tailwind Traders** 프로젝트가 표시된 웹 브라우저로 전환하여 Azure DevOps 포털 맨 왼쪽의 세로 메뉴 모음에 있는 **Pipelines**를 클릭합니다. 그런 다음 **파이프라인** 섹션에서 **릴리스**를 클릭하고 릴리스 목록에서 **Website-CD** 항목을 클릭한 후에 **Website-CD** 항목을 선택한 상태로 웹 브라우저 창에서 이 파이프라인의 URL을 클립보드에 복사합니다.

    >**참고**: 이 URL은 `https://dev.azure.com/<organization_name>/Tailwind%20Traders/_release?_a=releases&view=mine&definitionId=2` 형식입니다. 여기서 `<organization_name>`은 Azure DevOps 조직의 이름을 나타내는 자리 표시자입니다.

1.  **게시물** 탭을 선택한 상태로 **Tailwind Traders** 팀의 **일반** 채널에 `@Azure Pipelines subscribe https://dev.azure.com/<organization_name>/Tailwind%20Traders/_release?_a=releases&view=mine&definitionId=2`를 게시하여 릴리스 파이프라인을 구독합니다. 이때 `<organization_name>` 자리 표시자는 DevOps 조직 이름으로 바꿔야 합니다.

    >**참고**: 릴리스 파이프라인의 경우 채널이 **배포 시작됨**, **배포 완료됨** 및 **배포 승인 보류 중** 알림을 구독하게 됩니다.
 
#### 작업 3: 필터를 사용하여 Microsoft Teams에서 Azure Pipelines 구독 사용자 지정

이 작업에서는 Microsoft Teams에서 Azure Pipelines 구독을 사용자 지정합니다.

>**참고**: 사용자가 파이프라인을 구독하면 필터가 적용되지 않고 구독 몇 개가 기본적으로 작성됩니다. 사용자는 이러한 구독을 사용자 지정해야 하는 경우가 많습니다. 빌드가 실패하거나 배포가 프로덕션 환경으로 푸시될 때만 알림을 받으려는 경우를 예로 들 수 있습니다. Azure Pipelines 앱에서는 필터가 지원되므로 채널에 표시되는 알림을 사용자 지정할 수 있습니다.

>**참고**: `@Azure Pipelines subscriptions` 명령을 사용하여 구독을 나열하고 관리할 수 있습니다. 이 명령은 채널의 현재 구독을 모두 나열합니다.

1.  **게시물** 탭을 선택한 상태로 **Tailwind Traders** 팀의 **일반** 채널에 `@Azure Pipelines subscriptions` 명령을 게시합니다. 그런 다음 **Azure Pipelines** 봇의 회신에서 **구독 추가**를 클릭합니다.
1.  **Azure Pipelines** **구독 추가** 패널의 **이벤트 선택** 드롭다운 목록에서 **빌드 완료됨**이 선택되어 있는지 확인하고 **다음**을 클릭합니다.
1.  **Azure Pipelines** **구독 추가** 패널의 **파이프라인 선택** 드롭다운 목록에서 **Website-CI**가 선택되어 있는지 확인하고 **다음**을 클릭합니다.
1.  **Azure Pipelines** **구독 추가** 패널의 **빌드 상태** 드롭다운 목록에서 **[모두]** 가 선택되어 있는지 확인하고 **제출**을 클릭합니다.
1.  **Azure Pipelines** **구독 추가** 패널에서 **확인**을 클릭하여 확인 메시지를 승인합니다.
1.  **Azure Pipelines** **구독 보기** 패널에서 구독 목록을 검토하고 패널을 닫습니다.

### 연습 3: DevOps 시나리오에서 Microsoft Teams 공동 작업 기능 검토

이 연습에서는 DevOps 시나리오에서 더욱 효율적으로 활용 가능한 Microsoft Teams의 공동 작업 기능을 검토합니다.

#### 작업 1: Microsoft Teams 대화 기능 검토

이 작업에서는 몇 가지 기본적인 Microsoft Teams 대화 기능을 검토합니다.

## 공동 작업 환경

>**참고**: Microsoft Teams 게시 기능을 사용하면 간편하게 대화에 연결하고 대화 기록을 보관할 수 있습니다. 대화형 작업 개선을 위해 이모지, 스티커, GIF도 지원됩니다.

1.  대화를 시작하려면 Microsoft Teams 창에서 **게시물** 탭을 클릭합니다.

    >**참고**: Teams에서는 Azure DevOps 작업 항목을 쉽게 찾아서 참조할 수 있으며 Teams 내에서 대화 및 공동 작업을 진행할 수 있습니다. 예를 들어 사용자 스토리 관련 사항을 논의하려는 경우 해당 스토리를 검색하여 대화에 추가한 다음 설명을 입력하면 됩니다.

1.  게시물 항목 바로 아래의 아이콘 목록에서 **Boards** 아이콘을 클릭합니다. 그러면 **Azure Boards** 팝업 창이 자동으로 표시됩니다.
1.  필요한 경우 **Azure Boards** 팝업 창에서 **설정** 링크를 클릭한 후 메시지가 표시되면 **Azure Boards에 연결할 조직 선택** 팝업 창의 **조직** 드롭다운 목록에서 Azure DevOps 조직을 선택하고 **계속**을 클릭합니다. 그런 다음 **프로젝트** 드롭다운 목록에서 **Tailwind Traders**를 선택하고 **계속**을 클릭합니다.
1.  게시물 항목으로 돌아와 작업 항목 참조를 게시물에 추가합니다.

#### 작업 2: Microsoft Teams에서 채널 만들기

이 작업에서는 Microsoft Teams에서 채널을 만드는 프로세스를 단계별로 진행합니다.

>**참고**: **채널**은 사용자의 기본 설정에 따라 특정 주제, 프로젝트, 분야를 기준으로 하여 대화를 정리하는 팀 내의 전용 섹션입니다. 팀 채널은 팀의 모든 구성원이 개방적으로 대화를 할 수 있는 위치입니다. 비공개 채팅은 채팅 참가자에게만 표시됩니다. 채널 사용 시에는 탭, 커넥터, 봇 등의 추가 앱을 함께 활용하면 가장 효율적입니다.

1.  Microsoft Teams 창에서 이 랩 앞부분에서 만든 **Tailwind Traders** 팀을 찾아 팀 오른쪽의 줄임표 기호를 클릭하고 드롭다운 메뉴에서 **채널 추가**를 클릭합니다.
1.  **"Tailwind Traders" 팀의 채널 만들기** 팝업 창의 **채널 이름** 텍스트 상자에 **DevOps 게시물**을 입력하고 **설명(선택 사항)** 텍스트 상자는 비워 둡니다. 그런 다음 **비공개** 드롭다운 목록에서 **표준 - 팀의 모든 사용자가 액세스 가능**을 선택하고 **다음**을 클릭합니다.

#### 작업 3: Microsoft Teams에서 콘텐츠 공유

이 작업에서는 Microsoft Teams에서 Azure DevOps Wiki를 공유하는 프로세스를 단계별로 진행합니다.

>**참고**: 여러 팀원들이 함께 작업을 하는 과정에서는 특정 파일을 공유하고 해당 파일에서 공동 작업을 수행해야 할 가능성이 높습니다. Microsoft Teams를 사용하면 채널 내에서 파일을 쉽게 공유할 수 있습니다. Word, Excel, PowerPoint, Visio 등의 표준 Microsoft Office 애플리케이션으로 만든 파일은 Teams 내에서 바로 확인 및 편집하고 공동 작업을 진행할 수 있습니다. 

1.  Microsoft Teams 창에서 **DevOps 게시물** 채널을 선택한 상태로 **파일** 탭을 클릭하고 도구 모음에 **업로드**, **동기화** 및 **다운로드** 항목이 포함되어 있음을 확인합니다. 

    >**참고**: **끌어서 놓기** 방식으로 파일을 업로드할 수도 있습니다.

    >**참고**: Teams 내에서 Azure DevOps의 콘텐츠를 탭으로 공유할 수도 있습니다. 이러한 콘텐츠의 한 가지 예가 프로젝트 목표, 대규모 사용자 스토리, 사양, 릴리스 정보, 모범 사례 등이 문서로 작성되어 있는 Azure DevOps Wiki입니다.

1.  랩 컴퓨터에서 Azure DevOps 포털이 표시된 브라우저 창으로 전환하여 Azure DevOps 포털 맨 왼쪽의 세로 메뉴 모음에 있는 **개요**를 클릭하고 **개요** 섹션에서 **Wiki**를 클릭합니다.
1.  **Wiki** 메뉴 항목을 선택한 상태로 **코드를 Wiki로 게시**를 클릭합니다.
1.  **코드를 Wiki로 게시** 창의 드롭다운 메뉴에 **TailwindTraders-Website**가 표시되어 있는지 확인하고 **분기** 드롭다운 목록에서 **main**을 선택합니다. 그런 다음 **폴더** 값을 **/Documents**로 설정하고 **Wiki 이름**에 **TailWindTraders-Website wiki**를 입력한 후에 **게시**를 클릭합니다.
1.  **TailWindTraders-Website wiki** 페이지가 표시된 웹 브라우저 창에서 해당 페이지의 URL을 클립보드에 복사합니다.
1.  Microsoft Teams 창으로 전환하여 새로 만든 팀인 **Tailwind Traders**의 **DevOps 게시물** 채널이 선택되어 있는지 확인하고 **DevOps 게시물** 창 위쪽 섹션에서 더하기 기호를 클릭합니다. 그러면 **탭 추가** 패널이 표시됩니다.
1.  **탭 추가** 패널에서 **웹 사이트**를 클릭하고 **웹 사이트** 패널에서 **탭 이름**은 **Tailwind Traders DevOps Wiki**로, **URL**은 방금 클립보드에 복사한 URL로 설정한 후에 **저장**을 클릭합니다.
1.  Microsoft Teams 창에서 **Tailwind Traders** 팀의 **DevOps 게시물** 채널을 선택한 상태로 위쪽 메뉴의 탭 목록에서 새로 추가한 **Tailwind Traders DevOps Wiki** 탭을 클릭합니다. 그리고 Azure DevOps 포털에서 사용 가능한 **TailWindTraders-Website wiki** Wiki와 일치하는 콘텐츠가 이 탭에 포함되어 있는지 확인합니다.

    >**참고**: 이제 Microsoft Teams와 Azure DevOps를 연결했으므로 Microsoft Teams를 통해 아래와 같은 다른 유형의 정보를 표시할 수 있습니다. 

    - [Teams에 OneNote Notebook 추가](https://support.office.com/ko-kr/article/Add-a-OneNote-notebook-to-Teams-0ec78cc3-ba3b-4279-a88e-aa40af9865c2): **스프린트 계획 회의**, **회고 회의** 등의 회의 메모를 보관할 수 있습니다. 
    - [Power BI에 Azure DevOps 연결](https://docs.microsoft.com/ko-kr/azure/devops/report/powerbi/?view=azure-devops) 및 [Power BI 탭 추가](https://support.office.com/ko-kr/article/add-a-powerbi-tab-to-teams-708ce6fe-0318-40fa-80f5-e9174f841918): Azure DevOps의 고급 보고서나 프로젝트 관련 기타 데이터가 표시되는 탭을 추가할 수 있습니다.

## 복습

이 랩에서는 Azure DevOps Services와 Microsoft Teams 간의 통합 시나리오를 구현했습니다.