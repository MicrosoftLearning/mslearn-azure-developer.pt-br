---
lab:
  topic: Secure solutions in Azure
  title: Recuperar definições de configuração da Configuração de Aplicativos do Azure
  description: 'Saiba como criar um recurso Configuração de Aplicativos do Azure e definir informações de configuração com a CLI do Azure. Em seguida, use o **ConfigurationBuilder** para recuperar as configurações do aplicativo.'
---

# Recuperar definições de configuração da Configuração de Aplicativos do Azure

Neste exercício, você criará um recurso de Configuração de Aplicativos do Azure, armazenará definições de configuração usando a CLI do Azure e criará um aplicativo de console .NET que usa o **ConfigurationBuilder** para recuperar valores de configuração. Você aprenderá a organizar as configurações com chaves hierárquicas e a autenticar seu aplicativo para acessar dados de configuração baseados em nuvem.

Tarefas realizadas neste exercício:

* Criar um recurso Configuração de Aplicativos do Azure aplicativo
* Armazenar informações de configuração da cadeia de conexão
* Criar um aplicativo de console .NET para recuperar as informações de configuração
* Limpar os recursos

Este exercício levará aproximadamente **15** minutos para ser concluído.

## Criar um recurso Configuração de Aplicativos do Azure e adicionar informações de configuração

Nesta seção do exercício, você criará os recursos necessários no Azure com a CLI do Azure.

1. No navegador, vá par o portal do Azure [https://portal.azure.com](https://portal.azure.com). Faça login com suas credenciais do Azure se for solicitado.

1. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um novo Cloud Shell no portal do Azure, selecionando um ambiente do ***Bash***. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure. Se você for solicitado a selecionar uma conta de armazenamento na qual manter seus arquivos, selecione **Nenhuma conta de armazenamento necessária**, sua assinatura e **Aplicar**.

    > **Observação**: se você já criou uma Cloud Shell que usa um ambiente *PowerShell*, troque-o pelo ***Bash***.

1. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para a versão clássica** (isso é necessário para usar o editor de código).

1. Crie um grupo de recursos para os recursos que este exercício requer. Se você já tiver um grupo de recursos que deseja usar, vá para a próxima etapa. Substitua **myResourceGroup** por um nome que você quer usar para o grupo de recursos. Você pode substituir **eastus** por uma região perto de você, se necessário.

    ```
    az group create --name myResourceGroup --location eastus
    ```

1. Muitos comandos exigem nomes exclusivos e usam os mesmos parâmetros. Criar algumas variáveis reduzirá as alterações necessárias para os comandos que criam recursos. Execute os comandos a seguir para criar as variáveis necessárias. Substitua **myResourceGroup** pelo nome que você está usando para este exercício. Se você alterou o local na etapa anterior, faça a mesma alteração na variável **location**.

    ```
    resourceGroup=myResourceGroup
    location=eastus
    appConfigName=appconfigname$RANDOM
    ```

1. Execute o comando a seguir para obter o nome do recurso de Configuração de Aplicativos. Registre o nome, você precisará dele mais tarde no exercício.

    ```
    echo $appConfigName
    ```

1. Execute o comando a seguir para garantir que o provedor **Microsoft.AppConfiguration** esteja registrado para sua assinatura.

    ```
    az provider register --namespace Microsoft.AppConfiguration
    ```

1. O registro pode demorar alguns minutos. Execute o comando a seguir para verificar o status do registro. Prossiga para a próxima etapa quando os resultados retornarem **Registrado**.

    ```
    az provider show --namespace Microsoft.AppConfiguration --query "registrationState"
    ```

1. Execute o comando a seguir para criar um recurso de Configuração de Aplicativos do Azure. Isso pode levar alguns minutos para ser executado.

    ```
    az appconfig create --location $location \
        --name $appConfigName \
        --resource-group $resourceGroup
        --sku Free
    ```

    >**Dica:** se houver um problema ao criar o Recurso de AppConfig devido a restrições de cota usando o valor da SKU **Gratuita**, use **Desenvolvedor** em vez disso.
    

### Atribuir uma função ao nome de usuário do Microsoft Entra

Para recuperar informações de configuração, você precisa atribuir o usuário do Microsoft Entra à função **Leitor de Dados de Configuração de Aplicativos**. 

1. Execute o comando a seguir para recuperar o **userPrincipalName** da sua conta. Isso representa a quem a função será atribuída.

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. Execute o comando a seguir para recuperar a ID do recurso do serviço de Configuração de Aplicativos. A ID do recurso define o escopo da atribuição de função.

    ```
    resourceID=$(az appconfig show --resource-group $resourceGroup \
        --name $appConfigName --query id --output tsv)
    ```

1. Execute o comando a seguir para criar e atribuir a função **Leitor de Dados de Configuração de Aplicativos**.

    ```
    az role assignment create --assignee $userPrincipal \
        --role "App Configuration Data Reader" \
        --scope $resourceID
    ```

Depois, adicione uma cadeia de conexão de espaço reservado à Configuração de Aplicativos.

### Adicionar informações de configuração com a CLI do Azure

No Configuração de Aplicativos do Azure, uma chave como **Dev:conStr** é uma chave hierárquica ou com namespace. Os dois-pontos (:) servem como um delimitador que cria uma hierarquia lógica, em que:

* **Dev** representa o namespace ou prefixo de ambiente (indicando que essa configuração é para o ambiente de desenvolvimento)
* **conStr** representa o nome da configuração

Essa estrutura hierárquica permite organizar as definições de configuração por ambiente, recurso ou componente de aplicativo, facilitando o gerenciamento e a recuperação de configurações relacionadas.

Execute o comando a seguir para armazenar a cadeia de conexão de espaço reservado. 

```
az appconfig kv set --name $appConfigName \
    --key Dev:conStr \
    --value connectionString \
    --yes
```

Este comando retorna um JSON. A última linha contém o valor em texto sem formatação. 

```json
"value": "connectionString"
```

## Criar um aplicativo de console .NET para recuperar informações de configuração

Agora que os recursos necessários estão implantados no Azure, a próxima etapa é configurar o aplicativo de console. As etapas a seguir são executadas no Cloud Shell.

>**Dica:** redimensione o Cloud Shell para exibir mais informações e código arrastando a borda superior. Você também pode usar os botões minimizar e maximizar para alternar entre o Cloud Shell e a interface do portal principal.

1. Execute os comandos a seguir para criar um diretório para conter o projeto e altere-o para o diretório do projeto.

    ```
    mkdir appconfig
    cd appconfig
    ```

1. Crie o aplicativo de console .NET.

    ```
    dotnet new console
    ```

1. Execute os comandos a seguir para adicionar os pacotes **Azure.Identity** e **Microsoft.Extensions.Configuration.AzureAppConfiguration** ao projeto.

    ```
    dotnet add package Azure.Identity
    dotnet add package Microsoft.Extensions.Configuration.AzureAppConfiguration
    ```

### Adicionar o código para o projeto

1. Execute o seguinte comando no Cloud Shell para começar a editar o aplicativo.

    ```
    code Program.cs
    ```

1. Substitua qualquer conteúdo pelo código a seguir. Substitua **YOUR_APP_CONFIGURATION_NAME** pelo nome que você registrou antes e leia os comentários no código.

    ```csharp
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.Configuration.AzureAppConfiguration;
    using Azure.Identity;
    
    // Set the Azure App Configuration endpoint, replace YOUR_APP_CONFIGURATION_NAME
    // with the name of your actual App Configuration service
    
    string endpoint = "https://YOUR_APP_CONFIGURATION_NAME.azconfig.io"; 
    
    // Configure which authentication methods to use
    // DefaultAzureCredential tries multiple auth methods automatically
    DefaultAzureCredentialOptions credentialOptions = new()
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true
    };
    
    // Create a configuration builder to combine multiple config sources
    var builder = new ConfigurationBuilder();
    
    // Add Azure App Configuration as a source
    // This connects to Azure and loads configuration values
    builder.AddAzureAppConfiguration(options =>
    {
        
        options.Connect(new Uri(endpoint), new DefaultAzureCredential(credentialOptions));
    });
    
    // Build the final configuration object
    try
    {
        var config = builder.Build();
        
        // Retrieve a configuration value by key name
        Console.WriteLine(config["Dev:conStr"]);
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error connecting to Azure App Configuration: {ex.Message}");
    }
    ```

1. Pressione **ctrl+s** para salvar o arquivo e **ctrl+q** para sair do editor.

## Execute o aplicativo e entre no Azure.

1. No Cloud Shell, insira o comando a seguir para entrar no Azure.

    ```
    az login
    ```

    **<font color="red">Você deve entrar no Azure, mesmo que a sessão do Cloud Shell já esteja autenticada.</font>**

    > **Observação**: na maioria dos cenários, apenas usar *az login* será suficiente. No entanto, se você tiver assinaturas em vários locatários, talvez seja necessário especificar o locatário usando o parâmetro *--tenant* . Confira [Entrar no Azure interativamente usando a CLI do Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obter detalhes.

1. Execute o comando a seguir para iniciar o aplicativo de console. O aplicativo exibirá o valor **connectionString** que você atribuiu à configuração **Dev:conStr** antes no exercício.

    ```
    dotnet run
    ```

    O aplicativo exibirá o valor **connectionString** que você atribuiu à configuração **Dev:conStr** antes no exercício.

## Limpar os recursos

Agora que você concluiu o exercício, exclua os recursos de nuvem que criou para evitar uso desnecessário de recursos.

1. No navegador, vá par o portal do Azure [https://portal.azure.com](https://portal.azure.com). Faça login com suas credenciais do Azure se for solicitado.
1. Navegue até o grupo de recursos que você criou e exiba o conteúdo dos recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.

> **CUIDADO:** excluir o grupo de recursos excluirá todos os recursos que ele contém. Se você escolher um grupo de recursos para este exercício, todos os recursos fora do escopo deste exercício também serão excluídos.
