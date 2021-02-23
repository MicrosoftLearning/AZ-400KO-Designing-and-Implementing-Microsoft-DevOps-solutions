---
lab:
    title: '랩: Azure Artifacts로 패키지 관리'
    az400Module: '모듈 9: 종속성 관리 전략 디자인 및 구현'
---

# 랩: Azure Artifacts로 패키지 관리
# 학생 랩 매뉴얼

## 랩 개요

Azure Artifacts를 활용하면 Azure DevOps에서 NuGet, npm 및 Maven 패키지를 쉽게 검색, 설치 및 게시할 수 있습니다. Azure Artifacts는 Build 등의 기타 Azure DevOps 기능과 긴밀하게 통합되므로 기존 워크플로에서 패키지 관리를 원활하게 진행할 수 있습니다.

이 랩에서는 다음 단계를 수행하여 Azure Artifacts 사용 방법을 알아봅니다.

- 피드 만들기 및 피드에 연결
- NuGet 패키지 만들기 및 게시
- NuGet 패키지 가져오기
- NuGet 패키지 업데이트

## 목표

이 랩을 완료하면 다음 작업을 수행할 수 있습니다.

-   Azure Active Directory(Azure AD) 서비스 주체 만들기
-   Azure Key Vault 만들기 
-   Azure DevOps 파이프라인을 통해 끌어오기 요청 추적

## 랩 소요 시간

-   예상 시간: **40분**

## 지침

### 시작하기 전

#### 랩 가상 머신에 로그인

다음 자격 증명을 사용하여 Windows 10 가상 머신에 로그인해야 합니다.
    
-   사용자 이름: **Student**
-   암호: **Pa55w.rd**

#### 이 랩에 필요한 애플리케이션 검토

이 랩에서 사용할 애플리케이션을 확인합니다.
    
-   Microsoft Edge
-   [Visual Studio 다운로드 페이지](https://visualstudio.microsoft.com/downloads/)에서 제공되는 Visual Studio 2019 Community Edition. Visual Studio 2019 설치에는 **ASP.NET 및 웹 개발**, **Azure 개발** 및 **NET Core 플랫폼 간 개발** 워크로드가 포함되어야 합니다. 이러한 워크로드는 랩 컴퓨터에 미리 설치되어 있습니다.

#### Azure DevOps 조직 설정

이 랩에 사용할 수 있는 Azure DevOps 조직이 아직 없으면 [조직 또는 프로젝트 컬렉션 만들기](https://docs.microsoft.com/ko-kr/azure/devops/organizations/accounts/create-organization?view=azure-devops)에서 제공되는 지침에 따라 조직을 만듭니다.

### 연습 0: 랩 필수 구성 요소 구성

이 연습에서는 랩의 필수 구성 요소를 설정합니다. 구체적으로는 Azure DevOps 데모 생성기 템플릿과 Visual Studio 구성을 기반으로 하여 미리 구성된 Parts Unlimited 팀 프로젝트를 설정합니다.

#### 작업 1: 팀 프로젝트 구성

이 작업에서는 Azure DevOps 데모 생성기를 사용하여 **PartsUnlimited** 템플릿을 기반으로 새 프로젝트를 생성합니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [Azure DevOps 데모 생성기](https://azuredevopsdemogenerator.azurewebsites.net)로 이동합니다. 이 유틸리티 사이트에서는 계정 내에서 새 Azure DevOps 프로젝트를 만드는 프로세스를 자동으로 진행할 수 있습니다. 이 프로젝트에는 랩에 필요한 작업 항목, 리포지토리 등의 콘텐츠가 미리 입력되어 있습니다. 

    > **참고**: 이 사이트에 대한 자세한 내용은 https://docs.microsoft.com/ko-kr/azure/devops/demo-gen을 참조하세요.

1.  **로그인**을 클릭하고 Azure DevOps 구독과 연결된 Microsoft 계정을 사용하여 로그인합니다.
1.  필요한 경우 **Azure DevOps 데모 생성기** 페이지에서 **수락**을 클릭하여 Azure DevOps 구독에 액세스하는 데 필요한 권한 요청을 수락합니다.
1.  **새 프로젝트 만들기** 페이지의 **새 프로젝트 이름** 텍스트 상자에 **Azure Artifacts로 패키지 관리**를 입력합니다. 그런 다음 **조직 선택** 드롭다운 목록에서 Azure DevOps 조직을 선택하고 **템플릿 선택**을 클릭합니다.
1.  **템플릿 선택** 페이지의 템플릿 목록에서 **PartsUnlimited** 템플릿을 클릭한 다음 **템플릿 선택**을 클릭합니다.
1.  **새 프로젝트 만들기** 페이지로 돌아와서 **프로젝트 만들기**를 클릭합니다.

    > **참고**: 프로세스가 완료될 때까지 기다립니다. 약 2분이 소요됩니다. 프로세스가 실패하면 DevOps 조직으로 이동하여 프로젝트를 삭제한 후에 다시 시도합니다.

1.  **새 프로젝트 만들기** 페이지에서 **프로젝트로 이동**을 클릭합니다.

#### 작업 2: Visual Studio에서 Parts Unlimited 솔루션 구성

이 작업에서는 랩에서 사용할 수 있도록 Visual Studio를 구성합니다.

1.  Azure DevOps 포털에서 **Azure Artifacts로 패키지 관리** 팀 프로젝트가 표시되어 있는지 확인합니다. 

    > **참고**: [https://dev.azure.com/`<your-Azure-DevOps-account-name>`/Package%20Management%20with%20Azure%20Artifacts](https://dev.azure.com/`<your-Azure-DevOps-account-name>`/Package%20Management%20with%20Azure%20Artifacts) URL로 직접 이동하여 프로젝트 페이지에 액세스할 수 있습니다. 여기서 `<your-Azure-DevOps-account-name>` 자리 표시자는 사용자의 계정 이름을 나타냅니다. 

1.  **Azure Artifacts로 패키지 관리** 창 왼쪽의 세로 메뉴에 있는 **Repos**를 클릭합니다.
1.  **파일** 창에서 **복제**를 클릭하고 **VS Code에서 복제**를 클릭한 다음 드롭다운 메뉴에서 **Visual Studio**를 선택합니다.
1.  작업을 계속 진행할지를 묻는 메시지가 표시되면 **열기**를 클릭합니다. 
1.  메시지가 표시되면 Azure DevOps 조직을 설정하는 데 사용한 사용자 계정으로 로그인합니다.
1.  Visual Studio 인터페이스 내의 **Azure DevOps** 팝업 창에서 기본 로컬 경로를 그대로 적용하고 **복제**를 클릭합니다. 그러면 Visual Studio로 프로젝트 가져오기가 자동으로 진행되며, 새 웹 브라우저 탭이 열리고 마이그레이션 보고서 페이지가 표시됩니다.

    > **참고**: **프로젝트 및 솔루션 변경 내용 검토** 대화 상자에서 지원되지 않는 프로젝트 형식 관련 경고를 검토하고 **확인**을 클릭합니다. 

1.  마이그레이션 보고서 페이지가 표시된 웹 브라우저 탭을 닫습니다.
1.  Visual Studio 창은 랩에서 사용할 수 있도록 열어 둡니다.

### 연습 1: Azure Artifacts 사용

이 연습에서는 다음 단계를 수행하여 Azure Artifacts 사용 방법을 알아봅니다.

- 피드 만들기 및 피드에 연결
- NuGet 패키지 만들기 및 게시
- NuGet 패키지 가져오기
- NuGet 패키지 업데이트

#### 작업 1: 피드 만들기 및 피드에 연결

이 작업에서는 피드를 만들어 해당 피드에 연결합니다.

1.  Azure DevOps 포털의 프로젝트 설정이 표시된 웹 브라우저 창의 세로 탐색 창에서 **Artifacts**를 클릭합니다.
1.  **Artifacts** 허브를 표시한 상태로 창 위쪽의 **피드 만들기**를 클릭합니다. 

    > **참고**: 조직 내의 사용자에게 제공되는 NuGet 패키지 컬렉션인 이 피드는 공용 NuGet 피드와 함께 피어로 저장됩니다. 이 랩에서는 Azure Artifacts 사용을 위한 워크플로를 중점적으로 진행하므로 아키텍처 및 개발과 관련된 실제 결정 사항은 설명을 위해서만 제시되는 것입니다.  이 피드에는 조직의 여러 프로젝트에서 공유할 수 있는 공통 기능이 포함됩니다. 

1.  **새 피드 만들기** 창의 **이름** 텍스트 상자에 **PartsUnlimitedShared**를 입력하고 **범위** 섹션에서는 **조직** 옵션을 선택합니다. 다른 설정은 기본값으로 유지하고 **만들기**를 클릭합니다. 

    > **참고**: 이 NuGet 피드에 연결하려는 모든 사용자는 환경을 구성해야 합니다. 

1.  **Artifacts** 허브로 돌아와서 **피드에 연결**을 클릭합니다.
1.  **피드에 연결** 창의 **NuGet** 섹션에서 **Visual Studio**를 선택하고 **Visual Studio** 창에서 **소스** URL을 복사합니다. 
1.  **Visual Studio** 창으로 다시 전환합니다.
1.  Visual Studio 창에서 **도구** 메뉴 머리글을 클릭하고, 드롭다운 메뉴에서 **NuGet 패키지 관리자**를 선택합니다. 그런 다음 계단식 메뉴에서 **패키지 관리자 설정**을 선택합니다.
1.  **옵션** 대화 상자에서 **패키지 소스**를 클릭하고 더하기 기호를 클릭하여 새 패키지 소스를 추가합니다.
1.  대화 상자 아래쪽의 **이름** 텍스트 상자에 있는 **Package source**를 **PartsUnlimitedShared**로 바꾸고 **소스** 텍스트 상자에 Azure DevOps 포털에서 복사한 URL을 붙여넣습니다. 
1.  **업데이트**와 **확인**을 차례로 클릭하여 추가를 완료합니다.

    > **참고**: 이제 Visual Studio가 새 피드에 연결되었습니다.

1.  아티팩트 소스 업데이트가 반영되도록 PartsUnlimited 리포지토리를 복제하는 데 사용한 다른 Visual Studio 인스턴스를 닫았다가 다시 연 후에 **PartsUnlimited** 솔루션을 엽니다. 이 연습의 세 번째 작업에서 해당 솔루션이 필요합니다.

#### 작업 2: NuGet 패키지 만들기 및 게시

이 작업에서는 NuGet 패키지를 만들고 게시합니다.

1.  새 패키지 소스를 구성하는 데 사용한 Visual Studio 창의 기본 메뉴에서 **파일**을 클릭하고 드롭다운 메뉴에서 **새로 만들기**를 클릭한 다음 계단식 메뉴에서 **프로젝트**를 클릭합니다. 

    > **참고**: 여기서는 NuGet 패키지로 게시되는 공유 어셈블리를 만듭니다. 이렇게 하면 다른 팀이 프로젝트 소스를 직접 사용하지 않고도 해당 패키지를 통합하여 최신 정보를 파악할 수 있습니다.

1.  **새 프로젝트 만들기** 창의 **최근 프로젝트 템플릿** 페이지에서 검색 텍스트 상자를 사용해 **클래스 라이브러리(.NET Framework)** 템플릿을 찾아서 선택한 후에 **다음**을 클릭합니다. 
1.  **새 프로젝트 만들기** 창의 **클래스 라이브러리(.NET Framework)** 페이지에서 다음 설정을 지정하고 **만들기**를 클릭합니다.

    | 설정 | 값 |
    | --- | --- |
    | 프로젝트 이름 | **PartsUnlimited.Shared** |
    | 위치 | 기본값 수락 |
    | 솔루션 | **새 솔루션 만들기** |
    | 솔루션 이름 | **PartsUnlimited.Shared** |
    | 프레임워크 | **NET Framework 4.5.1** |

    > **참고**: **NET Standard**는 선택하지 마세요.

1.  Visual Studio 인터페이스 내의 **솔루션 탐색기** 창에서 **Class1.cs**를 마우스 오른쪽 단추로 클릭한 다음 오른쪽 클릭 메뉴에서 **삭제**를 선택합니다. 삭제를 확인하라는 메시지가 표시되면 **확인**을 클릭합니다.
1.  Visual Studio 인터페이스 내의 **솔루션 탐색기** 창에서 **PartsUnlimited.Shared** 프로젝트 노드를 마우스 오른쪽 단추로 클릭하고 **속성**을 선택합니다.
1.  **PartsUnlimited.Shared** 속성 창 내에서 **대상 프레임워크**가 **NET Framework 4.5.1**로 설정되어 있는지 확인합니다.
1.  **Ctrl+Shift+B**를 눌러 프로젝트를 빌드합니다. 

    > **참고**: 다음 작업에서는 **NuGet.exe**를 사용하여 빌드 프로젝트에서 직접 NuGet 패키지를 생성합니다. 이렇게 하려면 프로젝트를 먼저 작성해야 합니다.

1.  Azure DevOps 포털이 표시된 웹 브라우저로 전환합니다. 
1.  **피드에 연결** 창으로 이동하여 **NuGet** 섹션에서 **NuGet.exe**를 선택합니다. 그러면 **NuGet.exe** 창이 표시됩니다.
1.  **NuGet.exe** 창에서 **도구 가져오기**를 클릭합니다.
1.  **도구 가져오기** 창에서 **최신 NuGet 다운로드** 링크를 클릭합니다. 그러면 다른 브라우저 탭이 자동으로 열리고 **사용 가능한 NuGet 배포 버전** 페이지가 표시됩니다.
1.  **사용 가능한 NuGet 배포 버전** 페이지에서 최신 nuget 버전의 링크를 클릭한 다음 실행 파일을 로컬 **Downloads** 폴더에 다운로드합니다.
1.  **Visual Studio** 창으로 전환합니다. **솔루션 탐색기** 창에서 **PartsUnlimited.Shared** 프로젝트 노드를 마우스 오른쪽 단추로 클릭하고 오른쪽 클릭 메뉴에서 **파일 탐색기에서 폴더 열기**를 선택합니다.
1.  파일 탐색기 창 내에서 다운로드한 **nuget.exe** 파일을 **Downloads** 폴더에서 **csproj** 파일이 들어 있는 폴더로 이동합니다.
1.  같은 파일 탐색기 창에서 **파일** 메뉴 머리글을 선택하고 드롭다운 메뉴에서 **Windows PowerShell 열기**를 선택합니다. 그런 다음 계단식 메뉴에서 **관리자 권한으로 Windows PowerShell 열기**를 클릭합니다. 
1.  **관리자: Windows PowerShell** 창에서 다음 명령을 실행하여 프로젝트에서 **nupkg** 파일을 만듭니다. 

    > **참고**: 이 방법을 사용하면 배포용 NuGet 비트를 빠르게 패키지로 만들 수 있습니다. NuGet은 매우 자세하게 사용자 지정할 수 있습니다. 자세한 내용은 [NuGet 패키지 만들기 페이지](https://docs.microsoft.com/ko-kr/nuget/create-packages/overview-and-workflowhttps:/docs.microsoft.com/ko-kr/nuget/create-packages/overview-and-workflow)를 참조하세요.

    ```
    ./nuget.exe pack ./PartsUnlimited.Shared.csproj
    ```

    > **참고**: **관리자: Windows PowerShell** 창에 표시되는 경고는 무시하세요.

    > **참고**: NuGet이 프로젝트에서 확인 가능한 정보를 토대로 하여 최소 패키지를 빌드합니다. 예를 들어 이 패키지의 이름은 **PartsUnlimited.Shared.1.0.0.nupkg**인데, 이 버전 번호는 어셈블리에서 검색한 것입니다.

1.  **Visual Studio** 창으로 다시 전환하여 **솔루션 탐색기** 창에서 **PartsUnlimited.Shared\Properties** 노드를 확장합니다. 그런 다음 **AssemblyInfo.cs**를 클릭하여 Visual Studio 창의 가운데 창에서 해당 파일을 열고 내용을 검토합니다.

    > **참고**: **AssemblyVersion** 특성은 어셈블리에서 기본 제공할 버전 번호를 지정합니다. 각 NuGet 릴리스에는 고유한 버전 번호가 필요합니다. 그러므로 이 방법을 계속 사용하여 패키지를 만드는 경우에는 빌드 전에 이 번호를 높여야 합니다.

1.  **관리자: Windows PowerShell** 창으로 전환한 후 다음 명령을 실행하여 **PartsUnlimitedShared** 피드에 패키지를 게시합니다.

    > **참고**: 여기서 **API 키**를 입력해야 합니다. 비어 있지 않은 아무 문자열이나 입력하면 됩니다. 여기서는 **VSTS**를 사용합니다. 메시지가 표시되면 Azure DevOps 조직에 로그인합니다. 

    ```
    ./nuget.exe push -source "PartsUnlimitedShared" -ApiKey VSTS PartsUnlimited.Shared.1.0.0.nupkg
    ```
1.  Azure DevOps 포털이 표시된 웹 브라우저 창으로 전환하여 세로 탐색 창에서 **Artifacts**를 선택합니다.
1.  **Artifacts** 허브 창 왼쪽 위의 드롭다운 목록을 클릭하고 피드 모록에서 **PartsUnlimitedShared** 항목을 선택합니다.

    > **참고**: **PartsUnlimitedShared** 피드에는 새로 게시한 NuGet 패키지가 포함되어 있습니다. 

1.  NuGet 패키지를 클릭하여 세부 정보를 표시합니다.

#### 작업 3: NuGet 패키지 가져오기

이 작업에서는 NuGet 패키지를 가져옵니다.

1.  **Parts Unlimited** 솔루션이 표시된 **Visual Studio** 창으로 전환합니다.
1.  **솔루션 탐색기** 창에서 **PartsUnlimitedWebsite** 프로젝트 아래의 **References** 노드를 마우스 오른쪽 단추로 클릭한 후에 오른쪽 클릭 메뉴에서 **NuGet 패키지 관리**를 선택합니다. 
그러면 솔루션 탐색기 창의 가운데 창에 **NuGet: PartsUnlimitedWebsite** 탭이 열립니다.
1.  **NuGet: PartsUnlimitedWebsite** 창에서 **찾아보기** 탭을 클릭한 다음 창 오른쪽 위의 **패키지 소스** 드롭다운 목록에서 **PartsUnlimitedShared**를 선택합니다. 

    > **참고**: 패키지 목록에는 방금 추가한 패키지 하나만 포함되어 있습니다.

1.  해당 패키지를 선택하고 **PartsUnlimited.Shared** 창에서 **설치**를 클릭하여 프로젝트에 패키지를 추가합니다.
1.  메시지가 표시되면 **변경 내용 미리 보기** 대화 상자에서 **확인**을 클릭합니다.
1.  **Ctrl+Shift+B**를 눌러 프로젝트를 빌드하고 빌드가 정상적으로 완료되었는지 확인합니다. 

    > **참고**: 아직은 NuGet 패키지를 활용할 수 없습니다. 이제 워크플로가 정상적으로 진행되는지를 확인하겠습니다. 

#### 작업 4: NuGet 패키지 업데이트

이 작업에서는 NuGet 패키지를 업데이트합니다.

1.  NuGet 소스 프로젝트가 포함된 **PartsUnlimited.Shared** 프로젝트가 열려 있는 **Visual Studio** 창으로 전환합니다.
1.  **솔루션 탐색기** 창에서 **PartsUnlimited.Shared** 프로젝트 노드를 마우스 오른쪽 단추로 클릭하고 오른쪽 클릭 메뉴에서 **추가**를 선택한 후에 계단식 메뉴에서 **새 항목**을 선택합니다.
1.  **새 항목 추가 - PartsUnlimitedShared** 대화 상자의 **Visual C# 항목** 목록에서 **클래스** 템플릿이 선택되어 있는지 확인합니다. 그런 다음 대화 상자 아래쪽의 **이름** 텍스트 상자에 **TaxService.cs**를 입력하고 **추가**를 클릭하여 클래스를 추가합니다. 

    > **참고**: 여기서는 다른 팀이 NuGet 패키지를 손쉽게 사용할 수 있도록 세금 계산 기능을 이 공유 클래스에 통합하여 중앙에서 관리한다고 가정합니다.

1.  가운데 창의 **TaxService.cs** 클래스 코드에서 기존의 클래스 정의를 다음 코드로 바꾸고 파일을 저장합니다.

    ```c#
    namespace PartsUnlimited.Shared
    {
        public class TaxService
        {
            static public decimal CalculateTax(decimal taxable, string postalCode)
            {
                return taxable * (decimal).1;
            }
        }
    }
    ```

    > **참고**: 여기서는 어셈블리와 패키지를 업데이트할 것이므로 어셈블리 버전을 업데이트해야 합니다.

1.  Visual Studio 창의 가운데 창에서 **AssemblyInfo.cs** 탭을 클릭하여 해당 파일의 내용을 표시합니다. 
1.  **AssemblyInfo.cs** 파일에서 `[assembly: AssemblyVersion("1.0.0.0")]`을 `[assembly: AssemblyVersion("1.1.0.0")]`으로 변경하고 파일을 저장합니다.
1.  **Ctrl+Shift+B**를 눌러 프로젝트를 빌드합니다.
1.  **관리자: Windows PowerShell** 창으로 전환한 후 다음 명령을 실행하여 NuGet 패키지를 다시 패키지합니다. 

    > **참고**: 새 패키지의 버전 번호가 업데이트됩니다.

    ```
    ./nuget.exe pack PartsUnlimited.Shared.csproj
    ```
1.  **관리자**로부터:** Windows PowerShell** 창에서 다음 명령을 실행하여 업데이트된 패키지를 게시합니다. 

    > **참고**: 패키지 버전 업데이트를 반영하여 게시된 아티팩트 버전 번호가 변경됩니다.

    ```
    ./nuget.exe push -source "PartsUnlimitedShared" -ApiKey VSTS PartsUnlimited.Shared.1.1.0.nupkg
    ```

1.  Azure DevOps 포털에서 **PartsUnlimitedShared 1.0.0** 아티팩트 창이 표시된 웹 브라우저 창으로 전환합니다.
1.  **PartsUnlimitedShared 1.0.0** 아티팩트 창에서 **버전** 탭을 클릭하여 **1.0.0** 및 **1.1.0** 버전이 포함되어 있는지 확인합니다. 
1.  **PartsUnlimited** 프로젝트가 표시된 **Visual Studio** 창으로 다시 전환합니다.
1.  **솔루션 탐색기** 창에서 **PartsUnlimitedWebsite\Utils\DefaultShippingTaxCalculator.cs**로 이동하여 해당 파일을 선택합니다. 그러면 솔루션 탐색기 창의 가운데 창에서 해당 파일이 자동으로 열립니다.
1.  **DefaultShippingTaxCalculator.cs** 파일의 코드 내 줄 **20**에서 **CalculateTax** 호출을 찾은 다음 `tax = CalculateTax(subTotal + shipping, postalCode);`를 `tax = PartsUnlimited.Shared.TaxService.CalculateTax(subTotal + shipping, postalCode);`로 바꿉니다.

    > **참고**: 원래 코드는 이 클래스 내부의 메서드를 호출했습니다. 이 줄 시작 부분에 추가한 코드는 해당 메서드를 NuGet 어셈블리의 코드로 리디렉션합니다. 그러나 이 프로젝트의 NuGet 패키지는 아직 업데이트되지 않았으므로 프로젝트가 계속 1.0.0.0 버전을 참조하며 새 변경 내용을 사용할 수 없습니다. 따라서 코드가 정상적으로 빌드되지 않습니다.

1.  **솔루션 탐색기** 창에서 **References** 노드를 마우스 오른쪽 단추로 클릭한 후에 오른쪽 클릭 메뉴에서 **NuGet 패키지 관리**를 선택합니다.

    > **참고**: **업데이트** 탭의 내용에 나와 있는 것처럼 NuGet이 업데이트를 인식했습니다. 

1.  **NuGet: PartsUnlimitedWebsite** 창에서 **업데이트** 탭을 클릭하고 검색 텍스트 상자에 **PartsUnlimited.Shared**를 입력합니다. 그런 다음 창 오른쪽의 버전 **안정적인 최신 버전 1.1.0** 드롭다운 목록에서 **업데이트**를 클릭하여 새 버전을 설치합니다. 

    > **참고**: 여러 NuGet 업데이트가 제공될 수도 있지만 여기서는 **PartsUnlimited.Shared**만 업데이트해야 합니다. 패키지가 완전히 업데이트 가능한 상태로 설정되려면 다소 시간이 걸릴 수 있습니다. 오류가 발생하면 잠시 기다렸다가 다시 시도하세요.

1.  메시지가 표시되면 **변경 내용 미리 보기** 대화 상자에서 **확인**을 클릭합니다.
1.  **F5** 키를 눌러 사이트를 빌드하고 실행하여 정상적으로 작동하는지 확인합니다.

#### 복습

이 랩에서는 다음 단계를 수행하여 Azure Artifacts 사용 방법을 알아보았습니다.

- 피드 만들기 및 피드에 연결
- NuGet 패키지 만들기 및 게시
- NuGet 패키지 가져오기
- NuGet 패키지 업데이트
