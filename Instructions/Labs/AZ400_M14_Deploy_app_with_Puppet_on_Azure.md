---
lab:
    title: '랩: Azure에서 Puppet을 사용하여 앱 배포'
    module: '모듈 14: Azure에서 사용 가능한 타사 IaC(코드형 인프라) 도구'
---

# 랩: Azure에서 Puppet을 사용하여 앱 배포
# 학생 랩 매뉴얼

## 랩 개요

이 랩에서는 Puppet Labs의 Puppet을 사용하여 Azure VM에 PartsUnlimted MRP 애플리케이션(PU MRP 앱)을 배포합니다. 

Puppet은 IaC(코드 제공 인프라) 상태 설명을 제공하여 머신 프로비전과 구성을 자동화할 수 있는 구성 관리 시스템입니다. 

이 랩에서는 대략적으로 다음과 같은 단계를 진행합니다.

- Azure Resource Manager 템플릿을 사용하여 Azure VM에서 Puppet 마스터 및 노드 프로비전
- 노드에 Puppet 에이전트 설치
- Puppet 프로덕션 환경 구성
- 프로덕션 환경 구성 테스트
- PU MRP 앱의 필수 구성 요소 설명을 제공하는 Puppet 프로그램 만들기
- 노드에서 Puppet 구성 실행

## 랩 환경

랩 환경에는 Azure VM 2개와 랩 컴퓨터가 포함되어 있습니다. Azure VM에 포함된 구성 요소는 다음과 같습니다.

- PU MRP 앱을 호스트할 Puppet 노드. 노드에서는 Puppet 에이전트 설치 작업만 수행하면 됩니다. Puppet 에이전트는 Linux나 Windows에서 실행할 수 있습니다. 이 랩에서는 Ubuntu Server를 사용합니다.
- Puppet 프로그램을 통해 노드에 적용한 구성을 관리하는 Puppet 마스터. Puppet 마스터는 Linux에서 실행해야 합니다. 이 랩에서는 에이전트와 마찬가지로 Ubuntu Server를 사용합니다.

Azure Resource Manager 템플릿을 사용하여 Azure VM 2개를 배포합니다. 랩 컴퓨터는 Puppet 마스터에 연결하여 마스터를 관리하는 데 사용할 관리 워크스테이션으로 사용됩니다. 

## 목표

이 랩을 완료하면 다음 작업을 수행할 수 있습니다.

- Azure Resource Manager 템플릿을 사용하여 Azure VM에서 Puppet 마스터 및 노드 프로비전
- 노드에 Puppet 에이전트 설치
- Puppet 프로덕션 환경 구성
- 프로덕션 환경 구성 테스트
- Puppet 프로그램 만들기
- 노드에서 Puppet 구성 실행

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

### 연습 1: Puppet을 사용하여 Azure VM에 PartsUnlimted MRP 애플리케이션 배포

이 연습에서는 Puppet을 사용하여 Azure VM에 PartsUnlimted MRP 애플리케이션을 배포합니다.

#### 작업 1: Puppet 마스터로 사용할 Azure VM 및 해당 노드 프로비전

이 작업에서는 Azure Resource Manager 템플릿을 사용하여 Azure VM 2개를 배포하고 구성합니다. 첫 번째 VM이 Puppet 마스터로 사용되며 두 번째 VM은 마스터의 관리 노드가 됩니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [**Azure Portal**](https://portal.azure.com)로 이동한 다음 이 랩에서 사용하는 Azure 구독에서 Contributor 이상의 역할이 지정된 사용자 계정으로 로그인합니다.
1.  다른 브라우저 탭을 열고 [Azure Resource Manager 템플릿 링크](
https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FPartsUnlimitedMRP%2Fmaster%2FLabfiles%2FAZ-400T05-ImplemntgAppInfra%2FLabfiles%2FM04%2FPuppet%2Fenv%2FPuppetPartsUnlimitedMRP.json)로 이동합니다. 그러면 Azure Portal의 **사용자 지정 배포** 블레이드로 자동 리디렉션됩니다.

1.  Azure Portal의 **사용자 지정 배포** 블레이드에서 **템플릿 편집**을 클릭합니다.
1.  **템플릿 편집** 블레이드의 템플릿 개요 창에서 **변수** 섹션을 확장하고 **vmSize**를 클릭합니다.
1.  템플릿 세부 정보 섹션에서 `"pmVmSize": "Standard_D2_V2",`를 `"vmSize": "Standard_D2s_v3",`으로 변경합니다.
1.  템플릿 세부 정보 섹션에서 `"mrpVmSize": "Standard_A2",`를 `"vmSize": "Standard_D2s_v3",`으로 변경하고 **저장**을 클릭합니다.
1.  **사용자 지정 배포** 블레이드로 돌아와서 다음 설정을 지정합니다. 그 외의 설정은 기본값으로 유지합니다.

    | 설정 | 값 |
    | --- | --- |
    | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
    | 리소스 그룹 | 새 리소스 그룹의 이름 **az400m14l03a-RG** |
    | 지역 | Azure VM을 배포할 수 있는 Azure 지역의 이름 |
    | PM 관리자 사용자 이름 | **azureuser** |
    | PM 관리자 암호 | **Pa55w.rd1234** |
    | 공용 IP용 PM DNS 이름 | 문자로 시작하며 문자와 숫자 3~64자가 포함된 고유 문자열 |
    | PM 콘솔 암호 | **Pa55w.rd1234** |
    | MRP 관리자 사용자 이름 | **azureuser** |
    | MRP 관리자 암호 | **Pa55w.rd1234** |
    | 공용 IP용 MRP DNS 이름 | 문자로 시작하며 문자와 숫자 3~64자가 포함된 고유 문자열 |

    >**참고**: 배포 중에 지정한 DNS 이름을 적어 두세요 첫 번째 이름(**PM**)이 Puppet 마스터에 할당됩니다. 두 번째 이름(**MRP**)은 Puppet 노드에 할당됩니다. 

1.  **사용자 지정 배포** 블레이드에서 **검토 + 만들기**를 클릭하고 유효성 검사 프로세스가 완료되었는지 확인한 다음 **만들기**를 클릭합니다.

    >**참고**: 배포가 완료될 때까지 기다립니다. 약 10분이 소요됩니다.

1.  배포가 완료되면 Azure Portal에서 페이지 위쪽의 **리소스, 서비스 및 문서 검색** 텍스트 상자를 사용해 **가상 머신** 리소스를 검색하여 선택합니다. 그런 다음 **가상 머신** 블레이드에서 Puppet 노드로 사용할 새로 배포된 Azure VM을 나타내는 항목을 선택합니다.
1.  가상 머신 블레이드에서 **DNS 이름** 속성의 값을 확인한 다음 적어 둡니다.
1.  **가상 머신** 블레이드로 돌아와서 Puppet 마스터를 호스트하는 새로 배포된 Azure VM을 나타내는 항목을 선택합니다.
1.  가상 머신 블레이드에서 **DNS 이름** 항목 위에 마우스 포인터를 놓고 이름을 클립보드에 복사합니다. 
1.  새 브라우저 탭을 열고 방금 클립보드에 복사한 DNS 이름으로 이동합니다. **https** 접두사를 사용하세요. 그러면 Puppet 마스터 콘솔 로그인 페이지가 표시됩니다.

    >**참고**: 인증서 오류는 정상적인 현상이므로 무시하세요. 

1.  Puppet 마스터 콘솔 로그인 페이지가 표시된 웹 브라우저 창에서 사용자 이름으로 **admin**을, 암호로 **Pa55w.rd1234**를 사용하여 로그인합니다. 그러면 Puppet 구성 관리 콘솔이 표시됩니다.

    >**참고**: 배포 중에 지정한 사용자 이름으로는 Puppet 마스터 콘솔에 로그인할 수 없으며 기본 제공 관리 사용자 계정을 대신 사용해야 합니다.

#### 작업 2: 노드 Azure VM에 Puppet 에이전트 설치

이 작업에서는 두 번째 Azure VM을 Puppet 마스터가 관리하는 노드로 추가합니다. 

1.  Puppet 구성 관리 콘솔이 표시된 웹 브라우저 왼쪽의 세로 메뉴에서 **노드**, **서명되지 않은 인증서**를 차례로 클릭하고 Puppet Enterprise에서 관리하는 노드를 추가할 수 있는 명령을 적어 둡니다. 

    >**참고**: 이 작업 뒷부분에서 이 명령을 사용합니다. 이 명령은 다음과 비슷한 형식입니다. 여기서 `<Pm_Dns_Name_for_Public_IP>` 자리 표시자는 이전 작업에서 배포했던 두 번째 Azure VM의 IP 주소에 할당한 이름을 나타냅니다.

    ```bash
    curl -k https://<Pm_Dns_Name_for_Public_IP>.0h03b1jj0ewetnu40a35rbm0qg.bx.internal.cloudapp.net:8140/packages/current/install.bash | sudo bash
    ``` 

1.  랩 컴퓨터의 웹 브라우저 창에서 [PuTTY 다운로드 페이지](https://putty.org/)로 이동하여 PuTTY 설치 프로그램을 다운로드한 다음 기본 설정으로 설치를 실행합니다.
1.  설치 후에 **시작** 메뉴에서 **PuTTY (64-bit)** 폴더를 확장하고 **PuTTY** 아이콘을 클릭하여 **PuTTY Configuration** 창을 엽니다.
1.  **PuTTY Configuration** 창의 **Host Name (or IP address)** 텍스트 상자에 이전 작업의 끝부분에서 확인한 Puppet 노드를 호스트하려는 Azure VM의 DNS 이름을 입력하고 **Open**을 클릭합니다.
1.  메시지가 표시되면 **PuTTY Security Alert** 창에서 **Yes**를 클릭합니다.
1.  메시지가 표시되면 Puppet 노드로 연결되는 **PuTTY** 세션에서 사용자 이름 **azureuser**, 암호 **Pa55w.rd1234**를 사용하여 로그인합니다.
1.  로그인 후에 Puppet 노드로 연결되는 PuTTY 세션에서 이 작업 앞부분에서 적어 두었던 명령을 실행합니다.

    >**참고**: 명령이 Puppet 에이전트 및 모든 종속성을 노드에 설치할 때까지 기다립니다. 완료되려면 약 2분이 소요됩니다. 이제부터는 Puppet 마스터만 사용하여 노드를 구성합니다.

    >**참고**: Azure Marketplace의 Puppet 에이전트 확장을 사용하면 Azure VM에서 Puppet 에이전트를 설치하고 구성하는 과정을 자동화할 수 있습니다.

    >**참고**: 다음으로는 새로 설치한 노드를 관리하기 위해 Puppet 구성 관리 콘솔에서 보류 중인 요청을 수락해야 합니다.

1.  Puppet 구성 관리 콘솔이 표시된 웹 브라우저 창으로 다시 전환한 다음 **서명되지 않은 인증서** 페이지를 새로 고쳐서 페이지에 보류 중인 서명되지 않은 인증서 요청이 표시되는지 확인합니다. 그런 다음 **수락**을 클릭해 요청을 승인하여 노드를 인벤토리에 추가합니다.

    >**참고**: 이 요청은 Puppet 마스터와 노드 간의 통신을 보호하는 데 사용할 인증서에 대한 권한 부여 요청입니다.

1.  Puppet 구성 관리 콘솔이 표시된 웹 브라우저 창에서 **노드**를 클릭한 다음 각각 Puppet 마스터와 해당 노드를 나타내는 항목 2개가 표시되는지 확인합니다.

    >**참고**: Parts Unlimited MRP 애플리케이션(PU MRP 앱)은 Java 애플리케이션입니다. PU MRP 앱을 사용하려면 Puppet 노드로 구성된 Azure VM에서 MongoDB 및 Apache Tomcat을 설치하고 구성해야 합니다. 하지만 여기서는 MongoDB 및 Tomcat을 수동으로 설치하고 구성하는 대신 노드를 자동 구성하는 Puppet 프로그램을 작성합니다. Puppet 마스터의 특정 디렉터리에 저장되는 Puppet 프로그램은 대상 노드 하나 이상의 필요한 상태를 설명하는 매니페스트로 구성됩니다. 매니페스트는 사전 패키지된 Puppet 프로그램인 모듈을 사용할 수 있습니다. 사용자는 모듈을 직접 만들 수도 있고 Puppet Labs에서 유지 관리하는 모듈인 'The Forge'를 Marketplace를 통해 사용할 수도 있습니다. The Forge에는 공식 지원되는 모듈도 있고 커뮤니티에서 업로드하는 오픈 소스 모듈도 있습니다. Puppet 프로그램은 환경별로 구성되어 있습니다. 따라서 개발, 테스트, 프로덕션 등의 각 환경용 Puppet 프로그램을 관리할 수 있습니다.

    >**참고**: 이 랩에서는 새로 온보딩한 노드를 사용하여 프로덕션 환경을 에뮬레이트합니다. 그리고 The Forge의 모듈도 다운로드하여 노드를 구성하는 데 사용합니다.

#### 작업 3: Puppet 프로덕션 환경 구성

이 작업에서는 에뮬레이트한 Puppet 프로덕션 환경을 구성합니다. 

>**참고**: Azure에 Puppet을 배포하는 데 사용하는 템플릿은 Puppet 마스터에 프로덕션 환경 관리용 디렉터리도 구성합니다. 해당 디렉터리는 **etc/puppetlabs/code/environments/production**입니다.

>**참고**: 먼저 프로덕션 모듈을 검사해야 합니다.

1.  랩 컴퓨터의 **시작** 메뉴에서 **PuTTY (64-bit)** 폴더를 확장하고 **PuTTY** 아이콘을 클릭하여 **PuTTY Configuration** 창을 엽니다.
1.  **PuTTY Configuration** 창의 **Host Name (or IP address)** 텍스트 상자에 Puppet 마스터를 호스트하는 Azure VM의 DNS 이름을 입력하고 **Open**을 클릭합니다.
1.  메시지가 표시되면 **PuTTY Security Alert** 창에서 **Yes**를 클릭합니다.
1.  메시지가 표시되면 Puppet 마스터로 연결되는 **PuTTY** 세션에서 사용자 이름 **azureuser**, 암호 **Pa55w.rd1234**를 사용하여 로그인합니다.
1.  정상적으로 인증이 되면 Puppet 마스터로 연결되는 PuTTY 세션에서 다음 명령을 실행하여 현재 작업 디렉터리를 프로덕션 디렉터리 **etc/puppetlabs/code/environments/production**으로 변경합니다.

    ```bash
    cd /etc/puppetlabs/code/environments/production
    ```

1.  Puppet 마스터로 연결되는 PuTTY 세션에서 `ls` 명령을 실행하여 프로덕션 디렉터리의 콘텐츠 목록을 표시합니다. 

    >**참고**: **manifests** 및 **modules** 디렉터리가 표시됩니다. **manifests** 디렉터리에는 이 랩의 뒷부분에서 샘플 노드에 적용할 구성의 설명이 포함되어 있습니다. **modules** 디렉터리에는 매니페스트가 참조하는 모듈이 포함되어 있습니다.

    >**참고**: 다음으로는 샘플 노드를 구성하는 데 필요한 추가 Puppet 모듈을 The Forge에서 설치합니다. 

1.  Puppet 마스터로 연결되는 PuTTY 세션 내에서 다음 명령을 실행하여 필요한 모듈을 설치합니다.

    ```bash
    sudo puppet module install puppetlabs-mongodb
    sudo puppet module install puppetlabs-tomcat
    sudo puppet module install maestrodev-wget
    sudo puppet module install puppetlabs-accounts
    sudo puppet module install puppetlabs-java
    ```

    >**참고**: The Forge의 mongodb 및 tomcat 모듈은 공식 지원 모듈입니다. wget 모듈은 공식 지원되지 않는 사용자 모듈입니다. accounts 모듈은 Linux에서 사용자와 그룹을 만들고 관리하는 데 필요한 클래스를 Puppet에 제공합니다. 마지막으로 java 모듈은 추가 Java 기능을 Puppet에 제공합니다.

    >**참고**: 다음으로는 Puppet 마스터의 **modules** 디렉터리에 사용자 지정 모듈 **mrpapp**를 만듭니다. 이 사용자 지정 모듈이 PU MRP 앱을 구성합니다. 

1.  Puppet 마스터로 연결되는 PuTTY 세션에서 다음 명령을 실행하여 현재 디렉터리를 **etc/puppetlabs/code/environments/production/modules**로 변경합니다.

    ```bash
    cd /etc/puppetlabs/code/environments/production/modules
    ```

1.  Puppet 마스터로 연결되는 PuTTY 세션에서 다음 명령을 실행하여 **mrpapp** 모듈을 만듭니다.

    ```bash
    sudo puppet module generate partsunlimited-mrpapp
    ```

    >**참고**: 그러면 마법사가 시작되며 모듈 작성 과정에서 일련의 질문이 표시됩니다. 

1.  **mrapp** 모듈 작성을 완료하려면 마법사가 완료될 때까지 각 질문에서 **Enter** 키를 눌러 빈 값이나 기본값을 그대로 적용합니다.

    >**참고**: `ls -la` 명령을 사용하면 새 모듈이 작성되었는지를 확인할 수 있습니다. `ls -la` 명령을 실행하면 숨김 파일(`-a`)을 포함한 디렉터리의 콘텐츠가 긴 목록 형식(`-l`)으로 나열됩니다.

    >**참고**: mrpapp 모듈은 노드 구성을 정의합니다. 프로덕션 환경의 노드 구성은 **site.pp** 파일에서 정의됩니다. **site.pp** 파일은 **manifests** 디렉터리에 있습니다. **pp** 파일 이름 확장명은 **Puppet Program(Puppet 프로그램)** 의 머리글자어입니다. 여기서는 노드의 구성을 추가하여 **site.pp** 파일을 편집합니다.

1.  Puppet 마스터로 연결되는 PuTTY 세션에서 다음 명령을 실행하여 Nano 텍스트 편집기에서 **site.pp**를 엽니다.

    ```bash
    sudo nano /etc/puppetlabs/code/environments/production/manifests/site.pp
    ```

1.  Nano 편집기 인터페이스 내에서 파일 아래쪽으로 스크롤하여 기존 `node default` 섹션을 다음 섹션으로 바꿉니다.

    ```bash
    node default {
       class { 'mrpapp': }
    }
    ```

1.  Nano 편집기 인터페이스 내에서 **Ctrl+O** 키 조합, **Enter** 키, **Ctrl+X** 키 조합을 차례로 눌러 변경 내용을 저장하고 파일을 닫습니다.

    >**참고**: **site.pp** 파일에 적용한 편집 내용은 기본적으로 **mrpapp** 모듈을 사용하여 모든 노드를 구성하도록 Puppet에 명령합니다. 현재는 비어 있는 상태인 **mrpapp** 모듈은 프로덕션 환경의 **modules** 디렉터리에 있으므로 Puppet은 모듈을 찾을 위치를 알 수 있습니다.

#### 작업 4: 프로덕션 환경 구성 테스트 

이 작업에서는 프로덕션 환경의 구성을 테스트합니다. 

>**참고**: 샘플 노드용 PU MRP 앱의 전체 설명을 제공하기 전에 **mrpapp** 모듈에서 더미 파일을 설정하여 구성을 테스트합니다. Puppet이 더미 파일을 정상적으로 실행하고 생성하는 경우 **mrpapp** 모듈의 체 구성을 진행할 수 있습니다.

>**참고**: 여기서는 먼저 **mrpapp** 모듈의 진입점인 **init.pp** 파일을 편집합니다. 이 파일은 Puppet 마스터의 **mrpapp/manifests** 디렉터리에 있습니다. 

1.  Puppet 마스터로 연결되는 PuTTY 세션에서 다음 명령을 실행하여 Nano 텍스트 편집기에서 **init.pp** 파일을 엽니다.

    ```bash
    sudo nano /etc/puppetlabs/code/environments/production/modules/mrpapp/manifests/init.pp
    ```

1.  Nano 편집기 인터페이스 내에서 `class: mrpapp` 선언으로 스크롤하여 해당 선언을 다음과 같이 수정합니다.

    ```puppet
    class mrpapp {
       file { '/tmp/dummy.txt':
          ensure => 'present',
          content => 'Puppet rules!'
       }
    }
    ```

    >**참고**: 지침과 같이 class 정의 앞의 주석 태그(`#`)를 제거해야 합니다.

1.  Nano 편집기 인터페이스 내에서 **Ctrl+O** 키 조합, **Enter** 키, **Ctrl+X** 키 조합을 차례로 눌러 변경 내용을 저장하고 파일을 닫습니다.

    >**참고**: Puppet 프로그램의 클래스는 개체 지향 프로그래밍의 클래스와는 달리 노드에 구성되어 있는 리소스를 정의합니다. 여기서는 **init.pp** 파일에 `mrpapp` 클래스(리소스)를 추가하여 /tmp/dummy.txt 경로에 파일이 있으며 해당 파일에 "Puppet rules!"라는 내용이 있는지를 확인하도록 Puppet에 명령했습니다. 랩을 진행하면서 `mrpapp` 클래스 내에서 고급 리소스를 더 정의해 보겠습니다.

    >**참고**: 이제 새로 설정한 구성의 테스트를 수행합니다. 

1.  Puppet 노드로 연결되는 PuTTY 세션으로 전환한 후에 다음 명령을 실행하여 새 구성을 테스트합니다.

    ```bash
    sudo puppet agent --test --debug
    ```

    >**참고**: 기본적으로 Puppet 에이전트는 30분마다 Puppet 마스터를 쿼리하여 구성을 확인합니다. 그런 다음 Puppet 마스터가 지정한 구성을 기준으로 현재 구성을 테스트합니다. 그리고 필요한 경우에는 Puppet 마스터가 지정한 구성과 일치하도록 구성을 수정합니다. 여기서 호출한 명령은 로컬 Puppet 에이전트가 Puppet 마스터를 기준으로 수행하는 구성 확인을 즉시 트리거합니다. 여기서는 구성에 **tmp/dummy.txt** 파일이 있어야 하므로 노드는 이 파일을 만듭니다.

1.  Puppet 노드로 연결되는 PuTTY 세션 내에서 다음 명령을 실행하여 **tmp/dummy.txt** 파일이 있는지를 확인하고 해당 파일 내용의 목록을 표시합니다.

    ```bash
    cat /tmp/dummy.txt
    ```

1.  명령의 출력으로 "Puppet rules!" 메시지가 나타나는지 확인합니다. 

    >**참고**: 다음으로는 구성 드리프트를 수정합니다. Puppet은 Puppet 에이전트가 실행될 때마다 환경이 올바른 상태인지를 확인하여 올바른 상태가 아니면 관련 클래스를 필요한 대로 다시 적용합니다. 이 프로세스를 통해 Puppet은 구성 드리프트를 검색하여 수정할 수 있습니다. 여기서는 노드에서 더미 파일 **dummy.txt**를 삭제하여 구성 드리프트를 시뮬레이트합니다. 

1.  Puppet 노드로 연결되는 PuTTY 세션 내에서 다음 명령을 실행하여 **dummy.txt** 파일을 삭제합니다.

    ```bash
    sudo rm /tmp/dummy.txt
    ```

1.  Puppet 노드로 연결되는 PuTTY 세션 내에서 다음 명령을 실행하여 Puppet 에이전트의 실행을 트리거합니다.

    ```bash
    sudo puppet agent --test
    ```

1.  Puppet 노드로 연결되는 PuTTY 세션 내에서 다음 명령을 실행하여 **tmp/dummy.txt** 파일이 있는지를 확인하고 해당 파일 내용의 목록을 표시합니다.

    ```bash
    cat /tmp/dummy.txt
    ```

#### 작업 5: PU MRP 앱의 필수 구성 요소 설명을 제공하는 Puppet 프로그램 만들기

이 작업에서는 PU MRP 앱의 필수 구성 요소 설명을 제공하는 Puppet 프로그램을 만듭니다.

>**참고**: 앞에서 Puppet 마스터에 노드를 후크했으므로 PU MRP 앱의 필수 구성 요소 설명을 제공하는 Puppet 프로그램을 작성할 수 있습니다. 실제 환경에서는 대형 구성 솔루션의 각 부분이 대개 여러 매니페스트나 노드로 분할됩니다. 구성을 여러 파일로 분할하여 "모듈화"하면 코드를 쉽게 재사용할 수 있습니다. 이 랩에서는 작업을 쉽게 수행할 수 있도록 앞에서 만든 mrpapp 모듈 내의 Puppet 프로그램 파일 **init.pp** 하나에 전체 구성의 설명을 포함합니다. 이 작업에서는 **init.pp** 파일 작성 프로세스를 단계별로 진행합니다.

>**참고**: 완성된 **init.pp** 파일은 [Microsoft PU MRP GitHub 리포지토리](https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/M04/Puppet/final/init.pp)에서 확인할 수 있습니다. GitHub에서 **init.pp** 파일 내용을 복사할 수도 있고 이 작업에서 편집을 단계별로 진행할 수도 있습니다.

>**참고**: Tomcat 서비스에 액세스하고 서비스를 실행하려면 노드에 Linux 사용자 및 그룹이 있어야 합니다. 이러한 사용자와 그룹에 Tomcat이 사용하는 기본 이름은 `Tomcat`입니다. 이러한 구성은 여러 가지 방식으로 구현할 수 있습니다. 여기서는 이러한 용도로 **init.pp** 파일에서 별도의 클래스를 사용합니다. 이 구현을 위해 **init.pp** 파일을 편집하는 동시에 다음 수정 작업을 수행합니다.

- PU MRP 앱을 간편하게 배포할 수 있도록 **war.pp** 파일을 편집합니다.
- 이 작업을 마칠 때 **war** 파일을 추출할 수 있도록 **~/tomcat7/webapps** 디렉터리에 대한 권한을 편집합니다. 이러한 권한을 편집하면 **pp** 파일 실행 시 Tomcat 서비스가 다시 시작될 때 **war** 파일을 자동으로 추출할 수 있습니다.

>**참고**: 이 작업의 개별 섹션에 포함된 단계를 진행할 때 Puppet 프로그램을 만들려면 수행해야 하는 모든 작업의 설명이 제공됩니다. 

##### 섹션 5.1: MongoDB 구성

>**참고**: MongoDB를 구성한 후에는 Puppet이 PU MRP 애플리케이션의 데이터베이스용 데이터가 포함된 Mongo 스크립트를 다운로드하도록 해야 합니다. 이 작업은 MongoDB 설정의 일부분으로 포함할 예정입니다. 이러한 요구 사항을 구현하려면 MongoDB를 구성하는 클래스를 추가합니다.

1.  Puppet 마스터로 연결되는 PuTTY 세션 내에서 다음 명령을 실행하여 Nano 텍스트 편집기에서 **mrpapp 모듈** 내의 **init.pp** 파일을 엽니다.

    ```bash
    sudo nano /etc/puppetlabs/code/environments/production/modules/mrpapp/manifests/init.pp
    ```

1.  Nano 편집기 인터페이스 내에서 파일 아래쪽으로 스크롤하여 새 줄에 `configuremongodb`를 추가합니다.

    ```puppet
    class configuremongodb {
      include wget
      class { 'mongodb': }->

      wget::fetch { 'mongorecords':
        source => 'https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/deploy/MongoRecords.js',
        destination => '/tmp/MongoRecords.js',
        timeout => 0,
      }->
      exec { 'insertrecords':
        command => 'mongo ordering /tmp/MongoRecords.js',
        path => '/usr/bin:/usr/sbin',
        unless => 'test -f /tmp/initcomplete'
      }->
      file { '/tmp/initcomplete':
        ensure => 'present',
      }
    }
    ```

1.  Nano 편집기 인터페이스 내에서 **Ctrl+O** 키 조합과 **Enter** 키를 차례로 눌러 변경 내용을 저장하고 파일은 열어 둡니다. 

    >**참고**: 여기서 잠시 `configuremongodb` 클래스를 살펴보겠습니다.

    줄 1. `configuremongodb`라는 `class(리소스)`를 만듭니다.
    줄 2. `wget`을 통해 파일을 다운로드할 수 있도록 `wget` 모듈을 포함합니다.
    줄 3. 앞에서 다운로드한 `mongodb` 모듈에서 `mongodb 리소스`를 호출합니다. 그러면 Puppet Mongo DB 모듈에 정의된 기본값을 사용하여 Mongo DB가 설치됩니다. 이 작업만 수행하면 Mongo DB를 설치할 수 있습니다.
    줄 5. `wget` 모듈에서 `fetch` 리소스를 호출하고 이 fetch 리소스를 통해 `mongorecords` 리소스를 호출합니다.
    줄 6. PU MRP GitHub에서 다운로드해야 하는 파일의 원본을 설정합니다.
    줄 7. 파일을 다운로드해야 하는 대상을 설정합니다.
    줄 10. 기본 제공 Puppet 리소스 `exec`를 사용하여 명령을 실행합니다.
    줄 11. 실행할 명령을 지정합니다.
    줄 12. 명령 호출 경로를 설정합니다.
    줄 13. `unless` 키워드를 사용하여 조건을 지정합니다. 명령은 한 번만 실행할 것이므로 레코드를 삽입한 후에 `tmp` 파일을 만듭니다(줄 15). 이 파일이 있으면 명령을 다시 실행하지 않습니다.

    >**참고**: 줄 3, 9, 14에 `->`로 표기된 코드는 순서 지정 화살표입니다. 이 화살표는 오른쪽 리소스를 호출하기 전에 왼쪽 리소스를 적용해야 한다는 것을 Puppet에 지시합니다. 따라서 실행 순서를 제어할 수 있습니다.

##### 섹션 5.2: Java 구성

1.  Nano 편집기 인터페이스 내에서 **init.pp** 파일 끝부분에 있는 `configuremongodb` 클래스 정의 바로 아래에 다음 `configurejava` 클래스 정의를 추가하여 Java 요구 사항을 설정합니다.

    ```puppet
    class configurejava {
      include apt
      $packages = ['openjdk-8-jdk', 'openjdk-8-jre']

      apt::ppa { 'ppa:openjdk-r/ppa': }->
      package { $packages:
         ensure => 'installed',
      }
    }
    ```

1.  Nano 편집기 인터페이스 내에서 **Ctrl+O** 키 조합과 **Enter** 키를 차례로 눌러 변경 내용을 저장하고 파일은 열어 둡니다. 

    >**참고**: 여기서 잠시 `configurejava` 클래스를 살펴보겠습니다.

    줄 2. `apt` 모듈을 포함합니다. 이 모듈을 사용하면 새 **PPA(개인 패키지 아카이브)** 를 구성할 수 있습니다.
    줄 3. 설치해야 하는 패키지 배열을 만듭니다.
    줄 5. **PPA**를 추가합니다.
    줄 6-8. 패키지가 설치되었는지 확인하도록 Puppet에 명령합니다. 그러면 Puppet이 배열을 확장하여 배열의 각 패키지를 설치합니다.

    >**참고**: Puppet 패키지 대상을 사용하여 Java를 설치할 수는 없습니다. 이렇게 하면 Java 7만 설치되기 때문입니다. 여기서는 Java 8을 설치하기 위해 PPA를 추가하고 `apt` 모듈을 사용합니다.

##### 섹션 5.3: 사용자 및 그룹 만들기

1.  Nano 편집기 인터페이스 내에서 **init.pp** 파일 끝부분에 다음 `createuserandgroup` 클래스를 추가하여 서비스 구성, 배포, 시작, 중지에 사용할 Linux 사용자와 그룹을 지정합니다.

    ```puppet
    class createuserandgroup {

    group { 'tomcat':
      ensure => 'present',
      gid    => '10003',
     }

    user { 'tomcat':
      ensure           => 'present',
      gid              => '10003',
      home             => '/tomcat',
      password         => '!',
      password_max_age => '99999',
      password_min_age => '0',
      uid              => '1003',
     }
    }
    ```

1.  Nano 편집기 인터페이스 내에서 **Ctrl+O** 키 조합과 **Enter** 키를 차례로 눌러 변경 내용을 저장하고 파일은 열어 둡니다. 

    >**참고**: 여기서 잠시 `createuserandgroup` 클래스를 살펴보겠습니다.

    줄 1. **init.pp** 파일 시작 부분에서 정의한 `createuserandgroup` 클래스를 호출합니다. 이 클래스는 `user` 및 `group` 리소스를 사용하기 위해 정의한 것입니다.
    줄 3-6. `group` 명령을 사용하여 `Tomcat` 그룹을 만듭니다. 매개 변수 값 `ensure = > present`를 사용하여 `Tomcat` 그룹이 있는지를 확인하고 필요하면 해당 그룹을 만듭니다. 그런 다음 그룹에 `gid` 값을 할당합니다. 이 값을 반드시 할당할 필요는 없지만 할당하는 경우 여러 머신에서 그룹을 쉽게 관리할 수 있습니다.
    줄 8-17. `user` 명령을 사용하여 `Tomcat` 사용자를 만듭니다. 그리고 `ensure` 명령을 사용하여 `Tomcat` 사용자가 있는지를 확인하고 필요하면 해당 사용자를 만듭니다. 또한 사용자를 쉽게 관리할 수 있도록 `gid` 및 `uid` 값을 할당합니다. 그런 다음 `Tomcat` 사용자의 홈 디렉터리 경로를 설정하고 사용자 암호 설정을 할당합니다. 여기서는 손쉬운 사용을 위해 `password` 값은 지정하지 않은 상태로 유지합니다. 실제로는 엄격한 암호 정책을 구현해야 합니다.

    >**참고**: `user` 및 `group` 명령은 이 랩 앞부분에서 Puppet 마스터에 설치한 `puppetlabs-accounts` 모듈을 통해 제공됩니다. 

##### 섹션 5.4: Tomcat 구성

1.  Nano 편집기 인터페이스 내에서 **init.pp** 파일 끝부분에 다음 `configuretomcat` 클래스를 추가하여 Tomcat을 구성합니다.

    ```puppet
    class configuretomcat {
      class { 'tomcat': }
      require createuserandgroup

     tomcat::instance { 'default':
      catalina_home => '/var/lib/tomcat7',
      install_from_source => false,
      package_name => ['tomcat7','tomcat7-admin'],
     }->

     tomcat::config::server::tomcat_users {
     'tomcat':
       catalina_base => '/var/lib/tomcat7',
       element  => 'user',
       password => 'password',
       roles => ['manager-gui','manager-jmx','manager-script','manager-status'];
     'tomcat7':
       catalina_base => '/var/lib/tomcat7',
       element  => 'user',
       password => 'password',
       roles => ['manager-gui','manager-jmx','manager-script','manager-status'];
     }->

     tomcat::config::server::connector { 'tomcat7-http':
      catalina_base => '/var/lib/tomcat7',
      port => '9080',
      protocol => 'HTTP/1.1',
      connector_ensure => 'present',
      server_config => '/etc/tomcat7/server.xml',
     }->

     tomcat::service { 'default':
      use_jsvc => false,
      use_init => true,
      service_name => 'tomcat7',
     }
    }
    ```

1.  Nano 편집기 인터페이스 내에서 **Ctrl+O** 키 조합과 **Enter** 키를 차례로 눌러 변경 내용을 저장하고 파일은 열어 둡니다. 

    >**참고**: 여기서 잠시 `configuretomcat` 클래스를 살펴보겠습니다.

    줄 1. `configuretomcat`이라는 `class(리소스)`를 만듭니다.
    줄 2. 앞에서 다운로드한 Tomcat 모듈에서 `tomcat` 리소스를 호출합니다.
    줄 3. Tomcat을 구성하려면 `Tomcat` 사용자와 그룹을 사용 가능하도록 설정해야 하므로 `createuserandgroup` 클래스를 필수로 표시합니다.
    줄 6-9. 앞에서 다운로드한 Tomcat 모듈에서 `Tomcat` 패키지를 설치합니다. 설치를 위해 여기서 패키지 이름인 `Tomcat7`을 호출합니다. 그리고 기본 설치를 수행할 홈 디렉터리를 지정합니다. 그런 다음 원본에서 설치를 수행하지 않음을 지정합니다. 다운로드한 패키지에서 설치할 때는 이와 같이 지정해야 합니다. 원하는 경우 웹 원본에서 직접 패키지를 설치할 수도 있습니다. `tomcat7-admin` 패키지도 설치합니다. 이 랩에서는 `tomcat7-admin` 패키지를 사용하지 않지만 Tomcat Manager 웹 사용자 인터페이스를 제공하려는 경우에는 이 패키지를 사용할 수 있습니다.
    줄 13-23. Tomcat에서 사용하도록 할 사용자 이름 2개를 만듭니다. 이러한 이름은 Tomcat 관리 및 구성용으로 Tomcat 애플리케이션 내에서 생성됩니다. 구체적으로는 **var/lib/tomcat7/conf/tomcat-users.xml** 파일 내에서 이름이 생성되어 호출됩니다. 
    줄 26-31. Tomcat 커넥터를 구성하고 사용할 프로토콜과 포트 번호를 정의합니다. 그리고 설정한 포트의 요청을 처리할 수 있는 커넥터가 있는지도 확인합니다. 또한 Puppet이 Tomcat **server.xml** 파일에 데이터를 쓸 수 있도록 커넥터 속성도 지정합니다. 여기서도 Puppet 마스터의 **var/lib/tomcat7/conf/server.xml**에서 파일을 열어 해당 내용을 확인할 수 있습니다.
    줄 35-38. Tomcat 서비스를 구성합니다. Tomcat은 한 번만 설치하므로 구성 대상으로 `default`를 지정합니다. 또한 `use_init` 값을 설정하여 서비스가 실행 중인지 확인합니다. 그런 후에 PU MRP 앱용 커넥터 이름을 `tomcat7`로 지정합니다.

##### 섹션 5.5: WAR 파일 배포

>**참고**: PU MRP 앱은 .war 파일로 컴파일됩니다. Tomcat에서는 이 파일을 사용하여 페이지를 제공합니다. 여기서는 PU MRP 앱 사이트용 .war 파일을 배포하기 위한 리소스를 지정해야 합니다.

1.  Nano 편집기 인터페이스 내에서 **init.pp** 파일 끝부분에 다음 `deploywar` 클래스를 추가합니다.

    ```puppet
    class deploywar {
      require configuretomcat

      tomcat::war { 'mrp.war':
        catalina_base => '/var/lib/tomcat7',
        war_source => 'https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/builds/mrp.war',
      }

     file { '/var/lib/tomcat7/webapps/':
       path => '/var/lib/tomcat7/webapps/',
       ensure => 'directory',
       recurse => 'true',
       mode => '777',
     }
    }
    ```

1.  Nano 편집기 인터페이스 내에서 **Ctrl+O** 키 조합과 **Enter** 키를 차례로 눌러 변경 내용을 저장하고 파일은 열어 둡니다. 

    >**참고**: 여기서 잠시 deploywar 클래스를 살펴보겠습니다.

    줄 1. `deploywar`라는 `class(리소스)`를 만듭니다.
    줄 2. `deploywar` 클래스를 호출하기 전에 `configuretomcat`이 완료되었는지 확인하도록 Puppet에 명령합니다.
    줄 5. Puppet이 Tomcat 서비스에 .war을 배포하도록 `catalina_base` 디렉터리를 설정합니다.
    줄 6. Tomcat 모듈의 `war` 리소스를 사용하여 `war_source`에서 .war을 배포합니다.
    줄 9. Tomcat 모듈에 정의된 `file` 클래스를 호출합니다. `file` 클래스를 통해 시스템에서 파일을 호출할 수 있습니다.
    줄 10. PU MRP 앱용 웹 디렉터리의 경로를 지정합니다.
    줄 11. 구성을 적용하기 전에 지정한 경로에 디렉터리가 있는지 확인합니다.
    줄 12. 구성이 `~/tomcat7/webapps/` 디렉터리 내의 모든 파일과 하위 디렉터리에 적용되도록 구성을 재귀적으로 적용해야 함을 지정합니다.
    줄 13. `~/tomcat7/webapps/` 디렉터리 내의 모든 파일과 하위 디렉터리에 대한 `777` 권한을 지정합니다. 대상 디렉터리 내의 모든 콘텐츠에 대한 읽기, 쓰기, 실행 권한을 허용하려면 이러한 권한을 설정해야 합니다. 이 랩에서는 `777` 권한을 설정하면 됩니다. 프로덕션 환경에서는 각 사용자, 그룹 및 액세스 수준에 따라 항상 더 제한적인 권한을 설정해야 합니다.

    >**참고**: 일반적으로는 .war 파일을 배포하려면 다음 작업을 수행해야 합니다.

    1. Tomcat **var/lib/tomcat7/webapps** 디렉터리에 .war 파일을 복사합니다.
    2. **webapps** 디렉터리에 대한 올바른 권한을 설정합니다.
    3. **var/lib/tomcat7/conf/server.xml**의 Tomcat 서버 구성 파일에서 `unpackWARS` 및 `autoDeploy`의 값을 지정합니다.
    4. Tomcat 파일 **var/lib/tomcat7/conf/tomcat-users.xml**에 유효한 사용자와 사용 가능한 사용자의 목록을 저장합니다.

    >**참고**: 서비스가 시작되면 Tomcat이 .war을 추출한 다음 배포합니다. 그러므로 **init.pp** 파일에서 서비스를 다시 시작해야 합니다. 이 랩을 완료할 때 Tomcat 파일 **server.xml** 및 **tomcat-users.xml**을 열어서 내용을 확인할 수 있습니다.

##### 섹션 5.6: 오더링 서비스 시작

>**참고**: PU MRP 앱은 Mongo DB에서 순서를 관리하는 REST API인 오더링 서비스를 호출합니다. 오더링 서비스는 .jar 파일로 컴파일됩니다. 이 .jar 파일을 노드에 복사해야 합니다. 그러면 REST API 요청을 수신 대기할 수 있도록 노드가 백그라운드에서 오더링 서비스를 반환합니다.

1.  Nano 편집기 인터페이스 내에서 **init.pp** 파일 끝부분에 다음 `orderingservice` 클래스를 추가하여 오더링 서비스가 실행되는지 확인합니다.

    ```puppet
    class orderingservice {
      package { 'openjdk-7-jre':
        ensure => 'installed',
      }

      file { '/opt/mrp':
        ensure => 'directory'
      }->
      wget::fetch { 'orderingsvc':
        source => 'https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/builds/ordering-service-0.1.0.jar',
        destination => '/opt/mrp/ordering-service.jar',
        cache_dir => '/var/cache/wget',
        timeout => 0,
      }->
  
      exec { 'stoporderingservice':
        command => "pkill -f ordering-service",
        path => '/bin:/usr/bin:/usr/sbin',
        onlyif => "pgrep -f ordering-service"
      }->

      exec { 'stoptomcat':
        command => 'service tomcat7 stop',
        path => '/bin:/usr/bin:/usr/sbin',
        onlyif => "test -f /etc/init.d/tomcat7",
      }->
      exec { 'orderservice':
        command => 'java -jar /opt/mrp/ordering-service.jar &',
        path => '/usr/bin:/usr/sbin:/usr/lib/jvm/java-8-openjdk-amd64/bin',
      }->
      exec { 'wait':
        command => 'sleep 20',
        path => '/bin',
        notify => Tomcat::Service['default']
      }
    }
    ```

1.  Nano 편집기 인터페이스 내에서 **Ctrl+O** 키 조합과 **Enter** 키를 차례로 눌러 변경 내용을 저장하고 파일은 열어 둡니다. 

    >**참고**: 여기서 잠시 orderingservice 클래스를 살펴보겠습니다.

    줄 1. `orderingservice`라는 `class(리소스)`를 만듭니다.
    줄 2-4. Puppet의 패키지 리소스를 사용하여 애플리케이션을 실행하는 데 필요한 JRE(Java Runtime Environment)를 설치합니다.
    줄 6-8. `/opt/mrp` 디렉터리가 있는지를 확인하고 필요하면 해당 디렉터리를 만듭니다.
    줄 9-14. `wget`을 사용하여 오더링 서비스 바이너리를 검색한 다음 `/opt/mrp`에 저장합니다.
    줄 12. 파일이 한 번만 다운로드되도록 캐시 디렉터리를 지정합니다.
    줄 15-19. orderingservice가 실행되고 있으면 중지합니다.
    줄 20-24. tomcat7 서비스가 실행되고 있으면 중지합니다.
    줄 25-28. 오더링 서비스를 시작합니다.
    줄 29-33. 오더링 서비스가 시작될 때까지 20초 동안 기다렸다가 tomcat 서비스에 알림을 보냅니다. 이러한 작업을 수행하면 서비스 새로 고침이 트리거됩니다. 그러면 Puppet이 서비스용으로 정의한 상태를 다시 적용합니다. 예를 들어 서비스가 아직 실행되고 있지 않으면 서비스를 시작합니다.

    >**참고**: java 명령을 실행한 후에는 잠시 기다려야 합니다. 이 서비스가 실행 중이어야 Tomcat을 시작할 수 있기 때문입니다. 이렇게 하지 않으면 오더링 서비스가 들어오는 API 요청을 수신 대기하는 데 필요한 포트를 Tomcat이 사용하게 됩니다.

##### 섹션 5.7: mrpapp 리소스 완성

1.  Nano 편집기 인터페이스 내에서 **init.pp** 파일 위쪽으로 스크롤한 다음 랩 앞부분에서 테스트했던 **dummy.txt** 파일 관련 내용을 삭제해 `mrpapp` 클래스 정의를 제거합니다. 
1.  Nano 편집기 인터페이스에서 업데이트된 `mrpapp` 클래스 정의를 추가합니다.

    ```puppet
    class mrpapp {
      class { 'configuremongodb': }
      class { 'configurejava': }
      class { 'createuserandgroup': }
      class { 'configuretomcat': }
      class { 'deploywar': }
      class { 'orderingservice': }
    }
    ```

    >**참고**: `mrpapp` 클래스의 수정 내용은 리소스를 실행하려면 **init.pp** 파일에 필요한 클래스를 지정합니다.

1.  Nano 편집기 인터페이스 내에서 **Ctrl+O** 키 조합, **Enter** 키, **Ctrl+X** 키 조합을 차례로 눌러 변경 내용을 저장하고 파일을 닫습니다.

    >**참고**: 필요한 사항을 모두 수정한 **init.pp** 파일은 [Microsoft PU MRP GitHub 리포지토리](https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/M04/Puppet/final/init.pp)의 **init.pp** 파일과 같아야 합니다. 원하는 경우 GitHub에서 **init.pp** 파일 내용을 복사하여 로컬 **init.pp** 파일에 붙여넣을 수도 있습니다.

##### 섹션 5.8: .war 파일 추출 권한 구성

>**참고**: **init.pp** 파일에 정의한 구성에는 PU MRP 앱용 .war 파일을 **var/lib/tomcat7/webapps** 디렉터리에 복사하는 작업이 포함됩니다. Tomcat은 webapps 디렉터리에서 .war 파일을 자동으로 추출합니다. 그러나 이 랩에서는 파일이 정상적으로 추출되도록 **war.pp** 파일에 전체 읽기, 쓰기 및 액세스 권한을 설정합니다.

1.  Puppet 마스터로 연결되는 PuTTY 세션 내에서 다음 명령을 실행하여 Nano 편집기에서 **war.pp** 파일을 엽니다.

    ```bash
    sudo nano /etc/puppetlabs/code/environments/production/modules/tomcat/manifests/war.pp
    ```

1.  Nano 편집기 인터페이스 내에서 파일 아래쪽으로 스크롤한 다음 기존 `mode => '0640'` 항목을 `mode => '0777'로 바꿔 원하는 권한을 수정합니다. 그러면 다음 콘텐츠가 반환됩니다.

    ```puppet
        file { "tomcat::war ${name}":
          ensure    => file,
          path      => "${_deployment_path}/${_war_name}",
          owner     => $user,
          group     => $group,
          mode      => '0777',
          subscribe => Archive["tomcat::war ${name}"],
        }
      }
    }
    ```

1.  Nano 편집기 인터페이스 내에서 **Ctrl+O** 키 조합, **Enter** 키, **Ctrl+X** 키 조합을 차례로 눌러 변경 내용을 저장하고 파일을 닫습니다.

    >**참고**: 여기서 잠시 war.pp 파일의 이 섹션을 살펴보겠습니다.

    줄 1. Tomcat 모듈 `tomcat::war`를 통해 제공되는 파일 이름을 지정합니다.
    줄 2. 작업을 수행하기 전에 파일이 있는지 확인합니다.
    줄 3. 이전에 `/var/lib/tomcat7/webapps`로 설정한 웹 디렉터리의 경로를 정의합니다. PU MRP 앱용 .war 파일은 **init.pp** 파일에서 지정한 대로 이 디렉터리에 복사됩니다.
    줄 4-5. 파일의 사용자 및 그룹 소유자를 설정합니다.
    줄 6. 파일 권한 모드를 `0777`로 설정합니다. 그러면 모든 사용자에게 읽기, 쓰기 및 실행 권한이 부여됩니다. 프로덕션 환경에서는 `777` 권한을 사용하면 안 됩니다.

#### 작업 6: Puppet 노드에서 Puppet 구성 실행

이 작업에서는 Puppet 노드에서 구성 업데이트를 트리거한 다음 이 업데이트를 트리거하면 PU MRP 앱이 정상적으로 배포되는지를 확인합니다.

1.  Puppet 노드로 연결되는 PuTTY 세션으로 전환한 후에 다음 명령을 실행하여 새 구성을 테스트합니다.

    ```bash
    sudo puppet agent --test
    ```

    >**참고**: 이 명령을 처음 실행하면 모든 필수 구성 요소를 다운로드하여 설치해야 하므로 몇 분 정도 걸릴 수 있습니다. 후속 실행에서는 Puppet 에이전트가 기존 환경이 올바르게 구성되었는지만 확인합니다. 

    >**참고**: `openjdk-7-jre` 관련 오류 메시지는 정상적인 현상이므로 무시하세요. OpenJDK 6 및 7은 Ubuntu 16.04에서 지원되지 않습니다.

1.  Tomcat이 올바르게 실행되고 있는지를 확인하려면 웹 브라우저를 시작한 다음 `http:\\` 접두사+Puppet 노드를 호스트하는 Azure VM의 DNS 이름+':9080' 형식의 URL로 이동합니다. 브라우저에 Tomcat 방문 페이지가 표시되는지 확인합니다.
1.  PU MRP 앱이 올바르게 실행되고 있는지를 확인하려면 같은 브라우저 탭 내에서 URL 끝에 `/mrp` 접미사를 추가합니다. 그런 다음 브라우저에 PU MRP 앱 환영 페이지가 표시되는지 확인합니다.

### 연습 3: Azure 랩 리소스 제거

이 연습에서는 예상치 못한 비용이 발생하지 않도록 이 랩에서 프로비전한 Azure 리소스를 제거합니다. 

>**참고**: 더 이상 사용하지 않는 새로 만든 Azure 리소스를 제거해야 합니다. 사용하지 않는 리소스를 제거하면 예기치 않은 비용이 발생하지 않습니다.

#### 작업 1: Azure 랩 리소스 제거

이 작업에서는 Azure Cloud Shell을 사용하여 불필요한 비용이 발생하지 않도록 이 랩에서 프로비전한 Azure 리소스를 제거합니다. 

1.  Azure Portal의 **Cloud Shell** 창에서 **Bash** 세션을 시작합니다.
1.  다음 명령을 실행하여 이 모듈의 전체 랩에서 생성된 모든 리소스 그룹을 나열합니다.

    ```sh
    az group list --query "[?starts_with(name,'az400m14l03a-RG')].name" --output tsv
    ```

1.  다음 명령을 실행하여 이 모듈의 전체 랩에서 만든 모든 리소스 그룹을 삭제합니다.

    ```sh
    az group list --query "[?starts_with(name,'az400m14l03a-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**참고**: 명령은 비동기적으로 실행되므로(--nowait 매개 변수에 의해 결정됨) 동일한 Bash 세션 내에서 즉시 다른 Azure CLI 명령을 실행할 수 있지만 리소스 그룹이 실제로 제거되기까지 몇 분 정도 걸립니다.

## 복습

이 랩에서는 Puppet Labs의 Puppet을 사용하여 Azure VM에 PartsUnlimted MRP 애플리케이션(PU MRP 앱)을 배포하는 방법을 배웠습니다. 
