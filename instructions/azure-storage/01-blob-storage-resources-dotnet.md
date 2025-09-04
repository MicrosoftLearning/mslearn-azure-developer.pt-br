---
lab:
  topic: Azure Storage
  title: Criar recursos de armazenamento de Blobs com a biblioteca de clientes .NET
  description: 'Saiba como usar a biblioteca de clientes .NET do Armazenamento do Azure para criar contêineres, carregar e listar blobs e excluir contêineres.'
---

# Criar recursos de armazenamento de Blobs com a biblioteca de clientes .NET

Neste exercício, você cria uma conta de Armazenamento do Azure e um aplicativo de console .NET usando a biblioteca de clientes do Armazenamento de Blobs do Azure para criar contêineres, carregar arquivos no armazenamento de blobs, listar blobs e baixar arquivos. Você aprenderá a autenticar com o Azure, executar operações de armazenamento de blobs programaticamente e verificar os resultados no portal do Azure.

Tarefas realizadas neste exercício:

* Preparar os recursos do Azure
* Criar um aplicativo de console para criar e baixar dados
* Executar o aplicativo e verificar os resultados
* Limpar os recursos

Este exercício levará aproximadamente **30** minutos para ser concluído.

## Crie uma conta de Armazenamento do Azure

Nesta seção do exercício, você criará os recursos necessários no Azure com a CLI do Azure.

1. No navegador, vá par o portal do Azure [https://portal.azure.com](https://portal.azure.com). Faça login com suas credenciais do Azure se for solicitado.

1. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um novo Cloud Shell no portal do Azure, selecionando um ambiente do ***Bash***. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure. Se você for solicitado a selecionar uma conta de armazenamento na qual manter seus arquivos, selecione **Nenhuma conta de armazenamento necessária**, sua assinatura e **Aplicar**.

    > **Observação**: se você já criou uma Cloud Shell que usa um ambiente *PowerShell*, troque-o pelo ***Bash***.

1. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para a versão clássica** (isso é necessário para usar o editor de código).

1. Crie um grupo de recursos para os recursos que este exercício requer. Substitua **myResourceGroup** por um nome que você quer usar para o grupo de recursos. Você pode substituir **eastus2** por uma região perto de você, se necessário. Se você já tiver um grupo de recursos que deseja usar, vá para a próxima etapa.

    ```
    az group create --location eastus2 --name myResourceGroup
    ```

1. Muitos comandos exigem nomes exclusivos e usam os mesmos parâmetros. Criar algumas variáveis reduzirá as alterações necessárias para os comandos que criam recursos. Execute os comandos a seguir para criar as variáveis necessárias. Substitua **myResourceGroup** pelo nome que você está usando para este exercício.

    ```
    resourceGroup=myResourceGroup
    location=eastus
    accountName=storageacct$RANDOM
    ```

1. Execute os comandos a seguir para criar a conta de Armazenamento do Azure. Cada nome de conta deve ser exclusivo. O primeiro comando cria uma variável com um nome exclusivo para sua conta de armazenamento. Registre o nome da sua conta da saída do comando de **eco**. 

    ```
    az storage account create --name $accountName \
        --resource-group $resourceGroup \
        --location $location \
        --sku Standard_LRS 
    
    echo $accountName
    ```

### Atribuir uma função ao nome de usuário do Microsoft Entra

Para permitir que seu aplicativo crie recursos e itens, atribua o usuário do Microsoft Entra à função de **Proprietário de Dados do Blob de Armazenamento**. Execute as etapas a seguir no Cloud Shell.

>**Dica:** redimensione o Cloud Shell para exibir mais informações e código arrastando a borda superior. Você também pode usar os botões minimizar e maximizar para alternar entre o Cloud Shell e a interface do portal principal.

1. Execute o comando a seguir para recuperar o **userPrincipalName** da sua conta. Isso representa a quem a função será atribuída.

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. Execute o comando a seguir para recuperar a ID de recurso da conta de armazenamento. A ID do recurso define o escopo da atribuição de função para um namespace específico.

    ```
    resourceID=$(az storage account show --name $accountName \
        --resource-group $resourceGroup \
        --query id --output tsv)
    ```
1. Execute o comando a seguir para criar e atribuir a função de **Proprietário de Dados do Blob de Armazenamento**. Essa função fornece as permissões para gerenciar contêineres e itens.

    ```
    az role assignment create --assignee $userPrincipal \
        --role "Storage Blob Data Owner" \
        --scope $resourceID
    ```

## Criar um aplicativo de console .NET para criar contêineres e itens

Agora que os recursos necessários estão implantados no Azure, a próxima etapa é configurar o aplicativo de console. As etapas a seguir são executadas no Cloud Shell.

1. Execute os comandos a seguir para criar um diretório para conter o projeto e altere-o para o diretório do projeto.

    ```
    mkdir azstor
    cd azstor
    ```

1. Crie o aplicativo de console .NET.

    ```
    dotnet new console
    ```

1. Execute os comandos a seguir para adicionar os pacotes necessários no aplicativo.

    ```
    dotnet add package Azure.Storage.Blobs
    dotnet add package Azure.Identity
    ```

1. Execute o comando a seguir para criar uma pasta de **dados** em seu projeto. 

    ```
    mkdir data
    ```

Agora é hora de adicionar o código para o projeto.

### Adicionar o código inicial para o projeto

1. Execute o seguinte comando no Cloud Shell para começar a editar o aplicativo.

    ```
    code Program.cs
    ```

1. Substitua qualquer conteúdo pelo código a seguir. Examine os comentários no código.

    ```csharp
    using Azure.Storage.Blobs;
    using Azure.Storage.Blobs.Models;
    using Azure.Identity;
    
    Console.WriteLine("Azure Blob Storage exercise\n");
    
    // Create a DefaultAzureCredentialOptions object to configure the DefaultAzureCredential
    DefaultAzureCredentialOptions options = new()
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true
    };
    
    // Run the examples asynchronously, wait for the results before proceeding
    await ProcessAsync();
    
    Console.WriteLine("\nPress enter to exit the sample application.");
    Console.ReadLine();
    
    async Task ProcessAsync()
    {
        // CREATE A BLOB STORAGE CLIENT
        
    
    
        // CREATE A CONTAINER
        
    
    
        // CREATE A LOCAL FILE FOR UPLOAD TO BLOB STORAGE
        
    
    
        // UPLOAD THE FILE TO BLOB STORAGE
        
    
    
        // LIST BLOBS IN THE CONTAINER
        
    
    
        // DOWNLOAD THE BLOB TO A LOCAL FILE
        
    
    }
    ```

1. Pressione **ctrl+s** para salvar as alterações e vá para a próxima etapa.


## Adicionar código para concluir o projeto

No restante do exercício, você adiciona código em áreas especificadas para criar o aplicativo completo. 

1. Localize o comentário **// CRIAR UM CLIENTE DO ARMAZENAMENTO DE BLOBS** e adicione o seguinte código diretamente abaixo dele. O **BlobServiceClient** atua como o ponto de entrada principal para gerenciar contêineres e blobs em uma conta de armazenamento. O cliente usa a *DefaultAzureCredential* para autenticação. Substitua **YOUR_ACCOUNT_NAME** pelo nome que você registrou antes.

    ```csharp
    // Create a credential using DefaultAzureCredential with configured options
    string accountName = "YOUR_ACCOUNT_NAME"; // Replace with your storage account name
    
    // Use the DefaultAzureCredential with the options configured at the top of the program
    DefaultAzureCredential credential = new DefaultAzureCredential(options);
    
    // Create the BlobServiceClient using the endpoint and DefaultAzureCredential
    string blobServiceEndpoint = $"https://{accountName}.blob.core.windows.net";
    BlobServiceClient blobServiceClient = new BlobServiceClient(new Uri(blobServiceEndpoint), credential);
    ```

1. Pressione **ctrl+s** para salvar as alterações e vá para a próxima etapa.

1. Localize o comentário **// CRIAR UM CONTÊINER** e adicione o código a seguir logo abaixo do comentário. Criar um contêiner inclui a criação de uma instância da classe **BlobServiceClient** e chamar o método **CreateBlobContainerAsync** para criar o contêiner em sua conta de armazenamento. Um valor GUID é anexado ao nome do contêiner para garantir que ele seja exclusivo. O método **CreateBlobContainerAsync** falhará se o contêiner já existir.

    ```csharp
    // Create a unique name for the container
    string containerName = "wtblob" + Guid.NewGuid().ToString();
    
    // Create the container and return a container client object
    Console.WriteLine("Creating container: " + containerName);
    BlobContainerClient containerClient = 
        await blobServiceClient.CreateBlobContainerAsync(containerName);
    
    // Check if the container was created successfully
    if (containerClient != null)
    {
        Console.WriteLine("Container created successfully, press 'Enter' to continue.");
        Console.ReadLine();
    }
    else
    {
        Console.WriteLine("Failed to create the container, exiting program.");
        return;
    }
    ```

1. Pressione **ctrl+s** para salvar as alterações e vá para a próxima etapa.

1. Localize o comentário **// CRIAR UM ARQUIVO LOCAL PARA UPLOAD PARA O ARMAZENAMENTO DE BLOBS** e adicione o código a seguir diretamente abaixo do comentário. Isso cria um arquivo no diretório de dados que é carregado no contêiner.

    ```csharp
    // Create a local file in the ./data/ directory for uploading and downloading
    Console.WriteLine("Creating a local file for upload to Blob storage...");
    string localPath = "./data/";
    string fileName = "wtfile" + Guid.NewGuid().ToString() + ".txt";
    string localFilePath = Path.Combine(localPath, fileName);
    
    // Write text to the file
    await File.WriteAllTextAsync(localFilePath, "Hello, World!");
    Console.WriteLine("Local file created, press 'Enter' to continue.");
    Console.ReadLine();
    ```

1. Pressione **ctrl+s** para salvar as alterações e vá para a próxima etapa.

1. Localize o comentário **// CARREGAR O ARQUIVO NO ARMAZENAMENTO DE BLOBS** e adicione o código a seguir diretamente abaixo do comentário. O código obtém uma referência a um objeto **BlobClient** chamando o método **GetBlobClient** no contêiner criado na seção anterior. Em seguida, ele carrega um arquivo local gerado usando o método **UploadAsync**. Esse método criará o blob se ele ainda não existir e o substituirá se já existir.

    ```csharp
    // Get a reference to the blob and upload the file
    BlobClient blobClient = containerClient.GetBlobClient(fileName);
    
    Console.WriteLine("Uploading to Blob storage as blob:\n\t {0}", blobClient.Uri);
    
    // Open the file and upload its data
    using (FileStream uploadFileStream = File.OpenRead(localFilePath))
    {
        await blobClient.UploadAsync(uploadFileStream);
        uploadFileStream.Close();
    }
    
    // Verify if the file was uploaded successfully
    bool blobExists = await blobClient.ExistsAsync();
    if (blobExists)
    {
        Console.WriteLine("File uploaded successfully, press 'Enter' to continue.");
        Console.ReadLine();
    }
    else
    {
        Console.WriteLine("File upload failed, exiting program..");
        return;
    }
    ```

1. Pressione **ctrl+s** para salvar as alterações e vá para a próxima etapa.

1. Localize o comentário **// LISTAR BLOBS NO CONTÊINER** e adicione o código a seguir diretamente abaixo do comentário. Liste os blobs no contêiner com o método **GetBlobsAsync**. Nesse caso, apenas um blob foi adicionado ao contêiner, pelo que a operação de listagem devolve apenas esse blob. 

    ```csharp
    Console.WriteLine("Listing blobs in container...");
    await foreach (BlobItem blobItem in containerClient.GetBlobsAsync())
    {
        Console.WriteLine("\t" + blobItem.Name);
    }
    
    Console.WriteLine("Press 'Enter' to continue.");
    Console.ReadLine();
    ```

1. Pressione **ctrl+s** para salvar as alterações e vá para a próxima etapa.

1. Localize o comentário **// BAIXAR O BLOB PARA UM ARQUIVO LOCAL** e adicione o código a seguir diretamente abaixo do comentário. O código usa o método **DownloadAsync** para baixar o blob criado antes no sistema de arquivos local. O código de exemplo adiciona um sufixo de "DOWNLOADED" ao nome do blob para que você possa ver os dois arquivos no sistema de arquivos local. 

    ```csharp
    // Adds the string "DOWNLOADED" before the .txt extension so it doesn't 
    // overwrite the original file
    
    string downloadFilePath = localFilePath.Replace(".txt", "DOWNLOADED.txt");
    
    Console.WriteLine("Downloading blob to: {0}", downloadFilePath);
    
    // Download the blob's contents and save it to a file
    BlobDownloadInfo download = await blobClient.DownloadAsync();
    
    using (FileStream downloadFileStream = File.OpenWrite(downloadFilePath))
    {
        await download.Content.CopyToAsync(downloadFileStream);
    }
    
    Console.WriteLine("Blob downloaded successfully to: {0}", downloadFilePath);
    ```

1. Pressione **ctrl+s** para salvar o arquivo e **ctrl+q** para sair do editor.

## Execute o aplicativo e entre no Azure.

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para entrar no Azure.

    ```
    az login
    ```

    **<font color="red">Você deve entrar no Azure, mesmo que a sessão do Cloud Shell já esteja autenticada.</font>**

    > **Observação**: na maioria dos cenários, apenas usar *az login* será suficiente. No entanto, se você tiver assinaturas em vários locatários, talvez seja necessário especificar o locatário usando o parâmetro *--tenant* . Confira [Entrar no Azure interativamente usando a CLI do Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obter detalhes.

1. Execute o comando a seguir para iniciar o aplicativo de console. O aplicativo será pausado muitas vezes durante a execução, aguardando que você pressione qualquer tecla para continuar. Com isso, você consegue exibir as mensagens no portal do Azure.

    ```
    dotnet run
    ```

1. No portal do Azure, navegue até a conta de Armazenamento do Azure que você criou. 

1. Expanda **> Armazenamento de dados** no painel de navegação esquerdo e selecione **Contêineres**.

1. Selecione o contêiner que o aplicativo criou e você pode exibir o blob que foi carregado.

1. Execute os dois comandos abaixo para alterar para o diretório de **dados** e listar os arquivos que foram carregados e baixados.

    ```
    cd data
    ls
    ```

## Limpar os recursos

Agora que você concluiu o exercício, exclua os recursos de nuvem que criou para evitar uso desnecessário de recursos.

1. Navegue até o grupo de recursos que você criou e exiba o conteúdo dos recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.

> **CUIDADO:** excluir o grupo de recursos excluirá todos os recursos que ele contém. Se você escolheu um grupo de recursos para este exercício, todos os recursos fora do escopo deste 

