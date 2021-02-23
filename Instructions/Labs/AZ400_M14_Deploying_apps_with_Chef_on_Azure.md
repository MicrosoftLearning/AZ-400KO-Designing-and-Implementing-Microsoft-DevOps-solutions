---
lab:
    title: '랩: Azure에서 Chef를 사용하여 앱 배포'
    az400Module: '모듈 14: Azure에서 사용 가능한 타사 IaC(코드형 인프라) 도구'
---

# 랩: Azure에서 Chef를 사용하여 앱 배포
# 학생 랩 매뉴얼

## 랩 개요

이 랩에서는 Chef Server를 사용하여 Azure VM에 PartsUnlimted MRP 애플리케이션(PU MRP 앱)을 배포합니다. 그리고 Azure VM 기반 배포에 사용할 수 있는 Chef의 주요 기능도 살펴봅니다. 

Chef는 다음과 같은 배포 옵션을 지원합니다.

- Chef Server로 구성된 지원 대상 Linux 배포. 표준 Ubuntu 이미지 기반 Azure VM을 이러한 용도로 사용할 수 있습니다.
- 미리 구성된 Chef Automate 이미지. Chef Automate 이미지 기반 Azure VM을 배포할 수 있습니다. 해당 이미지에 Chef Server가 포함됩니다.
- 호스트형 Chef Server. https://manage.chef.io에서 호스트형 Chef Server에 가입한 다음 호스트형 서버를 사용하여 Chef 환경을 구성할 수 있습니다.

이 랩에서는 Chef Automate 이미지 기반 Azure VM을 배포하고 30일 무료 평가판 라이선스를 사용합니다. 

이 랩에서는 대략적으로 다음과 같은 단계를 진행합니다.

- Azure에서 Chef Automate 서버 배포
- Chef Automate 서버와 연결하여 상호 작용하도록 워크스테이션 구성
- 설치 자동화를 위해 PU MRP 애플리케이션 및 해당 종속성용 쿡북과 레시피 작성
- Chef Automate 서버에 쿡북 업로드
- 여러 서버에 적용할 수 있는 쿡북 및 특성의 기준 세트를 정의하는 역할 작성
- Linux VM에 구성을 배포하는 노드 작성
- 노드 Linux VM을 부트스트랩하여 역할 추가. 이 VM은 Chef를 사용하여 PU MRP 애플리케이션을 배포합니다.

## 랩 환경

랩 환경에는 Azure VM 2개와 랩 컴퓨터가 포함되어 있습니다. Azure VM에 포함된 구성 요소는 다음과 같습니다.

- Chef Server가 포함된 Chef Automate
- Chef를 사용하여 PU MRP 애플리케이션을 배포할 Chef 노드

Azure Resource Manager 템플릿을 사용하여 Azure VM 2개를 배포합니다. 랩 컴퓨터는 Chef Server에 연결하여 Chef Server를 관리하는 데 사용할 Chef 워크스테이션으로 사용됩니다. Chef 워크스테이션에서는 최신 버전 Windows Server나 클라이언트 운영 체제, Linux 또는 MacOS를 실행할 수 있습니다. 

## 목표

이 랩을 완료하면 다음 작업을 수행할 수 있습니다.

- Azure Resource Manager 템플릿을 사용하여 Chef Automate 서버 및 클라이언트 배포
- Chef 워크스테이션 구성
- Chef 쿡북과 레시피를 작성한 다음 Chef Automate 서버에 업로드
- Chef 역할을 작성하여 클라이언트에 배포

## 랩 소요 시간

-   예상 시간: **90분**

## 지침

### 시작하기 전

#### 랩 가상 머신에 로그인

다음 자격 증명을 사용하여 Windows 10 컴퓨터에 로그인했는지 확인합니다.
    
-   사용자 이름: **Student**
-   암호: **Pa55w.rd**

#### 이 랩에 필요한 애플리케이션 검토

이 랩에서 사용할 애플리케이션을 확인합니다.
    
-   Microsoft Edge

#### Azure 구독 준비

-   기존 Azure 구독을 확인하거나 새 구독을 만듭니다.
-   Azure 구독의 소유자 역할, 그리고 Azure 구독과 연결된 Azure AD 테넌트의 전역 관리자 역할이 지정되어 있는 Microsoft 계정 또는 Azure AD 계정이 있는지 확인합니다. 자세한 내용은 [Azure Portal을 사용하여 Azure 역할 할당 목록 표시](https://docs.microsoft.com/ko-kr/azure/role-based-access-control/role-assignments-list-portal) 및 [Azure Active Directory에서 관리자 역할 확인 및 할당](https://docs.microsoft.com/ko-kr/azure/active-directory/roles/manage-roles-portal#view-my-roles)을 참조하세요.

### 연습 1: Chef Server를 사용하여 Azure VM에 PartsUnlimted MRP 애플리케이션 배포 

이 연습에서는 Chef Server를 사용하여 Azure VM에 PartsUnlimted MRP 애플리케이션을 배포합니다.

#### 작업 1: Azure VM에 Chef Automate 서버 배포

이 작업에서는 Azure Resource Manager 템플릿을 사용하여 Azure VM을 배포하고 구성합니다. 이 Azure VM은 Chef Automate 서버로 사용됩니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [**Azure Portal**](https://portal.azure.com)로 이동한 다음 이 랩에서 사용하는 Azure 구독에서 Contributor 이상의 역할이 지정된 사용자 계정으로 로그인합니다.
1.  다른 브라우저 탭을 열고 [Azure Resource Manager 템플릿 링크](
https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FPartsUnlimitedMRP%2Fmaster%2FLabfiles%2FAZ-400T05-ImplemntgAppInfra%2FLabfiles%2FM04%2FDeployusingChef%2Fenv%2Fdeploychef.json)로 이동합니다. 그러면 Azure Portal의 **사용자 지정 배포** 블레이드로 자동 리디렉션됩니다.
1.  Azure Portal의 **사용자 지정 배포** 블레이드에서 다음 설정을 지정합니다. 그 외의 설정은 기본값으로 유지합니다.

    | 설정 | 값 |
    | --- | --- |
    | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
    | 리소스 그룹 | 새 리소스 그룹의 이름 **az400m14l02a-RG** |
    | 지역 | Azure VM을 배포할 수 있는 Azure 지역의 이름 |
    | 관리자 사용자 이름 | **azureuser** |
    | 관리자 암호 | **Pa55w.rd1234** |
    | 인증 유형 | **password** |
    | 진단 스토리지 계정 이름 | 문자로 시작하며 문자와 숫자 3~24자가 포함된 고유 문자열 |
    | 진단 스토리지 계정(신규 또는 기존 계정) | **신규** |
    | 위치 | Azure VM을 배포하도록 선택한 것과 같은 Azure 지역의 이름(공백 없이 입력) |
    | 공용 IP 주소 이름 | **az400m14l02b-pip** |
    | 공용 IP DNS 이름 | 문자로 시작하며 문자와 숫자 3~64자가 포함된 고유 문자열 |
    | 스토리지 계정 이름 | 진단 스토리지 계정용으로 선택한 것과 같은 이름 |
    | Virtual Network 이름 | **az400m14l02b-vnet** |
    | VM 이름 | **az400m14chs-vm** |

1.  **사용자 지정 배포** 블레이드에서 **검토 + 만들기**를 클릭하고 유효성 검사 프로세스가 완료되었는지 확인한 다음 **만들기**를 클릭합니다.

    >**참고**: 배포가 완료될 때까지 기다립니다. 약 10분이 소요됩니다.

1.  배포가 완료되면 Azure Portal에서 페이지 위쪽의 **리소스, 서비스 및 문서 검색** 텍스트 상자를 사용해 **가상 머신** 리소스를 검색하여 선택합니다. 그런 다음 **가상 머신** 블레이드에서 새로 배포된 **az400m14chs-vm** 가상 머신을 나타내는 항목을 선택합니다.
1.  **az400m14chs-vm** 블레이드에서 **DNS 이름** 항목 위에 마우스 포인터를 놓고 이름을 클립보드에 복사합니다. 
1.  새 브라우저 탭을 열고 방금 클립보드에 복사한 DNS 이름으로 이동합니다. **https** 접두사를 사용하세요. 

    >**참고**: 인증서 오류는 정상적인 현상이므로 무시하세요. 

1.  Chef Automate 로그인 메시지에 액세스할 수 있는지 확인하세요.

    >**참고**: Chef Automate에 로그인하려면 다음 작업을 완료해야 합니다. 

#### 작업 2: Chef 워크스테이션 구성

이 작업에서는 Chef Starter Kit를 설치하여 랩 컴퓨터를 Chef 워크스테이션으로 구성합니다. Chef 워크스테이션에서 Chef Server에 연결해 관리 작업을 수행하게 됩니다.

1.  랩 컴퓨터에서 다른 웹 브라우저 창을 시작하고 [Chef DK(Development Kit)](https://downloads.chef.io/chefdk) 다운로드 페이지로 이동하여 랩 가상 머신과 일치하는 Chef Development Kit 버전(Windows 10)을 선택합니다. 그런 후에 **다운로드**를 클릭하고 **ChefDK 다운로드** 팝업 창에서 요청된 연락처 정보를 입력한 후에 **다운로드**를 다시 클릭합니다.

    >**참고**: 2020년 12월 현재 Chef Development Kit의 최신 버전은 4.12.0입니다. 이 랩에서는 Windows 10용 Chef Development Kit 4.12.0-1-x64의 테스트가 완료되었습니다.

1.  Chef Development Kit 설치 관리자를 다운로드한 후 기본 설정으로 설치를 완료합니다. 그러면 표시되는 바탕 화면 바로 가기를 두 번 클릭하여 **관리자: ChefDK** PowerShell 창을 시작합니다. 
1.  **관리자: ChefDK** PowerShell 창에서 다음 명령을 실행하여 설치를 확인합니다.

    ```
    chef verify
    ```

1.  제품 라이선스를 수락하라는 메시지가 표시되면 **yes**를 입력하고 **Enter** 키를 누릅니다.
1.  확인이 실패하면 **관리자: ChefDK** PowerShell 창에서 다음 명령을 실행하여 이름과 이메일 주소를 입력해 전역 Git 변수를 구성한 다음 `chef verify`를 다시 실행합니다. 

    ```
    git config --global user.name "azureuser"
    git config --global user.email "azureuser@partsunlimitedmrp.local"
    ```

    >**참고**: 이 랩에서는 Git를 사용하여 Chef 구성 세부 정보를 저장합니다.

    >**참고**: 이 랩 뒷부분에서 PowerShell을 다시 사용할 것이므로 PowerShell 창은 열어 둡니다.

    >**참고**: Chef Automate에 연결하여 관리 작업을 수행하려면 설치한 Chef Automate를 호스트하는 Azure VM의 vmId 특성을 확인해야 합니다. vmId는 Azure에서 각 VM에 할당되는 고유 값입니다. Azure PowerShell, Azure CLI 등의 여러 가지 방법을 통해 vmId를 가져올 수 있습니다. 이 랩에서는 PuTTY를 사용합니다.

1.  랩 컴퓨터의 웹 브라우저 창에서 [PuTTY 다운로드 페이지](https://putty.org/)로 이동하여 PuTTY 설치 프로그램을 다운로드한 다음 기본 설정으로 설치를 실행합니다.
1.  설치 후에 **시작** 메뉴에서 **PuTTY (64-bit)** 폴더를 확장하고 **PuTTY** 아이콘을 클릭하여 **PuTTY Configuration** 창을 엽니다.
1.  **PuTTY Configuration** 창의 **Host Name (or IP address)** 텍스트 상자에 이전 작업 끝부분에서 확인한 DNS 이름을 입력하고 **Open**을 클릭합니다.
1.  메시지가 표시되면 **PuTTY Security Alert** 창에서 **Yes**를 클릭합니다.
1.  메시지가 표시되면 **PuTTY** 콘솔 창에서 사용자 이름 **azureuser**, 암호 **Pa55w.rd1234**를 사용하여 로그인합니다.
1.  로그인 후에 PuTTY 콘솔 창에서 다음 명령을 실행하여 vmId를 확인합니다.

    ```bash
    sudo chef-marketplace-ctl show-instance-id
    ```

    >**참고**: vmId의 값을 적어 두세요. 이 작업 뒷부분에서 해당 값이 필요합니다.

1.  PuTTY 콘솔 창에서 다음 명령을 실행하여 세션을 종료합니다.

    ```bash
    exit
    ```

    >**참고**: 이전 작업에서 설치한 Azure VM에서 실행되는 Chef Automate에 액세스하려면 Chef Starter Kit가 필요합니다. Chef Starter Kit에 액세스하려면 이전 작업 끝부분에서 확인한 DNS 이름에 /biscotti/setup을 추가합니다. 그러면 다음 형식의 URL이 생성됩니다. 여기서 `<DNS_name>` 자리 표시자는 이전 작업 끝부분에서 확인한 DNS 이름을 나타냅니다.

    ```
    https://<DNS_name>/biscotti/setup
    ```

1.  웹 브라우저 창에서 이전 단계에서 생성한 URL로 이동합니다. 인증서 관련 오류는 무시하고 **설정 권한 부여** 페이지로 이동합니다. 
1.  **설정 권한 부여** 페이지에서 **vmId**를 입력하라는 메시지가 표시되면 **PuTTY** 세션 내에서 확인한 값을 입력하고 **권한 부여**를 클릭합니다.
1.  Chef Automate 설치 세부 정보를 입력하라는 메시지가 표시되면 다음 설정을 지정하고 **Chef Automate 설정**을 클릭합니다.

    | 설정 | 값 |
    | --- | --- |
    | 이름 입력 | **Azure** |
    | 성 입력 | **사용자** |
    | 사용자 이름 입력 | **azureuser** |
    | 이메일 입력 | **azureuser@partsunlimitedmrp.local** |
    | 암호 입력 | **Pa55w.rd1234** |
    | Chef Automate 구독에 포함된 Chef Base Support 사용을 위한 지원 계정 만들기 | **꺼짐** |
    | EULA 및 마스터 계약을 읽었으며 동의합니다. | **켜기** |

    >**참고**: 이 작업을 수행하면 조직 이름으로 **default**가 적용됩니다. Chef Automate 구성 설정 과정에서는 조직 이름을 지정하지 않기 때문입니다. 원하는 경우 Chef Automate 서버 내에서 조직 이름을 지정할 수 있습니다. 이 조직 값은 랩 뒷부분에서 **knife.rb** 파일에 포함됩니다. 

1.  메시지가 표시되면 **Starter Kit 다운로드**를 클릭합니다. 그러면 starter-kit.zip 파일 다운로드가 트리거됩니다. 이 압축 파일에는 Chef를 쉽게 설정할 수 있는 미리 구성된 파일이 들어 있습니다. 

    >**참고**: Chef Starter Kit에는 사용자 자격 증명 및 인증서 세부 정보가 포함되어 있습니다. 초기 등록 시에 입력한 사용자 및 조직 세부 정보에 따라 다른 자격 증명과 세부 정보가 생성됩니다. 여러 사용자 또는 등록에서 Chef Starter Kit를 재사용하지 마세요.

    >**참고**: Chef Starter Kit를 다운로드하면 **Chef Automate에 로그인** 단추를 사용할 수 있게 됩니다. 

1.  웹 브라우저 창에서 **Chef Automate에 로그인** 단추를 클릭한 다음 **Chef Automate 설정** 웹 페이지에서 입력했던 정보(사용자 **azureuser**, 암호 **Pa55w.rd1234**)를 사용하여 Chef Automate에 로그인합니다. 그러면 Chef Automate 대시보드가 표시됩니다.
1.  랩 컴퓨터에서 파일 탐색기를 열고 **다운로드** 폴더에 있는 **Chef Starter Kit** zip 보관 파일의 압축을 풀어 해당 내용을 **C:\\Labfiles\\chef** 폴더에 저장합니다.
1.  파일 탐색기에서 **C:\\Labfiles\\chef\\chef-repo\\.chef** 폴더로 이동한 다음 메모장에서 **knife.rb** 파일을 엽니다. 
1.  **knife.rb** 파일에서 **chef_server_url** 이름에 포함되어 있는 정규화된 도메인 이름이 이전 작업에서 확인한 Azure VM DNS 이름과 일치하며 조직 이름이 **default**로 설정되어 있는지 확인한 후에 메모장 창을 닫습니다.
1.  **관리자: ChefDK** PowerShell 창으로 다시 전환한 후 다음 명령을 실행하여 새 리포지토리를 시작합니다.

    ```powershell
    Set-Location -Path 'C:\Labfiles\chef\chef-repo'
    git init
    git add -A
    git commit -m "starter kit commit"
    ```

    >**참고**: Chef Server에는 신뢰할 수 없는 SSL 인증서가 포함되어 있습니다. 그러므로 Chef 워크스테이션이 Chef Server와 통신할 수 있도록 SSL 인증서를 신뢰 항목으로 수동 설정해야 합니다. 신뢰 설정을 변경하기 위해 다음 명령을 실행하여 유효한 Chef용 SSL 인증서를 가져옵니다.

    ```chef
    knife ssl fetch
    ```

1.  **관리자: ChefDK** PowerShell 창에서 다음 명령을 실행하여 chef-repo의 현재 내용을 나열합니다.

    ```powershell
    Get-ChildItem -Path '.\' -Recurse
    ```

1.  **관리자: ChefDK** PowerShell 창에서 다음 명령을 실행하여 Chef 워크스테이션 리포지토리의 내용을 Chef Automate Azure VM 기반 리포지토리의 내용과 동기화합니다. 

    ```chef
    knife download /
    ```

    >**참고**: 이 명령을 실행하면 Chef Automate 서버에서 전체 Chef 리포지토리가 다운로드됩니다.

    >**참고**: 사용자 계정 세부 정보에 따라 acls 관련 오류가 표시될 수도 있는데 이 랩에서는 해당 오류를 무시해도 됩니다.

1.  **관리자: Chef DK** PowerShell 창에서 다음 명령을 다시 실행하여 `knife download /` 실행 후의 현재 chef-repo 내용을 나열합니다. 그리고 chef-repo 디렉터리에 추가로 작성된 파일과 폴더를 적어 둡니다.

    ```powershell
    Get-ChildItem -Path '.\' -Recurse
    ```

1.  **관리자: ChefDK** PowerShell 창에서 다음 명령을 실행하여 새로 추가된 파일을 Git 리포지토리에 커밋합니다.

    ```git
    git add -A
    git commit -m "knife download commit"
    ```

#### 작업 3: Chef 쿡북과 레시피를 작성한 다음 Chef Automate 서버에 업로드

이 작업에서는 PU MRP 애플리케이션 설치를 자동화하기 위해 PU MRP 앱 종속성용 쿡북과 레시피를 작성한 다음 Chef 서버에 업로드합니다.

1.  **관리자: ChefDK** PowerShell 창에서 다음 명령을 실행하여 Chef knife 도구를 사용해 **chef-repo**의 **cookbooks** 하위 디렉터리에 쿡북 템플릿을 생성합니다.

    ```chef
    Set-Location -Path '.\cookbooks'
    chef generate cookbook mrpapp
    ```

    >**참고**: 쿡북은 애플리케이션 또는 기능 구성용 작업 세트입니다. 쿡북은 시나리오와 해당 시나리오를 지원하는 데 필요한 모든 것을 정의합니다. 쿡북 내에는 쿡북이 수행하도록 할 작업 세트를 정의하는 일련의 레시피가 포함되어 있습니다. 쿡북과 레시피는 Ruby 언어로 작성됩니다. 위에서 chef generate cookbook 명령을 실행한 결과 **chef-repo\\cookbooks** 디렉터리에 **mrpapp** 디렉터리가 작성되었습니다. mrpapp 디렉터리에는 쿡북과 기본 레시피를 정의하는 모든 상용구 코드가 포함되어 있습니다.

1.  **관리자: ChefDK** PowerShell 창에서 다음 명령을 실행하여 편집을 위해 **metadata.rb** 파일을 엽니다.

    ```powershell
    notepad .\mrpapp\metadata.rb
    ```

    >**참고**: 쿡북과 레시피는 다른 쿡북과 레시피를 활용할 수 있습니다. 이 랩에서 사용하는 쿡북은 APT 리포지토리 관리용 기존 레시피를 사용합니다. 

1.  **metadata.rb** 파일의 내용이 표시된 메모장 창에서 파일 끝에 다음 줄을 추가하고 파일을 저장한 후에 메모장 창을 닫습니다.

    ```chef
    depends 'apt'
    ```

    >**참고**: 다음으로는 레시피용 종속성 3개(**apt** 쿡북, **windows** 쿡북, **chef-client** 쿡북)를 설치해야 합니다. 이 3개 쿡북은 [공식 Chef 쿡북 리포지토리](https://supermarket.chef.io/cookbooks)에서 다운로드한 다음 `knife cookbook site` 명령을 사용하여 설치할 수 있습니다.

1.  **관리자: ChefDK** PowerShell 창에서 다음 명령을 실행하여 `knife cookbook site` 명령을 사용해 쿡북을 다운로드한 다음 설치합니다.

    ```chef
    knife cookbook site install apt
    knife cookbook site install windows
    knife cookbook site install chef-client
    ```

    >**참고**: 다음으로는 [Microsoft Parts Unlimited MRP GitHub 리포지토리](https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/M04/DeployusingChef/final/default.rb)에서 **default.rb** 레시피의 전체 내용을 복사해야 합니다.

1.  랩 컴퓨터에서 다른 웹 브라우저 창을 시작하고 **https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/M04/DeployusingChef/final/default.rb**로 이동하여 **default.rb** 레시피를 RAW 형식으로 표시한 다음 웹 페이지의 내용을 클립보드에 복사합니다.
1.  **관리자: ChefDK** PowerShell 창에서 다음 명령을 실행하여 **C:\\Labfiles\\chef\\chef-repo\\cookbooks\\mrpapp\\recipes\\default.rb** 파일을 메모장에서 엽니다.

    ```powershell
    notepad .\mrpapp\recipes\default.rb
    ```

1.  메모장 창에서 파일 내용을 검토하여 다음과 같은지 확인합니다.

    ```chef
    #
    # Cookbook:: mrpapp
    # Recipe:: default
    #
    # Copyright:: 2020, The Authors, All Rights Reserved.
    ```

1.  메모장 창에서 클립보드의 내용을 파일 끝에 추가(새 줄 시작)하고 변경 내용을 저장한 후에 메모장 창을 닫습니다.

    >**참고**: 복사한 레시피가 애플리케이션을 프로비전하는 방법, 그리고 **default.rb** 파일에 포함된 각 코드 줄의 의미는 뒷부분에서 설명합니다.

    >**참고**: 레시피는 `apt` 리소스를 실행합니다. 따라서 레시피가 실행되기 전에 `apt-get update` 명령이 실행됩니다. 이 명령은 로컬 패키지가 최신 버전인지를 확인합니다.

    ```chef
     	# Runs apt-get update
     	include_recipe "apt"
    ```

    >**참고**: 다음으로는 **OpenJDK** 리포지토리가 apt 리포지토리에 포함되어 있으며 최신 상태인지를 확인하기 위해 `apt_repository` 리소스를 추가합니다.

    ```chef
 	# Add the Open JDK apt repo
 	apt_repository 'openJDK' do
 		uri 'ppa:openjdk-r/ppa'
 		distribution 'trusty'
 	end
    ```

    >**참고**: 다음으로는 `apt-package` 레시피를 사용하여 **OpenJDK** 및 **OpenJRE**가 설치되었는지를 확인합니다. 여기서는 헤드리스 버전을 사용합니다. 전체 버전을 사용하려면 레거시 패키지가 필요한데, 이 패키지는 사용하지 않을 것이기 때문입니다.

    ```chef
 	# Install JDK and JRE
 	apt_package 'openjdk-8-jdk-headless' do
 		action :install
 	end

 	apt_package 'openjdk-8-jre-headless' do
 		action :install
     	end
    ```

    >**참고**: 다음으로는 `JAVA_HOME` 및 `PATH` 환경 변수가 OpenJDK를 참조하도록 설정합니다.

    ```chef
 	# Set Java environment variables
 	ENV['JAVA_HOME'] = "/usr/lib/jvm/java-8-openjdk-amd64"
     	ENV['PATH'] = "#{ENV['PATH']}:/usr/lib/jvm/java-8-openjdk-amd64/bin"
    ```

    >**참고**: 다음으로는 MongoDB 데이터베이스 엔진과 Tomcat 웹 서버를 설치합니다.

    ```chef
 	# Install MongoDB
 	apt_package 'mongodb' do
 		action :install
 	end

 	# Install Tomcat 7
 	apt_package 'tomcat7' do
 		action :install
     	end
    ```

    >**참고**: 이제 종속성이 모두 설치되었습니다. 따라서 애플리케이션 구성을 시작할 수 있습니다. 먼저 MongoDB 데이터베이스에 기본 데이터가 있는지를 확인해야 합니다. `remote_file` 리소스는 지정된 위치에 파일을 다운로드합니다. 이 작업은 idempotent 방식으로 진행됩니다. 즉, 서버의 파일과 로컬 파일의 체크섬이 같으면 작업이 수행되지 않습니다. 그리고 `notifies` 명령도 사용합니다. 그러면 리소스 실행 시 파일의 새 버전이 있는 경우 지정된 리소스에 새 파일을 실행하라는 알림이 전송됩니다.

    ```chef
 	# Load MongoDB data
 	remote_file 'mongodb_data' do
 		source 'https://github.com/Microsoft/PartsUnlimitedMRP/tree/master/deploy/MongoRecords.js'
 		path './MongoRecords.js'
 		action :create
 		notifies :run, "script[mongodb_import]", :immediately
     	end
    ```

    >**참고**: 다음으로는 `script` 리소스를 사용하여 명령줄 스크립트를 설정합니다. 이 스크립트는 이전 단계에서 다운로드한 MongoDB 데이터를 로드합니다. `script` 리소스의 `action` 매개 변수는 `nothing`으로 설정됩니다. 따라서 스크립트는 실행 알림 수신 시에만 실행됩니다. 즉, 이 리소스는 이전 단계에서 지정한 `remote_file` 리소스에서 알림을 받는 경우에만 실행됩니다. 그러므로 **MongoRecord.js** 파일의 새 버전을 업로드할 때마다 레시피가 해당 파일을 다운로드하여 가져옵니다. **MongoRecords.js** 파일이 변경되지 않으면 다운로드나 가져오기는 진행되지 않습니다.

    ```chef
 	script 'mongodb_import' do
 		interpreter "bash"
 		action :nothing
 		code "mongo ordering MongoRecords.js"
     	end
    ```

    >**참고**: 다음으로는 Tomcat이 PU MRP 애플리케이션을 실행할 포트를 설정합니다. 여기서는 `script` 리소스를 사용해 정규식을 호출하여 **etc/tomcat7/server.xml** 파일을 업데이트합니다. `not_if` 작업은 가드 문입니다. `not_if` 작업의 코드가 `true`를 반환하면 리소스는 실행되지 않습니다. 그러므로 스크립트는 필요할 때만 실행됩니다.

    >**참고**: 여기서는 `#{node['tomcat']['mrp_port']}` 특성을 참조합니다. 이 값은 아직 정의하지 않았으며 다음 작업에서 정의합니다. 특성을 사용하면 변수를 설정할 수 있으므로 각 서버에서 서로 다른 포트에 PU MRP 애플리케이션을 배포할 수 있습니다. 포트가 변경되면 `notifies`를 사용하여 서비스 다시 시작을 호출합니다.

    ```chef
 	# Set tomcat port
 	script 'tomcat_port' do
 		interpreter "bash"
 		code "sed -i 's/Connector port=\".*\" protocol=\"HTTP\\/1.1\"$/Connector port=\"#{node['tomcat']['mrp_port']}\" protocol=\"HTTP\\/1.1\"/g' /etc/tomcat7/server.xml"
 		not_if "grep 'Connector port=\"#{node['tomcat']['mrp_port']}\" protocol=\"HTTP/1.1\"$' /etc/tomcat7/server.xml"
 		notifies :restart, "service[tomcat7]", :immediately
     	end
    ```

    >**참고**: 이제 PU MRP 애플리케이션을 다운로드한 다음 Tomcat에서 실행을 시작할 수 있습니다. 새 버전을 사용할 수 있게 되면 Tomcat 서비스에 애플리케이션을 다시 시작하라는 알림이 전송됩니다.

    ```chef
 	# Install the PU MRP app, restart the Tomcat service if necessary
 	remote_file 'mrp_app' do
 		source 'https://github.com/Microsoft/PartsUnlimitedMRP/tree/master/builds/mrp.war'
 		action :create
 		notifies :restart, "service[tomcat7]", :immediately
     	end
    ```

    >**참고**: Tomcat 서비스의 필요한 상태(여기서는 '서비스가 실행 중이어야 함')를 정의할 수 있습니다. 그러면 스크립트가 Tomcat 서비스의 상태를 확인한 다음 필요한 경우 서비스를 시작합니다.

    ```chef
 	# Ensure Tomcat is running
 	service 'tomcat7' do
 		action :start
 	end
    ```

    >**참고**: 마지막으로 `ordering_service`가 실행되고 있는지를 확인할 수 있습니다. 이 확인 과정에서는 `remote_file` 및 `script` 리소스 조합을 사용해 ordering_service를 종료했다가 다시 시작해야 하는지를 확인합니다. 

    ```chef
 	remote_file 'ordering_service' do
 		source 'https://github.com/Microsoft/PartsUnlimitedMRP/tree/master/builds/ordering-service-0.1.0.jar'
 		path './ordering-service-0.1.0.jar'
 		action :create
 		notifies :run, "script[stop_ordering_service]", :immediately
 	end

 	# Kill the ordering service
 	script 'stop_ordering_service' do
 		interpreter "bash"
 	# Only run when notifed
 		action :nothing
 		code "pkill -f ordering-service"
 		only_if "pgrep -f ordering-service"
 	end

 	# Start the ordering service.
 	script 'start_ordering_service' do
 		interpreter "bash"
 		code "/usr/lib/jvm/java-8-openjdk-amd64/bin/java -jar ordering-service-0.1.0.jar &"
 		not_if "pgrep -f ordering-service"
 	end
    ```

1.  **관리자: ChefDK** PowerShell 창에서 다음 명령을 실행하여 추가한 파일을 Git 리포지토리에 커밋합니다.

    ```git
    git add .
    git commit -m "mrp cookbook commit"
    ```

    >**참고**: 레시피를 만들고 종속성을 설치했으므로 쿡북과 레시피를 Chef Automate 서버에 업로드할 수 있습니다.

1.  **관리자: ChefDK** PowerShell 창에서 다음 명령을 실행하여 `knife cookbook upload` 명령을 사용해 쿡북과 레시피를 Chef Automate 서버에 업로드합니다.

    ```chef
    knife cookbook upload mrpapp --include-dependencies
    knife cookbook upload chef-client --include-dependencies
    ```

#### 작업 4: Chef 역할 만들기

이 연습에서는 `knife` 명령을 사용하여 역할을 만듭니다. 이 역할은 여러 서버에 적용할 수 있는 쿡북 및 특성의 기준 세트를 정의합니다.

>**참고**: 자세한 내용은 [Chef 설명서 페이지의 Knife 역할](https://docs.chef.io/knife_role.html)을 참조하세요.

1.  **관리자: ChefDK** PowerShell 창에서 다음 명령을 실행하여 **knife.rb** 파일을 메모장에서 엽니다.

    ```powershell
    notepad C:\Labfiles\chef\chef-repo\.chef\knife.rb
    ```

1.  메모장 창에서 다음 줄을 파일 끝에 추가(새 줄 시작)하고 변경 내용을 저장한 후에 메모장 창을 닫습니다.

    ```chef
    knife[:editor] = "notepad"
    ```

    >**참고**: 이 줄을 추가하면 다음 단계에서 역할을 만들 때 메모장이 knife.rb를 여는 편집기로 지정됩니다. knife.rb에서 편집기를 지정하지 않으면 편집기 환경 변수를 설정하라는 오류 메시지가 표시됩니다.

1.  **관리자: ChefDK** PowerShell 창에서 다음 명령을 실행하여 역할을 만듭니다. 역할의 이름은 **partsrole**입니다.

    ```chef
    knife role create partsrole
    ```

    >**참고**: 이 명령을 실행하면 메모장 창이 열리고 역할 정의 구조를 나타내는 JSON 콘텐츠가 표시됩니다.

    ```json
    {
      "name": "partsrole",
      "description": "",
      "json_class": "Chef::Role",
      "default_attributes": {

      },
      "override_attributes": {

      },
      "chef_type": "role",
      "run_list": [

      ],
      "env_run_lists": {

      }
    }
    ```

1.  메모장 창에서 `default_attributes` 섹션을 다음과 같이 변경합니다.

    ```json
      "default_attributes": {
          "tomcat": {
              "mrp_port": 9080
          }
      },
    ```

1. override_attributes를 다음과 같이 업데이트합니다.

    ```json
      "override_attributes": {
          "chef_client": {
              "interval": "60",
              "splay": "1"
          }
      },
    ```

1. run_list를 다음과 같이 업데이트합니다.

    ```json
      "run_list": [
          "recipe[mrpapp]",
          "recipe[chef-client::service]"
      ],
    ```

1.  완성된 파일의 내용이 다음과 같은지 확인하고 변경 내용을 저장한 후에 메모장 창을 닫습니다.

    ```json
    {
      "name": "partsrole",
      "description": "",
      "json_class": "Chef::Role",
      "default_attributes": {
          "tomcat": {
              "mrp_port": 9080
          }
      },
      "override_attributes": {
          "chef_client": {
              "interval": "60",
              "splay": "1"
          }
      },
      "chef_type": "role",
      "run_list": [
          "recipe[mrpapp]",
          "recipe[chef-client::service]"
      ],
      "env_run_lists": {

      }
    }
    ```

1.  **관리자: ChefDK** PowerShell 창으로 다시 전환한 후 다음 명령을 실행하여 `knife role create partsrole`이 정상적으로 완료되었는지 확인합니다. 

    >**참고**: 메모장 창은 닫으세요. 명령이 정상적으로 완료되면 `Created role[partsrole]` 메시지가 표시됩니다. 

#### 작업 5: PU MRP 앱 서버 부트스트랩 및 PU MRP 애플리케이션 배포

이 연습에서는 **knife**를 사용하여 PU MRP 애플리케이션 서버를 부트스트랩한 다음 PU MRP 애플리케이션 역할에 할당합니다.

>**참고**: 먼저 Linux VM을 프로비전합니다. 이 VM은 Chef 클라이언트로 구성되며, 이전 단계에서 만든 역할을 사용하여 이 VM에 PU MRP 애플리케이션을 배포합니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [Azure Resource Manager 템플릿 링크](
https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FPartsUnlimitedMRP%2Fmaster%2FLabfiles%2FAZ-400T05-ImplemntgAppInfra%2FLabfiles%2FM04%2FDeployusingChef%2Fenv%2Fdeploylinux.json)로 이동합니다. 그러면 Azure Portal의 **사용자 지정 배포** 블레이드로 자동 리디렉션됩니다.
1.  Azure Portal의 **사용자 지정 배포** 블레이드에서 **템플릿 편집**을 클릭합니다.
1.  **템플릿 편집** 블레이드의 템플릿 개요 창에서 **변수** 섹션을 확장하고 **vmSize**를 클릭합니다.
1.  템플릿 세부 정보 섹션에서 `"vmSize": "Standard_A1",`을 "vmSize": "Standard_D2s_v3",`으로 변경하고 **저장**을 클릭합니다.
1.  **사용자 지정 배포** 블레이드로 돌아와서 다음 설정을 지정합니다. 그 외의 설정은 기본값으로 유지합니다.

    | 설정 | 값 |
    | --- | --- |
    | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
    | 리소스 그룹 | 새 리소스 그룹의 이름 **az400m14l02b-RG** |
    | 지역 | 이 랩의 앞부분에서 Chef Automate Azure VM을 배포한 Azure 지역의 이름 |
    | 관리자 사용자 이름 | **azureuser** |
    | 관리자 암호 | **Pa55w.rd1234** |
    | DNS 레이블 접두사 | 문자로 시작하며 문자와 숫자 3~64자가 포함된 고유 문자열 |
    | Ubuntu OS 버전 | **16.04.0-LTS** |

1.  **사용자 지정 배포** 블레이드에서 **검토 + 만들기**를 클릭하고 유효성 검사 프로세스가 완료되었는지 확인한 다음 **만들기**를 클릭합니다.

    >**참고**: 배포가 완료될 때까지 기다립니다. 약 2분이 소요됩니다.

1.  배포가 완료되면 Azure Portal에서 페이지 위쪽의 **리소스, 서비스 및 문서 검색** 텍스트 상자를 사용해 **가상 머신** 리소스를 검색하여 선택합니다. 그런 다음 **가상 머신** 블레이드에서 새로 배포된 **mrpUbuntuVM** 가상 머신을 나타내는 항목을 선택합니다.
1.  **mrpUbuntuVM** 블레이드에서 **DNS 이름** 항목 위에 마우스 포인터를 놓고 이름을 클립보드에 복사합니다. 
1.  **관리자: Chef DK** PowerShell 창으로 다시 전환한 후 다음 명령을 실행하여 **knife** 도구를 사용해 새로 배포한 Azure VM을 부트스트랩합니다. 여기서 `<DNS_name>` 자리 표시자는 이전 작업 끝부분에서 확인한 DNS 이름을 나타냅니다.

    ```chef
    knife bootstrap <DNS_name> --ssh-user azureuser --ssh-password Pa55w.rd1234 --node-name mrp-app --run-list role[partsrole] --sudo --verbose
    ```
1.  연결을 계속할지 묻는 메시지가 표시되면 **Y**를 입력하고 **Enter** 키를 누릅니다.

    >**참고**: 온보딩 스크립트가 완료될 때까지 기다립니다. 약 3분이 소요됩니다. 스크립트에서 수행하는 단계는 다음과 같습니다.

    - Chef 클라이언트 구성 요소 설치
    - `mrp` Chef 역할 할당
    - `mrpapp` 레시피 실행

1.  스크립트가 완료되면 Azure Portal이 표시된 브라우저 창으로 전환하여 다른 브라우저 탭을 열고 **http://** 접두사+`mrpUbuntuVM` Azure VM의 DNS 이름+`:9080/mrp/` 접미사가 포함된 URL로 이동합니다. 최종 URL의 형식은 `http://<DNS_name>:9080/mrp/`입니다. 여기서 `<DNS_name>` 자리 표시자는 이전 작업 끝부분에서 확인한 DNS 이름을 나타냅니다.
1.  브라우저 탭에 PU MRP 애플리케이션의 방문 페이지가 표시되는지 확인합니다.
1.  Chef Automate 서버 대시보드가 표시된 브라우저 창으로 전환하여 대시보드 페이지 위쪽에서 **노드**를 클릭한 다음 레이블이 **mrp-app**인 노드 하나가 포함되어 있는지 확인합니다.

### 연습 2: Azure 랩 리소스 제거

이 연습에서는 예상치 못한 비용이 발생하지 않도록 이 랩에서 프로비전한 Azure 리소스를 제거합니다. 

>**참고**: 더 이상 사용하지 않는 새로 만든 Azure 리소스를 제거해야 합니다. 사용하지 않는 리소스를 제거하면 예기치 않은 비용이 발생하지 않습니다.

#### 작업 1: Azure 랩 리소스 제거

이 작업에서는 Azure Cloud Shell을 사용하여 불필요한 비용이 발생하지 않도록 이 랩에서 프로비전한 Azure 리소스를 제거합니다. 

1.  Azure Portal의 **Cloud Shell** 창에서 **Bash** 세션을 시작합니다.
1.  다음 명령을 실행하여 이 모듈의 전체 랩에서 생성된 모든 리소스 그룹을 나열합니다.

    ```sh
    az group list --query "[?starts_with(name,'az400m14l02')].name" --output tsv
    ```

1.  다음 명령을 실행하여 이 모듈의 전체 랩에서 만든 모든 리소스 그룹을 삭제합니다.

    ```sh
    az group list --query "[?starts_with(name,'az400m14l02')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**참고**: 명령은 비동기적으로 실행되므로(--nowait 매개 변수에 의해 결정됨) 동일한 Bash 세션 내에서 즉시 다른 Azure CLI 명령을 실행할 수 있지만 리소스 그룹이 실제로 제거되기까지 몇 분 정도 걸립니다.

## 복습

이 랩에서는 Chef Server를 사용하여 Azure VM에 PartsUnlimted MRP 애플리케이션(PU MRP 앱)을 배포하는 방법을 배웠습니다. 