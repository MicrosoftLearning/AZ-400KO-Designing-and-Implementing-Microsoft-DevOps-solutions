---
lab:
    title: '랩: Azure Key Vault와 Azure DevOps 통합'
    module: '모듈 7: 애플리케이션 구성 및 비밀 관리'
---

# 랩: Azure Key Vault와 Azure DevOps 통합
# 학생 랩 매뉴얼

## 랩 개요

Azure Key Vault에서는 키, 암호, 인증서 등의 중요한 데이터를 안전하게 저장하고 관리할 수 있습니다. Azure Key Vault는 하드웨어 보안 모듈뿐 아니라 다양한 암호화 알고리즘과 키 길이를 지원합니다. Azure Key Vault를 사용하면 소스 코드를 통해 중요한 데이터가 노출될 가능성을 최소화할 수 있습니다. 이러한 데이터 노출은 개발자가 흔히 저지르는 실수 중 하나입니다. Azure Key Vault에 액세스하려면 적절한 인증과 권한 부여를 진행해야 합니다. 따라서 Azure Key Vault에 저장된 콘텐츠에 대해 세분화된 권한을 적용할 수 있습니다.

이 랩에서는 다음 단계를 수행하여 Azure DevOps 파이프라인과 Azure Key Vault를 통합하는 방법을 살펴봅니다.

- MySQL 서버 암호를 비밀로 저장할 Azure Key Vault 만들기
- Azure 서비스 주체를 만들어 Azure Key Vault의 비밀 액세스 권한 제공
- 서비스 주체가 비밀을 읽을 수 있도록 권한 구성
- Azure Key Vault에서 암호를 검색하여 후속 작업에 전달하도록 파이프라인 구성

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

#### Azure 구독 준비

-   기존 Azure 구독을 확인하거나 새 구독을 만듭니다.
-   Azure 구독의 소유자 역할, 그리고 Azure 구독과 연결된 Azure AD 테넌트의 전역 관리자 역할이 지정되어 있는 Microsoft 계정 또는 Azure AD 계정이 있는지 확인합니다. 자세한 내용은 [Azure Portal을 사용하여 Azure 역할 할당 목록 표시](https://docs.microsoft.com/ko-kr/azure/role-based-access-control/role-assignments-list-portal) 및 [Azure Active Directory에서 관리자 역할 확인 및 할당](https://docs.microsoft.com/ko-kr/azure/active-directory/roles/manage-roles-portal#view-my-roles)을 참조하세요.

#### Azure DevOps 조직 설정

이 랩에 사용할 수 있는 Azure DevOps 조직이 아직 없으면 [조직 또는 프로젝트 컬렉션 만들기](https://docs.microsoft.com/ko-kr/azure/devops/organizations/accounts/create-organization?view=azure-devops)에서 제공되는 지침에 따라 조직을 만듭니다.

### 연습 0: 랩 필수 구성 요소 구성

이 연습에서는 랩의 필수 구성 요소를 설정합니다. 구체적으로는 Azure DevOps 데모 생성기 템플릿을 기반으로 하여 미리 구성된 Parts Unlimited 팀 프로젝트를 설정합니다.

#### 작업 1: 팀 프로젝트 구성

이 작업에서는 Azure DevOps 데모 생성기를 사용하여 **Azure Key Vault** 템플릿을 기반으로 새 프로젝트를 생성합니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [Azure DevOps 데모 생성기](https://azuredevopsdemogenerator.azurewebsites.net)로 이동합니다. 이 유틸리티 사이트에서는 계정 내에서 새 Azure DevOps 프로젝트를 만드는 프로세스를 자동으로 진행할 수 있습니다. 이 프로젝트에는 랩에 필요한 작업 항목, 리포지토리 등의 콘텐츠가 미리 입력되어 있습니다. 

    > **참고**: 이 사이트에 대한 자세한 내용은 https://docs.microsoft.com/ko-kr/azure/devops/demo-gen을 참조하세요.

1.  **로그인**을 클릭하고 Azure DevOps 구독과 연결된 Microsoft 계정을 사용하여 로그인합니다.
1.  필요한 경우 **Azure DevOps 데모 생성기** 페이지에서 **수락**을 클릭하여 Azure DevOps 구독에 액세스하는 데 필요한 권한 요청을 수락합니다.
1.  **새 프로젝트 만들기** 페이지의 **새 프로젝트 이름** 텍스트 상자에 **Azure Key Vault와 Azure DevOps 통합**을 입력합니다. 그런 다음 **조직 선택** 드롭다운 목록에서 Azure DevOps 조직을 선택하고 **템플릿 선택**을 클릭합니다.
1.  **템플릿 선택** 페이지의 머리글 메뉴에서 **DevOps** 랩을 클릭하고 템플릿 목록에서 **Azure Key Vault** 템플릿을 클릭한 다음 **템플릿 선택**을 클릭합니다.
1.  **새 프로젝트 만들기** 페이지로 돌아와서 **ARM 출력** 레이블 아래의 체크박스를 선택하고 **프로젝트 만들기**를 클릭합니다.

    > **참고**: 프로세스가 완료될 때까지 기다립니다. 약 2분이 소요됩니다. 프로세스가 실패하면 DevOps 조직으로 이동하여 프로젝트를 삭제한 후에 다시 시도합니다.

1.  **새 프로젝트 만들기** 페이지에서 **프로젝트로 이동**을 클릭합니다.

### 연습 1: Azure DevOps와 Azure Key Vault 통합

- Azure Key Vault의 비밀 액세스 권한을 제공할 Azure 서비스 주체 만들기
- MySQL 서버 암호를 비밀로 저장할 Azure Key Vault 만들기
- 서비스 주체가 비밀을 읽을 수 있도록 권한 구성
- Azure Key Vault에서 암호를 검색하여 후속 작업에 전달하도록 파이프라인 구성

#### 작업 1: 서비스 주체 만들기 

이 작업에서는 Azure CLI를 사용하여 서비스 주체를 만듭니다. 

> **참고**: 서비스 주체가 이미 있으면 다음 작업을 바로 진행해도 됩니다.

Azure Pipelines에서 Azure 리소스에 앱을 배포하려면 서비스 주체가 필요합니다. 여기서는 파이프라인의 비밀을 검색할 것이므로 Azure Key Vault를 만들 때 서비스에 대한 권한을 부여해야 합니다. 

프로젝트 설정 페이지에서 새 서비스 연결을 만들거나 파이프라인 정의 내에서 Azure 구독에 연결하면 Azure Pipelines에서 서비스 주체가 자동으로 작성됩니다. Azure Portal이나 Azure CLI를 통해 서비스 주체를 수동으로 만든 다음 여러 프로젝트에서 재사용할 수도 있습니다. 미리 정의된 권한 집합을 적용하려는 경우에는 기존 서비스 주체를 사용하는 것이 좋습니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하여 [**Azure Portal**](https://portal.azure.com)로 이동한 다음 이 랩에서 사용할 Azure 구독의 소유자 역할, 그리고 해당 구독과 연결된 Azure AD 테넌트의 전역 관리자 역할이 지정된 사용자 계정으로 로그인합니다.
1.  Azure Portal의 페이지 위쪽 검색 텍스트 상자 바로 오른쪽에 있는 **Cloud Shell** 아이콘을 클릭합니다. 
1.  **Bash** 또는 **PowerShell**을 선택하라는 메시지가 표시되면 **Bash**를 선택합니다. 

   >**참고**: **Cloud Shell**을 처음 시작하고 **탑재된 스토리지가 없음** 메시지를 받으면, 이 랩에서 사용하는 구독을 선택하고 **스토리지 만들기**를 선택합니다. 

1.  **Cloud Shell** 창의 **Bash** 프롬프트에서 다음 명령을 실행하여 서비스 주체를 만듭니다. 여기서 `<service-principal-name>`은 문자와 숫자로 구성된 고유한 문자열로 바꿉니다.

    ```
    az ad sp create-for-rbac --name <service-principal-name>
    ```

    > **참고**: 이 명령을 실행하면 JSON 출력이 생성됩니다. 텍스트 파일에 출력을 복사해 두세요. 이 랩의 뒷부분에서 해당 내용이 필요합니다.

1.  **Cloud Shell** 창의 **Bash** 프롬프트에서 다음 명령을 실행하여 Azure 구독 ID 및 구독 이름 특성의 값을 검색합니다. 

    ```
    az account show --query id --output tsv
    az account show --query name --output tsv
    ```

    > **참고**: 두 값을 모두 텍스트 파일에 복사해 두세요. 이 랩의 뒷부분에서 두 값이 필요합니다.


#### 작업 2: Azure Key Vault 만들기

이 작업에서는 Azure Portal을 사용하여 Azure Key Vault를 만듭니다.

이 랩 시나리오에서는 앱이 MySQL 데이터베이스에 연결됩니다. 이 MySQL 데이터베이스의 암호를 키 자격 증명 모음에 비밀로 저장하려고 합니다.

1.  Azure Portal의 **리소스, 서비스 및 문서 검색** 텍스트 상자에 **키 자격 증명 모음**을 입력하고 **Enter** 키를 누릅니다. 
1.  **키 자격 증명 모음** 블레이드에서 **+ 추가**를 클릭합니다. 
1.  **키 자격 증명 모음 만들기** 블레이드의 **기본** 탭에서 다음 설정을 지정하고 **다음: 액세스 정책**을 클릭합니다.

    | 설정 | 값 |
    | --- | --- |
    | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
    | 리소스 그룹 | 새 리소스 그룹의 이름 **az400m07l01-RG** |
    | Key Vault 이름 | 고유하고 유효한 이름 |
    | 지역 | 랩 환경의 위치와 가까운 Azure 지역 |
    | 가격 책정 계층 | **표준** |
    | 삭제된 자격 증명 모음을 보존할 기간(일) | **7** |
    | 제거 보호 | **제거 보호 사용 안 함** |

1.  **키 자격 증명 모음 만들기** 블레이드의 **액세스 정책** 탭에서 **+ 액세스 정책 추가**를 클릭하여 새 정책을 설정합니다.

    > **참고**: 권한이 부여된 애플리케이션과 사용자에게만 액세스를 허용하는 방식으로 키 자격 증명 액세스 보안을 유지해야 합니다. 이렇게 하면 자격 증명 모음의 데이터에 액세스할 때 파이프라인에서 인증에 사용할 서비스 주체에 읽기(가져오기) 권한을 제공해야 합니다. 

1.  **액세스 정책 추가** 블레이드에서 **보안 주체 추가** 레이블 바로 아래에 있는 **선택된 항목 없음** 링크를 클릭합니다. 
1.  **보안 주체** 블레이드에서 이전 연습에서 만든 보안 주체를 검색하여 선택하고 **선택**을 클릭합니다. 

    > **참고**: 보안 주체는 이름이나 ID로 검색할 수 있습니다.

1.  **액세스 정책 추가** 블레이드로 돌아와서 **비밀 권한** 드롭다운 목록에서 **가져오기** 및 **나열** 권한 옆의 체크박스를 선택하고 **추가**를 클릭합니다. 
1.  **키 자격 증명 모음 만들기** 블레이드의 **액세스 정책** 탭으로 돌아와서 **검토 + 만들기**를 클릭하고 **검토 + 만들기** 블레이드에서 **만들기**를 클릭합니다. 

    > **참고**: Azure Key Vault가 프로비전될 때까지 기다립니다. 1분 미만이 소요됩니다.

1.  **배포가 완료됨** 블레이드에서 **리소스로 이동**을 클릭합니다.
1.  Azure Key Vault 블레이드 왼쪽의 세로 메뉴에 있는 **설정** 메뉴에서 **비밀**을 클릭합니다. 
1.  **비밀** 블레이드에서 **생성/가져오기**를 클릭합니다.
1.  **비밀 만들기** 블레이드에서 다음 설정을 지정한 후에 **만들기**를 클릭합니다. 다른 값은 모두 기본값으로 유지합니다.

    | 설정 | 값 |
    | --- | --- |
    | 업로드 옵션 | **수동** |
    | 이름 | **sqldbpassword** |
    | 값 | 유효한 MySQL 암호 값 |


#### 작업 3: Azure Pipelines 확인

이 작업에서는 Azure Key Vault에서 비밀을 검색하도록 Azure Pipelines를 구성합니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 이전 연습에서 만든 Azure DevOps 프로젝트 **Azure Key Vault와 Azure DevOps 통합**으로 이동합니다.
1.  Azure DevOps 포털의 세로 탐색 창에서 **Pipelines**를 선택하고 **Pipelines** 창이 표시되는지 확인합니다.
1.  **파이프라인** 창에서 **SmartHotel-CouponManagement-CI** 파이프라인을 나타내는 항목을 클릭하고 **SmartHotel-CouponManagement-CI** 창에서 **파이프라인 실행**을 클릭합니다.
1.  **파이프라인 실행** 창에서 기본 설정을 수락하고 **실행**을 클릭하여 빌드를 트리거합니다.
1.  Azure DevOps 포털의 세로 탐색 창에 있는 **Pipelines** 섹션에서 **릴리스**를 선택합니다. 
1.  **SmartHotel-CouponManagement-CI** 창 오른쪽 위의 **편집**을 클릭합니다.
1.  **모든 파이프라인 > SmartHotel-CouponManagement-CI** 창에서 **작업** 탭을 선택하고 드롭다운 메뉴에서 **개발**을 선택합니다.

    > **참고**: **개발** 단계의 릴리스 정의에 **Azure Key Vault** 작업이 포함되어 있습니다. 이 작업에서는 Azure Key Vault에서 *비밀*을 다운로드합니다. 랩 앞부분에서 만든 Azure Key Vault 리소스와 구독을 지정해야 합니다.

    > **참고**: Azure에 배포할 수 있는 권한을 파이프라인에 부여해야 합니다. Azure Pipelines는 새 서비스 주체를 사용하여 서비스 연결을 자동으로 만들 수 있습니다. 하지만 여기서는 앞에서 만든 연결을 사용합니다. 

1.  **Azure Key Vault** 작업을 선택하고 오른쪽의 **Azure Key Vault** 작업 속성에서 **Azure 구독** 레이블 옆에 있는 **관리**를 클릭합니다. 
그러면 다른 브라우저 탭이 열리고 Azure DevOps 포털의 **서비스 연결** 창이 표시됩니다.
1.  **서비스 연결** 창에서 **새 서비스 연결**을 클릭합니다. 
1.  **새 서비스 연결** 창에서 **Azure Resource Manager** 옵션을 선택하고 **다음**을 클릭합니다. 그런 후에 **서비스 주체(수동)** 를 선택하고 **다음**을 다시 클릭합니다.
1.  **새 서비스 연결** 창에서 다음 설정을 지정합니다. 이 연습의 첫 번째 작업에서 Azure CLI를 사용해 서비스 주체를 만든 후 텍스트 파일에 복사한 정보를 사용하세요.

- 구독 ID: `az account show --query id --output tsv` 실행 결과로 반환된 값
- 구독 이름: `z account show --query name --output tsv` 실행 결과로 반환된 값
- 서비스 주체 ID: `az ad sp create-for-rbac --name <service-principal-name>` 실행 결과 생성된 출력에서 레이블이 **appId**인 값
- 서비스 주체 키: `az ad sp create-for-rbac --name <service-principal-name>` 실행 결과 생성된 출력에서 레이블이 **password**인 값
- 테넌트 ID: `az ad sp create-for-rbac --name <service-principal-name>` 실행 결과 생성된 출력에서 레이블이 **tenant**인 값

1.  **새 서비스 연결** 창에서 **확인**을 클릭하여 입력한 정보가 유효한지를 확인합니다. 
1.  **확인 성공** 응답이 표시되면 **서비스 연결 이름** 텍스트 상자에 **kv-service-connection**을 입력하고 **확인 및 저장**을 클릭합니다.
1.  **Azure Key Vault** 작업이 표시된 웹 브라우저 탭으로 다시 전환합니다. 
1.  **Azure Key Vault** 작업을 선택한 상태로 **Azure Key Vault** 창에서 **새로 고침** 단추를 클릭하고 **Azure 구독** 드롭다운 목록에서 **kv-service-connection** 항목을 선택합니다. 그런 다음 **키 자격 증명 모음** 드롭다운 목록에서 첫 번째 작업에서 만든 Azure Key Vault를 나타내는 항목을 선택하고 **비밀 필터** 텍스트 상자에 **sqldbpassword**를 입력합니다. 마지막으로 **출력 변수** 섹션을 확장하고 **참조 이름** 텍스트 상자에 **sqldbpassword**를 입력합니다. 

    > **참고**: 이제 Azure Pipelines에서 런타임에 비밀의 최신 값을 가져온 다음 작업 변수 **$(sqldbpassword)** 로 설정합니다. 후속 작업에서 해당 변수를 참조하는 방식으로 이 작업을 사용할 수 있습니다.  

1.  이 변수를 확인하려면 다음 작업인 **Azure 배포**를 선택합니다. 이 작업에서는 ARM 템플릿을 배포한 다음 **템플릿 매개 변수 재정의** 텍스트 상자의 내용을 검토합니다. 

    ```
    -webAppName $(webappName) -mySQLAdminLoginName "azureuser" -mySQLAdminLoginPassword $(sqldbpassword)
    ```

    > **참고**: **템플릿 매개 변수 재정의** 콘텐츠는 **sqldbpassword** 변수를 참조하여 mySQL 관리자 암호를 설정합니다. 그러면 키 자격 증명 모음에서 지정한 암호를 사용하여 ARM 템플릿에 정의된 MySQL 데이터베이스가 프로비전됩니다. 

    > **참고**: 작업의 위치와 구독을 지정하여 파이프라인 정의를 완료할 수 있습니다. 이렇게 하려면 **Azure App Service 배포**. 파이프라인의 마지막 작업에서 같은 과정을 반복하세요. 그리고 마지막으로 파이프라인을 저장하여 새 릴리스를 만들고 배포를 시작하면 됩니다.

### 연습 2: Azure 랩 리소스 제거

이 연습에서는 예상치 못한 비용이 발생하지 않도록 이 랩에서 프로비전한 Azure 리소스를 제거합니다. 

>**참고**: 더 이상 사용하지 않는 새로 만든 Azure 리소스를 제거해야 합니다. 사용하지 않는 리소스를 제거하면 예기치 않은 비용이 발생하지 않습니다.

#### 작업 1: Azure 랩 리소스 제거

이 작업에서는 Azure Cloud Shell을 사용하여 불필요한 비용이 발생하지 않도록 이 랩에서 프로비전한 Azure 리소스를 제거합니다. 

1.  Azure Portal의 **Cloud Shell** 창에서 **Bash** 세션을 시작합니다.
1.  다음 명령을 실행하여 이 모듈의 전체 랩에서 생성된 모든 리소스 그룹을 나열합니다.

    ```sh
    az group list --query "[?starts_with(name,'az400m07l01-RG')].name" --output tsv
    ```

1.  다음 명령을 실행하여 이 모듈의 전체 랩에서 만든 모든 리소스 그룹을 삭제합니다.

    ```sh
    az group list --query "[?starts_with(name,'az400m07l01-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**참고**: 명령은 비동기적으로 실행되므로(--nowait 매개 변수에 의해 결정됨) 동일한 Bash 세션 내에서 즉시 다른 Azure CLI 명령을 실행할 수 있지만 리소스 그룹이 실제로 제거되기까지 몇 분 정도 걸립니다.

#### 복습

이 랩에서는 다음 단계를 수행하여 Azure DevOps 파이프라인과 Azure Key Vault를 통합했습니다.

- MySQL 서버 암호를 비밀로 저장할 Azure Key Vault 만들기
- Azure 서비스 주체를 만들어 Azure Key Vault의 비밀 액세스 권한 제공
- 서비스 주체가 비밀을 읽을 수 있도록 권한 구성
- Azure Key Vault에서 암호를 검색하여 후속 작업에 전달하도록 파이프라인 구성
