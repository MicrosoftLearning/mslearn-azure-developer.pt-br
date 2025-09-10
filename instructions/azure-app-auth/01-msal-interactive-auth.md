---
lab:
  topic: Azure authentication and authorization
  title: Implementar autenticação interativa com MSAL.NET
  description: Saiba como implementar a autenticação interativa usando o SDK MSAL.NET e adquirir um token.
---

# Implementar autenticação interativa com MSAL.NET

Neste exercício, você registrará um aplicativo no Microsoft Entra ID e criará um aplicativo de console .NET que usa o MSAL.NET para executar a autenticação interativa e adquirir um token de acesso para Microsoft Graph. Você aprenderá a configurar escopos de autenticação, manipular o consentimento do usuário e ver como os tokens são armazenados em cache para execuções subsequentes. 

Tarefas realizadas neste exercício:

* Registrar um aplicativo na plataforma de identidade da Microsoft
* Crie um aplicativo de console .NET que implemente  a classe **PublicClientApplicationBuilder** para configurar a autenticação.
* Adquira um token interativamente usando a permissão **user.read** Microsoft Graph.

Este exercício levará aproximadamente **15** minutos para ser concluído.

## Antes de começar

Para realizar o exercício, você precisará do seguinte:

* Uma assinatura do Azure. Caso ainda não tenha uma avaliação gratuita, [inscreva-se em uma](https://azure.microsoft.com/).

* [Visual Studio Code](https://code.visualstudio.com/) em uma das [plataformas compatíveis](https://code.visualstudio.com/docs/supporting/requirements#_platforms).

* [.NET 8](https://dotnet.microsoft.com/en-us/download/dotnet/8.0) ou superior.

* [Kit de Desenvolvimento em C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csdevkit) para o Visual Studio Code.

## Registrar um novo aplicativo

1. No navegador, vá par o portal do Azure [https://portal.azure.com](https://portal.azure.com). Faça login com suas credenciais do Azure se for solicitado.

1. No portal, pesquise e selecione **Registros de aplicativo**. 

1. Selecione **+ Novo registro** e, quando a página **Registrar um aplicativo** for exibida, insira as informações de registro do aplicativo:

    | Campo | Valor |
    |--|--|
    | **Nome** | Inserir `myMsalApplication`  |
    | **Tipos de conta compatíveis** | Selecione **Contas somente neste diretório organizacional** |
    | **Redirecionar URI (opcional)** | Selecione **Cliente público/nativo (móvel e desktop)** e insira `http://localhost` na caixa à direita. |

1. Selecione **Registrar**. O Microsoft Entra ID atribui uma ID de aplicativo (cliente) exclusiva ao seu aplicativo e você será levado para a página **Visão geral** do aplicativo. 

1. Na seção **Essentials** da página **Visão geral**, registre a **ID do Aplicativo (cliente)** e a **ID do Diretório (locatário)**. As informações são necessárias para o aplicativo.

    ![Captura de tela mostrando o local dos campos a serem copiados.](./media/01-app-directory-id-location.png)
 
## Criar um aplicativo de console .NET para adquirir um token

Agora que os recursos necessários estão implantados no Azure, a próxima etapa é configurar o aplicativo de console. As etapas a seguir são executadas em seu ambiente local.

1. Crie uma pasta chamada **authapp** ou um nome de sua escolha para o projeto.

1. Inicie o **Visual Studio Code** e selecione **Arquivo > Abrir pasta…** e selecione a pasta do projeto.

1. Selecione **Exibir > Terminal** para abrir um terminal.

1. Execute o comando a seguir no terminal VS Code para criar o aplicativo de console .NET.

    ```
    dotnet new console
    ```

1. Execute os comandos a seguir para adicionar os pacotes **Microsoft.Identity.Client**e **dotenv.net** ao projeto.

    ```
    dotnet add package Microsoft.Identity.Client
    dotnet add package dotenv.net
    ```

### Configurar o aplicativo de console

Nesta seção, você criará e editará um arquivo **.env** para armazenar os segredos gravados antes. 

1. Selecione **Arquivo > Novo arquivo…** e crie um arquivo chamado *.env* na pasta do projeto.

1. Abra o arquivo **.env** e adicione o código a seguir. Substitua **YOUR_CLIENT_ID** e **YOUR_TENANT_ID** pelos valores que você registrou antes.

    ```
    CLIENT_ID="YOUR_CLIENT_ID"
    TENANT_ID="YOUR_TENANT_ID"
    ```

1. Pressione **ctrl+s** para salvar as alterações.

### Adicionar o código inicial para o projeto

1. Abra o arquivo *Program.cs* e substitua qualquer conteúdo pelo código a seguir. Examine os comentários no código.

    ```csharp
    using Microsoft.Identity.Client;
    using dotenv.net;
    
    // Load environment variables from .env file
    DotEnv.Load();
    var envVars = DotEnv.Read();
    
    // Retrieve Azure AD Application ID and tenant ID from environment variables
    string _clientId = envVars["CLIENT_ID"];
    string _tenantId = envVars["TENANT_ID"];
    
    // ADD CODE TO DEFINE SCOPES AND CREATE CLIENT 
    
    
    
    // ADD CODE TO ACQUIRE AN ACCESS TOKEN
    
    
    ```

1. Pressione **ctrl+s** para salvar as alterações.

### Adicionar código para concluir o aplicativo

1. Localize o comentário **// ADICIONAR CÓDIGO PARA DEFINIR ESCOPOS E CRIAR CLIENTE** e adicione o código a seguir diretamente após o comentário. Examine os comentários no código.

    ```csharp
    // Define the scopes required for authentication
    string[] _scopes = { "User.Read" };
    
    // Build the MSAL public client application with authority and redirect URI
    var app = PublicClientApplicationBuilder.Create(_clientId)
        .WithAuthority(AzureCloudInstance.AzurePublic, _tenantId)
        .WithDefaultRedirectUri()
        .Build();
    ```

1. Localize o comentário **// ADICIONAR CÓDIGO PARA ADQUIRIR UM TOKEN DE ACESSO** e adicione o código a seguir diretamente após o comentário. Examine os comentários no código.

    ```csharp
    // Attempt to acquire an access token silently or interactively
    AuthenticationResult result;
    try
    {
        // Try to acquire token silently from cache for the first available account
        var accounts = await app.GetAccountsAsync();
        result = await app.AcquireTokenSilent(_scopes, accounts.FirstOrDefault())
                    .ExecuteAsync();
    }
    catch (MsalUiRequiredException)
    {
        // If silent token acquisition fails, prompt the user interactively
        result = await app.AcquireTokenInteractive(_scopes)
                    .ExecuteAsync();
    }
    
    // Output the acquired access token to the console
    Console.WriteLine($"Access Token:\n{result.AccessToken}");
    ```

1. Pressione **ctrl+s** para salvar o arquivo e **ctrl+q** para sair do editor.

## Executar o aplicativo

Agora que o aplicativo está concluído, é hora de executar. 

1. Inicie o aplicativo executando o seguinte comando:

    ```
    dotnet run
    ```

1. O aplicativo abrirá o navegador padrão, solicitando que você selecione a conta com a qual deseja autenticar. Se houver várias contas listadas, selecione a associada ao locatário usado no aplicativo.

1. Se esta for a primeira vez que você se autentica no aplicativo registrado, receberá uma notificação de **Permissões solicitadas** pedindo que você aprove o aplicativo para entrar e ler seu perfil e manter o acesso aos dados aos quais você deu acesso. Selecione **Aceitar**.

    ![Captura de tela mostrando a notificação de permissões solicitadas](./media/01-granting-permission.png)

1. Você deverá ver os resultados semelhantes ao exemplo abaixo no console.

    ```
    Access Token:
    eyJ0eXAiOiJKV1QiLCJub25jZSI6IlZF.........
    ```

1. Inicie o aplicativo uma segunda vez e observe que você não recebe mais a notificação de **Permissões solicitadas**. A permissão que você concedeu antes foi armazenada em cache.

## Limpar os recursos

Agora que você concluiu o exercício, exclua os recursos de nuvem que criou para evitar uso desnecessário de recursos.

1. No navegador, vá par o portal do Azure [https://portal.azure.com](https://portal.azure.com). Faça login com suas credenciais do Azure se for solicitado.
1. Navegue até o grupo de recursos que você criou e exiba o conteúdo dos recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.

> **CUIDADO:** excluir o grupo de recursos excluirá todos os recursos que ele contém. Se você escolher um grupo de recursos para este exercício, todos os recursos fora do escopo deste exercício também serão excluídos.
