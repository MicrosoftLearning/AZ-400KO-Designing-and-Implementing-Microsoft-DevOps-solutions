---
lab:
    title: '랩: 에이전트 풀 구성 및 파이프라인 스타일 이해'
    module: '모듈 5: Azure Pipelines 구성'
---

# 랩: 에이전트 풀 구성 및 파이프라인 스타일 이해
# 학생 랩 매뉴얼

## 랩 개요

YAML 기반 파이프라인을 사용하면 CD/CI를 코드로 완벽하게 구현할 수 있습니다. 이 구현에서는 파이프라인 정의가 Azure DevOps 프로젝트에 포함된 코드와 같은 리포지토리에 저장됩니다. YAML 기반 파이프라인은 끌어오기 요청, 코드 검토, 기록, 분기, 템플릿 등 클래식 파이프라인에서 제공되는 폭넓은 기능을 지원합니다. 

어떤 파이프라인 스타일을 선택하든 Azure Pipelines를 사용하여 솔루션을 배포하거나 코드를 작성하려면 에이전트가 필요합니다. 작업을 한 번에 하나씩 실행하는 에이전트는 컴퓨팅 리소스를 호스트합니다. 작업은 에이전트의 호스트 머신에서 바로 실행할 수도 있고 컨테이너에서 실행할 수도 있습니다. Microsoft에서 호스트하는 에이전트(자동으로 관리됨)를 사용하여 작업을 실행하거나 직접 설정 및 관리하는 자체 호스팅 에이전트를 구현할 수 있습니다. 

이 랩에서는 클래식 파이프라인을 YAML 기반 파이프라인으로 변환한 다음, 먼저 Microsoft에서 호스트된 에이전트를 사용하여 실행한 다음 자체 호스팅 에이전트를 사용하여 동일 작업을 수행하는 프로세스를 단계별로 진행합니다.

## 목표

이 랩을 완료하면 다음 작업을 수행할 수 있습니다.

- YAML 기반 파이프라인 구현
- 자체 호스팅 에이전트 구현

## 랩 소요 시간

-   예상 시간: **90분**

## 지침

### 시작하기 전

#### 랩 가상 머신에 로그인

다음 자격 증명을 사용하여 Windows 10 컴퓨터에 로그인했는지 확인합니다.
    
-   사용자 이름: **Student**
-   암호: **Pa55w.rd**

#### 설치된 애플리케이션 검토

Windows 10 데스크톱에서 작업 표시줄을 찾습니다. 작업 표시줄에는 이 랩에서 사용할 애플리케이션에 대한 아이콘이 포함되어 있습니다.
    
-   Microsoft Edge
-   [Visual Studio Code](https://code.visualstudio.com/). 이 랩의 필수 구성 요소로 설치됩니다.

#### Azure DevOps 조직 설정

이 랩에 사용할 수 있는 Azure DevOps 조직이 아직 없으면 [조직 또는 프로젝트 컬렉션 만들기](https://docs.microsoft.com/ko-kr/azure/devops/organizations/accounts/create-organization?view=azure-devops)에서 제공되는 지침에 따라 조직을 만듭니다.

#### Azure 구독 준비

-   기존 Azure 구독을 확인하거나 새 구독을 만듭니다.
-   Azure 구독의 소유자 역할, 그리고 Azure 구독과 연결된 Azure AD 테넌트의 전역 관리자 역할이 지정되어 있는 Microsoft 계정 또는 Azure AD 계정이 있는지 확인합니다. 자세한 내용은 [Azure Portal을 사용하여 Azure 역할 할당 목록 표시](https://docs.microsoft.com/ko-kr/azure/role-based-access-control/role-assignments-list-portal) 및 [Azure Active Directory에서 관리자 역할 확인 및 할당](https://docs.microsoft.com/ko-kr/azure/active-directory/roles/manage-roles-portal#view-my-roles)을 참조하세요.

#### GitHub 계정 설정

[새 GitHub 계정 등록](https://docs.github.com/en/free-pro-team@latest/github/getting-started-with-github/signing-up-for-a-new-github-account)에서 제공되는 지침을 따르세요.

### 연습 0: 랩 필수 구성 요소 구성

이 연습에서는 랩의 필수 구성 요소를 설정합니다. 구체적으로는 Azure DevOps 데모 생성기 템플릿을 기반으로 하여 미리 구성된 Parts Unlimited 팀 프로젝트를 설정합니다.

#### 작업 1: 팀 프로젝트 구성

이 작업에서는 Azure DevOps 데모 생성기를 사용하여 **PartsUnlimited** 템플릿을 기반으로 새 프로젝트를 생성합니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [Azure DevOps 데모 생성기](https://azuredevopsdemogenerator.azurewebsites.net)로 이동합니다. 이 유틸리티 사이트에서는 계정 내에서 새 Azure DevOps 프로젝트를 만드는 프로세스를 자동으로 진행할 수 있습니다. 이 프로젝트에는 랩에 필요한 작업 항목, 리포지토리 등의 콘텐츠가 미리 입력되어 있습니다. 

    > **참고**: 이 사이트에 대한 자세한 내용은 https://docs.microsoft.com/ko-kr/azure/devops/demo-gen을 참조하세요.

1.  **로그인**을 클릭하고 Azure DevOps 구독과 연결된 Microsoft 계정을 사용하여 로그인합니다.
1.  필요한 경우 **Azure DevOps 데모 생성기** 페이지에서 **수락**을 클릭하여 Azure DevOps 구독에 액세스하는 데 필요한 권한 요청을 수락합니다.
1.  **새 프로젝트 만들기** 페이지의 **새 프로젝트 이름** 텍스트 상자에 **에이전트 풀 구성 및 파이프라인 스타일 이해**를 입력합니다. 그런 다음 **조직 선택** 드롭다운 목록에서 Azure DevOps 조직을 선택하고 **템플릿 선택**을 클릭합니다.
1.  **템플릿 선택** 페이지에서 **PartsUnlimited** 템플릿을 클릭한 다음 **템플릿 선택**을 클릭합니다.
1.  **새 프로젝트 만들기** 페이지로 돌아와서 **ARM 출력** 레이블 아래의 체크박스를 선택하고 **프로젝트 만들기**를 클릭합니다.

    > **참고**: 프로세스가 완료될 때까지 기다립니다. 약 2분이 소요됩니다. 프로세스가 실패하면 DevOps 조직으로 이동하여 프로젝트를 삭제한 후에 다시 시도합니다.

1.  **새 프로젝트 만들기** 페이지에서 **프로젝트로 이동**을 클릭합니다.

#### 작업 2: Visual Studio Code 설치

이 작업에서는 Visual Studio Code를 설치합니다. 이 필수 구성 요소를 이미 구현한 경우에는 다음 작업을 바로 진행해도 됩니다.

1.  Visual Studio Code를 아직 설치하지 않았다면 웹 브라우저 창에서 [Visual Studio Code 다운로드 페이지](https://code.visualstudio.com/)로 이동하여 Visual Studio Code를 다운로드한 다음 설치합니다. 

#### 작업 3: 이미지 배포를 위해 랩 환경 준비

이 작업에서는 자체 호스팅 에이전트를 프로비전하는 데 사용할 이미지 배포를 위해 랩 환경을 준비합니다.

> **참고**: 자체 호스팅 에이전트와 해당 구성의 위치는 요구 사항에 따라 크게 달라집니다. 이 연습에서는 Microsoft에서 호스트하는 에이전트에 사용되는 것과 같은 이미지를 기반으로 하는 Azure VM을 사용합니다. 이미지는 [가상 환경 GitHub 리포지토리](https://github.com/actions/virtual-environments)에서 제공됩니다. GitHub Actions에서 호스트하는 실행기용 이미지도 이 페이지에서 확인할 수 있습니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [GitHub](https://github.com)로 이동한 다음 GitHub 계정에 로그인합니다.
1.  GitHub 계정이 표시된 웹 브라우저의 오른쪽 위에서 사용자 프로필을 나타내는 둥근 아이콘을 클릭한 다음 드롭다운 메뉴에서 **설정**을 클릭합니다.
1.  **공개 프로필** 페이지 왼쪽의 세로 메뉴에서 **개발자 설정**을 클릭합니다.
1.  **GitHub 앱** 페이지 왼쪽의 세로 메뉴에서 **개인용 액세스 토큰**을 클릭합니다.
1.  **개인용 액세스 토큰** 페이지의 오른쪽 위에서 **새 토큰 생성**을 클릭합니다.
1.  **새 개인용 액세스 토큰** 페이지의 **메모** 상자에 **az400m05l05a lab**을 입력하고 **범위 선택** 섹션에서 **read:packages** 체크박스를 선택한 후에 **토큰 생성**을 클릭합니다.
1.  **개인용 액세스 토큰** 페이지에서 새로 생성된 토큰을 확인하고 해당 값을 적어 둡니다. 이 작업 뒷부분에서 해당 토큰이 필요합니다.

    > **참고**: 지금 개인용 액세스 토큰을 적어 두세요. 현재 페이지에서 다른 위치로 이동하면 토큰의 값을 검색할 수 없습니다. 

1.  랩 컴퓨터의 웹 브라우저에서 [Packer 다운로드 페이지](https://www.packer.io/downloads)로 이동하여 최신 버전의 Windows 64비트 버전 Packer를 다운로드하여 **packer.exe** 이진 파일이 포함된 보관 파일을 열어 **C:\Windows** 디렉터리에 압축을 풉니다. 
1.  랩 컴퓨터에서 Windows PowerShell ISE를 관리자 권한으로 시작하고 **관리자: Windows PowerShell ISE** 창의 콘솔 창에서 다음 명령을 실행하여 Chocolatey를 설치합니다.

    ```powershell
    Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
    ```

1.  **관리자: Windows PowerShell ISE** 창의 콘솔 창에서 다음 명령을 실행하여 git를 설치합니다.

    ```powershell
    choco install git -params '"/GitOnlyOnPath"' -y
    ```

1.  **관리자: Windows PowerShell ISE** 창의 콘솔 창에서 다음 명령을 실행하여 packer를 설치합니다.

    ```powershell
    choco install packer -y
    ```

1.  **관리자: Windows PowerShell ISE** 창의 콘솔 창에서 다음 명령을 실행하여 Azure CLI를 설치합니다.

    ```powershell
    Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; rm .\AzureCLI.msi
    ```

1.  **관리자: Windows PowerShell ISE** 창의 콘솔 창에서 다음 명령을 실행하여 PowerShell을 설치합니다. 설치를 계속 진행할지를 확인하라는 메시지가 표시되면 **모두 예**를 클릭합니다.

    ```powershell
    Install-Module -Name Az
    ```

1.  **관리자: Windows PowerShell ISE** 창의 콘솔 창에서 다음 명령을 실행하여 Azure 구독에 로그인합니다. 자격 증명을 입력하라는 메시지가 표시되면 Azure 구독의 소유자 역할, 그리고 Azure 구독과 연결된 Azure AD 테넌트의 전역 관리자 역할이 지정되어 있는 Microsoft 계정 또는 Azure AD 계정을 사용하여 인증합니다.

    ```powershell
    Add-AzAccount
    ```

1.  **관리자: Windows PowerShell ISE** 창의 콘솔 창에서 다음 명령을 실행하여 git를 설치합니다.

    ```powershell
    New-Item -Type Directory -Path 'C:\Labfiles' -Force
    Set-Location c:\Labfiles
    git clone https://github.com/actions/virtual-environments.git virtual-environments -q
    ```

1.  랩 컴퓨터에서 파일 탐색기를 시작하여 **C:\\Labfiles\\virtual-environments\\images\\win** 디렉터리로 이동한 다음 메모장에서 **windows2019.json** 파일을 열어 해당 내용을 검토합니다.

    > **참고**: 이 파일은 이미지의 내용을 정의합니다. **provisioners** 섹션을 수정하여 이 파일의 구성을 사용자 지정하면 이미지 프로비전 시간을 조정할 수 있습니다.

1.  랩 컴퓨터의 파일 탐색기에서 **C:\\Labfiles\\virtual-environments\\helpers** 디렉터리로 이동한 다음 메모장에서 **GenerateResourcesAndImage.ps1** 파일을 엽니다. 
1.  **GenerateResourcesAndImage.ps1** 파일의 내용이 표시된 메모장에서 `New-AzStorageAccount -ResourceGroupName $ResourceGroupName -AccountName $storageAccountName -Location $AzureLocation -SkuName "Standard_LRS"`` 명령이 포함된 줄을 찾은 다음 `"Standard_LRS"`를 `"Premium_LRS"`로 바꾸고 변경 내용을 저장한 후에 파일을 닫습니다.

    > **참고**: 이 변경은 이미지를 더 빠르게 프로비전하기 위한 작업입니다.

1.  랩 컴퓨터의 파일 탐색기에서 **C:\\Labfiles\\virtual-environments\\images\\win** 디렉터리로 이동한 다음 메모장에서 **windows2016.json** 파일을 엽니다. 그런 후에 아래 내용을 붙여넣어 기존 내용을 바꾸고 변경 내용을 저장한 다음 파일을 닫습니다.

    > **참고**: 이러한 변경은 이 랩 시나리오에서 이미지를 더 빠르게 프로비전하려면 반드시 진행해야 하는 작업입니다. 일반적으로는 요구 사항과 일치하도록 이미지를 조정해야 합니다. 

    ```json
    {
        "variables": {
            "client_id": "{{env `ARM_CLIENT_ID`}}",
            "client_secret": "{{env `ARM_CLIENT_SECRET`}}",
            "subscription_id": "{{env `ARM_SUBSCRIPTION_ID`}}",
            "tenant_id": "{{env `ARM_TENANT_ID`}}",
            "object_id": "{{env `ARM_OBJECT_ID`}}",
            "resource_group": "{{env `ARM_RESOURCE_GROUP`}}",
            "storage_account": "{{env `ARM_STORAGE_ACCOUNT`}}",
            "temp_resource_group_name": "{{env `TEMP_RESOURCE_GROUP_NAME`}}",
            "location": "{{env `ARM_RESOURCE_LOCATION`}}",
            "virtual_network_name": "{{env `VNET_NAME`}}",
            "virtual_network_resource_group_name": "{{env `VNET_RESOURCE_GROUP`}}",
            "virtual_network_subnet_name": "{{env `VNET_SUBNET`}}",
            "private_virtual_network_with_public_ip": "{{env `PRIVATE_VIRTUAL_NETWORK_WITH_PUBLIC_IP`}}",
            "vm_size": "Standard_D8s_v3",
            "run_scan_antivirus": "false",
            "root_folder": "C:",
            "toolset_json_path": "{{env `TEMP`}}\\toolset.json",
            "image_folder": "C:\\image",
            "imagedata_file": "C:\\imagedata.json",
            "helper_script_folder": "C:\\Program Files\\WindowsPowerShell\\Modules\\",
            "psmodules_root_folder": "C:\\Modules",
            "install_user": "installer",
            "install_password": null,
            "capture_name_prefix": "packer",
            "image_version": "dev",
            "image_os": "win16"
        },
        "sensitive-variables": [
            "install_password",
            "client_secret"
        ],
        "builders": [
            {
                "name": "vhd",
                "type": "azure-arm",
                "client_id": "{{user `client_id`}}",
                "client_secret": "{{user `client_secret`}}",
                "subscription_id": "{{user `subscription_id`}}",
                "object_id": "{{user `object_id`}}",
                "tenant_id": "{{user `tenant_id`}}",
                "os_disk_size_gb": "256",
                "location": "{{user `location`}}",
                "vm_size": "{{user `vm_size`}}",
                "resource_group_name": "{{user `resource_group`}}",
                "storage_account": "{{user `storage_account`}}",
                "temp_resource_group_name": "{{user `temp_resource_group_name`}}",
                "capture_container_name": "images",
                "capture_name_prefix": "{{user `capture_name_prefix`}}",
                "virtual_network_name": "{{user `virtual_network_name`}}",
                "virtual_network_resource_group_name": "{{user `virtual_network_resource_group_name`}}",
                "virtual_network_subnet_name": "{{user `virtual_network_subnet_name`}}",
                "private_virtual_network_with_public_ip": "{{user `private_virtual_network_with_public_ip`}}",
                "os_type": "Windows",
                "image_publisher": "MicrosoftWindowsServer",
                "image_offer": "WindowsServer",
                "image_sku": "2016-Datacenter",
                "communicator": "winrm",
                "winrm_use_ssl": "true",
                "winrm_insecure": "true",
                "winrm_username": "packer"
            }
        ],
        "provisioners": [
            {
                "type": "powershell",
                "inline": [
                    "New-Item -Path {{user `image_folder`}} -ItemType Directory -Force"
                ]
            },
            {
                "type": "file",
                "source": "{{ template_dir }}/scripts/ImageHelpers",
                "destination": "{{user `helper_script_folder`}}"
            },
            {
                "type": "file",
                "source": "{{ template_dir }}/scripts/SoftwareReport",
                "destination": "{{user `image_folder`}}"
            },
            {
                "type": "file",
                "source": "{{ template_dir }}/post-generation",
                "destination": "C:/"
            },
            {
                "type": "file",
                "source": "{{ template_dir }}/scripts/Tests",
                "destination": "{{user `image_folder`}}"
            },
            {
                "type": "file",
                "source": "{{template_dir}}/toolsets/toolset-2016.json",
                "destination": "{{user `toolset_json_path`}}"
            },
            {
                "type": "windows-shell",
                "inline": [
                    "net user {{user `install_user`}} {{user `install_password`}} /add /passwordchg:no /passwordreq:yes /active:yes /Y",
                    "net localgroup Administrators {{user `install_user`}} /add",
                    "winrm set winrm/config/service/auth @{Basic=\"true\"}",
                    "winrm get winrm/config/service/auth"
                ]
            },
            {
                "type": "powershell",
                "inline": [
                    "if (-not ((net localgroup Administrators) -contains '{{user `install_user`}}')) { exit 1 }"
                ]
            },
            {
                "type": "powershell",
                "environment_vars": [
                    "ImageVersion={{user `image_version`}}",
                    "TOOLSET_JSON_PATH={{user `toolset_json_path`}}",
                    "PSMODULES_ROOT_FOLDER={{user `psmodules_root_folder`}}"
                ],
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-PowerShellModules.ps1",
                    "{{ template_dir }}/scripts/Installers/Initialize-VM.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-WebPlatformInstaller.ps1"
                ],
                "execution_policy": "unrestricted"
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Update-DotnetTLS.ps1"
                ]
            },
            {
                "type": "windows-restart",
                "restart_timeout": "30m"
            },
            {
                "type": "powershell",
                "inline": [
                    "setx ImageVersion {{user `image_version` }} /m",
                    "setx ImageOS {{user `image_os` }} /m"
                ]
            },
            {
                "type": "powershell",
                "environment_vars": [
                    "IMAGE_VERSION={{user `image_version`}}",
                    "IMAGEDATA_FILE={{user `imagedata_file`}}"
                ],
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Update-ImageData.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-PowershellCore.ps1"
                ]
            },
            {
                "type": "windows-restart",
                "restart_timeout": "30m"
            },
            {
                "type": "powershell",
                "valid_exit_codes": [
                    0,
                    3010
                ],
                "environment_vars": [
                    "TOOLSET_JSON_PATH={{user `toolset_json_path`}}"
                ],
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-VS.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-NET48.ps1"
                ],
                "elevated_user": "{{user `install_user`}}",
                "elevated_password": "{{user `install_password`}}"
            },
                {
                "type": "powershell",
                "environment_vars": [
                    "TOOLSET_JSON_PATH={{user `toolset_json_path`}}"
                ],
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-Nuget.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-Vsix.ps1"
                ]
            },
            {
                "type": "windows-restart",
                "restart_timeout": "30m"
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-NodeLts.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-7zip.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-Packer.ps1"
                ]
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-OpenSSL.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-Git.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-GitHub-CLI.ps1"
                ]
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Enable-DeveloperMode.ps1"
                ],
                "elevated_user": "{{user `install_user`}}",
                "elevated_password": "{{user `install_password`}}"
            },
            {
                "type": "windows-shell",
                "inline": [
                    "wmic product where \"name like '%%microsoft azure powershell%%'\" call uninstall /nointeractive"
                ]
            },
            {
                "type": "powershell",
                "environment_vars": [
                    "TOOLSET_JSON_PATH={{user `toolset_json_path`}}"
                ],
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-Cmake.ps1"
                ]
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-GitVersion.ps1"
                ]
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Configure-DynamicPort.ps1"
                ],
                "elevated_user": "{{user `install_user`}}",
                "elevated_password": "{{user `install_password`}}"
            },
            {
                "type": "windows-restart",
                "restart_timeout": "30m"
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Finalize-VM.ps1"
                ]
            },
            {
                "type": "windows-restart",
                "restart_timeout": "30m"
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Configure-Antivirus.ps1",
                    "{{ template_dir }}/scripts/Installers/Disable-JITDebugger.ps1"
                ]
            },
            {
                "type": "powershell",
                "inline": [
                    "if( Test-Path $Env:SystemRoot\\System32\\Sysprep\\unattend.xml ){ rm $Env:SystemRoot\\System32\\Sysprep\\unattend.xml -Force}",
                    "& $env:SystemRoot\\System32\\Sysprep\\Sysprep.exe /oobe /generalize /quiet /quit",
                    "while($true) { $imageState = Get-ItemProperty HKLM:\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Setup\\State | Select ImageState; if($imageState.ImageState -ne 'IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE') { Write-Output $imageState.ImageState; Start-Sleep -s 10  } else { break } }"
                ]
            }
        ]
    }
    ```

1.  랩 컴퓨터의 파일 탐색기에서 **C:\\Labfiles\\virtual-environments\\images\\win\\scripts\\Installers** 디렉터리로 이동하여 메모장에서 **VisualStudio.Tests.ps1** 파일을 열고 다음 섹션을 찾습니다.

    ```powershell
    $workLoads = @(
	"--allWorkloads --includeRecommended"
	$requiredComponents
	"--remove Component.CPython3.x64"
    )
    ```

1.  **VisualStudio.Tests.ps1** 파일의 내용이 표시된 메모장 창에서 이전 단계에서 확인한 섹션을 다음 내용을 바꿉니다.

    ```powershell
    $workLoads = @(
	"--add Microsoft.VisualStudio.Workload.NetWeb --add Component.GitHub.VisualStudio --includeRecommended"
	"--remove Component.CPython3.x64"
    )
    ```

    > **참고**: 이 변경은 이 랩 시나리오에서 이미지를 더 빠르게 프로비전하려면 반드시 진행해야 하는 작업입니다. 일반적으로는 요구 사항과 일치하도록 설치할 Visual Studio 구성 요소를 조정해야 합니다. 

1.  랩 컴퓨터의 파일 탐색기에서 **C:\\Labfiles\\virtual-environments\\images\\win\\scripts\\Tests** 디렉터리로 이동하여 메모장에서 **VisualStudio.Tests.ps1** 파일을 열고 `$expectedComponents = Get-ToolsetContent | Select-Object -ExpandProperty visualStudio | Select-Object -ExpandProperty workloads` 줄을 `$expectedComponents = "Microsoft.VisualStudio.Workload.NetWeb"`으로 바꾸고 변경 내용을 저장한 후에파일을 닫습니다.

    > **참고**: 이 변경은 이 랩 시나리오에서 이미지를 더 빠르게 프로비전하려면 반드시 진행해야 하는 작업입니다. 일반적으로는 요구 사항과 일치하도록 테스트를 조정해야 합니다. 

1.  **관리자: Windows PowerShell ISE** 창으로 다시 전환하여 콘솔 창에서 다음 명령을 실행해 Packer PowerShell 모듈을 가져옵니다. 다음 단계에서 자체 호스팅 에이전트를 프로비전하는 데 사용할 운영 체제 이미지를 생성하는 데 이 모듈을 사용합니다.

    ```powershell
    Set-Location c:\Labfiles\virtual-environments
    Import-Module .\helpers\GenerateResourcesAndImage.ps1
    ```

1.  **관리자: Windows PowerShell ISE** 창의 스크립트 창에서 다음 명령을 실행하여 Packer PowerShell 모듈을 사용해 자체 호스팅 에이전트를 프로비전하는 데 사용할 운영 체제 이미지를 생성합니다. 여기서 `<Azure_region>` 자리 표시자는 Azure VM을 배포할 Azure 지역의 이름으로 바꾸고, `<GitHub_PAT>` 자리 표시자는 이 작업의 앞부분에서 생성한 GitHub 개인용 액세스 토큰의 값으로 바꿉니다.

    ```powershell
    $location = '<Azure_region>'
    $githubPAT = '<GitHub_PAT>'
    $random = (-join (((48..57)+(65..90)+(97..122)) * 80 | Get-Random -Count 6 |%{[char]$_})).ToLower()
    $rgName = "az400m05$random-RG"
    $subscriptionId = (Get-AzSubscription).SubscriptionId  
    GenerateResourcesAndImage -SubscriptionId $subscriptionId -ResourceGroupName $rgName -ImageGenerationRepositoryRoot "$pwd" -ImageType 'Windows2016' -AzureLocation $location -GitHubFeedToken $githubPAT
    ```

1.  메시지가 표시되면 앞에서 사용한 것과 같은 사용자 계정으로 로그인하여 Azure 구독과 연결된 Azure AD 테넌트에 인증합니다.

    > **참고**: 스크립트가 완료될 때까지 기다리지 말고 다음 연습을 진행하세요. 이미지 프로비전은 60분 정도 걸릴 수 있습니다.

### 연습 1: YAML 기반 Azure DevOps 파이프라인 작성

이 연습에서는 클래식 Azure DevOps 파이프라인을 YAML 기반 파이프라인으로 변환합니다. 

#### 작업 1: Azure DevOps YAML 파이프라인 만들기

이 작업에서는 템플릿 기반 Azure DevOps YAML 파이프라인을 만듭니다.

1.  **에이전트 풀 구성 및 파이프라인 스타일 이해** 프로젝트가 열린 Azure DevOps 포털이 표시되어 있는 웹 브라우저 왼쪽의 세로 탐색 창에서 **Pipelines**를 클릭합니다. 
1.  **파이프라인** 창의 **최근** 탭에서 **새 파이프라인**을 클릭합니다.
1.  **코드 위치** 창에서 **Azure Repos Git**를 클릭합니다. 
1.  **리포지토리 선택** 창에서 **PartsUnlimited**를 클릭합니다.
1.  **파이프라인 구성** 창에서 **시작 파이프라인**을 클릭합니다.
1.  **파이프라인 YAML 검토** 페이지에서 샘플 파이프라인을 검토하고 **저장 및 실행** 단추 옆의 아래쪽 캐럿 기호를 클릭한 다음 **저장**을 클릭하고 **저장** 창에서 **저장**을 클릭합니다.

    > **참고**: 이렇게 하면 프로젝트 코드를 호스트하는 리포지토리의 루트 디렉터리에 **azure-pipelines.yml** 파일이 생성됩니다.

    > **참고**: 여기서는 샘플 YAML 파이프라인 내용을 클래식 편집기로 생성한 파이프라인의 코드로 바꾼 다음, 클래식 파이프라인과 YAML 파이프라인의 차이를 반영하여 해당 코드를 수정할 것입니다.

#### 작업 2: 클래식 파이프라인을 YAML 파이프라인으로 변환

이 작업에서는 클래식 파이프라인을 YAML 파이프라인으로 변환합니다.

1.  **에이전트 풀 구성 및 파이프라인 스타일 이해** 프로젝트가 열린 Azure DevOps 포털이 표시되어 있는 웹 브라우저 왼쪽의 세로 탐색 창에서 **Pipelines**를 클릭합니다. 
1.  **파이프라인** 창의 **최근** 탭에서 **PartsUnlimitedE2E** 항목이 포함된 항목의 오른쪽 모서리 위에 마우스를 놓으면 **자세히** 메뉴를 나타내는 세로 줄임표가 표시됩니다. 이 줄임표 기호를 클릭하고 드롭다운 메뉴에서 **편집**을 클릭합니다. 그러면 랩을 시작할 때 생성한 프로젝트에 포함되어 있는 빌드 파이프라인이 표시됩니다. 
1.  **PartsUnlimitedE2E** 편집 창의 **작업** 탭에서 **트리거**를 클릭하고 **PartsUnlimited** 창 오른쪽에서 **연속 통합 사용**의 선택을 취소합니다. 그런 다음 **저장 및 큐에 넣기** 단추 옆에 있는 아래쪽 캐럿을 클릭하고 드롭다운 메뉴에서 **저장**을 클릭한 후에 **빌드 파이프라인 저장**에서 **저장**을 클릭합니다.

    > **참고**: 이렇게 하면 이 랩에서 리포지토리 변경으로 인해 자동 빌드가 의도치 않게 실행되는 상황을 방지할 수 있습니다.

    > **참고**: 기존 파이프라인의 내용을 새 파이프라인에 복사한 후에 기존 파이프라인을 삭제해도 됩니다.

1.  Azure DevOps 포털 왼쪽의 세로 탐색 창에 있는 **Pipelines** 섹션에서 **파이프라인**을 클릭합니다. 
1.  **파이프라인** 창의 **최근** 탭에서 **PartsUnlimitedE2E** 항목을 클릭합니다. 
1.  **PartsUnlimitedE2E** 창의 **실행** 탭 오른쪽 위에서 세로 줄임표 기호를 클릭한 다음 드롭다운 메뉴에서 **YAML로 내보내기**를 클릭합니다. 이렇게 하면 로컬 **다운로드** 폴더에 **build.yml** 파일이 자동으로 다운로드됩니다.

    > **참고**: **YAML로 내보내기** 기능은 이전에 Azure DevOps 포털 내 파이프라인 편집기 창에서 제공되었던 **YAML 보기** 옵션 대신 제공됩니다. 이전 옵션 사용 시에는 YAML 내용을 한 번에 한 작업씩만 확인할 수 있었습니다. 새 기능은 기존의 클래식 파이프라인 인프라와 YAML 파이프라인 인프라(YAML 구문 분석 라이브러리 포함)를 모두 활용하므로 두 인프라 간의 더욱 정확한 변환이 가능합니다. 이 기능은 다음과 같은 파이프라인 구성 요소를 지원합니다.

    - 단일 작업/여러 작업
    - 체크 아웃 옵션
    - 실행 계획 병렬 처리
    - 시간 제한 및 이름 메타데이터
    - 수요
    - 일정 및 기타 트리거
    - 풀 선택(기본값과 다른 작업 포함)
    - 모든 작업과 모든 입력(기본 입력 최적화 포함)
    - 작업 및 단계 조건
    - 작업 그룹 언롤

    > **참고**: 새 기능에서 지원되지 않는 구성 요소는 변수 및 표준 시간대 변환뿐입니다. YAML에서 정의된 변수는 Azure DevOps 포털 사용자 인터페이스에 설정된 변수를 재정의합니다. **YAML로 내보내기** 기능 사용 시 클래식 파이프라인 변수가 검색되면 새로 생성되는 YAML 파이프라인 정의 내의 주석에 해당 변수가 명시적으로 포함됩니다. 마찬가지로 YAML의 cron 일정은 UTC로 표시되므로 조직 표준 시간대를 사용하는 클래식 일정 역시 주석에 포함됩니다.

    > **참고**: 이 기능에 대한 자세한 내용은 ["YAML 보기" 대신 제공되는 기능](https://devblogs.microsoft.com/devops/replacing-view-yaml/)을 참조하세요.

1.  랩 컴퓨터에서 Visual Studio Code를 시작한 다음 Visual Studio Code를 사용하여 **build.yml** 파일을 엽니다. 이 파일의 내용은 다음과 같습니다.

    ```yaml
    name: $(date:yyyyMMdd)$(rev:.r)
    jobs:
    - job: Phase_1
      displayName: Phase 1
      cancelTimeoutInMinutes: 1
      pool:
        vmImage: vs2017-win2016
      steps:
      - checkout: self
      - task: NuGetInstaller@0
        name: NuGetInstaller_1
        displayName: NuGet restore
        inputs:
          solution: '**\*.sln'
      - task: VSBuild@1
        name: VSBuild_2
        displayName: Build solution
        inputs:
          vsVersion: 15.0
          msbuildArgs: /p:DeployOnBuild=true /p:WebPublishMethod=Package /p:PackageAsSingleFile=true /p:SkipInvalidConfigurations=true /p:PackageLocation="$(build.stagingDirectory)" /p:IncludeServerNameInBuildInfo=True /p:GenerateBuildInfoConfigFile=true /p:BuildSymbolStorePath="$(SymbolPath)" /p:ReferencePath="C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\Extensions\Microsoft\Pex"
          platform: $(BuildPlatform)
          configuration: $(BuildConfiguration)
      - task: VSTest@1
        name: VSTest_3
        displayName: Test Assemblies
        inputs:
          testAssembly: '**\$(BuildConfiguration)\*test*.dll;-:**\obj\**'
          codeCoverageEnabled: true
          vsTestVersion: latest
          platform: $(BuildPlatform)
          configuration: $(BuildConfiguration)
      - task: CopyFiles@2
        name: CopyFiles1
        displayName: Copy Files
        inputs:
          SourceFolder: $(build.sourcesdirectory)
          Contents: '**/*.json'
          TargetFolder: $(build.artifactstagingdirectory)
      - task: PublishBuildArtifacts@1
        name: PublishBuildArtifacts_5
        displayName: Publish Artifact
        inputs:
          PathtoPublish: $(build.artifactstagingdirectory)
          TargetPath: '\\my\share\$(Build.DefinitionName)\$(Build.BuildNumber)'
    ...
    ```

1.  Visual Studio Code 창 최상위 메뉴에서 **선택**을 클릭하고 드롭다운 메뉴에서 **모두 선택**을 클릭합니다.
1.  Visual Studio Code 창 최상위 메뉴에서 **편집**을 클릭하고 드롭다운 메뉴에서 **복사**를 클릭합니다.
1.  Azure DevOps 포털이 표시된 브라우저 창으로 전환하여 왼쪽의 세로 탐색 창에 있는 **Pipelines** 섹션에서 **파이프라인**을 클릭합니다. 
1.  **파이프라인** 창의 **최근** 탭에서 **PartsUnlimited**를 선택하고 **PartsUnlimited** 창에서 **편집**을 선택합니다.
1.  **PartsUnlimited** 편집 창에서 파이프라인의 기존 YAML 내용을 선택한 후 클립보드의 내용으로 바꿉니다.
1.  **PartsUnlimited** 편집 창 오른쪽 위의 **저장**을 클릭하고 **저장** 창에서 **저장**을 클릭합니다. 그러면 이 파이프라인을 기반으로 하는 빌드가 자동으로 트리거됩니다. 
1.  Azure DevOps 포털 왼쪽의 세로 탐색 창에 있는 Pipelines 섹션에서 파이프라인을 클릭합니다.
1.  **파이프라인** 창의 **최근** 탭에서 **PartsUnlimited** 항목을 클릭하고 **PartsUnlimited** 창의 **실행** 탭에서 최신 실행을 선택합니다. 그런 다음 해당 실행의 **요약** 창에서 아래쪽으로 스크롤하여 **작업** 섹션에서 **1단계**를 클릭하고 작업이 정상적으로 완료될 때까지 모니터링합니다. 

### 연습 2: Azure DevOps 에이전트 풀 관리

이 연습에서는 자체 호스팅 Azure DevOps 에이전트를 구현합니다.

> **참고**: 이 연습을 시작하기 전에 랩 첫부분에서 시작한 이미지 프로비전 스크립트의 실행이 정상적으로 완료되었는지 확인하세요.

#### 작업 1: Azure VM 기반 자체 호스팅 에이전트 배포

이 작업에서는 Azure VM을 배포합니다. 해당 VM은 이 랩의 시작 부분에서 생성한 사용자 지정 이미지를 사용하여 자체 호스팅 에이전트를 프로비전하는 데 사용됩니다.

1.  랩 컴퓨터에서 **관리자: Windows PowerShell ISE** 창으로 전환한 다음 이 랩의 시작 부분에서 호출한 **GenerateResourcesAndImage** 스크립트의 출력을 검토해 스토리지 계정 이름의 값을 확인합니다. 이렇게 하려면 생성된 vhd 파일의 전체 경로를 나타내는 **OSDiskUri** 문자열의 첫 부분을 살펴봅니다.

    > **참고**: 출력은 `OSDiskUri: https://az400m05l05xrg001.blob.core.windows.net/system/Microsoft.Compute/Images/images/packer-osDisk.40096600-7cac-47f4-8573-ffaa113c78b1.v
hd' 값과 같은 형식입니다. 여기서 `az400m05l05xrg001`이 스토리지 계정 이름을 나타냅니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [**Azure Portal**](https://portal.azure.com)로 이동한 다음 이전 작업에서 Azure VM을 프로비전하는 데 사용한 사용자 계정으로 로그인합니다. 
1.  Azure Portal에서 **스토리지 계정** 리소스 종류를 검색하여 선택하고 **스토리지 계정** 블레이드에서 이 작업 앞부분에서 확인한 스토리지 계정의 이름을 클릭합니다. 
1.  스토리지 계정 블레이드에서 스토리지 계정이 포함된 리소스 그룹의 이름을 적어 둡니다. 이 작업 뒷부분에서 해당 그룹이 필요합니다.
1.  스토리지 계정 블레이드 왼쪽의 세로 메뉴에 있는 **Blob service** 섹션에서 **컨테이너**를 클릭합니다.
1.  컨테이너 블레이드에서 **+ 컨테이너**를 클릭하고 **새 컨테이너** 블레이드의 **이름** 텍스트 상자에 **disks**를 입력합니다. 그런 다음 **공용 액세스 수준** 드롭 상자에 **개인(익명 권한 없음)** 항목이 있는지 확인하고 **만들기**를 클릭합니다.
1.  컨테이너 블레이드로 돌아온 다음 컨테이너 목록에서 **images**를 클릭하고 **images** 블레이드에서 새로 생성한 이미지를 나타내는 항목을 클릭합니다.
1.  이미지 속성이 표시된 블레이드의 **URL** 항목 옆에 있는 **클립보드에 복사** 아이콘을 클릭하고 복사한 값을 메모장에 붙여넣습니다. 다음 단계에서 해당 값이 필요합니다. 
1.  랩 컴퓨터에서 **관리자: Windows PowerShell ISE** 창으로 전환한 다음 스크립트 창에서 다음 명령을 실행하여 Azure Resource Manager 템플릿을 사용한 Azure VM 배포에 필요한 변수를 설정합니다. 여기서 `<resource_group_name>` 자리 표시자는 이 작업 앞부분에서 확인한 리소스 그룹의 이름으로, `<storage_account_name>` 자리 표시자는 이 작업 앞부분에서 확인한 스토리지 계정의 이름으로, 그리고 `<image_url>` 자리 표시자는 이 작업 앞부분에서 확인한 이미지 URL의 이름으로 바꿉니다.

    ```powershell
    $rgName = `<resource_group_name>'
    $storageAccountName = '<storage_account_name>'
    $imageUrl = '<image_url>'
    ```

1.  랩 컴퓨터에서 파일 탐색기 창으로 전환하여 **C:\\Labfiles** 폴더에 다음과 같은 내용을 새 파일 **az400m05-vm0.deployment.template.json**을 만듭니다. 그런 다음 해당 파일을 저장하고 닫습니다. 

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
          "vmSize": {
            "type": "string",
            "defaultValue": "Standard_D2s_v3",
            "metadata": {
              "description": "Virtual machine size"
            }
          },
          "adminUsername": {
            "type": "string",
            "metadata": {
              "description": "Admin username"
            }
          },
          "adminPassword": {
            "type": "securestring",
            "metadata": {
              "description": "Admin password"
            }
          },
          "storageAccountName": {
            "type": "string",
            "metadata": {
               "description": "storage account name"
            }
          },
          "imageUrl": {
            "type": "string",
            "metadata": {
               "description": "image Url"
            }
          },
          "diskContainer": {
            "type": "string",
            "defaultValue": "disks",
            "metadata": {
              "description": "disk container"
            }
          }
        },
      "variables": {
        "vmName": "az400m05-vm0",
        "osType": "Windows",
        "osDiskVhdName": "[concat('http://',parameters('storageAccountName'),'.blob.core.windows.net/',parameters('diskContainer'),'/',variables('vmName'),'-osDisk.vhd')]",
        "nicName": "az400m05-vm0-nic0",
        "virtualNetworkName": "az400m05-vnet0",
        "publicIPAddressName": "az400m05-vm0-nic0-pip",
        "nsgName": "az400m05-vm0-nic0-nsg",
        "vnetIpPrefix": "10.0.0.0/22", 
        "subnetIpPrefix": "10.0.0.0/24", 
        "subnetName": "subnet0",
        "subnetRef": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('virtualNetworkName'), variables('subnetName'))]",
        "computeApiVersion": "2018-06-01",
        "networkApiVersion": "2018-08-01"
      },
        "resources": [
            {
                "name": "[variables('vmName')]",
                "type": "Microsoft.Compute/virtualMachines",
                "apiVersion": "[variables('computeApiVersion')]",
                "location": "[resourceGroup().location]",
                "dependsOn": [
                    "[variables('nicName')]"
                ],
                "properties": {
                    "osProfile": {
                        "computerName": "[variables('vmName')]",
                        "adminUsername": "[parameters('adminUsername')]",
                        "adminPassword": "[parameters('adminPassword')]",
                        "windowsConfiguration": {
                            "provisionVmAgent": "true"
                        }
                    },
                    "hardwareProfile": {
                        "vmSize": "[parameters('vmSize')]"
                    },
                    "storageProfile": {
                        "osDisk": {
                            "name": "[concat(variables('vmName'),'-osDisk')]",
                            "osType": "[variables('osType')]",
                            "caching": "ReadWrite",
                            "image": {
                                "uri": "[parameters('imageUrl')]"
                            },
                            "vhd": {
                                "uri": "[variables('osDiskVhdName')]"
                            },
                            "createOption": "fromImage"
                        },
                        "dataDisks": []
                    },
                    "networkProfile": {
                        "networkInterfaces": [
                            {
                                "properties": {
                                    "primary": true
                                },
                                "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
                            }
                        ]
                    }
                }
            },
            {
                "type": "Microsoft.Network/virtualNetworks",
                "name": "[variables('virtualNetworkName')]",
                "apiVersion": "[variables('networkApiVersion')]",
                "location": "[resourceGroup().location]",
                "comments": "Virtual Network",
                "properties": {
                    "addressSpace": {
                        "addressPrefixes": [
                            "[variables('vnetIpPrefix')]"
                        ]
                    },
                    "subnets": [
                        {
                            "name": "[variables('subnetName')]",
                            "properties": {
                                "addressPrefix": "[variables('subnetIpPrefix')]"
                            }
                        }
                    ]
                }
            },
            {
                "name": "[variables('nicName')]",
                "type": "Microsoft.Network/networkInterfaces",
                "apiVersion": "[variables('networkApiVersion')]",
                "location": "[resourceGroup().location]",
                "comments": "Primary NIC",
                "dependsOn": [
                    "[variables('publicIpAddressName')]",
                    "[variables('nsgName')]",
                    "[variables('virtualNetworkName')]"
                ],
                "properties": {
                    "ipConfigurations": [
                        {
                            "name": "ipconfig1",
                            "properties": {
                                "subnet": {
                                    "id": "[variables('subnetRef')]"
                                },
                                "privateIPAllocationMethod": "Dynamic",
                                "publicIpAddress": {
                                    "id": "[resourceId('Microsoft.Network/publicIpAddresses', variables('publicIpAddressName'))]"
                                }
                            }
                        }
                    ],
                    "networkSecurityGroup": {
                        "id": "[resourceId('Microsoft.Network/networkSecurityGroups', variables('nsgName'))]"
                    }
                }
            },
            {
                "name": "[variables('publicIpAddressName')]",
                "type": "Microsoft.Network/publicIpAddresses",
                "apiVersion": "[variables('networkApiVersion')]",
                "location": "[resourceGroup().location]",
                "comments": "Public IP for Primary NIC",
                "properties": {
                    "publicIpAllocationMethod": "Dynamic"
                }
            },
            {
                "name": "[variables('nsgName')]",
                "type": "Microsoft.Network/networkSecurityGroups",
                "apiVersion": "[variables('networkApiVersion')]",
                "location": "[resourceGroup().location]",
                "comments": "Network Security Group (NSG) for Primary NIC",
                "properties": {
                    "securityRules": [
                        {
                            "name": "default-allow-rdp",
                            "properties": {
                                "priority": 1000,
                                "sourceAddressPrefix": "*",
                                "protocol": "Tcp",
                                "destinationPortRange": "3389",
                                "access": "Allow",
                                "direction": "Inbound",
                                "sourcePortRange": "*",
                                "destinationAddressPrefix": "*"
                            }
                        }
                    ]
                }
            }
        ],
        "outputs": {}
    }
    ```

1.  랩 컴퓨터의 파일 탐색기 창에서 **C:\\Labfiles** 폴더에 다음과 같은 내용을 새 파일 **az400m05-vm0.deployment.template.parameters.json**을 만듭니다. 그런 다음 해당 파일을 저장하고 닫습니다. 

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "vmSize": {
                "value": "Standard_D2s_v3"
            },
            "adminUsername": {
                "value": "Student"
            },
            "adminPassword": {
                "value": "Pa55w.rd1234"
            }
        }
    }
    ```    

1.  **관리자: Windows PowerShell ISE** 창의 스크립트 창에서 다음 명령을 실행하여 Azure VM을 배포합니다. VM은 이 랩의 시작 부분에서 생성한 사용자 지정 이미지를 사용하여 자체 호스팅 에이전트를 프로비전하는 데 사용됩니다.

    ```powershell
    Set-Location -Path 'C:\Labfiles'
    New-AzResourceGroupDeployment -ResourceGroupName $rgName -Name az400m05-vm0-deployment -TemplateFile .\az400m05-vm0.deployment.template.json -TemplateParameterFile .\az400m05-vm0.deployment.template.parameters.json -storageAccountName $storageAccountName -imageUrl $imageUrl
    ```

    > **참고**: 프로비전 프로세스가 완료될 때까지 기다립니다. 이 프로세스는 5분 정도 걸립니다. 


#### 작업 2: Azure DevOps 자체 호스팅 에이전트 구성

이 작업에서는 새로 프로비전한 Azure VM을 Azure DevOps 자체 호스팅 에이전트로 구성한 다음 빌드 파이프라인을 실행하는 데 사용합니다.

1.  랩 컴퓨터에서 Azure Portal이 표시된 웹 브라우저 창으로 전환합니다. 그런 다음 Azure Portal에서 **가상 머신** 리소스 종류를 검색하여 선택하고 **가상 머신** 블레이드에서 **az400m05-vm0**을 클릭합니다.
1.  **az400m05-vm0** 블레이드에서 **연결**을 클릭하고 드롭다운 메뉴에서 **RDP**를 클릭합니다. 그런 다음 **az400m05-vm0 \| 연결** 블레이드에서 **RDP 파일 다운로드**를 클릭하고 다운로드한 RDP 파일을 열어서 원격 데스크톱을 사용해 **az400m05-vm0** Azure VM에 연결합니다. 로그인하라는 메시지가 표시되면 사용자 이름으로는 **Student**, 암호로는 **Pa55w.rd1234**를 입력합니다.
1.  **az400m05-vm0**에 연결된 원격 데스크톱 세션 내에서 웹 브라우저를 시작하고 [Azure DevOps 포털](https://dev.azure.com)로 이동한 다음 Azure DevOps 조직과 연결된 Microsoft 계정을 사용하여 로그인합니다.
1.  Azure DevOps 포털에서 **에이전트 가져오기** 패널을 닫고 Azure DevOps 페이지 오른쪽 위의 **사용자 설정** 아이콘을 클릭합니다. 그런 다음 드롭다운 메뉴에서 **개인용 액세스 토큰**을 클릭하고 **개인용 액세스 토큰** 창에서 **+ 새 토큰**을 클릭합니다.
1.  **새 개인용 액세스 토큰 만들기** 창에서 **모든 범위 표시** 링크를 클릭하고 다음 설정을 지정한 후에 **만들기**를 클릭합니다. 다른 값은 모두 기본값으로 유지합니다.

    | 설정 | 값 |
    | --- | --- |
    | 이름 | **에이전트 풀 구성 및 파이프라인 스타일 이해 랩** |
    | 범위 | **에이전트 풀** |
    | 사용 권한 | **읽기 및 관리** |

1.  **성공** 창에서 개인용 액세스 토큰의 값을 클립보드에 붙여넣습니다.

    > **참고**: 지금 토큰을 복사해야 합니다. 이 창을 닫으면 토큰을 검색할 수 없습니다. 

1.  **성공** 창에서 **닫기**를 클릭합니다.
1.  Azure DevOps 포털에서 **개인용 액세스 토큰** 창 왼쪽 위의 **Azure DevOps** 기호를 클릭한 다음 왼쪽 아래의 **조직 설정** 레이블을 클릭합니다.
1.  **개요** 창 왼쪽의 세로 메뉴에 있는 **Pipelines** 섹션에서 **에이전트 풀**을 클릭합니다.
1.  **에이전트 풀** 창의 오른쪽 위에서 **풀 추가**를 클릭합니다. 
1.  **에이전트 풀 추가** 창의 **풀 유형** 드롭다운 목록에서 **자체 호스팅**을 선택하고 **이름** 텍스트 상자에 **az400m05l05a-pool**을 입력한 다음 **만들기**를 클릭합니다.
1.  **에이전트 풀** 창으로 돌아와서 새로 만든 **az400m05l05a-pool**을 나타내는 항목을 클릭합니다. 
1.  **az400m05l05a-pool** 창의 **작업** 탭에서 **에이전트** 머리글을 클릭합니다.
1.  **az400m05l05a-pool** 창의 **에이전트** 탭에서 **새 에이전트** 단추를 클릭합니다.
1.  **에이전트 가져오기** 창에서 **Windows** 및 **x64** 탭이 선택되어 있는지 확인하고 **다운로드**를 클릭하여 에이전트 이진 파일이 포함된 zip 보관 파일을 사용자 프로필 내의 로컬 **다운로드** 폴더에 다운로드합니다.

    > **참고**: 이 시점에서 현재 시스템 설정으로 인해 파일을 다운로드할 수 없다는 오류 메시지가 표시되면 Internet Explorer 창 오른쪽 위에 있는 **설정** 메뉴 머리글을 나타내는 톱니바퀴 기호를 클릭합니다. 그런 다음 드롭다운 메뉴에서 **인터넷 옵션**을 선택하고 **인터넷 옵션** 대화 상자에서 **고급**을 클릭합니다. 그리고 나서 **고급** 탭의 **원래대로**를 클릭하고 **Internet Explorer 기본 설정 복원** 대화 상자에서 **다시 설정**을 다시 클릭한 후에 **닫기**를 클릭하고 다운로드를 다시 시도해 봅니다. 

1.  **az400m05-vm0**에 연결된 원격 데스크톱 세션 내에서 Windows PowerShell을 관리자 권한으로 시작합니다. 그런 다음 **관리자: Windows PowerShell** 콘솔에서 다음 명령을 실행하여 **C:\\agent** 디렉터리를 만든 후에 다운로드한 보관 파일의 압축을 풀어 해당 내용을 이 디렉터리에 저장합니다.

    ```powershell
    New-Item -Type Directory -Path 'C:\agent'
    Set-Location -Path 'C:\agent'
    Add-Type -AssemblyName System.IO.Compression.FileSystem
    [System.IO.Compression.ZipFile]::ExtractToDirectory("$HOME\Downloads\vsts-agent-win-x64-2.179.0.zip", "$PWD")
    ```

1.  **az400m05-vm0**에 연결된 원격 데스크톱 세션 내의 **관리자: Windows PowerShell** 콘솔에서 다음 명령을 실행하여 에이전트를 구성합니다.

    ```powershell
    .\config.cmd
    ```

1.  메시지가 표시되면 다음 설정의 값을 지정합니다.

    | 설정 | 값 |
    | ------- | ------ |
    | 서버 URL 입력 | **https://dev.azure.com/`<organization_name>`** 형식의 Azure DevOps 조직 URL. 여기서 `<organization_name>`은 Azure DevOps 조직의 이름을 나타냅니다. |
    | 인증 유형 입력(PAT의 경우 Enter 키 누르기) | **Enter** 키 | 
    | 개인용 액세스 토큰 입력 | 이 작업의 앞부분에서 적어 둔 액세스 토큰 |
    | 에이전트 풀 입력(기본값을 사용하려면 Enter 키 누르기) | **az400m05l05a-pool** |
    | 에이전트 이름 입력 | **az400m05-vm0** |
    | 작업 폴더 입력(_work를 사용하려면 Enter 키 누르기) | **C:\agent** |
    | 각 단계의 작업에서 압축 풀기 수행 여부 입력 (N을 입력하려면 Enter 키 누르기) | **Enter** |
    | 에이전트를 서비스로 실행할지 여부 입력 (Y/N)(N을 입력하려면 Enter 키 누르기) | **Enter** |
    | 자동 로그온 구성 및 시작 시 에이전트 실행 여부 입력(Y/N)(N을 입력하려면 Enter 키 누르기) | **Enter** |

    > **참고**: 자체 호스팅 에이전트는 서비스로 실행할 수도 있고 대화형 프로세스로 실행할 수도 있습니다. 처음에는 대화형 모드를 사용하는 것이 좋습니다. 이렇게 하면 에이전트 기능을 쉽게 확인할 수 있기 때문입니다. 프로덕션 환경에서는 자동 로그온을 사용하도록 설정한 대화형 프로세스나 서비스로 에이전트를 실행해야 합니다. 이 두 모드에서는 실행 상태가 영구 보존되며, 운영 체제를 다시 시작할 때 에이전트가 자동으로 다시 시작되기 때문입니다.

1.  **az400m05-vm0**에 연결된 원격 데스크톱 세션 내의 **관리자: Windows PowerShell** 콘솔에서 다음 명령을 실행하여 대화형 모드에서 에이전트를 시작합니다.

    ```powershell
    .\run.cmd
    ```

    > **참고**: 에이전트가 **작업 수신 중** 상태를 보고하는지 확인합니다.

1.  Azure DevOps 포털이 표시된 브라우저 창으로 전환하여 **에이전트 가져오기** 창을 닫습니다.
1.  **az400m05l05a-pool** 창의 **에이전트** 탭으로 돌아와서 새로 구성한 에이전트가 **온라인** 상태로 목록에 표시되는지 확인합니다.
1.  Azure DevOps 포털이 표시된 웹 브라우저 창의 왼쪽 위에서 **Azure DevOps** 레이블을 클릭합니다.
1.  프로젝트 목록이 표시된 브라우저 창에서 **에이전트 풀 구성 및 파이프라인 스타일 이해** 프로젝트를 나타내는 타일을 클릭합니다. 
1.  **에이전트 풀 구성 및 파이프라인 스타일 이해** 창 왼쪽의 세로 탐색 창에 있는 **Pipelines** 섹션에서 **파이프라인**을 클릭합니다. 
1.  **파이프라인** 창의 **최근** 탭에서 **PartsUnlimited**를 선택하고 **PartsUnlimited** 창에서 **편집**을 선택합니다.
1.  **PartsUnlimited** 편집 창의 기존 YAML 기반 파이프라인에서 대상 에이전트 풀을 지정하는 줄 **6** `vmImage: vs2017-win2016`을 새로 만든 자체 호스팅 에이전트 풀을 지정하는 다음 내용으로 바꿉니다.

    ```yaml
    name: az400m05l05a-pool
    demands:
    - agent.name -equals az400m05-vm0
    ```

1.  **PartsUnlimited** 편집 창 오른쪽 위의 **저장**을 클릭하고 **저장** 창에서 **저장**을 다시 클릭합니다. 그러면 이 파이프라인을 기반으로 하는 빌드가 자동으로 트리거됩니다. 
1.  Azure DevOps 포털 왼쪽의 세로 탐색 창에 있는 Pipelines 섹션에서 파이프라인을 클릭합니다.
1.  **파이프라인** 창의 **최근** 탭에서 **PartsUnlimited** 항목을 클릭하고 **PartsUnlimited** 창의 **실행** 탭에서 최신 실행을 선택합니다. 그런 다음 해당 실행의 **요약** 창에서 아래쪽으로 스크롤하여 **작업** 섹션에서 **1단계**를 클릭하고 작업이 정상적으로 완료될 때까지 모니터링합니다. 

### 연습 3: Azure 랩 리소스 제거

이 연습에서는 예상치 못한 비용이 발생하지 않도록 이 랩에서 프로비전한 Azure 리소스를 제거합니다. 

>**참고**: 더 이상 사용하지 않는 새로 만든 Azure 리소스를 제거해야 합니다. 사용하지 않는 리소스를 제거하면 예기치 않은 비용이 발생하지 않습니다.

#### 작업 1: Azure 랩 리소스 제거

이 작업에서는 Azure Cloud Shell을 사용하여 불필요한 비용이 발생하지 않도록 이 랩에서 프로비전한 Azure 리소스를 제거합니다. 

1.  Azure Portal의 **Cloud Shell** 창에서 **Bash** 세션을 시작합니다.
1.  다음 명령을 실행하여 이 모듈의 전체 랩에서 생성된 모든 리소스 그룹을 나열합니다.

    ```sh
    az group list --query "[?starts_with(name,'az400m05')].name" --output tsv
    ```

1.  다음 명령을 실행하여 이 모듈의 전체 랩에서 만든 모든 리소스 그룹을 삭제합니다.

    ```sh
    az group list --query "[?starts_with(name,'az400m05')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**참고**: 명령은 비동기적으로 실행되므로(--nowait 매개 변수에 의해 결정됨) 동일한 Bash 세션 내에서 즉시 다른 Azure CLI 명령을 실행할 수 있지만 리소스 그룹이 실제로 제거되기까지 몇 분 정도 걸립니다.

#### 복습

이 랩에서는 클래식 파이프라인을 YAML 기반 파이프라인으로 변환하는 방법과 자체 호스팅 에이전트를 구현 및 사용하는 방법을 배웠습니다.
