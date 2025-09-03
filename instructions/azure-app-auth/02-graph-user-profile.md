---
lab:
  topic: Azure authentication and authorization
  title: Recuperar informações de perfil de usuário com o SDK Microsoft Graph
  description: Saiba como recuperar informações de perfil de usuário do Microsoft Graph.
---

# Recuperar informações de perfil de usuário com o SDK Microsoft Graph

Neste exercício, crie um aplicativo .NET para autenticação com o Microsoft Entra ID e solicite um token de acesso. Depois, chame a API do Microsoft Graph para recuperar e exibir suas informações de perfil de usuário. Você aprenderá a configurar permissões e interagir com Microsoft Graph no seu aplicativo.

Tarefas realizadas neste exercício:

* Registrar um aplicativo na plataforma de identidade da Microsoft
* Crie um aplicativo de console .NET que implemente a autenticação interativa e use a classe **GraphServiceClient** para recuperar informações de perfil do usuário.

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
    | **Nome** | Inserir `myGraphApplication`  |
    | **Tipos de conta compatíveis** | Selecione **Contas somente neste diretório organizacional** |
    | **Redirecionar URI (opcional)** | Selecione **Cliente público/nativo (móvel e desktop)** e insira `http://localhost` na caixa à direita. |

1. Selecione **Registrar**. O Microsoft Entra ID atribui uma ID de aplicativo (cliente) exclusiva ao seu aplicativo e você será levado para a página **Visão geral** do aplicativo. 

1. Na seção **Essentials** da página **Visão geral**, registre a **ID do Aplicativo (cliente)** e a **ID do Diretório (locatário)**. As informações são necessárias para o aplicativo.

    ![Captura de tela mostrando o local dos campos a serem copiados.](./media/01-app-directory-id-location.png)
 
## Criar um aplicativo de console .NET para enviar e receber mensagens

Agora que os recursos necessários estão implantados no Azure, a próxima etapa é configurar o aplicativo de console. As etapas a seguir são executadas em seu ambiente local.

1. Crie uma pasta chamada **graphapp**ou um nome de sua escolha para o projeto.

1. Inicie o **Visual Studio Code** e selecione **Arquivo > Abrir pasta…** e selecione a pasta do projeto.

1. Selecione **Exibir > Terminal** para abrir um terminal.

1. Execute o comando a seguir no terminal VS Code para criar o aplicativo de console .NET.

    ```
    dotnet new console
    ```

1. Execute os comandos a seguir para adicionar os pacotes **Azure.Identity**, **Microsoft.Graph** e **dotenv.net** ao projeto.

    ```
    dotnet add package Azure.Identity
    dotnet add package Microsoft.Graph
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

1. Pressione **ctrl+s** para salvar o arquivo.

### Adicionar o código inicial para o projeto

1. Abra o arquivo *Program.cs* e substitua qualquer conteúdo pelo código a seguir. Examine os comentários no código.

    ```csharp
    using Microsoft.Graph;
    using Azure.Identity;
    using dotenv.net;
    
    // Load environment variables from .env file (if present)
    DotEnv.Load();
    var envVars = DotEnv.Read();
    
    // Read Azure AD app registration values from environment
    string clientId = envVars["CLIENT_ID"];
    string tenantId = envVars["TENANT_ID"];
    
    // Validate that required environment variables are set
    if (string.IsNullOrEmpty(clientId) || string.IsNullOrEmpty(tenantId))
    {
        Console.WriteLine("Please set CLIENT_ID and TENANT_ID environment variables.");
        return;
    }
    
    // ADD CODE TO DEFINE SCOPE AND CONFIGURE AUTHENTICATION
    
    
    
    // ADD CODE TO CREATE GRAPH CLIENT AND RETRIEVE USER PROFILE
    
    
    ```

1. Pressione **ctrl+s** para salvar as alterações.

### Adicionar código para concluir o aplicativo

1. Localize o comentário **// ADICIONAR CÓDIGO PARA DEFINIR ESCOPO E CONFIGURAR AUTENTICAÇÃO** e adicione o código a seguir diretamente após o comentário. Examine os comentários no código.

    ```csharp
    // Define the Microsoft Graph permission scopes required by this app
    var scopes = new[] { "User.Read" };
    
    // Configure interactive browser authentication for the user
    var options = new InteractiveBrowserCredentialOptions
    {
        ClientId = clientId, // Azure AD app client ID
        TenantId = tenantId, // Azure AD tenant ID
        RedirectUri = new Uri("http://localhost") // Redirect URI for auth flow
    };
    var credential = new InteractiveBrowserCredential(options);
    ```

1. Localize o comentário **// ADICIONAR CÓDIGO PARA CRIAR CLIENTE DO GRAPH E RECUPERAR PERFIL DE USUÁRIO** e adicione o código a seguir diretamente após o comentário. Examine os comentários no código.

    ```csharp
    // Create a Microsoft Graph client using the credential
    var graphClient = new GraphServiceClient(credential);
    
    // Retrieve and display the user's profile information
    Console.WriteLine("Retrieving user profile...");
    await GetUserProfile(graphClient);
    
    // Function to get and print the signed-in user's profile
    async Task GetUserProfile(GraphServiceClient graphClient)
    {
        try
        {
            // Call Microsoft Graph /me endpoint to get user info
            var me = await graphClient.Me.GetAsync();
            Console.WriteLine($"Display Name: {me?.DisplayName}");
            Console.WriteLine($"Principal Name: {me?.UserPrincipalName}");
            Console.WriteLine($"User Id: {me?.Id}");
        }
        catch (Exception ex)
        {
            // Print any errors encountered during the call
            Console.WriteLine($"Error retrieving profile: {ex.Message}");
        }
    }
    ```

1. Pressione **ctrl+s** para salvar o arquivo.

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
    Retrieving user profile...
    Display Name: <Your account display name>
    Principal Name: <Your principal name>
    User Id: 9f5...
    ```

1. Inicie o aplicativo uma segunda vez e observe que você não recebe mais a notificação de **Permissões solicitadas**. A permissão que você concedeu antes foi armazenada em cache.

## Limpar os recursos

Agora que você concluiu o exercício, deve excluir o registro de aplicativo criado antes.

1. No portal do Azure, navegue até o registro de aplicativo que você criou.
1. Na barra de ferramentas, selecione **Excluir**.
1. Confirme a exclusão.
