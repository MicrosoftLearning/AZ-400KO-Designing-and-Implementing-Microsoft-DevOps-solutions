---
lab:
    title: '랩: Azure에서 Ansible 사용'
    az400Module: '모듈 14: Azure에서 사용 가능한 타사 IaC(코드형 인프라) 도구'
---

# 랩: Ansible과 Azure
# 학생 랩 매뉴얼

## 랩 개요

이 랩에서는 Ansible을 사용하여 Azure 리소스를 배포, 구성 및 관리합니다. 

선언형 구성 관리 소프트웨어인 Ansible은 관리 컴퓨터에 적용되는 적절한 구성의 설명(플레이북 형식)을 확인한 다음 해당 구성을 자동으로 적용하고 지속적으로 유지 관리함으로써 구성의 불일치 발생 가능성을 방지합니다. YAML을 사용하여 플레이북의 서식을 지정합니다.

Puppet, Chef 등 흔히 사용되는 기타 구성 관리 도구와는 달리 Ansible에는 에이전트가 없으므로 관리 컴퓨터에 소프트웨어를 설치하지 않아도 됩니다. Ansible은 SSH를 사용해 Linux 서버를, PowerShell 원격을 사용해 Windows 서버와 클라이언트를 관리합니다.

Azure Resource Manager를 통해 액세스 가능한 Azure 리소스 등 운영 체제 이외의 리소스와 상호 작용할 수 있도록 Ansible은 모듈이라는 확장을 지원합니다. Ansible은 Python으로 작성되었으므로 모듈도 Python 라이브러리로 구현됩니다. 그리고 Azure 리소스를 관리하기 위해 [GitHub에서 호스트되는 모듈](https://github.com/ansible-collections/azure)을 사용합니다.

Ansible을 사용하려면 관리되는 리소스가 호스트 인벤토리에 문서로 작성되어 있어야 합니다. Ansible은 Azure 등의 특정 시스템용 동적 인벤토리를 지원하므로 런타임 시 호스트 인벤토리가 동적으로 생성됩니다.

이 랩에서는 대략적으로 다음과 같은 단계를 진행합니다.

- Azure Cloud Shell을 사용하여 Azure VM 2개 배포
- 동적 인벤토리 생성
- Azure CLI를 사용하여 Azure에서 Ansible 제어 VM 만들기
- Azure VM에서 Ansible 설치 및 구성
- Ansible 구성 및 샘플 플레이북 파일 다운로드
- Azure AD에서 서비스 주체 만들기 및 구성
- Ansible에 사용할 Azure AD 자격 증명 및 SSH 구성
- Ansible 플레이북을 사용하여 Azure VM 배포
- Ansible 플레이북을 사용하여 Azure VM 구성
- Azure VM에서 구성 및 필요한 상태 관리에 Ansible 플레이북 사용
- Azure 리소스의 필요한 상태 및 구성 관리를 원활하게 진행하기 위해 Ansible 및 Azure Resource Manager 템플릿 사용

## 목표

이 랩을 완료하면 다음 작업을 수행할 수 있습니다.

- Azure Cloud Shell을 사용하여 Azure VM 배포
- Ansible 동적 인벤토리 생성
- Azure CLI를 사용하여 Azure에서 Ansible 제어 VM 만들기
- Azure VM에서 Ansible 설치 및 구성
- Ansible 구성 및 샘플 플레이북 파일 다운로드
- Azure AD에서 서비스 주체 만들기 및 구성
- Ansible에 사용할 Azure AD 자격 증명 및 SSH 구성
- Ansible 플레이북을 사용하여 Azure VM 배포
- Ansible 플레이북을 사용하여 Azure VM 구성
- Azure VM에서 구성 및 필요한 상태 관리에 Ansible 플레이북 사용
- Azure 리소스의 필요한 상태 및 구성 관리를 원활하게 진행하기 위해 Ansible 및 Azure Resource Manager 템플릿 사용

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

### 연습 1: Ansible을 사용하여 Azure VM 배포, 구성 및 관리

이 연습에서는 Ansible을 사용하여 Azure VM을 배포, 구성 및 관리합니다.

#### 작업 1: Azure Cloud Shell을 사용하여 Azure VM 2개 만들기

이 작업에서는 Azure Cloud Shell을 사용하여 Azure VM 2개를 만듭니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [**Azure Portal**](https://portal.azure.com)로 이동한 다음 이 랩에서 사용하는 Azure 구독에서 Contributor 이상의 역할이 지정된 사용자 계정으로 로그인합니다.
1.  Azure Portal의 도구 모음에서 검색 텍스트 상자 바로 오른쪽에 있는 **Cloud Shell** 아이콘을 클릭합니다. 

    >**참고**: [https://shell.azure.com](https://shell.azure.com)로 이동하여 Cloud Shell에 바로 액세스할 수도 있습니다.

1.  **Bash** 또는 **PowerShell**을 선택하라는 메시지가 표시되면 **Bash**를 선택합니다. 

    >**참고**: **Cloud Shell**을 처음 시작하고 **탑재된 스토리지가 없음** 메시지를 받으면, 이 랩에서 사용하는 구독을 선택하고 **스토리지 만들기**를 선택합니다. 

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 Azure VM을 호스트할 리소스 그룹을 만듭니다. 여기서 `<Azure_region>` 자리 표시자는 이 랩에서 리소스를 배포할 Azure 지역 이름으로 바꿉니다.

    ```bash
    LOCATION=<Azure_region>
    RGNAME=az400m14l03arg
    az group create --name $RGNAME --location $LOCATION
    ```

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 Ubuntu를 실행하는 첫 번째 Azure VM을 이전 단계에서 만든 리소스 그룹에 배포합니다.

    ```bash
    VM1NAME=az400m1403vm1
    az vm create --resource-group $RGNAME --name $VM1NAME --image UbuntuLTS --generate-ssh-keys --no-wait
    ```

    >**참고**: 배포가 완료되기까지 기다리지 말고 다음 단계로 진행하십시오. 

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 Ubuntu를 실행하는 두 번째 Azure VM을 같은 리소스 그룹에 배포합니다.

    ```bash
    VM2NAME=az400m1403vm2
    az vm create --resource-group $RGNAME --name $VM2NAME --image UbuntuLTS --generate-ssh-keys
    ```

    >**참고**: 배포가 완료될 때까지 기다린 후 다음 작업 진행합니다. 완료되려면 약 2분이 소요됩니다.

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 첫 번째 Azure VM을 Nginx 웹 서버로 식별할 수 있도록 해당 VM에 Nginx 태그를 지정합니다.

    ```bash
    VM1ID=$(az vm show --resource-group $RGNAME --name $VM1NAME --query id --output tsv)
    az resource tag --tags Ansible=nginx --id $VM1ID
    ```

#### 작업 2: 동적 인벤토리 생성

이 작업에서는 이전 작업에서 배포한 Azure VM 2개를 대상으로 동적 Ansible 인벤토리를 생성합니다.

>**참고**: Ansible 2.8부터는 Azure 동적 인벤토리 플러그 인이 제공됩니다.

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 새 파일 **myazure_rm.yml**을 만든 다음 Nano 텍스트 편집기에서 엽니다.

    ```bash
    nano ./myazure_rm.yml
    ```

1.  Nano 편집기 인터페이스에 다음 내용을 붙여넣습니다.

    ```bash
    plugin: azure_rm
    include_vm_resource_groups:
    - az400m14l03arg
    auth_source: auto

    keyed_groups:
    - prefix: tag
      key: tags
    ```

1.  Nano 편집기 인터페이스 내에서 **Ctrl+O** 키 조합, **Enter** 키, **Ctrl+X** 키 조합을 차례로 눌러 변경 내용을 저장하고 파일을 닫습니다.
1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 Ansible Azure 모듈을 설치합니다.

    ```bash
    curl -O https://raw.githubusercontent.com/ansible-collections/azure/dev/requirements-azure.txt
    pip install -r requirements-azure.txt
    rm requirements-azure.txt
    ansible-galaxy collection install azure.azcollection
    ```

    >**참고**: Ansible 버전 2.10부터는 Azure 모듈이 코어 모듈과 별도로 유지 관리됩니다. Ansible 버전을 확인하려면 `ansible --version`을 실행합니다.

1.  Cloud Shell 창의 Bash 세션에서 `exit`를 입력하고 **Enter** 키를 눌러 세션을 끝냅니다. 

    >**참고**: 모듈 설치를 적용하려면 세션을 끝내야 합니다. 

1.  Azure Portal의 도구 모음에서 **Cloud Shell** 아이콘을 클릭하여 Cloud Shell 세션을 다시 시작합니다. 
1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 ping 테스트를 수행하고 Azure에서 실행되는 모든 가상 머신의 동적 인벤토리를 생성합니다. **myazure_rm.yml** 파일에 이름을 입력했던 리소스 그룹에 인벤토리를 생성해야 합니다.

    ```bash
    ansible all -m ping -i ./myazure_rm.yml
    ```

    >**참고**: 명령을 처음 실행할 때는 **yes**를 입력하고 **Enter** 키를 눌러 대상 VM을 신뢰할 수 있음을 승인해야 합니다. 각 대상 Azure VM에서 이 작업을 수행해야 할 수 있습니다. 대상 VM의 신뢰성이 확인된 후부터는 추가 확인을 진행하지 않아도 명령 실행 출력이 정상적으로 반환됩니다. 필요한 경우 위의 명령을 몇 번 실행하여 정상 출력을 생성합니다.

    >**참고**: 사용 중단 관련 경고는 무시하세요. 

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 인벤토리에 포함된 호스트 목록을 표시합니다.

    ```bash
    ansible all -i ./myazure_rm.yml --list-hosts
    ```

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 인벤토리에 포함된 호스트의 태그 기반 그룹 목록을 표시합니다.

    ```bash
    ansible-inventory -i ./myazure_rm.yml --graph
    ```

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 인벤토리에서 특정 태그 값이 포함된 모든 호스트에 대한 연결을 테스트합니다.

    ```bash
    ansible -i ./myazure_rm.yml -m ping tag_Ansible_nginx
    ```

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 첫 번째 호스트 속성의 JSON 기반 표현 목록을 표시합니다. 여기서 `<Ansible_host_name>`은 `ansible all -i ./myazure_rm.yml --list-hosts` 명령에서 반환된 첫 번째 호스트의 이름으로 바꿉니다.

    ```bash
    ansible-inventory -i ./myazure_rm.yml --host <Ansible_host_name> | jq
    ```

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 인벤토리에 포함된 모든 호스트 속성의 JSON 기반 표현 목록을 표시합니다.

    ```bash
    ansible-inventory -i ./myazure_rm.yml --list | jq
    ```

1.  Cloud Shell 창을 닫습니다.

#### 작업 3: Ansible 제어 머신으로 사용할 Azure VM 프로비전

이 작업에서는 Azure CLI를 사용하여 Azure VM을 배포한 다음 Ansible 환경을 관리하도록 구성합니다.

>**참고**: Ansible 정적 호스트 인벤토리는 **etc/ansible/hosts** 파일을 사용합니다. Azure Cloud Shell에서는 루트 액세스 기능이 제공되지 않으므로 저장 옵션 사용 시 사용자 **$Home** 홈 디렉터리에만 저장이 가능합니다. 그러므로 여기서는 비동적 인벤토리를 사용하는 Ansible 관리 기능을 구현하기 위해 Linux를 실행하는 Azure VM을 배포한 후 Ansible 관리 시스템으로 구성합니다.

1.  Azure Portal의 도구 모음에서 검색 텍스트 상자 바로 오른쪽에 있는 **Cloud Shell** 아이콘을 클릭합니다. 

    >**참고**: [https://shell.azure.com](https://shell.azure.com)로 이동하여 Cloud Shell에 바로 액세스할 수도 있습니다.

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 새 Azure VM을 호스트할 리소스 그룹을 만듭니다.

    ```bash
    RG1NAME=az400m14l03arg
    VM1NAME=az400m1403vm1
    LOCATION=$(az vm show --resource-group $RG1NAME --name $VM1NAME --query location --output tsv)
    RG2NAME=az400m14l03brg
    az group create --name $RG2NAME --location $LOCATION
    ```

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 Ansible 관리 시스템으로 사용하려는 새 Azure VM을 호스트할 가상 네트워크를 만듭니다.

    ```bash
    VNETNAME=az400m1403-vnet
    SUBNETNAME=ansible-subnet
    az network vnet create \
    --name $VNETNAME \
    --resource-group $RG2NAME \
    --location $LOCATION \
    --address-prefixes 192.168.0.0/16 \
    --subnet-name $SUBNETNAME \
    --subnet-prefix 192.168.1.0/24
    ```

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 Ansible 관리 시스템으로 사용하려는 Azure VM의 네트워크 어댑터에 할당할 공용 IP 주소를 생성합니다.

    ```bash
    PIPNAME=az400m1403-pip
    az network public-ip create \
    --name $PIPNAME \
    --resource-group $RG2NAME \
    --location $LOCATION
    ```
 
1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 Ansible 제어 머신으로 사용하려는 Azure VM을 만듭니다.

    ```bash
    VM3NAME=az400m1403vm3
    az vm create \
    --name $VM3NAME \
    --resource-group $RG2NAME \
    --location $LOCATION \
    --image UbuntuLTS \
    --vnet-name $VNETNAME \
    --subnet $SUBNETNAME \
    --public-ip-address $PIPNAME \
    --authentication-type password \
    --admin-username azureuser \
    --admin-password Pa55w.rd1234
    ```

    >**참고**: 배포가 완료될 때까지 기다린 후 다음 작업 진행합니다. 완료되려면 약 2분이 소요됩니다.

    >**참고**: 프로비전이 완료되면 JSON 기반 출력에 포함된 **"publicIpAddress"** 속성의 값을 확인합니다. 

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 SSH를 사용해 새로 배포한 Azure VM에 연결합니다.

    ```bash
    PIP=$(az vm show --show-details --resource-group $RG2NAME --name $VM3NAME --query publicIps --output tsv)
    ssh azureuser@$PIP
    ```

1.  계속 진행할지를 확인하라는 메시지가 표시되면 **yes**를 입력하고 **Enter** 키를 누릅니다. 암호를 입력하라는 메시지가 표시되면 **Pa55w.rd1234**를 입력합니다.

#### 작업 4: Azure VM에서 Ansible 설치 및 구성

이 작업에서는 이전 작업에서 배포한 Azure VM에서 Ansible을 설치하고 구성합니다.

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 새로 배포한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 최신 버전 및 패키지 세부 정보가 포함되도록 Advanced Packaging Tool(apt) 패키지 목록을 업데이트합니다.

    ```bash
    sudo apt update
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 새로 배포한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 Ansible에 필요한 필수 구성 요소를 설치합니다.

    ```bash
    sudo apt-get install -y libssl-dev libffi-dev python-dev python-pip
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 새로 배포한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 Ansible 및 필수 Azure 모듈을 설치합니다.

    ```bash
    sudo apt-get install python3-pip
    sudo apt install ansible
    curl -O https://raw.githubusercontent.com/ansible-collections/azure/dev/requirements-azure.txt
    pip3 install -r requirements-azure.txt
    rm requirements-azure.txt
    ansible-galaxy collection install azure.azcollection
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 새로 배포한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 Ansible 플레이북이 배포 전에 DNS 이름을 확인할 수 있도록 dnspython 패키지를 설치합니다.

    ```bash
    sudo pip3 install dnspython 
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 새로 배포한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 **jq** JSON 구문 분석 도구를 설치합니다.

    ```bash
    sudo apt install jq
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 새로 배포한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 Azure CLI를 설치합니다.

    ```sh
    curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
    ```

#### 작업 5: Ansible 구성 및 샘플 플레이북 파일 다운로드

이 작업에서는 GitHub Ansible 구성 리포지토리와 샘플 랩 파일에서 몇 가지 파일을 다운로드합니다. 

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 새로 배포한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 **git**이 설치되어 있는지 확인합니다.

    ```bash
    sudo apt install git
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 새로 배포한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 GitHub에서 Ansible 소스 코드를 복제합니다.

    ```bash
    git clone git://github.com/ansible/ansible.git --recursive
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 새로 배포한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 GitHub에서 PartsUnlimitedMRP 리포지토리를 복제합니다.

    ```bash
    git clone https://github.com/Microsoft/PartsUnlimitedMRP.git
    ```

    >**참고**: 이 리포지토리에는 다양한 리소스를 만드는 데 사용할 수 있는 플레이북이 포함되어 있습니다. 이 랩에서는 해당 플레이북 중 몇 가지를 사용합니다.

#### 작업 6: Azure Active Directory 서비스 주체 만들기 및 구성

이 작업에서는 Azure 리소스에 액세스하려면 진행해야 하는 Ansible 비대화형 인증을 원활하게 진행하기 위해 Azure AD 서비스 주체를 생성합니다. 그리고 이전 작업에서 만든 리소스 그룹의 Contributor 역할을 해당 서비스 주체에 할당합니다.

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 새로 배포한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 Azure 구독과 연결된 Azure AD 테넌트에 로그인합니다.

    ```bash
    az login
    ```

1.  이전 명령의 출력으로 표시된 코드를 확인하고 랩 컴퓨터로 전환합니다. 그런 다음 랩 컴퓨터의 브라우저 창에서 다른 탭을 열고 Azure Portal이 표시되면 [Microsoft 디바이스 로그인 페이지](https://microsoft.com/devicelogin)로 이동합니다. 메시지가 표시되면 코드를 입력하고 **다음**을 선택합니다.

1.  다시 메시지가 표시되면 이 랩에서 사용하는 자격 증명을 사용하여 로그인한 다음 브라우저 탭을 닫습니다.

1.  Cloud Shell 창의 Bash 세션으로 다시 전환합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 서비스 주체를 생성합니다.

    ```bash
    ANSIBLESP=$(az ad sp create-for-rbac --name az400m1403Ansible)
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 새로 생성한 서비스 주체의 속성을 표시합니다.

    ```bash
    echo $ANSIBLESP | jq
    ```

1.  출력을 검토하고 **appId**, **password** 및 **tenant** 속성의 값을 적어 둡니다. 출력의 형식은 다음과 같습니다.

    ```bash
    {
      "appId": "0df61458-1a6e-4b4c-b02a-d4ae43cd981e",
      "displayName": "az400m1403Ansible",
      "name": "http://az400m1403Ansible",
      "password": "JAHxIbRUypM~-DiA27Guj58E4nC.S_u~_U",
      "tenant": "44bb87a9-c9c8-4c0f-b176-88d7814533ba"
    }
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 구독의 값을 확인합니다.

    ```bash
    SUBSCRIPTIONID=$(az account show --query id --output tsv)
    echo $SUBSCRIPTIONID
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 기본 제공 Azure 역할 기반 액세스 제어 Contributor 역할의 **ID** 속성 값을 검색합니다. 

    ```bash
    CONTRIBUTORID=$(az role definition list --name "Contributor" --query "[].id" --output tsv)
    echo $CONTRIBUTORID
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 이 랩의 앞부분에서 만든 리소스 그룹에 대한 Contributor 역할을 할당합니다. 

    ```bash
    APPID=$(echo $ANSIBLESP | jq '.appId' -r)

    RG2NAME=az400m14l03brg
    az role assignment create --assignee "$APPID" \
    --role "$CONTRIBUTORID" \
    --resource-group "$RG2NAME"
    ```

#### 작업 7: Ansible에 사용할 Azure AD 자격 증명 및 SSH 구성

이 작업에서는 이전 작업에서 만든 Azure AD 서비스 주체를 활용하여 Ansbile용 Azure 액세스 권한과 Ansible에 사용할 SSH를 구성합니다.

>**참고**: 지정된 Ansible 제어 시스템에서 원격 관리를 허용하려는 경우 자격 증명 파일을 만들거나 서비스 주체 세부 정보를 Ansible 환경 변수로 내보내면 됩니다. 여기서는 이 두 가지 옵션 중 첫 번째 옵션을 선택합니다. 자격 증명은 **~/.azure/credentials** 파일에 저장됩니다. 

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 자격 증명 파일을 만듭니다. 

    ```bash
    echo "[default]" > ~/.azure/credentials
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 이전 작업에서 확인한 Azure 구독 ID가 포함된 새 줄을 Ansible 자격 증명 파일에 추가합니다. 

    ```bash
    echo subscription_id=$SUBSCRIPTIONID >> ~/.azure/credentials
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 이전 작업에서 확인한 서비스 주체 ID가 포함된 새 줄을 Ansible 자격 증명 파일에 추가합니다. 

    ```bash
    echo client_id=$APPID >> ~/.azure/credentials
    ```
1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 이전 작업에서 확인한 서비스 주체 ID의 비밀이 포함된 새 줄을 Ansible 자격 증명 파일에 추가합니다. 

    ```bash
    SECRET=$(echo $ANSIBLESP | jq '.password' -r)
    echo secret=$SECRET >> ~/.azure/credentials
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 이전 작업에서 확인한 Azure AD 테넌트의 ID가 포함된 새 줄을 Ansible 자격 증명 파일에 추가합니다. 

    ```bash
    TENANT=$(echo $ANSIBLESP | jq '.tenant' -r)
    echo tenant=$TENANT >> ~/.azure/credentials
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 Ansible 자격 증명 파일의 내용을 확인합니다. 

    ```bash
    cat  ~/.azure/credentials
    ```

    >**참고**: 출력은 다음과 유사합니다.

    ```bash
    [default]
    subscription_id=1cfb08bc-a39e-46cc-9b1e-38a182b43d60
    client_id=0df61458-1a6e-4b4c-b02a-d4ae43cd981e
    secret=JAHxIbRUypM~-DiA27Guj58E4nC.S_u~_U
    tenant=44bb87a9-c9c8-4c0f-b176-88d7814533ba
    ```

    >**참고**: 이제 원격 SSH 연결용 공용 키/프라이빗 키 쌍을 만든 다음 키가 정상 작동하는지 테스트합니다. 

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 키 쌍을 생성합니다. 메시지가 표시되면 **Enter** 키를 누릅니다. 그러면 파일 위치의 기본값이 그대로 적용되며 암호는 설정되지 않습니다.

    ```bash
    ssh-keygen -t rsa
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 프라이빗 키를 호스트하는 **ssh** 폴더에 대한 읽기, 쓰기, 실행 권한을 부여합니다.

    ```bash
    chmod 755 ~/.ssh
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 **authorized_keys** 파일을 만들고 해당 파일에 대한 읽기 및 쓰기 권한을 설정합니다.

    ```bash
    touch ~/.ssh/authorized_keys
    chmod 644 ~/.ssh/authorized_keys
    ```

    >**참고**: 이 파일에 포함되어 있는 키를 제공하면 암호를 입력하지 않아도 액세스가 허용됩니다.

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 **authorized_keys** 파일에 암호를 추가합니다.

    ```bash
    ssh-copy-id azureuser@127.0.0.1
    ```

1. 메시지가 표시되면 **yes**를 입력하고 이 랩의 앞부분에서 세 번째 Azure VM을 배포할 때 지정했던 **azureuser** 사용자 계정의 암호 **Pa55w.rd1234**를 입력합니다. 
1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 암호를 입력하라는 메시지가 표시되지 않는지 확인합니다.

    ```bash
    ssh 127.0.0.1
    ```
1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 **exit**를 입력하고 **Enter** 키를 눌러 방금 설정한 루프백 연결을 종료합니다. 

>**참고**: Ansible 환경을 설정할 때는 암호 없는 SSH 인증 설정 단계를 반드시 수행해야 합니다. 

#### 작업 8: Ansible 플레이북을 사용하여 웹 서버 Azure VM 만들기

이 작업에서는 Ansible 플레이북을 사용하여 웹 서버를 호스트하는 Azure VM을 만듭니다. 

>**참고**: 제어 Azure VM에서 Ansible이 작동되어 실행되고 있으므로 관리되는 Azure VM을 만들고 구성하는 데 사용할 첫 번째 플레이북을 배포할 수 있습니다. 이 플레이북은 동적 인벤토리가 아닌 localhost 파일을 사용합니다. 배포에는 이 랩의 앞부분에서 GitHub 리포지토리에서 복제한 샘플 플레이북 **~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_vm_web.yml**을 사용합니다. 샘플 플레이북을 배포하기 전에 플레이북의 내용에 포함되어 있는 공용 SSH 키를 이전 작업에서 생성한 키로 바꿔야 합니다. 

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 이전 작업에서 생성한 로컬에 저장된 공개 키를 확인합니다.

    ```bash
    cat ~/.ssh/id_rsa.pub
    ```

1.  출력 문자열 끝부분의 사용자 이름을 포함하여 전체 출력을 적어 둡니다. 
1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 Nano 텍스트 편집기에서 **new_vm_web.yml** 파일을 엽니다.

    ```bash
    nano ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_vm_web.yml
    ```

1.  Nano 편집기 인터페이스 내에서 파일 끝부분의 key_data 값에 포함된 SSH 문자열을 찾은 다음 파일에 포함되어 있는 키를 삭제합니다.
1.  Nano 편집기에서 파일 끝부분의 `key_data` 항목에 포함된 SSH 문자열을 찾은 다음 기존 키 값을 삭제하고 이 작업의 앞부분에서 적어 둔 키 값으로 바꿉니다. 
1.  파일에 포함된 `admin_username` 항목의 값이 Ansible 제어 시스템을 호스트하는 Azure VM에 로그인하는 데 사용한 사용자 이름(**azureuser**)과 일치하는지 확인합니다.파일에 포함된 `admin_username` 항목의 값이 Ansible 제어 시스템을 호스트하는 Azure VM에 로그인하는 데 사용한 사용자 이름(**azureuser**)과 일치하는지 확인합니다. `ssh_public_keys` 섹션의 `path` 항목에도 같은 사용자 이름을 사용해야 합니다.
1.  `vm_size` 항목의 값을 `Standard_A0`에서 `Standard_D2s_v3`로 변경합니다.
1.  필요한 경우 `dnsname: '{{ vmname }}.westeurope.cloudapp.azure.com'` 항목의 지역 이름을 배포 대상 Azure 지역 이름으로 변경합니다.
1.  Nano 편집기 인터페이스 내에서 **Ctrl+O** 키 조합, **Enter** 키, **Ctrl+X** 키 조합을 차례로 눌러 변경 내용을 저장하고 파일을 닫습니다.

    >**참고**: 랩을 시작할 때 만든 리소스 그룹에 Azure VM을 배포할 것입니다. 배포에는 다음 값을 사용합니다. 

    | 설정 | 값 |
    | --- | --- |
    | 리소스 그룹 | **az400m14l03arg** |
    | 가상 네트워크 | **az400m1403vm1VNET** |
    | 서브넷 | **az400m1403vm1Subnet** |

    >**참고**: 참고: 변수는 플레이북 내에서 정의할 수도 있고 런타임에 입력할 수도 있습니다. 런타임에 변수를 입력하는 경우 `ansible-playbook` 명령을 호출할 때 `--extra-vars` 옵션을 포함하면 됩니다. VM 이름은 소문자와 숫자만 포함하여 15자 이하로 입력합니다. 하이픈, 밑줄 기호, 대문자는 사용하지 마세요. 그리고 해당 Azure VM과 연결된 공용 IP 주소의 DNS 이름을 생성할 때도 같은 이름이 사용되므로, 전역적으로 고유한 이름을 지정해야 합니다. 

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 Azure VM을 프로비전하는 샘플 Ansible 플레이북을 배포합니다. 여기서 `<VM_name>` 자리 표시자는 선택한 고유 VM 이름을 나타냅니다.

    ```bash
    ansible-playbook ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_vm_web.yml --extra-vars "vmname=<VM_name> resgrp=az400m14l03arg vnet=az400m1403vm1VNET subnet=az400m1403vm1Subnet"
    ```

    >**참고**: 잘못된 VM 이름을 입력하면 다음 오류가 발생할 수 있습니다.

    - `fatal: [localhost]: FAILED! => {"changed": false, "failed": true, "msg": "이름이 storageaccountname인 스토리지 계정은 이미 사용되었습니다. - Reason.already_exists"}`. 이 문제를 해결하려면 Azure VM에 다른 이름을 사용하십시오. 사용한 이름은 전역적으로 고유하지 않습니다.
    - `fatal: [localhost]: FAILED! => {"changed": false, "failed": true, "msg": "your-vm-name을(를) 만들거나 업데이트하는 동안 오류가 발생했습니다. Azure 오류: InvalidDomainNameLabel\nMessage: VM의 도메인 이름 레이블이 잘못되었습니다. 다음 정규식을 따라야 합니다. ^[a-z][a-z0-9-]{1,61}[a-z0-9]$.”}`. 이 문제를 해결하려면 Azure VM에 필요한 명명 규칙을 따르는 다른 이름을 사용하십시오. 

    >**참고**: 배포가 완료될 때까지 기다립니다. 완료되려면 약 3분이 소요됩니다. 

    >**참고**: 배포가 완료되면 동적 인벤토리를 실행하여 Ansible에서 새 VM이 검색되는지 확인할 수 있습니다. 

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 동적 인벤토리를 생성합니다.

    ```bash
    python /usr/local/lib/python2.7/dist-packages/ansible_collections/community/general/scripts/inventory/azure_rm.py --list | jq
    ```
1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 ping 테스트를 수행합니다. 이 테스트에서는 새로 배포된 Azure VM이 동적 인벤토리 파일에 포함되어 있는지를 확인합니다. 이전과 마찬가지로 테스트를 처음 실행할 때는 SSH 호스트 키를 확인해야 하지만 후속 실행에서는 별도의 상호 작용이 필요하지 않습니다.

    ```bash
    sudo chmod +x /usr/local/lib/python2.7/dist-packages/ansible_collections/community/general/scripts/inventory/azure_rm.py
    ansible -i /usr/local/lib/python2.7/dist-packages/ansible_collections/community/general/scripts/inventory/azure_rm.py all -m ping
    ```

    >**참고**: 앞에서 Azure Cloud Shell을 통해 배포한 Azure VM 2개는 인벤토리에 표시되기는 하지만 액세스할 수는 없습니다. Ansible 제어 머신에서 생성한 공용 키가 해당 VM에 포함되어 있지 않기 때문입니다. 새로 배포한 Azure VM에서 응답이 정상 수신되면 다음 작업을 계속 진행합니다. 

1.  동적 인벤토리가 정상적으로 완료되지 않으면 Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 새로 배포한 Azure VM에 SSH를 사용하여 연결합니다. 여기서 `<VM_name>` 자리 표시자는 새로 프로비전한 Azure VM에 할당한 이름을 나타냅니다.

    ```bash
    $RG2NAME='az400m14l03arg'
    $VM4NAME='<VM_name>'
    PIP=$(az vm show --show-details --resource-group $RG2NAME --name $VM4NAME --query publicIps --output tsv)
    ssh azureuser@$PIP
    ```

1.  계속 진행할지를 확인하라는 메시지가 표시되면 **yes**를 입력하고 **Enter** 키를 누릅니다. 연결이 설정되면 **exit**를 입력하고 **Enter** 키를 다시 눌러 연결을 종료합니다.

#### 작업 9: Ansible 플레이북을 사용하여 새로 배포한 Azure VM 구성

이 작업에서는 다른 Ansible 플레이북을 실행하여 새로 만든 머신을 구성합니다. 여기서는 소프트웨어 패키지 httpd를 설치하고 GitHub 리포지토리에서 HTML 페이지를 다운로드하는 플레이북을 사용합니다. 이 작업을 완료하면 정상 작동하는 웹 서버가 완성됩니다.

>**참고**: 여기서는 **~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/httpd.yml** 샘플 플레이북을 사용합니다. 그리고 **vmname** 변수를 활용하여 플레이북의 hosts 매개 변수를 수정합니다. 이 매개 변수는 동적 인벤토리 스크립트에서 반환된 호스트 중 플레이북이 대상으로 사용할 호스트를 정의합니다. 

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 새로 배포한 Azure VM에서 현재 웹 서비스를 실행하고 있지 않음을 확인합니다. 여기서 `<IP_address>` 자리 표시자는 새로 프로비전한 Azure VM의 네트워크 어댑터에 할당된 공용 IP 주소를 나타냅니다.

    ```bash
    curl http://<IP_address>
    ```

    >**참고**: 다음 스크립트를 실행하면 이 공용 IP 주소를 확인할 수 있습니다. 여기서 `<VM_name>` 자리 표시자는 새로 프로비전한 Azure VM에 할당한 이름을 나타냅니다.

    ```bash
    $RG2NAME='az400m14l03brg'
    $VM4NAME='<VM_name>'
    PIP=$(az vm show --show-details --resource-group $RG2NAME --name $VM4NAME --query publicIps --output tsv)
    echo $PIP
    ```

1.  응답이 `curl: (7) Failed to connect to 52.186.157.26 port 80: Connection refused` 형식인지 확인합니다.
1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 Ansible 플레이북을 사용해 HTTP 서비스를 설치합니다. 여기서 `<VM_name>` 자리 표시자는 새로 프로비전한 Azure VM에 할당한 이름을 나타냅니다. 

    ```bash
    ansible-playbook -i /usr/local/lib/python2.7/dist-packages/ansible_collections/community/general/scripts/inventory/azure_rm.py  ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/httpd.yml --extra-vars "vmname=<VM_name>"
    ```

    >**참고**: 설치가 완료될 때까지 기다립니다. 이 작업에는 1분 미만이 소요됩니다. 

1.  설치가 완료되면 Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 새로 배포한 Azure VM에서 이제 웹 서비스를 실행하고 있음을 확인합니다. 여기서 `<IP_address>` 자리 표시자는 새로 프로비전한 Azure VM의 네트워크 어댑터에 할당된 공용 IP 주소를 나타냅니다.

    ```bash
    curl http://<IP_address>
    ```

    >**참고**: 출력에는 다음 내용이 포함되어 있어야 합니다. 

    ```html
     <!DOCTYPE html>
     <html lang="en">
         <head>
             <meta charset="utf-8">
             <title>Hello World</title>
         </head>
         <body>
             <h1>Hello World</h1>
             <p>
                 <br>This is a test page
                 <br>This is a test page
                 <br>This is a test page
             </p>
         </body>
     </html>
    ```

#### 작업 10: Ansible 플레이북을 사용하여 Azure에서 구성 관리 및 필요한 상태 구현

이 작업에서는 Ansible 플레이북을 사용하여 Azure에서 구성 관리 및 필요한 상태를 구현합니다.

>**참고**: `ansible-playbook` 명령을 주기적으로 실행하여 구성이 해당 플레이북의 내용과 일치하는 상태로 유지되고 있는지를 확인할 수 있습니다. 여기서는 이 확인을 자동으로 수행하기 위해 Linux cron 기능을 활용합니다. 이 작업에서는 1분마다 명령을 실행하지만 프로덕션 환경에서는 실행 빈도를 낮춰야 할 가능성이 높습니다.

>**참고**: Ansible 플레이북을 사용하여 cron 작업을 설정합니다. 

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 Nano 텍스트 편집기에서 Ansible 구성 파일을 엽니다.

    ```bash
    sudo nano /etc/ansible/ansible.cfg
    ```

1.  Nano 편집기 인터페이스 내의 `[Default]` 섹션 내 `#host_key_checking = False` 줄에서 선행 해시 문자 `#`을 제거합니다.
1.  Nano 편집기 인터페이스 내에서 **Ctrl+O** 키 조합, **Enter** 키, **Ctrl+X** 키 조합을 차례로 눌러 변경 내용을 저장하고 파일을 닫습니다.
1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 Nano 텍스트 편집기에서 cron 작업을 구성하는 플레이북을 엽니다.

    ```bash
    nano ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/cron.yml
    ```

1.  Nano 편집기 인터페이스 내의 `job` 항목에서 `< your-vm-name >` 자리 표시자를 이전 작업에서 구성한 Azure VM의 이름으로 바꿉니다. 
1.  Nano 편집기 인터페이스 내에서 `job` 항목을 `ansible-playbook -i /usr/local/lib/python2.7/dist-packages/ansible_collections/community/general/scripts/inventory/azure_rm.py  /home/azureuser/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/httpd.yml --extra-vars "vmname=<VM_name>"`으로 설정합니다. 여기서 `<VM_name>` 자리 표시자는 관리되는 Azure VM에 할당한 이름을 나타냅니다.
1.  Nano 편집기 인터페이스 내에서 **Ctrl+O** 키 조합, **Enter** 키, **Ctrl+X** 키 조합을 차례로 눌러 변경 내용을 저장하고 파일을 닫습니다.
1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 cron 작업을 만듭니다.

    ```bash
    ansible-playbook ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/cron.yml
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 cron 작업이 실행되고 있는지 확인합니다.

    ```bash
    sudo tail /var/log/syslog
    ```

    >**참고**: 모든 사용자가 실행하는 모든 cron 작업은 같은 파일에 로깅되므로 루트 권한이 필요합니다. 

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 cron 편집기 인터페이스에서 cron 작업을 확인합니다.

    ```bash
    crontab -e
    ```

1.  메시지가 표시되면 nano에 해당하는 **1**을 선택하고 **Enter** 키를 누릅니다.
1.  **crontab** 편집기 내에서 **usr/bin/ansible-playbook** 항목이 있는지 확인하고 **Ctrl+X** 키 조합을 눌러 편집기를 닫습니다.

    >**참고**: 별표 5개는 작업이 1분마다 실행됨을 나타냅니다.

    >**참고**: 이제 설정 작업을 확인해 보겠습니다. 

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 다음 명령을 실행하여 관리되는 Azure VM에 SSH를 통해 연결합니다. 여기서 `<IP_address>` 자리 표시자는 해당 Azure VM의 네트워크 어댑터에 할당된 공용 IP 주소를 나타냅니다.

    ```bash
    ssh azureuser@<IP_address>
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 웹 서버로 구성된 Azure VM에 연결된 SSH 세션을 표시하고 다음 명령을 실행하여 웹 사이트가 계속 작동하고 있는지 확인합니다.

    ```bash
    curl http://127.0.0.1
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 웹 서버로 구성된 Azure VM에 연결된 SSH 세션을 표시하고 다음 명령을 실행하여 웹 사이트의 홈 페이지를 삭제합니다.

    ```bash
    rm /var/www/html/index.html
    ```

1.  1분 동안 기다렸다가 Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 웹 서버로 구성된 Azure VM에 연결된 SSH 세션을 표시하고 다음 명령을 실행하여 웹 사이트의 홈 페이지가 복원되었는지 확인합니다.

    ```bash
    curl http://127.0.0.1
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 웹 서버로 구성된 Azure VM에 연결된 SSH 세션을 표시하고 **exit**를 입력한 후 **Enter** 키를 눌러 Ansible 제어 머신에 연결된 SSH 세션으로 돌아옵니다.

    >**참고**: Ansible 플레이북을 주기적으로 실행하면 Azure VM의 상태를 플레이북에 정의된 필요한 구성에서 다른 상태로 바꾸는 변경 내용을 수정할 수 있습니다. 이 방식을 사용하면 구성 드리프트를 방지하고 필요한 상태를 유지할 수 있습니다. 같은 방식에 따라 Azure 리소스를 대상으로 하는 Ansible 플레이북도 주기적으로 실행할 수 있습니다. 그러면 인프라가 적절하게 배포 및 구성되었는지를 확인할 수 있습니다. 

#### 작업 11: Azure 리소스의 필요한 상태 및 구성 관리를 원활하게 진행하기 위해 Ansible 및 Azure Resource Manager 템플릿 사용

이 작업에서는 Azure 리소스의 필요한 상태 및 구성 관리를 원활하게 진행하기 위해 Ansible 및 Azure Resource Manager 템플릿을 함께 사용합니다.

>**참고**: 이전 작업에서 확인했듯이, Ansible을 사용하면 해당 모듈에서 지원하는 기존 리소스의 구성 편차를 수정할 수 있습니다. 그러나 Ansible을 사용해 Azure Resource Manager 템플릿을 참조하는 플레이북을 배포할 수도 있습니다. 그러면 Azure Resource Manager에서 제공하는 리소스와 기능에 직접 액세스할 수 있습니다. 

>**참고**: 여기서는 Azure VM을 하나 더 배포합니다. 단, 이번에는 Azure Resource Manager 템플릿을 참조하는 Ansible 플레이북을 사용합니다. 이 과정을 쉽게 이해할 수 있도록 [스토리지 계정 하나를 프로비전하는 매우 단순한 Azure 빠른 시작 템플릿](https://github.com/Azure/azure-quickstart-templates/tree/master/101-storage-account-create)을 사용하겠습니다. **PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_ARM_deployment.yml** 플레이북을 검토하면 관련 플레이북 구문을 파악할 수 있습니ㅏㄷ.

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 웹 서버로 구성된 Azure VM에 연결된 SSH 세션을 표시하고 다음 명령을 실행하여 Azure Resource Manager 템플릿을 호출하는 플레이북을 Nano 텍스트 편집기에서 엽니다.

    ```bash
    nano  ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_ARM_deployment.yml
    ```

1.  Nano 편집기 인터페이스의 `templateLink:` 항목에서 현재 URL을 `https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json`으로 바꿉니다.
1.  Nano 편집기 인터페이스 내에서 **Ctrl+O** 키 조합, **Enter** 키, **Ctrl+X** 키 조합을 차례로 눌러 변경 내용을 저장하고 파일을 닫습니다.
1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 웹 서버로 구성된 Azure VM에 연결된 SSH 세션을 표시하고 다음 명령을 실행하여 플레이북을 실행합니다. 이때 `<Azure_region>` 자리 표시자는 이 랩의 모든 리소스를 배포한 Azure 지역의 이름으로 바꿉니다.

    ```bash
    ansible-playbook ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_ARM_deployment.yml --extra-vars "resgrp=az400m14l03brg location=<Azure_region>"
    ```

    >**참고**: AR 템플릿 배포는 idempotent 작업입니다. 즉, ARM 템플릿을 한 번만 배포해도 여러 번 배포하는 경우와 결과는 같습니다. 따라서 같은 리소스 그룹에 ARM 템플릿을 다시 배포해도 중복 리소스는 생성되지 않습니다. 그러므로 이전 작업에서 살펴본 것처럼 Linux cron 유틸리티 등을 사용하여 ARM 템플릿이 일정한 간격으로 다시 배포되도록 예약할 수 있습니다. 

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 웹 서버로 구성된 Azure VM에 연결된 SSH 세션을 표시하고 다음 명령을 실행하여 같은 플레이북을 실행합니다. 이때 `<Azure_region>` 자리 표시자는 이 랩의 모든 리소스를 배포한 Azure 지역의 이름으로 바꿉니다.

    ```bash
    ansible-playbook ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_ARM_deployment.yml --extra-vars "resgrp=az400m14l03brg location=<Azure_region>"
    ```

    >**참고**: 이제 템플릿을 사용하여 배포한 스토리지 계정을 수정합니다. 이러한 변경 내용은 검색이 어려울 수도 있지만, 환경에 좋지 않은 영향을 줄 가능성이 있습니다. 따라서 이러한 변경 내용을 자동으로 필요한 상태로 되돌리는 메커니즘이 있으면 유용합니다. 여기서는 스토리지 계정의 복제 설정을 변경합니다. 

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 웹 서버로 구성된 Azure VM에 연결된 SSH 세션을 표시하고 다음 명령을 실행하여 이 작업의 앞부분에서 배포한 스토리지 계정의 현재 설정을 확인합니다.

    ```bash
    RGNAME=az400m14l03brg
    STORAGEACCOUNTNAME=$(az storage account list --resource-group $RGNAME --query "[].name" --output tsv)
    az storage account show \
    --name $STORAGEACCOUNTNAME \
    --resource-group $RGNAME \
    --query sku
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 웹 서버로 구성된 Azure VM에 연결된 SSH 세션을 표시하고 다음 명령을 실행하여 SKU를 `Standard_LRS`에서 `Standard_GRS`로 변경하고 변경이 수행되었는지를 확인합니다. 

    ```bash
    az storage account update \
    --name $STORAGEACCOUNTNAME \
    --resource-group $RGNAME \
    --sku 'Standard_GRS'

    az storage account show \
    --name $STORAGEACCOUNTNAME \
    --resource-group $RGNAME \
    --query sku
    ```

    >**참고**: 이제 플레이북을 다시 실행하여 같은 템플릿을 다시 배포합니다. 

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 웹 서버로 구성된 Azure VM에 연결된 SSH 세션을 표시하고 다음 명령을 실행하여 같은 플레이북을 다시 실행합니다. 이때 `<Azure_region>` 자리 표시자는 이 랩의 모든 리소스를 배포한 Azure 지역의 이름으로 바꿉니다.

    ```bash
    ansible-playbook ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_ARM_deployment.yml --extra-vars "resgrp=az400m14l03brg location=<Azure_region>"
    ```

1.  Cloud Shell 창에서 Bash 세션을 표시합니다. 그런 다음 Ansible 제어 머신으로 구성한 Azure VM에 연결된 SSH 세션 내에서 웹 서버로 구성된 Azure VM에 연결된 SSH 세션을 표시하고 다음 명령을 실행하여 변경 내용이 이전 상태로 되돌아갔는지 확인합니다. 

    ```bash
    az storage account show \
    --name $STORAGEACCOUNTNAME \
    --resource-group $RGNAME \
    --query sku
    ```

### 연습 3: Azure 랩 리소스 제거

이 연습에서는 예상치 못한 비용이 발생하지 않도록 이 랩에서 프로비전한 Azure 리소스를 제거합니다. 

>**참고**: 더 이상 사용하지 않는 새로 만든 Azure 리소스를 제거해야 합니다. 사용하지 않는 리소스를 제거하면 예기치 않은 비용이 발생하지 않습니다.

#### 작업 1: Azure 랩 리소스 제거

이 작업에서는 Azure Cloud Shell을 사용하여 불필요한 비용이 발생하지 않도록 이 랩에서 프로비전한 Azure 리소스를 제거합니다. 

1.  Azure Portal의 **Cloud Shell** 창에서 **Bash** 세션을 시작합니다.
1.  다음 명령을 실행하여 이 모듈의 전체 랩에서 생성된 모든 리소스 그룹을 나열합니다.

    ```sh
    az group list --query "[?starts_with(name,'az400m14l03')].name" --output tsv
    ```

1.  다음 명령을 실행하여 이 모듈의 전체 랩에서 만든 모든 리소스 그룹을 삭제합니다.

    ```sh
    az group list --query "[?starts_with(name,'az400m14l03')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**참고**: 명령은 비동기적으로 실행되므로(--nowait 매개 변수에 의해 결정됨) 동일한 Bash 세션 내에서 즉시 다른 Azure CLI 명령을 실행할 수 있지만 리소스 그룹이 실제로 제거되기까지 몇 분 정도 걸립니다.

## 복습

이 랩에서는 Ansible을 사용하여 Azure 리소스를 배포, 구성 및 관리하는 방법을 배웠습니다. 

