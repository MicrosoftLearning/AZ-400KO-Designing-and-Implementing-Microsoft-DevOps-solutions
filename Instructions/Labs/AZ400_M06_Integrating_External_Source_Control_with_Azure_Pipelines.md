---
lab:
    title: '랩: Azure Pipelines를 사용하여 외부 소스 제어 기능 통합'
    module: '모듈 6: Azure Pipelines를 사용하여 연속 통합 구현'
---

# 랩: Azure Pipelines를 사용하여 외부 소스 제어 기능 통합
# 학생 랩 매뉴얼

## 랩 개요

Microsoft는 Azure DevOps의 출시와 함께 새로운 CI/CD(연속 통합/지속적인 업데이트) 서비스인 Azure Pipelines를 개발자에게 제공하고 있습니다. 이 서비스를 사용하면 애플리케이션을 지속적으로 빌드/테스트하여 모든 플랫폼이나 클라우드에 배포할 수 있습니다. Azure Pipelines에는 Linux/macOS/Windows용 클라우드 호스팅 에이전트, 컨테이너가 기본적으로 지원되는 유용한 워크플로, 그리고 Kubernetes/VM/서버리스 환경으로 애플리케이션을 유연하게 배포하는 기능이 포함되어 있습니다.

모든 GitHub 오픈 소스 프로젝트에서는 Azure Pipelines를 통해 CI/CD 서비스를 시간 제한 없이 무료로 사용할 수 있으며 병렬 작업을 10회 진행할 수 있습니다. 모든 오픈 소스 프로젝트는 유료 고객이 사용하는 것과 같은 인프라에서 실행됩니다. 그러므로 유료 서비스와 같은 우수한 성능과 품질이 제공됩니다. tom, CPython, Pipenv, Tox, Visual Studio Code, TypeScript 등의 대다수 유명 오픈 소스 프로젝트에서는 CI/CD에 Azure Pipelines가 이미 사용되고 있으며, Azure Pipelines를 채택하는 프로젝트는 계속 늘어나고 있습니다.

이 랩에서는 GitHub 프로젝트를 사용하여 Azure Pipelines를 설정하는 것이 얼마나 쉬운지, 그리고 즉시 혜택을 볼 수 있는 방법을 살펴보겠습니다.

## 목표

이 랩을 완료하면 다음 작업을 수행할 수 있습니다.

-   GitHub Marketplace에서 Azure Pipelines를 설치합니다.
-   GitHub 프로젝트를 Azure DevOps 파이프라인과 통합합니다.
-   파이프라인을 통해 끌어오기 요청을 추적합니다.

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

#### GitHub 계정 설정

이 랩에 사용할 수 있는 GitHub 계정이 아직 없으면 [새 GitHub 계정 등록](https://docs.github.com/en/free-pro-team@latest/github/getting-started-with-github/signing-up-for-a-new-github-account)에서 제공되는 지침에 따르세요.

### 연습 1: Azure Pipelines 시작

이 연습에서는 Marketplace에서 제공되는 신규 Azure Pipelines 통합 기능을 사용해 Azure DevOps와 GitHub 프로젝트를 통합합니다.

#### 작업 1: GitHub 리포지토리 포크 및 Azure Pipelines 설치

이 작업에서는 GitHub 리포지토리를 포크하고 GitHub 계정에 Azure Pipelines를 설치합니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [GitHub actionsdemos/calculator 사이트](https://github.com/actionsdemos/calculator)로 이동한 다음 GitHub에 아직 로그인하지 않았으면 지금 로그인합니다.

    > **참고**: 이 랩에서는 이 기준 프로젝트를 포크하여 사용합니다.

1.  **actionsdemos/calculator site** 페이지에서 **Fork**를 클릭하여 사용자의 GitHub 계정에 리포지토리를 포크합니다. 메시지가 표시되면 리포지토리를 포크할 계정을 선택합니다.

    > **참고**: **GitHub Marketplace**에서는 프로젝트 워크프로를 확장하는 데 사용할 수 있는 Microsoft와 타사의 다양한 도구를 제공합니다. 

1.  포크된 리포지토리가 표시된 페이지의 위쪽 메뉴에서 **Marketplace**를 클릭합니다.
1.  **Search for apps and actions**에 **Azure Pipelines**를 입력하고 **Enter** 키를 누른 다음 결과 목록에서 **Azure Pipelines**를 클릭합니다. 
1.  **Azure Pipelines** 페이지에서 **Read more**를 클릭하여 Azure Pipelines의 이점 설명을 확인합니다.

    > **참고**: 퍼블릭 리포지토리에서는 누구나 Azure Pipelines 서비스를 무료로 사용할 수 있습니다. 프라이빗 리포지토리 사용 시에는 빌드 큐 하나에서 Azure Pipelines 서비스를 무료로 사용할 수 있습니다. 

1.  **Azure Pipelines** 페이지에서 **Install it for free**를 클릭합니다. **GitHub** 계정이 여러 개이면 **Switch billing account** 드롭다운에서 calculator를 포크한 계정을 선택합니다.
1.  **Review your order** 페이지에서 **Complete order and begin installation**을 클릭합니다.
1.  **Install Azure Pipelines** 페이지에서 기본 옵션인 **All repositories**를 그대로 적용하고 **Install**을 클릭합니다.

    > **참고**: 포함할 리포지토리를 지정할 수도 있지만 이 랩에서는 모든 리포지토리를 포함합니다. Azure DevOps가 서비스를 제공하려면 권한 세트 목록이 필요합니다. 

1.  메시지가 표시되면 GitHub 암호를 입력해 인증을 하고 작업을 계속 진행합니다.
1.  메시지가 표시되면 **Setup your Azure Pipelines project** 페이지의 **Select your Azure DevOps organization** 드롭다운 목록에서 Azure DevOps 계정을 선택하고 **Create a new project**를 클릭합니다.
1.  메시지가 표시되면 **Setup your Azure Pipelines project** 페이지의 **Project name** 텍스트 상자에 **Integrating External Source Control with Azure Pipelines**를 입력합니다. **Project visibility**는 **Private**으로 설정된 상태로 유지하고 **Continue**를 클릭합니다.
1.  **Azure Pipelines by Microsoft would like permission to** 페이지에서 **Authorize Azure Pipelines**를 클릭합니다.

### 작업 2: Azure Pipelines 프로젝트 구성

이 작업에서는 이전 작업에서 만든 GitHub 리포지토리의 포크를 기반으로 하여 Azure Pipelines 프로젝트를 구성합니다.

> **참고**: 이제 Azure DevOps 사이트에서 Azure Pipelines 프로젝트를 설정해야 합니다. 

1.  Azure DevOps 포털 **파이프라인** 보기의 **리포지토리 선택** 창에서 이전 작업에서 만든 GitHub calculator 리포지토리의 포크를 선택합니다.

    > **참고**: Azure Pipelines에서 프로젝트를 분석하여 해당 프로젝트에 적합한 기존 템플이 있는지를 확인합니다. 여기서는 프로젝트 요구를 충족할 수 있는 **Node.js**용 템플릿을 사용하는 것이 좋습니다. 대체 템플릿도 제시되지만 이 랩에서는 권장 템플릿이 가장 적합합니다. 
1.  **파이프라인 구성**에서 **Node.js**를 선택합니다.

    > **참고**: 빌드 파이프라인은 이러한 프로세스를 정의하는 데 적합한 태그 구문인 **YAML**로 정의됩니다. 이 구문을 사용하면 리포지토리 내의 기타 모든 파일과 마찬가지로 파이프라인 구성을 관리할 수 있기 때문입니다. 이 템플릿은 빌드용 VM을 끌어올 풀, 빌드용으로 Node.js를 설치할 프로세스, 그리고 실제 빌드 자체를 확인하는 단순한 템플릿입니다. 

1.  **파이프라인 YAML 검토**에서 **저장 및 실행**을 클릭하여 파이프라인을 저장하고 새 빌드를 큐에 넣습니다.
1.  **저장 및 실행** 창에서 기본 설정을 수락하고 **저장 및 실행**을 클릭합니다.

    > **참고**: 이 랩에서는 master 분기에 이 새 파일을 바로 커밋할 수 있습니다. 

    > **참고**: 파이프라인 실행이 완료되려면 시간이 걸립니다. 이 시간 동안 파이프라인은 빌드 에이전트를 구성하고 GitHub에서 소스를 끌어온 다음, 파이프라인 정의에 따라 빌드합니다.

1.  빌드 작업 창의 **요약** 탭에서 빌드가 정상적으로 완료되었는지 확인합니다.

### 작업 3: YAML 빌드 파이프라인 정의 수정

이 작업에서는 포크한 GitHub 리포지토리에서 YAML 빌드 정의를 수정하고 이 수정 내용으로 인해 트리거되는 빌드 프로세스를 추적합니다.

> **참고**: 기본 파이프라인으로도 빌드를 시작할 수 있지만, 이 파이프라인에서는 자동화 가능한 작업이 수행되지 않습니다. 가령 변경 시 버그가 발생하지 않음을 확인할 수 있는 테스트도 실행된다면 매우 유용할 것입니다. 그러면 GitHub로 돌아가서 YAML을 수동으로 편집해 보겠습니다. 

1.  빌드 작업 창의 **요약** 탭에서 **리포지토리 및 버전** 레이블 옆에 있는 항목(이 랩 앞부분에서 만든 포크를 호스트하는 GitHub 프로젝트 리포지토리를 나타내는 항목)을 마우스 오른쪽 단추로 클릭하고 **새 탭에서 링크 열기**를 선택합니다. 그러면 새 브라우저 탭이 열리고 포크 내용이 포함된 GitHub 페이지가 표시됩니다.

    > **참고**: 이 랩에서는 GitHub와 Azure DevOps 간을 전환해 가면서 작업을 진행할 것이므로 GitHub와 Azure DevOps의 브라우저 탭을 열어 두면 랩을 더 쉽게 진행할 수 있습니다.

1.  포크 내용이 표시된 GitHub 페이지에서 **azure-pipelines.yml** 파일을 나타내는 항목을 클릭합니다. 그러면 파일이 자동으로 열리고 해당 내용이 표시됩니다. 
1.  **master/calculator/azure-pipelines.yml** 페이지에서 파일 내용이 표시된 창 오른쪽 위에 있는 연필 모양 **Edit this file** 아이콘을 클릭합니다. 

    > **참고**: 이 프로젝트에는 Mocha를 사용하여 작성된 테스트가 이미 포함되어 있으므로 파이프라인에서 해당 테스트를 실행만 하면 됩니다. 

1.  테스트 실행을 추가하려면 `npm run build` 명령 바로 아래에 같은 들여쓰기를 적용하여 `npm test` 명령을 추가합니다. 그리고 빌드의 각 작업 내용을 명확하게 표시하기 위해 `displayName` 항목은 `'npm install, build, and test'`로 업데이트합니다. 

    ```
        npm test
      displayName: 'npm install, build, and test'
    ```

1.  페이지 아래쪽으로 스크롤하여 기본 커밋 메시지를 **Adding npm test**로 바꾸고 **Commit changes**를 클릭합니다. 

    > **참고**: 이전과 마찬가지로 랩 환경에서는 이 변경 내용을 master 분기에 바로 커밋해도 됩니다.

1.  **Azure DevOps** 포털이 표시된 브라우저 탭으로 다시 전환한 다음 이동 경로를 탐색하여 **파이프라인** 보기의 **파이프라인** 창으로 이동합니다.
1.  **최근에 실행한 파이프라인** 목록의 **최근** 탭에 업데이트를 통해 트리거된 새 빌드가 이미 표시되어 있는지 확인합니다. 파이프라인에 해당하는 항목을 클릭하고 **실행** 탭에서 최신 실행을 선택한 후에 **작업** 섹션에서 **작업** 항목을 클릭합니다.
1.  작업 세부 정보가 표시된 창에서 작업의 개별 태스크를 클릭하여 완료될 때까지 진행합니다.

### 작업 4: GitHub 끌어오기 요청을 통해 변경 제안

이 작업에서는 잘못된 변경을 제안한 다음 끌어오기 요청을 통해 트리거되는 빌드의 결과를 검토합니다.

> **참고**: 이 파이프라인 설정에서 제공되는 유용한 이점 중 하나는, 변경을 커밋할 때마다 품질 게이트가 자동으로 실행된다는 것입니다. 그러므로 여러 품질 수준에서 다수의 참가자가 업데이트할 가능성이 있는 프로젝트를 훨씬 쉽게 관리할 수 있습니다. 

1.  **azure-pipelines.yml** 파일의 내용이 나와 있는 GitHub 페이지가 표시된 브라우저 탭으로 다시 전환합니다. 그런 후에 포크한 리포지토리의 내용이 나열된 페이지로 다시 이동하여 **azure-pipelines.yml**을 클릭합니다.
1.  **calculator/** 프롬프트에 **arithmeticController.js**를 입력하고 결과 목록에서 **api/controllers/arithmeticController.js**를 클릭합니다. 그러면 브라우저 세션이 **master/calculator/api/controllers/arithmeticController.js** 페이지로 자동 리디렉션되어 해당 파일의 내용이 표시됩니다.

    > **참고**: 이 컨트롤러에는 앱의 핵심 기능이 포함되어 있습니다. 하지만 **add** 작업의 코드를 명확하게 확인하기가 어렵습니다. 여기서는 JavaScript 사용법을 잘 모르는 개발자가 코드를 더 쉽게 알아볼 수 있도록 정리하여 개선하려 한다고 가정해 보겠습니다.

1.  **master/calculator/api/controllers/arithmeticController.js** 페이지에서 파일 내용이 표시된 창 오른쪽 위에 있는 연필 모양 **Edit this file** 아이콘을 클릭합니다. 
1.  `    'add':      function(a,b) { return +a + +b },` 줄을 `    'add':      function(a,b) { return a + b },`로 변경합니다.

    > **참고**: 이 변경은 잘못된 작업이므로 잘못된 결과가 생성됩니다.

1.  페이지 아래쪽으로 스크롤하여 기본 커밋 메시지를 **Modifying the add function**으로 바꾸고 **Create a new branch**를 선택하여 이름을 **addition-cleanup**으로 설정한 후에 **Propose file change**를 클릭합니다.
1.  **Open a pull request** 페이지에서 **Create pull request**를 클릭하여 테스트하지 않은 변경 내용을 프로덕션 코드로 가져오는 프로세스를 시작합니다. 

    > **참고**: Azure DevOps에서 변경 내용을 검색하여 빌드 파이프라인을 시작합니다. 그러면 확인이 실패하여 GitHub UI에서 업데이트가 트리거됩니다. 

    > **참고**: 이제 프로젝트 소유자로서 코드를 다시 올바르게 수정해 보겠습니다.

1.  **Modifying the add function #1** 끌어오기 요청 페이지의 **All checks have failed** 섹션에서 **Details**를 클릭하여 자세한 내용을 확인합니다.
1.  **ANNOTATIONS** 섹션을 검토한 후 해당 섹션 바로 아래의 **View more details on Azure Pipelines** 링크를 클릭합니다. 그러면 새 브라우저 탭이 열리고 Azure DevOps 포털의 작업에서 실패한 실행이 표시됩니다.
1.  Azure Portal의 실패한 작업 창에서 **작업** 항목을 클릭하여 작업 세부 정보를 표시합니다. 
1.  작업 태스크 목록에서 **npm 설치, 빌드 및 테스트** 태스크를 클릭하여 해당 출력을 확인합니다.
1.  실패하는 테스트가 나열된 섹션을 찾습니다. 

    > **참고**: 테스트가 실패한 이유를 바로 확인하지는 못할 수도 있습니다. 그러나 파이프라인에 누적된 모든 기록을 살펴보면 새 끌어오기 요청에서 특정 부분이 잘못되었음을 쉽게 파악할 수 있습니다. 다음 단계에서는 "21 + 21"의 결과로 "42"가 아닌 "2121"이 생성된 이유를 살펴봅니다.

1.  Azure DevOps 포털의 작업에서 실패한 실행이 표시된 탭을 닫습니다.

### 작업 5: 손상된 끌어오기 요청을 사용하여 프로젝트 개선

이 작업에서는 이전 작업에서 만든 끌어오기 요청에 추가했던 잘못된 변경 내용을 수정합니다.

> **참고**: 이제 프로젝트 소유자로서 코드를 다시 올바르게 수정해 보겠습니다.

1.  **Modifying the add function #1** GitHub 페이지가 표시된 브라우저 탭으로 돌아와 포크한 리포지토리의 내용이 나열된 기본 페이지를 다시 표시합니다. 
1.  리포지토리 파일 목록이 포함된 창의 위쪽에서 **Pull requests**를 클릭한 다음 최신 끌어오기 요청을 나타내는 항모을 클릭합니다.
1.  **Modifying the add function #1** GitHub 페이지에서 **File changed** 탭을 클릭하고 해당 내용을 검토합니다.

    > **참고**: 변수를 숫자 표현으로 강제 변환하려면 각 변수 앞에 더하기 기호가 필요함을 모르는 사람이 변경을 수행한 것으로 보입니다. 이 더하기 기호를 제거했기 때문에 JavaScript에서 가운데의 더하기 기호를 문자열 연결 연산자로 해석하여 실패한 테스트에서처럼 21 + 21 = 2121이 표시된 것입니다. 

1.  **Modifying the add function #1** GitHub 페이지에서 **Review changes** 단추 바로 아래의 줄임표 기호를 클릭하고 드롭다운 메뉴에서 **Edit file**을 클릭합니다. 
1.  **a** 및 **b** 변수 앞에 더하기 기호를 추가하여 원래 변경 내용을 되돌립니다. 즉, 코드를 'add':      function(a,b) { return +a + +b },`로 수정합니다. 그리고 이 코드 앞의 줄에 `// Using + operator to type cast variables as integers in order to prevent string concatenation` 주석을 포함합니다.
1.  페이지 아래쪽으로 스크롤하여 기본 커밋 메시지를 **Fixing the add function**으로 바꾸고 **Commit directly to the addition-cleanup branch** 옵션이 선택되어 있는지 확인한 후에 **Commit changes**를 클릭합니다.
1.  **Modifying the add function #1** GitHub 페이지에서 **Conversation** 탭을 선택합니다.

    > **참고**: Azure DevOps에서 변경 내용을 다시 검색하여 빌드 파이프라인을 시작합니다. 모든 검사에 통과할 때까지 기다립니다. 

1.  모든 검사에 통과하면 **끌어오기 요청 병합**을 클릭한 다음 **병합 확인**을 클릭합니다.

### 작업 6: 빌드 상태 배지 추가

이 작업에서는 GitHub 리포지토리에 빌드 상태 배지를 추가합니다.

> **참고**: 빌드 상태 배지는 품질 프로젝트에서 중요한 기호로 사용됩니다. 현재 프로젝트의 빌드 상태가 정상임을 나타내는 배지가 표시되어 있으면 프로젝트를 효율적으로 유지 관리하고 있다는 의미입니다. 

1.  **Azure DevOps** 포털이 표시된 브라우저 탭으로 다시 전환한 다음 이동 경로를 탐색하여 **파이프라인** 보기의 **파이프라인** 창 **최근** 탭으로 이동합니다.
1.  **최근** 탭의 **최근에 실행한 파이프라인** 목록에서 이 랩에서 사용한 파이프라인에 해당하는 항목을 클릭합니다. 
1.  파이프라인 창 오른쪽 위의 줄임표 기호를 클릭하고 드롭다운 목록에서 **상태 배지**를 선택합니다. 

    > **참고**: **상태 배지** UI를 사용하면 빌드 상태를 빠르고 쉽게 참조할 수 있습니다. 사용자 대시보드에서 제공된 URL을 사용하거나, Markdown 조각을 사용해 Wiki 페이지 등의 위치에 상태 배지를 추가하는 경우가 많습니다. 

1.  **상태 배지** 창에서 **샘플 Markdown**의 **클립보드에 복사** 단추를 클릭합니다.
1.  포크한 리포지토리의 내용이 나와 있는 GitHub 페이지가 표시된 브라우저 탭으로 다시 전환합니다. 그런 후에 필요한 경우 **<> Code** 탭을 클릭합니다.
1.  리포지토리 파일 목록에서 **README.md**를 클릭하고 **master/calculator/README.md** 페이지에서 파일 내용이 표시된 창 오른쪽 위에 있는 연필 모양 **Edit this file** 아이콘을 클릭합니다. 
1.  줄 6 위에 줄을 하나 더 추가하여 클립보드의 내용을 붙여넣습니다.
1.  페이지 아래쪽으로 스크롤하여 기본 커밋 메시지를 **Add an Azure Pipelines status badge**로 바꾸고 **Commit changes**를 클릭합니다. 

    > **참고**: 이제 프로젝트 전면 페이지에 동적 빌드 상태 배지가 추가되었습니다. 따라서 프로젝트를 효율적으로 관리하고 있음을 누구나 확인할 수 있습니다.

#### 복습

이 랩에서는 Marketplace에서 제공되는 신규 Azure Pipelines 통합 기능을 사용해 Azure DevOps와 GitHub 프로젝트를 통합했습니다.
