---
lab:
    title: '랩: Resource Manager 템플릿를 사용하여 Azure 배포'
    az400Module: '모듈 13: Azure Tools를 사용하여 인프라 및 구성 관리'
---

# 랩: Resource Manager 템플릿를 사용하여 Azure 배포
# 학생 랩 매뉴얼

## 랩 개요

이 랩에서는 연결된 템플릿 개념을 활용하여 ARM(Azure Resource Manager) 템플릿을 만들고 모듈화합니다. 그런 다음 기본 배포 템플릿을 수정하여 연결된 템플릿과 업데이트된 종속성을 호출하고, 마지막으로 Azure에 템플릿을 배포합니다.

## 목표

이 랩을 완료하면 다음 작업을 수행할 수 있습니다.

- Resource Manager 템플릿 만들기
- 스토리지 리소스용 연결된 템플릿 만들기
- Azure Blob Storage에 연결된 템플릿을 업로드하고 SAS 토큰 생성
- 기본 템플릿을 수정하여 연결된 템플릿 호출
- 기본 템플릿을 수정하여 종속성 업데이트
- 연결된 템플릿을 사용하여 Azure에 리소스 배포

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
-   [Visual Studio Code](https://code.visualstudio.com/). 이 랩의 필수 구성 요소로 설치됩니다. 

#### Azure 구독 준비

-   기존 Azure 구독을 확인하거나 새 구독을 만듭니다.
-   Azure 구독의 소유자 역할, 그리고 Azure 구독과 연결된 Azure AD 테넌트의 전역 관리자 역할이 지정되어 있는 Microsoft 계정 또는 Azure AD 계정이 있는지 확인합니다.

### 연습 0: 랩 필수 구성 요소 구성

이 연습에서는 Visual Studio Code를 비롯한 랩의 필수 구성 요소를 설정합니다.

#### 작업 1: Git 및 Visual Studio Code 설치 및 구성

이 작업에서는 Visual Studio Code를 설치합니다. 이 필수 구성 요소를 이미 구현한 경우에는 다음 작업을 바로 진행해도 됩니다.

1.  Visual Studio Code를 아직 설치하지 않았다면 랩 컴퓨터에서 웹 브라우저를 시작하고 [Visual Studio Code 다운로드 페이지](https://code.visualstudio.com/)로 이동하여 Visual Studio Code를 다운로드한 다음 설치합니다. 

### 연습 1: Azure Resource Manager 템플릿 작성 및 배포

이 랩에서는 연결된 템플릿을 사용하여 Azure Resource Manager 템플릿을 만들고 모듈화합니다. 그런 다음 기본 배포 템플릿을 수정하여 연결된 템플릿과 업데이트된 종속성을 호출하고, 마지막으로 Azure에 템플릿을 배포합니다.

#### 작업 1: Resource Manager 템플릿 만들기

이 작업에서는 Visual Studio Code를 사용하여 Resource Manager 템플릿을 만듭니다.

1.  랩 컴퓨터에서 Visual Studio Code를 시작하고 Visual Studio Code에서 **파일** 최상위 메뉴를 클릭합니다. 그런 다음 드롭다운 메뉴에서 **기본 설정**을 선택하고 계단식 메뉴에서 **확장**을 선택한 후에 **확장 검색** 텍스트 상자에 **Azure Resource Manager(ARM) 도구**를 입력합니다. 그 후에 해당하는 검색 결과를 선택하고 **설치**를 클릭하여 Azure Resource Manager 도구를 설치합니다.
1.  Visual Studio Code에서 **파일** 최상위 메뉴를 클릭하고 드롭다운 메뉴에서 **파일 열기**를 선택합니다. 그런 다음 파일 열기 대화 상자에 **https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json** URL을 입력하고 **열기**를 클릭합니다.

    > **참고**: 여기서는 템플릿을 처음부터 만들지 않고 [Azure 빠른 시작 템플릿](https://azure.microsoft.com/ko-kr/resources/templates/) 중 하나인 **간단한 Windows 템플릿 VM 배포**를 사용합니다. GitHub([101-vm-simple-windows](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows))에서 이 템플릿을 다운로드할 수 있습니다.

1.  랩 컴퓨터에서 파일 탐색기를 열고 템플릿을 저장할 다음 로컬 폴더를 만듭니다.

    - **C:\\templates** 
    - **C:\\templates\\storage** 

1.  azuredeploy.json 템플릿이 표시된 Visual Studio Code 창으로 다시 전환하여 **파일** 최상위 메뉴를 클릭하고 드롭다운 메뉴에서 **다른 이름으로 저장**을 클릭하여 새로 만든 로컬 폴더에 템플릿을 **azuredeploy.json**으로 저장합니다.
1.  템플릿을 검토하여 구조를 자세히 파악합니다. 이 템플릿에는 5가지 리소스 종류가 포함되어 있습니다.

    - Microsoft.Storage/storageAccounts
    - Microsoft.Network/publicIPAddresses
    - Microsoft.Network/virtualNetworks
    - Microsoft.Network/networkInterfaces
    - Microsoft.Compute/virtualMachines

1.  Visual Studio Code에서 파일을 다시 저장합니다. 이번에는 대상으로 **C:\\templates\\storage**를 선택하고 파일 이름으로는 **storage.json**을 입력합니다.

    > **참고**: 같은 JSON 파일 2개(**C:\\templates\\azuredeploy.json** 및 **C:\\templates\\storage\\storage.json**)를 만들었습니다.

#### 작업 2: 스토리지 리소스용 연결된 템플릿 만들기

이 작업에서는 이전 작업에서 저장한 템플릿을 수정합니다. 즉, 연결된 스토리지 템플릿 **storage.json**은 첫 번째 템플릿을 통해 실행하도록 호출되며 스토리지 계정만 만들도록 수정합니다. 연결된 스토리지 템플릿은 기본 템플릿인 **azuredeploy.json**에 값을 다시 전달해야 합니다. 연결된 스토리지 템플릿의 outputs 요소에서 이 값을 정의합니다.

1.  Visual Studio Code 창에 **storage.json** 파일이 표시된 상태로 **resources** 섹션 아래에서 **storageAccounts** 리소스를 제외한 모든 resource 요소를 제거합니다. resources 섹션이 다음과 같이 표시되어야 합니다.

    ```json
    "resources": [
      {
        "type": "Microsoft.Storage/storageAccounts",
        "name": "[variables('storageAccountName')]",
        "location": "[parameters('location')]",
        "apiVersion": "2018-07-01",
        "sku": {
           "name": "Standard_LRS"
        },
        "kind": "Storage",
        "properties": {}
      }
    ],
    ```

1.  storageAccount의 name 요소 이름을 varibles에서 parameteres로 바꿉니다.

    ```json
    "resources": [
      {
        "type": "Microsoft.Storage/storageAccounts",
        "name": "[parameters('storageAccountName')]",
        "location": "[parameters('location')]",
        "apiVersion": "2018-07-01",
        "sku": {
           "name": "Standard_LRS"
        },
        "kind": "Storage",
        "properties": {}
      }
    ],
    ```

1.  다음으로 전체 변수 섹션과 모든 변수 정의를 제거합니다.

    ```json
    "variables": {
      "storageAccountName": "[concat('bootdiags', uniquestring(resourceGroup().id))]",
      "nicName": "myVMNic",
      "addressPrefix": "10.0.0.0/16",
      "subnetName": "Subnet",
      "subnetPrefix": "10.0.0.0/24",
      "virtualNetworkName": "MyVNET",
      "subnetRef": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('virtualNetworkName'), variables('subnetName'))]",
      "networkSecurityGroupName": "default-NSG"
    },
    ```

1.  다음으로는 location을 제외한 모든 매개 변수 값을 제거하고 다음 매개 변수 코드를 추가합니다. 완성된 코드는 다음과 같아야 합니다.

    ```json
    "parameters": {
      "location": {
        "type": "string",
        "defaultValue": "[resourceGroup().location]",
        "metadata": {
          "description": "Location for all resources."
        }
      },
        "storageAccountName":{
        "type": "string",
        "metadata": {
          "description": "Azure Storage account name."
        }
      }
    },
    ```

1.  그런 다음 output 섹션을 업데이트하여 storageURI 출력 값을 정의합니다. storageUri 값은 기본 템플릿의 가상 머신 리소스 정의에 필요합니다. 이 값을 기본 템플릿에 출력 값으로 다시 전달합니다. 출력이 다음과 같이 표시되도록 수정합니다.

    ```json
    "outputs": {
      "storageUri": {
        "type": "string",
        "value": "[reference(parameters('storageAccountName')).primaryEndpoints.blob]"
      }
    }
    ```

1. 마지막으로 템플릿 스키마 버전을 2015-01-01에서 2019-04-01로 업데이트합니다. 이렇게 하려면 템플릿 정의 파일의 처음 몇 줄을 다음과 같이 업데이트합니다.

```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "storageAccountName":{
          "type": "string",
          "metadata": {
            "description": "Azure Storage account name."
          }
```

1.  storage.json 템플릿을 저장합니다. 이제 연결된 스토리지 템플릿의 코드는 다음과 같습니다.

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "storageAccountName":{
          "type": "string",
          "metadata": {
            "description": "Azure Storage account name."
          }
        },
        "location": {
          "type": "string",
          "defaultValue": "[resourceGroup().location]",
          "metadata": {
            "description": "Location for all resources."
          }
        }
      },
      "resources": [
        {
          "type": "Microsoft.Storage/storageAccounts",
          "name": "[parameters('storageAccountName')]",
          "apiVersion": "2016-01-01",
          "location": "[parameters('location')]",
          "sku": {
            "name": "Standard_LRS"
          },
          "kind": "Storage",
          "properties": {}
        }
      ],
      "outputs": {
      "storageUri": {
        "type": "string",
        "value": "[reference(parameters('storageAccountName')).primaryEndpoints.blob]"
        }
      }
    }
    ```

#### 작업 3: Azure Blob Storage에 연결된 템플릿을 업로드하고 SAS 토큰 생성

이 작업에서는 이전 작업에서 만든 연결된 템플릿을 Azure Blob Storage에 업로드하고, 후속 배포에서 해당 템플릿에 액세스할 수 있도록 SAS 토큰을 생성합니다.

> **참고**: 템플릿에 연결할 때는 Azure Resource Manager 서비스가 http 또는 https를 통해 이 템플릿에 액세스할 수 있어야 합니다. 여기서는 서비스가 템플릿에 액세스할 수 있도록 연결된 스토리지 템플릿인 **storage.json**을 Azure의 Blob Storage에 업로드합니다. 그런 다음 디지털 서명된 URL을 생성합니다. 이 URL에서 해당 Blob에 액세스(제한된 액세스)할 수 있습니다. Azure Cloud Shell에서 Azure CLI를 사용하여 이러한 단계를 수행합니다. Azure Portal을 통해 Blob 컨테이너를 수동으로 만들어 파일을 업로드한 다음 URL을 생성하거나, 랩 컴퓨터에 설치된 Azure CLI 또는 Azure PowerShell 모듈을 사용할 수도 있습니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [**Azure Portal**](https://portal.azure.com)로 이동한 다음 이 랩에서 사용할 Azure 구독에서 Contributor 이상의 역할이 지정된 사용자 계정으로 로그인합니다. 

1.  Azure Portal의 도구 모음에서 검색 텍스트 상자 바로 오른쪽에 있는 **Cloud Shell** 아이콘을 클릭합니다. 

    > **참고**: [Azure Cloud Shell](http://shell.azure.com)로 직접 이동할 수도 있습니다.

1.  **Bash** 또는 **PowerShell**을 선택하라는 메시지가 표시되면 **PowerShell**을 선택합니다. 

    >**참고**: **Cloud Shell**을 처음 시작하고 **탑재된 스토리지가 없음** 메시지를 받으면, 이 랩에서 사용하는 구독을 선택하고 **스토리지 만들기**를 선택합니다. 

1.  Cloud Shell 창의 PowerShell 세션에서 다음 명령을 실행하여 Blob Storage 컨테이너를 만들고 이전 작업에서 만든 템플릿 파일을 업로드한 다음 SAS 토큰을 생성합니다. 기본 템플릿에서 이 토큰을 참조하여 연결된 템플에 액세스합니다.

    >**참고**: 스크립트에서 생성된 최종 출력을 적어 두세요. 이 랩 뒷부분에서 해당 출력의 내용이 필요합니다.

    ```powershell
    # Provide the name of the closest Azure region in which you can provision Azure VMs
    $location = Read-Host -Prompt 'Enter the name of Azure region (i.e. centralus)'

    # This is a random string used to assign the name to the Azure storage account
    $suffix = Get-Random
  
    $resourceGroupName = 'az400m13l01-RG'
    $storageAccountName = 'az400m13blob' + $suffix

    # The name of the Blob container to be created
    $containerName = 'linktempblobcntr' 
    
    # A completed linked template used in this lab
    $linkedTemplateURL = "https://raw.githubusercontent.com/Microsoft/PartsUnlimited/master/Labfiles/AZ-400T05_Implementing_Application_Infrastructure/M01/storage.json" 

    # A file name used for downloading and uploading the linked template
    $fileName = 'storage.json' 
    
    # Download the lab linked template into in your Azure Cloud Shell home directory
    Invoke-WebRequest -Uri $linkedTemplateURL -OutFile "$home/$fileName" 
    
    # Create a resource group
    New-AzResourceGroup -Name $resourceGroupName -Location $location 
    
    # Create a storage account
    $storageAccount = New-AzStorageAccount `
      -ResourceGroupName $resourceGroupName `
      -Name $storageAccountName `
      -Location $location `
      -SkuName 'Standard_LRS'
    
    $context = $storageAccount.Context
    
    # Create a container
    New-AzureStorageContainer -Name $containerName -Context $context
        
    # Upload the linked template
    Set-AzureStorageBlobContent `
      -Container $containerName `
      -File "$home/$fileName" `
      -Blob $fileName `
      -Context $context
    
    # Generate a SAS token. We set an expiry time of 24 hours, but you could have shorter values for increased security.
    $templateURI = New-AzureStorageBlobSASToken `
      -Context $context `
      -Container $containerName `
      -Blob $fileName `
      -Permission r `
      -ExpiryTime (Get-Date).AddHours(24.0) `
      -FullUri
    
    "Resource Group Name: $resourceGroupName"
    "Linked template URI with SAS token: $templateURI"
    ```

    >**참고**: 이 명령의 출력 값은 다음과 같습니다.

    ```powershell
    Resource Group Name: az400m13l01-RG
    Linked template URI with SAS token: https://az400m13blob1677205310.blob.core.windows.net/linktempblobcntr/storage.json?sv=2018-03-28&sr=b&sig=B4hDLt9rFaWHZXToJlMwMjejAQGT7x0INdDR9bHBQnI%3D&se=2020-11-23T21%3A54%3A53Z&sp=r
    ```

    >**참고**: 보안 수준을 높여야 하는 시나리오에서는 기본 템플릿 배포 중에 SAS 토큰을 동적으로 생성한 다음 해당 토큰의 유효 기간을 더 짧게 할당할 수 있습니다.

1.  Cloud Shell 창을 닫습니다.

#### 작업 4: 기본 템플릿을 수정하여 연결된 템플릿 호출

이 작업에서는 이전 작업에서 Azure Blob Storage에 업로드한 연결된 템플릿을 참조하도록 기본 템플릿을 수정합니다.

> **참고**: 모든 스토리지 요소를 모듈화하여 변경된 템플릿 구조를 적용하려면 기본 템플릿이 새 스토리지 리소스 정의를 호출하도록 수정해야 합니다.

1.  Visual Studio Code에서 **파일** 최상위 메뉴를 클릭하고 드롭다운 메뉴에서 **파일 열기**를 선택합니다. 그런 다음 파일 열기 대화 상자에서 **C:\\templates\\azuredeploy.json**으로 이동하여 해당 파일을 선택하고 **열기**를 클릭합니다.
1.  **azuredeploy.json** 파일의 resource 섹션에서 스토리지 리소스 요소를 제거합니다.

    ```json
    {
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[variables('storageAccountName')]",
      "location": "[parameters('location')]",
      "apiVersion": "2018-07-01",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "properties": {}
    },
    ```

1.  다음으로 새로 삭제한 스토리지 리소스 요소가 있었던 곳과 같은 위치에 다음 코드를 직접 추가합니다.

    > **참고**: `<linked_template_URI_with_SAS_token>` 자리 표시자는 이전 작업 끝부분에서 적어 두었던 실제 값으로 바꾸세요.

    ```json
    {
      "name": "linkedTemplate",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2018-05-01",
      "properties": {
          "mode": "Incremental",
          "templateLink": {
              "uri":"<linked_template_URI_with_SAS_token>"
          },
          "parameters": {
              "storageAccountName":{"value": "[variables('storageAccountName')]"},
              "location":{"value": "[parameters('location')]"}
          }
       }
    },
    ```    

1.  기본 템플릿에서 다음 세부 정보를 검토합니다.

    - 기본 템플릿의 Microsoft.Resources/deployments 리소스는 다른 템플릿에 연결하는 데 사용됩니다.
    - 배포 리소스의 이름은 linkedTemplate입니다. 이 이름은 종속성 구성에 사용됩니다.
    - 연결된 템플릿을 호출할 때는 증분 배포 모드만 사용할 수 있습니다.
    - templateLink/uri에는 연결된 템플릿 URI가 포함되어 있습니다.
    - parameters를 사용하여 기본 템플릿의 값을 연결된 템플릿에 전달합니다.

1.  템플릿을 저장합니다.


#### 작업 5: 기본 템플릿을 수정하여 종속성 업데이트

이 작업에서는 업데이트해야 하는 나머지 종속성을 반영하여 기본 템플릿을 수정합니다.

> **참고**: 스토리지 계정이 연결된 스토리지 템플릿에 정의되어 있으므로 **Microsoft.Compute/virtualMachines** 리소스 정의를 업데이트해야 합니다. 

1.  virtual machines 요소의 resource 섹션에서 **dependsOn** 요소를 업데이트합니다. 이렇게 하려면 아래 코드를

    ```json
    "dependsOn": [
      "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
      "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
    ```

    아래 코드로 바꿉니다.

    ```json
   "dependsOn": [
     "linkedTemplate",
     "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
    ```

1.  resources 섹션의 **Microsoft.Compute/virtualMachines** 요소 아래에서 연결된 스토리지 템플릿에서 정의한 출력 값을 반영하여 **properties/diagnosticsProfile/bootDiagnostics/storageUri** 요소를 다시 구성합니다. 이렇게 하려면 아래 코드를

    ```json
    "diagnosticsProfile": {
      "bootDiagnostics": {
        "enabled": true,
        "storageUri": "[reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))).primaryEndpoints.blob]"
      }
    ```

    아래 코드로 바꿉니다.

    ```json
    "diagnosticsProfile": {
      "bootDiagnostics": {
        "enabled": true,
        "storageUri": "[reference('linkedtemplate').outputs.storageUri.value]"
      }
    ```

1.  업데이트된 기본 배포 템플릿을 저장합니다.

#### 작업 6: 연결된 템플릿을 사용하여 Azure에 리소스 배포

> **참고**: 템플릿을 여러 가지 방법으로 배포할 수 있습니다. 예를 들어 Azure Portal에서 바로 배포할 수도 있고, 로컬에 설치된 Azure CLI 또는 PowerShell을 사용할 수도 있으며, Azure Cloud Shell에서 배포할 수도 있습니다. 이 랩에서는 Azure Cloud Shell에서 Azure CLI를 사용합니다.  

> **참고**: 여기서는 Azure Cloud Shell을 사용하기 위해 기본 배포 템플릿인 azuredeploy.json을 Cloud Shell 홈 디렉터리에 업로드합니다. 연결된 템플릿을 업로드했던 것처럼 기본 템플릿을 Azure Blob Storage에 업로드한 다음 로컬 파일 시스템 경로가 아닌 템플릿의 URI를 사용하여 참조할 수도 있습니다.

1.  랩 컴퓨터에서 Azure Portal이 표시된 웹 브라우저의 **Cloud Shell** 아이콘을 클릭하여 Cloud Shell을 엽니다. 
1.  Cloud Shell 창에서 **PowerShell**을 클릭하고 드롭다운 메뉴에서 **Bash**를 클릭한 다음 메시지가 표시되면 **확인**을 클릭합니다. 
1.  Cloud Shell 창에서 **파일 업로드/다운로드** 아이콘을 클릭하고 드롭다운 메뉴에서 **업로드**를 클릭합니다. 
1.  **열기** 대화 상자에서 **C:\\templates\\azuredeploy.json**으로 이동하여 해당 파일을 선택한 후에 **열기**를 클릭합니다.
1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 새로 업로드한 템플릿을 사용해 배포를 수행합니다.

    ```bash
    az deployment group create --name az400m13l01deployment --resource-group az400m13l01-RG --template-file azuredeploy.json
    ```

1.  'adminUsername'의 값을 입력하라는 메시지가 표시되면 **Student**를 입력하고 **Enter** 키를 누릅니다.
1.  'adminPassword'의 값을 입력하라는 메시지가 표시되면 **Pa55w.rd1234**를 입력하고 **Enter** 키를 누릅니다.

1.  위의 명령을 실행하여 템플릿을 배포할 때 오류가 발생하면 다음 방법을 시도해 봅니다.

    - Azure 구독이 여러 개이면 리소스 그룹을 배포할 때 구독 컨텍스트를 올바르게 설정했는지 확인합니다.
    - 지정한 URI에서 연결된 템플릿에 액세스할 수 있는지 확인합니다.

    > **참고**: 다음 단계에서는 네트워크 및 가상 머신 리소스 정의와 같은 기본 배포 템플릿의 나머지 리소스 정의를 모듈화할 수 있습니다. 

    > **참고**: 배포한 리소스를 사용하지 않으려는 경우에는 관련 요금이 발생하지 않도록 해당 리소스를 삭제해야 합니다. 리소스 그룹 **az400m13l01-RG**를 삭제하면 됩니다.

### 연습 2: Azure 랩 리소스 제거

이 연습에서는 예상치 못한 비용이 발생하지 않도록 이 랩에서 프로비전한 Azure 리소스를 제거합니다. 

>**참고**: 더 이상 사용하지 않는 새로 만든 Azure 리소스를 제거해야 합니다. 사용하지 않는 리소스를 제거하면 예기치 않은 비용이 발생하지 않습니다.

#### 작업 1: Azure 랩 리소스 제거

이 작업에서는 Azure Cloud Shell을 사용하여 불필요한 비용이 발생하지 않도록 이 랩에서 프로비전한 Azure 리소스를 제거합니다. 

1.  Azure Portal의 **Cloud Shell** 창에서 **Bash** 세션을 시작합니다.
1.  다음 명령을 실행하여 이 모듈의 전체 랩에서 생성된 모든 리소스 그룹을 나열합니다.

    ```sh
    az group list --query "[?starts_with(name,'az400m13l01-RG')].name" --output tsv
    ```

1.  다음 명령을 실행하여 이 모듈의 전체 랩에서 만든 모든 리소스 그룹을 삭제합니다.

    ```sh
    az group list --query "[?starts_with(name,'az400m13l01-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**참고**: 명령은 비동기적으로 실행되므로(--nowait 매개 변수에 의해 결정됨) 동일한 Bash 세션 내에서 즉시 다른 Azure CLI 명령을 실행할 수 있지만 리소스 그룹이 실제로 제거되기까지 몇 분 정도 걸립니다.

## 복습

이 랩에서는 Azure Resource Manager 템플릿을 만든 다음 연결된 템플릿을 사용하여 해당 템플릿을 모듈화하는 방법을 알아보았습니다. 그런 다음 기본 배포 템플릿을 수정하여 연결된 템플릿과 업데이트된 종속성을 호출하고, 마지막으로 Azure에 템플릿을 배포했습니다.
