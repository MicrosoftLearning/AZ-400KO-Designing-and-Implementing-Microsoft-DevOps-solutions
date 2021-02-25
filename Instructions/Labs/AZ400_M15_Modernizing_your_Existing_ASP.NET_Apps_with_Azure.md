---
lab:
    title: '랩: Azure를 사용하여 기존 ASP.NET 앱 현대화'
    module: '모듈 15: Docker를 사용하여 컨테이너 관리'
---

# 랩: Azure를 사용하여 기존 ASP.NET 앱 현대화
# 학생 랩 매뉴얼

## 랩 개요

Azure로 마이그레이션하는 과정의 일환으로 웹 애플리케이션을 현대화하려는 경우 애플리케이션 전체를 다시 설계할 필요는 없습니다. 비용과 시간 등의 제약으로 인해 항상 마이크로 서비스 등의 고급 방식을 통해 애플리케이션을 다시 설계할 수 있는 것은 아니기 때문입니다. 하지만 컨테이너화와 Azure PaaS 서비스를 활용하면 앱을 현대화할 수 있습니다. 예를 들어 Azure SQL Database와 같은 관리형 서비스로 앱의 데이터 계층을 마이그레이션하는 경우 기본 코드는 변경하지 않고 연결 문자열만 업데이트하면 됩니다.

이 랩에서는 Nerd Dinner 애플리케이션을 사용하여 이러한 현대화 방식을 설명합니다. Nerd Dinner는 오픈 소스 ASP.NET MVC 프로젝트입니다. [http://www.nerddinner.com](http://www.nerddinner.com)에서 Nerd Dinnerd 사이트의 실제 실행 방식을 확인할 수 있습니다. 여기서는 애플리케이션 DB를 Azure SQL 인스턴스로 이동한 다음 Azure Container Instances에서 애플리케이션을 실행할 수 있도록 애플리케이션에 Docker 지원을 추가합니다.

## 목표

이 랩을 완료하면 다음 작업을 수행할 수 있습니다.

- Azure의 SQL Server로 LocalDB 마이그레이션
- Visual Studio의 Docker 도구를 사용하여 애플리케이션용 Docker 지원 추가
- ACR(Azure Container Registry)에 Docker 이미지 게시
- Docker 이미지를 ACR에서 ACI(Azure Container Instances)로 푸시

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
-   [Visual Studio 다운로드 페이지](https://visualstudio.microsoft.com/downloads/)에서 제공되는 Visual Studio 2019 Community Edition. Visual Studio 2019 설치에는 **ASP.NET 및 웹 개발**, **Azure 개발** 및 **NET Core 플랫폼 간 개발** 워크로드가 포함되어야 합니다. 이러한 워크로드는 랩 컴퓨터에 미리 설치되어 있습니다.
-   [Docker 설명서 사이트](https://docs.docker.com/docker-for-windows/install/#download-docker-for-windows)에서 제공되는 **Windows용 Docker**. 이 랩의 필수 구성 요소로 설치됩니다. 

#### Azure 구독 준비

-   기존 Azure 구독을 확인하거나 새 구독을 만듭니다.
-   Azure 구독의 소유자 역할, 그리고 Azure 구독과 연결된 Azure AD 테넌트의 전역 관리자 역할이 지정되어 있는 Microsoft 계정 또는 Azure AD 계정이 있는지 확인합니다. 자세한 내용은 [Azure Portal을 사용하여 Azure 역할 할당 목록 표시](https://docs.microsoft.com/ko-kr/azure/role-based-access-control/role-assignments-list-portal) 및 [Azure Active Directory에서 관리자 역할 확인 및 할당](https://docs.microsoft.com/ko-kr/azure/active-directory/roles/manage-roles-portal#view-my-roles)을 참조하세요.

### 연습 0: 랩 필수 구성 요소 구성

이 연습에서는 랩의 필수 구성 요소를 설정합니다. 구체적으로는 Nerd Dinner 애플리케이션을 복제하고 Docker Desktop을 구성합니다.

#### 작업 1: Nerd Dinner 애플리케이션 복제

이 작업에서는 Nerd Dinner 애플리케이션 복제본을 만든 다음 Visual Studio에서 엽니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [https://github.com/spboyer/nerddinner-mvc4](https://github.com/spboyer/nerddinner-mvc4)로 이동합니다. 
1.  [spboyer/nerddinner-mvc4](https://github.com/spboyer/nerddinner-mvc4) 페이지에서 **Code**를 클릭하고 드롭다운 목록에서 코드 리포지토리의 HTTPS URL 옆에 있는 클립보드 아이콘을 클릭합니다.
1.  랩 컴퓨터에서 Visual Studio를 시작한 다음 시작 페이지에서 **리포지토리 복제**를 클릭합니다.
1.  **리포지토리 복제** 페이지의 **리포지토리 위치** 텍스트 상자에 클립보드의 내용을 붙여넣고 **복제**를 클릭합니다.
1.  추가 구성 요소를 설치하라는 메시지가 표시되면 **설치**를 클릭합니다.
1.  Visual Studio 창의 위쪽 메뉴에서 **빌드**를 클릭하고 드롭다운 메뉴에서 **솔루션 다시 빌드**를 클릭합니다.
1.  Visual Studio 창의 위쪽 메뉴에서 **IIS Express**를 클릭합니다. 그러면 웹 브라우저가 열리고 웹 애플리케이션이 표시됩니다.
1.  애플리케이션이 로컬에서 실행되고 있는지 확인한 후실행 중인 애플리케이션이 표시된 웹 브라우저 창을 닫습니다.

#### 작업 2: Docker Desktop 구성 

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [Docker 설명서 사이트](https://docs.docker.com/docker-for-windows/install/#download-docker-for-windows)로 이동하여 Docker Desktop for Windows를 다운로드한 다음 기본 설정으로 설치합니다.
1.  랩 컴퓨터에서 작업 표시줄을 확장하고 **Docker** 아이콘을 마우스 오른쪽 단추로 클릭한 다음 오른쪽 클릭 메뉴에서 **Switch to Windows containers** 옵션을 선택합니다.
1.  작업을 확인하라는 메시지가 표시되면 **Switch**를 클릭합니다.

### 연습 1: Nerd Dinner 애플리케이션 현대화

이 연습에서는 Docker 기반 컨테이너화 및 Azure SQL 데이터베이스를 사용하여 Nerd Dinner 애플리케이션을 현대화합니다.

#### 작업 1: Azure SQL 데이터베이스 만들기

이 작업에서는 Azure SQL 데이터베이스를 만듭니다.

1.  랩 컴퓨터에서 웹 브라우저를 시작하고 [**Azure Portal**](https://portal.azure.com)로 이동한 다음 이 랩에서 사용하는 Azure 구독에서 Contributor 이상의 역할이 지정된 사용자 계정으로 로그인합니다.
1.  Azure Portal에서 **SQL 데이터베이스** 리소스 종류를 검색하여 선택하고 **SQL 데이터베이스** 블레이드에서 **추가**를 클릭합니다.
1.  **SQL 데이터베이스 만들기** 블레이드의 **기본** 탭에서 다음 설정을 지정합니다.

    | 설정 | 값 | 
    | --- | --- |
    | 구독 | Azure 구독의 이름 |
    | 리소스 그룹 | 새 리소스 그룹의 이름 **az400m15l01a-RG** |
    | 데이터베이스 이름 | **nerddinnerlab** | 

1.  **SQL Database 만들기** 블레이드의 **기본** 탭에서 **서버 선택** 드롭다운 목록 바로 아래의 **새로 만들기**를 클릭합니다. 
1.  **새 서버** 블레이드에서 다음 설정을 지정하고 **확인**을 클릭합니다.

    | 설정 | 값 | 
    | --- | --- |
    | 서버 이름 | 유효하고 전역적으로 고유한 서버 이름 | 
    | 서버 관리자 로그인 | **sqluser** |
    | 암호 | **Pa55w.rd1234** |
    | 위치 | Azure SQL 데이터베이스를 프로비전할 수 있으며 랩을 진행하는 위치와 가까운 Azure 지역 |

    > **참고**: 서버에 사용한 이름을 적어 두세요. 이 랩 뒷부분에서 해당 이름이 필요합니다.

1.  **SQL Database 만들기** 블레이드의 **기본** 탭으로 돌아와서 **컴퓨팅 + 스토리지** 레이블 옆의 **데이터베이스 구성**을 클릭합니다.
1.  **구성** 블레이드에서 **기본, 표준, 프리미엄을 찾고 계신가요?** 를 클릭하고 **적용**을 클릭합니다.
1.  **SQL Database 만들기** 블레이드의 **기본** 탭으로 돌아와 **다음: 네트워킹 >** 을 클릭합니다.
1.  **SQL Database 만들기** 블레이드의 **네트워킹** 탭에서 다음 설정을 지정합니다. 다른 설정은 기본값으로 유지하고 **추가 설정**을 클릭합니다.

    | 설정 | 값 | 
    | --- | --- |
    | 연결 방법 | **공용 엔드포인트** |
    | Azure 서비스 및 리소스에서 이 서버에 액세스할 수 있도록 허용 | **예** |
    | 현재 클라이언트 IP 주소 추가 | **예** |

1.  **SQL Database 만들기** 블레이드의 **추가 설정** 탭에서 다음 설정을 지정합니다. 다른 설정은 기본값으로 유지하고 **검토 + 만들기**를 클릭합니다.

    | 설정 | 값 | 
    | --- | --- |
    | 기존 데이터 사용 | **없음** |
    | Azure Defender 또는 SQL | **나중에** |

1.  **SQL 데이터베이스 만들기** 블레이드의 **검토 + 만들기**에서 **만들기**를 클릭합니다. 

    > **참고**: 배포가 완료될 때까지 기다립니다. 이 프로세스는 5분 정도 걸립니다.

#### 작업 2: LocalDB를 Azure의 SQL Server로 마이그레이션

이 작업에서는 이전 작업에서 만든 Azure SQL 데이터베이스로 애플리케이션 LocalDB를 마이그레이션합니다.

1.  Visual Studio의 위쪽 메뉴에서 **보기**를 클릭하고 드롭다운 메뉴에서 **SQL Server 개체 탐색기**를 클릭합니다.
1.  **SQL Server 개체 탐색기** 창에서 **서버 추가** 아이콘을 클릭하고 **연결** 대화 상자에서 다음 설정을 지정한 후에 **연결**을 클릭합니다. 여기서 **<server-name>** 은 이전 작업에서 만든 논리 서버의 이름으로 바꿉니다.

    | 설정 | 값 | 
    | --- | --- |
    | 서버 이름 | **<server-name>.database.windows.net** |
    | 인증 | **SQL Server 인증** |
    | 사용자 이름 | **sqluser** |
    | 암호 | **Pa55w.rd1234** |
    | 데이터베이스 이름 | **nerddinnerlab** | 

    > **참고**: 먼저 새로 프로비전한 Azure SQL 데이터베이스에 LocalDB 인스턴스의 스키마를 복사합니다.

1.  **SQL Server 개체 탐색기** 창에서 localDB 인스턴스를 나타내는 **localdb** 노드를 확장하고 이 노드의 **Databases** 폴더를 확장합니다.  
1.  데이터베이스 목록에서 Nerd Dinner 애플리케이션 리포지토리를 기반으로 만든 프로젝트에 포함된 데이터베이스를 마우스 오른쪽 단추로 클릭하고 오른쪽 클릭 메뉴에서 **스키마 비교**를 클릭합니다. 그러면 Visual Studio 창의 기본 창에 **SqlSchemaCompare1** 탭이 열립니다.
1.  **SqlSchemaCompare1** 탭 위쪽의 **대상 선택** 드롭다운 목록에서 **대상 선택**을 클릭합니다. 
1.  **대상 스키마 선택** 대화 상자에서 **데이터베이스** 옵션이 선택되어 있는지 확인하고 **연결 선택**을 클릭합니다. 그런 다음 **연결** 대화 상자에서 **nerddinnerlab** Azure SQL 데이터베이스를 선택하고 **연결**을 클릭한 후에 **대상 스키마 선택** 대화 상자로 돌아와서 **확인**을 클릭합니다.
1.  Visual Studio 창의 기본 창에 표시된 **SqlSchemaCompare1** 탭에서 **비교**를 클릭합니다. 비교가 완료될 때까지 기다립니다.
1.  비교가 완료되면 **SqlSchemaCompare1** 탭에서 **업데이트**를 클릭하고 업데이트를 확인하라는 메시지가 표시되면 **예**를 클릭합니다.
1.  **SqlSchemaCompare1** 탭을 닫습니다.

    > **참고**: 이제 새로 프로비전한 Azure SQL 데이터베이스에 LocalDB 인스턴스의 데이터를 복사합니다.

1.  **SQL Server 개체 탐색기** 창에서 LocalDB 인스턴스를 마우스 오른쪽 단추로 클릭하고 마우스 오른쪽 클릭 메뉴에서 **데이터 비교**를 클릭합니다. 그러면 **새 데이터 비교** 마법사가 시작되고 **SqlDataCompare1** 탭이 열립니다.
1.  **새 데이터 비교** 마법사의 **원본 및 대상 데이터베이스 선택** 페이지 **대상 데이터베이스** 섹션에서 **연결 선택**을 클릭하고 **연결** 대화 상자에서 **nerddinnerlab** Azurre SQL 데이터베이스를 선택합니다. 그런 다음 **연결**을 클릭하고 **원본 및 대상 데이터베이스 선택** 페이지로 돌아와 **다음**을 클릭합니다.
1.  **비교할 테이블, 필드 및 뷰 선택**에서 **테이블** 및 **뷰** 항목 옆의 체크박스를 선택하고 **마침**을 클릭합니다.
1.  **SqlDataCompare1** 탭의 위쪽에서 **대상 업데이트**를 클릭하고 업데이트를 확인하라는 메시지가 표시되면 **예**를 클릭합니다.
1.  **SqlDataCompare1** 탭을 닫습니다.

    > **참고**: 여기서는 코드가 변경되지 않도록 **web.config** 변환을 사용합니다. 구체적으로는 새 **web.release.config** 항목을 추가합니다. 

1.  Visual Studio의 **솔루션 탐색기** 창에서 **Web.config** 항목을 확장하고 **Web.Release.config**를 선택합니다. 그러면 Visual Studio 창의 가운데 창에 **Web.Release.config** 파일이 열립니다.
1.  **Web.Release.config** 창에서 `<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">` 줄 바로 아래에 다음 항목을 추가하고 변경 내용을 저장합니다. 여기서 `<server_name>` 자리 표시자는 이 연습의 첫 번째 작업에서 만든 서버의 이름으로 바꿉니다.

    ```csharp
    <connectionStrings>
      <add name="DefaultConnection" connectionString="Data Source=<server_name>.database.windows.net;Initial Catalog=nerddinnerlab;Integrated Security=False;User ID=sqluser;Password=Pa55w.rd1234;Connect Timeout=30;Encrypt=True;TrustServerCertificate=False;ApplicationIntent=ReadWrite;MultiSubnetFailover=False" providerName="System.Data.SqlClient" xdt:Transform="SetAttributes" xdt:Locator="Match(name)"/>
    </connectionStrings>
    ```

    > **참고**: 이제 애플리케이션 LocalDB를 Azure SQL 데이터베이스로 마이그레이션했으며 데이터베이스 위치 변경 내용을 반영하여 연결 문자열을 업데이트했습니다.

#### 작업 3: Visual Studio를 사용하여 Docker 지원 추가 및 Docker 컨테이너 내에서 로컬로 애플리케이션 디버그

이 작업에서는 Visual Studio에 Docker 지원을 추가한 다음 Docker 컨테이너 내에서 로컬로 애플리케이션을 디버그하는 데 사용합니다.

1.  Visual Studio를 사용하여 Docker를 통해 애플리케이션을 컨테이너화하려면 Visual Studio의 **솔루션 탐색기** 창에서 **NerdDinner** 프로젝트를 마우스 오른쪽 단추로 클릭하고 오른쪽 클릭 메뉴에서 **추가**를 선택한 다음 계단식 메뉴에서 **컨테이너 오케스트레이터 지원**을 클릭합니다.
1.  **컨테이너 오케스트레이션 지원 추가** 대화 상자의 **컨테이너 오케스트레이터** 드롭다운 목록에서 **Docker Compose**를 선택하고 **확인**을 클릭합니다. 그러면 Visual Studio 창의 기본 창에 **Dockerfile** 탭이 자동으로 열립니다.
1.  Docker Desktop을 시작할지를 묻는 메시지가 표시되면 **예**를 클릭합니다. 

    > **참고**: Visual Studio에서 **docker-compose** 프로젝트와 **Dockerfile**을 비롯한 필수 파일을 솔루션에 자동으로 추가합니다. 또한 프로젝트를 검사하여 프로젝트에 사용할 적절한 기본 이미지를 확인합니다. Nerd Dinner 솔루션의 경우에는 **microsoft/aspnet:4.8-windowsservercore-ltsc2019** 기본 이미지가 선택되었습니다.

    ```csharp
    FROM mcr.microsoft.com/dotnet/framework/aspnet:4.8-windowsservercore-ltsc2019
    ARG source
    WORKDIR /inetpub/wwwroot
    COPY ${source:-obj/Docker/publish} .
    ```
    > **참고**: Visual Studio를 사용하여 애플리케이션을 로컬로 실행하고 Docker 컨테이너 내에서 디버그한 후 Azure SQL 데이터베이스로의 연결을 테스트하려면 **docker-compose**를 시작 프로젝트로 설정한 후에 시작합니다.

1.  Visual Studio의 **솔루션 탐색기** 창에서 **docker-compose** 프로젝트를 마우스 오른쪽 단추로 클릭한 다음 오른쪽 클릭 메뉴에서 **시작 프로젝트로 설정**을 선택합니다.
1.  Visual Studio 최상위 메뉴에서 **Docker Compose**를 클릭합니다.

    > **참고**: Visual Studio에서 기본 이미지를 다운로드한 다음 배포용 이미지를 빌드합니다. 빌드가 완료되면 애플리케이션이 로컬 브라우저에서 시작됩니다.

    > **참고**: 사용 가능한 대역폭에 따라 다운로드에 시간이 매우 오래 걸릴 수도 있습니다.

1.  랩 컴퓨터에서 **명령 프롬프트**를 관리자 권한으로 시작하고 **관리자: C:\\windows\\system32\cmd.exe** 에서 다음 명령을 실행하여 로컬 docker 이미지 목록을 표시한 다음 **Dockerfile**에 참조되어 있는 이미지가 목록에 포함되어 있는지 확인합니다.

    ''cmd'
    docker images
    ```

#### 작업 4: Azure Container Registry에 Docker 이미지 게시

이 작업에서는 Visual Studio를 사용하여 이전 단계에서 빌드한 Docker 이미지를 Azure Container Registry에 게시합니다.

1.  Visual Studio의 **솔루션 탐색기** 창에서 **NerdDinner** 프로젝트를 마우스 오른쪽 단추로 클릭한 다음 오른쪽 클릭 메뉴에서 **게시**를 클릭합니다. 그러면 **게시** 마법사가 시작됩니다.
1.  **게시** 마법사의 **지금 게시하려는 위치는 어디입니까?** 페이지에서 **Azure** 옵션이 선택되어 있는지 확인하고 **다음**을 클릭합니다.
1.  **게시** 마법사의 **애플리케이션을 호스트하는 데 사용할 Azure 서비스는 무엇입니까?** 페이지에서 **Azure Container Registry** 옵션을 선택하고 **다음**을 클릭합니다.
1.  **게시** 마법사의 **기존 Azure Container Registry를 선택하거나 새 Azure Container Registry 만들기** 페이지에서 필요한 경우 **로그인**을 클릭하고 메시지가 표시되면 이 랩에서 사용 중인 Azure 구독에서 Contributor 이상의 역할이 지정된 사용자 계정으로 로그인합니다.
1.  **게시** 마법사의 **기존 Azure Container Registry를 선택하거나 새 Azure Container Registry 만들기** 페이지에서 더하기 기호를 클릭합니다.
1.  **새로 만들기** 페이지에서 다음 설정을 지정하고 **만들기**를 클릭합니다.

    | 설정 | 값 | 
    | --- | --- |
    | DNS 접두사 | 기본값 수락 |
    | 구독 | Azure 구독의 이름 |
    | 리소스 그룹 | **az400m15l01a-RG** |
    | SKU | **표준** |
    | 레지스트리 위치 | Azure SQL 데이터베이스 배포용으로 선택한 것과 같은 Azure 지역 |

1.  **게시** 마법사의 **기존 Azure Container Registry를 선택하거나 새 Azure Container Registry 만들기** 페이지로 돌아와 **마침**을 클릭합니다.
1.  Visual Studio 인터페이스의 **NerdDinner** 탭에서 **게시**를 클릭합니다.

    > **참고**: 게시 작업이 완료될 때까지 기다립니다. 게시 작업이 완료되면 Docker 이미지가 작성되어 Azure Container Registry에 게시됩니다.

1.  게시가 정상적으로 완료되면 Azure Portal이 표시된 웹 브라우저 창으로 전환하여 Azure Portal에서 **컨테이너 레지스트리**를 검색하여 선택합니다. 그런 다음 **Azure Container Registry**에서 새로 만든 Azure Container Registry를 나타내는 항목을 클릭하고, 해당 블레이드 왼쪽의 세로 메뉴에 있는 **서비스** 섹션에서 **리포지토리**를 클릭하여 **nerddinner** 항목이 포함되어 있는지 확인합니다.

#### 작업 5: 새 Docker 이미지를 ACR에서 ACI(Azure Container Instances)로 푸시

이 작업에서는 ACI(Azure Container instance)를 만든 다음 ACR에서 새로 업데이트된 Docker 이미지에 푸시합니다.

> **참고**: AKS(Azure Kubernetes Service) 또는 Service Fabric에 컨테이너를 배포할 수도 있습니다.

> **참고**: 여기서는 Azure CLI를 사용하여 Azure Container Instance를 만듭니다.

1.  Azure Portal의 도구 모음에서 검색 텍스트 상자 바로 오른쪽에 있는 **Cloud Shell** 아이콘을 클릭합니다. 
1.  **Bash** 또는 **PowerShell**을 선택하라는 메시지가 표시되면 **Bash**를 선택합니다. 

    >**참고**: **Cloud Shell**을 처음 시작하고 **탑재된 스토리지가 없음** 메시지를 받으면, 이 랩에서 사용하는 구독을 선택하고 **스토리지 만들기**를 선택합니다. 

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 ACI용으로 새 리소스 그룹을 만듭니다.

    ```bash
    RESOURCEGROUPNAME1='az400m15l01a-RG'
    LOCATION=$(az group show --name $RESOURCEGROUPNAME1 --query location --output tsv)
    RESOURCEGROUPNAME2='az400m15l02a-RG'
    az group create --name $RESOURCEGROUPNAME2 --location $LOCATION
    ```

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 ACI를 만든 다음 이전 작업에서 만든 ACR에서 해당 ACI에 NerdDinner 이미지를 푸시합니다.

    ```bash
    ACINAME='nerddinner'$RANDOM$RANDOM
    ACRNAME=$(az acr list --resource-group $RESOURCEGROUPNAME1 --query '[].name' --output tsv)
    ```

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 이전 작업에서 만든 ACR에 대한 **끌어오기** 권한이 있는 서비스 주체를 만들고 이 서비스 주체의 암호를 셸 변수에 저장합니다.

    ```bash
    SPPASSWORD=$(az ad sp create-for-rbac \
    --name http://$ACRNAME-pull \
    --scopes $(az acr show --name $ACRNAME --query id --output tsv) \
    --role acrpull \
    --query password \
    --output tsv)
    ```

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 이전 단계에서 만든 서비스 주체 이름을 검색한 다음 해당 서비스 주체의 암호를 셸 변수에 저장합니다.

    ```bash  
    SPUSERNAME=$(az ad sp show --id http://$ACRNAME-pull --query appId --output tsv)
    ```

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 이미지가 포함된 Azure Container Registry의 이름을 검색한 다음 셸 변수에 해당 이름을 저장합니다.

    ```bash  
    ACRLOGINSERVER=$(az acr show --name $ACRNAME --resource-group $RESOURCEGROUPNAME1 --query "loginServer" --output tsv)
    ```

1.  Cloud Shell 창의 Bash 세션에서 다음 명령을 실행하여 Azure Container Registry에 저장된 이미지를 기반으로 Azure Container instance를 만듭니다. 이 작업에서 만든 끌어오기 권한이 있는 서비스 주체를 사용하여 해당 인스턴스에 액세스합니다.

    ```bash
    az container create \
    --name $ACINAME \
    --resource-group $RESOURCEGROUPNAME2 \
    --image $ACRLOGINSERVER/nerddinner:latest \
    --registry-login-server $ACRLOGINSERVER \
    --registry-username $SPUSERNAME \
    --registry-password $SPPASSWORD \
    --dns-name-label $ACINAME \
    --os-type windows \
    --query ipAddress.fqdn
    ```

    > **참고**: 배포가 완료될 때까지 기다립니다. 완료되려면 약 5분이 소요됩니다. 출력에는 Azure Container instance에 할당된 정규화된 DNS 도메인 이름이 포함됩니다.

1.  랩 컴퓨터에서 다른 웹 브라우저 탭을 열고 Azure Container instance를 프로비전한 명령의 출력에서 확인한 IP 주소로 이동한 다음 NerdDinner 애플리케이션이 실행되고 있음을 확인합니다.

## 연습 2: Azure 랩 리소스 제거

이 연습에서는 예상치 못한 비용이 발생하지 않도록 이 랩에서 프로비전한 Azure 리소스를 제거합니다. 

>**참고**: 더 이상 사용하지 않는 새로 만든 Azure 리소스를 제거해야 합니다. 사용하지 않는 리소스를 제거하면 예기치 않은 비용이 발생하지 않습니다.

#### 작업 1: Azure 랩 리소스 제거

이 작업에서는 Azure Cloud Shell을 사용하여 불필요한 비용이 발생하지 않도록 이 랩에서 프로비전한 Azure 리소스를 제거합니다. 

1.  Azure Portal의 **Cloud Shell** 창에서 **Bash** 세션을 시작합니다.
1.  다음 명령을 실행하여 이 모듈의 전체 랩에서 생성된 모든 리소스 그룹을 나열합니다.

    ```sh
    az group list --query "[?starts_with(name,'az400m15l0')].name" --output tsv
    ```

1.  다음 명령을 실행하여 이 모듈의 전체 랩에서 만든 모든 리소스 그룹을 삭제합니다.

    ```sh
    az group list --query "[?starts_with(name,'az400m15l0')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**참고**: 명령은 비동기적으로 실행되므로(--nowait 매개 변수에 의해 결정됨) 동일한 Bash 세션 내에서 즉시 다른 Azure CLI 명령을 실행할 수 있지만 리소스 그룹이 실제로 제거되기까지 몇 분 정도 걸립니다.

## 복습

이 랩에서는 Azure 클라우드 및 Windows 컨테이너를 사용하여 코드와 구성을 최소한으로만 변경해 기존 .NET 애플리케이션을 현대화하는 방법을 알아보았습니다.
