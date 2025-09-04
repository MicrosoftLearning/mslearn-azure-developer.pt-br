---
lab:
  topic: Azure events and messaging
  title: Enviar e receber mensagens do Armazenamento de Filas do Azure
  description: Saiba como enviar e enviar mensagens do Armazenamento de Filas do Azure com o SDK do .NET Azure.StorageQueues.
---

# Enviar e receber mensagens do Armazenamento de Filas do Azure

Neste exercício, você criará e configurará Armazenamento de Filas do Azure e criará um aplicativo .NET para enviar e receber mensagens usando o SDK do **Azure.Storage.Queues**. Você aprenderá a provisionar recursos de armazenamento, gerenciar mensagens da fila e limpar seu ambiente quando terminar. 

Tarefas realizadas neste exercício:

* Criar recursos de Armazenamento de Filas do Azure
* Atribuir uma função ao nome de usuário do Microsoft Entra
* Criar um aplicativo de console .NET para enviar e receber mensagens
* Limpar os recursos

Este exercício levará aproximadamente **30** minutos para ser concluído.

## Criar recursos de Armazenamento de Filas do Azure

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
    storAcctName=storactname$RANDOM
    ```

1. Você precisará do nome atribuído à conta de armazenamento mais adiante neste exercício. Execute o comando e a saída do registro a seguir.

    ```
    echo $storAcctName
    ```

1. Execute o comando a seguir para criar uma conta de armazenamento usando a variável criada antes. A operação leva alguns minutos para ser concluída.

    ```bash
    az storage account create --resource-group $resourceGroup \
        --name $storAcctName --location $location --sku Standard_LRS
    ```

### Atribuir uma função ao nome de usuário do Microsoft Entra

Para permitir que seu aplicativo envie e receba mensagens, atribua o usuário Microsoft Entra à função **Colaborador de Dados da Fila de Armazenamento**. Isso dá à sua conta de usuário permissão para criar filas e enviar/receber mensagens usando o RBAC do Azure. Execute as etapas a seguir no Cloud Shell.

1. Execute o comando a seguir para recuperar o **userPrincipalName** da sua conta. Isso representa a quem a função será atribuída.

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. Execute o comando a seguir para recuperar a ID de recurso da conta de armazenamento. A ID do recurso define o escopo da atribuição de função para um namespace específico.

    ```
    resourceID=$(az storage account show --resource-group $resourceGroup \
        --name $storAcctName --query id --output tsv)
    ```

1. Execute o comando a seguir para criar e atribuir a função **Colaborador de Dados da Fila de Armazenamento**.

    ```
    az role assignment create --assignee $userPrincipal \
        --role "Storage Queue Data Contributor" \
        --scope $resourceID
    ```

## Criar um aplicativo de console .NET para enviar e receber mensagens

Agora que os recursos necessários estão implantados no Azure, a próxima etapa é configurar o aplicativo de console. As etapas a seguir são executadas no Cloud Shell.

>**Dica:** redimensione o Cloud Shell para exibir mais informações e código arrastando a borda superior. Você também pode usar os botões minimizar e maximizar para alternar entre o Cloud Shell e a interface do portal principal.

1. Execute os comandos a seguir para criar um diretório para conter o projeto e altere-o para o diretório do projeto.

    ```
    mkdir queuestor
    cd queuestor
    ```

1. Crie o aplicativo de console .NET.

    ```
    dotnet new console
    ```

1. Execute os comandos a seguir para adicionar os pacotes **Azure.Storage.Queues** e **Azure.Identity** ao projeto.

    ```
    dotnet add package Azure.Storage.Queues
    dotnet add package Azure.Identity
    ```

### Adicionar o código inicial para o projeto

1. Execute o seguinte comando no Cloud Shell para começar a editar o aplicativo.

    ```
    code Program.cs
    ```

1. Substitua qualquer conteúdo pelo código a seguir. Examine os comentários no código e substitua **<YOUR-STORAGE-ACCT-NAME>** pelo nome da conta de armazenamento que você registrou antes.

    ```csharp
    using Azure;
    using Azure.Identity;
    using Azure.Storage.Queues;
    using Azure.Storage.Queues.Models;
    using System;
    using System.Threading.Tasks;
    
    // Create a unique name for the queue
    // TODO: Replace the <YOUR-STORAGE-ACCT-NAME> placeholder 
    string queueName = "myqueue-" + Guid.NewGuid().ToString();
    string storageAccountName = "<YOUR-STORAGE-ACCT-NAME>";
    
    // ADD CODE TO CREATE A QUEUE CLIENT AND CREATE A QUEUE
    
    
    
    // ADD CODE TO SEND AND LIST MESSAGES
    
    
    
    // ADD CODE TO UPDATE A MESSAGE AND LIST MESSAGES
    
    
    
    // ADD CODE TO DELETE MESSAGES AND THE QUEUE
    
    
    ```

1. Pressione **ctrl+s** para salvar as alterações.

### Adicionar código para criar um cliente de fila e criar uma fila

Agora é hora de adicionar código para criar o cliente de armazenamento de filas e criar uma fila.

1. Localize o comentário **// ADICIONAR CÓDIGO PARA CRIAR UM CLIENTE DE FILA E CRIAR UMA FILA** e adicione o código a seguir diretamente após o comentário. Examine o código e os comentários.

    ```csharp
    // Create a DefaultAzureCredentialOptions object to exclude certain credentials
    DefaultAzureCredentialOptions options = new()
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true
    };
    
    // Instantiate a QueueClient to create and interact with the queue
    QueueClient queueClient = new QueueClient(
        new Uri($"https://{storageAccountName}.queue.core.windows.net/{queueName}"),
        new DefaultAzureCredential(options));
    
    Console.WriteLine($"Creating queue: {queueName}");
    
    // Create the queue
    await queueClient.CreateAsync();
    
    Console.WriteLine("Queue created, press Enter to add messages to the queue...");
    Console.ReadLine();
    ```

1. Pressione **ctrl+s** para salvar o arquivo e continue com o exercício.

### Adicionar código para enviar e listar mensagens em uma fila

1. Localize o comentário **// ADICIONAR CÓDIGO PARA ENVIAR E LISTAR MENSAGENS** e adicione o código a seguir diretamente após o comentário. Examine o código e os comentários.

    ```csharp
    // Send several messages to the queue with the SendMessageAsync method.
    await queueClient.SendMessageAsync("Message 1");
    await queueClient.SendMessageAsync("Message 2");
    
    // Send a message and save the receipt for later use
    SendReceipt receipt = await queueClient.SendMessageAsync("Message 3");
    
    Console.WriteLine("Messages added to the queue. Press Enter to peek at the messages...");
    Console.ReadLine();
    
    // Peeking messages lets you view the messages without removing them from the queue.
    
    foreach (var message in (await queueClient.PeekMessagesAsync(maxMessages: 10)).Value)
    {
        Console.WriteLine($"Message: {message.MessageText}");
    }
    
    Console.WriteLine("\nPress Enter to update a message in the queue...");
    Console.ReadLine();
    ```

1. Pressione **ctrl+s** para salvar o arquivo e continue com o exercício.

### Adicionar código para atualizar uma mensagem e listar os resultados

1. Localize o comentário **// ADICIONAR CÓDIGO PARA ATUALIZAR UMA MENSAGEM E LISTAR MENSAGENS** e adicione o código a seguir diretamente após o comentário. Examine o código e os comentários.

    ```csharp
    // Update a message with the UpdateMessageAsync method and the saved receipt
    await queueClient.UpdateMessageAsync(receipt.MessageId, receipt.PopReceipt, "Message 3 has been updated");
    
    Console.WriteLine("Message three updated. Press Enter to peek at the messages again...");
    Console.ReadLine();
    
    
    // Peek messages from the queue to compare updated content
    foreach (var message in (await queueClient.PeekMessagesAsync(maxMessages: 10)).Value)
    {
        Console.WriteLine($"Message: {message.MessageText}");
    }
    
    Console.WriteLine("\nPress Enter to delete messages from the queue...");
    Console.ReadLine();
    ```

1. Pressione **ctrl+s** para salvar o arquivo e continue com o exercício.

### Adicionar código para excluir mensagens e a fila

1. Localize o comentário **// ADICIONAR CÓDIGO PARA EXCLUIR MENSAGENS E FILA** e adicione o código a seguir diretamente após o comentário. Examine o código e os comentários.

    ```csharp
    // Delete messages from the queue with the DeleteMessagesAsync method.
    foreach (var message in (await queueClient.ReceiveMessagesAsync(maxMessages: 10)).Value)
    {
        // "Process" the message
        Console.WriteLine($"Deleting message: {message.MessageText}");
    
        // Let the service know we're finished with the message and it can be safely deleted.
        await queueClient.DeleteMessageAsync(message.MessageId, message.PopReceipt);
    }
    Console.WriteLine("Messages deleted from the queue.");
    Console.WriteLine("\nPress Enter key to delete the queue...");
    Console.ReadLine();
    
    // Delete the queue with the DeleteAsync method.
    Console.WriteLine($"Deleting queue: {queueClient.Name}");
    await queueClient.DeleteAsync();
    
    Console.WriteLine("Done");
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

1. Expanda **> Armazenamento de dados** no painel de navegação à esquerda e selecione **Filas**.

1. Selecione a fila que o aplicativo cria, e você pode exibir as mensagens enviadas e monitorar o que o aplicativo está fazendo.

## Limpar os recursos

Agora que você concluiu o exercício, exclua os recursos de nuvem que criou para evitar uso desnecessário de recursos.

1. Navegue até o grupo de recursos que você criou e exiba o conteúdo dos recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.

> **CUIDADO:** excluir o grupo de recursos excluirá todos os recursos que ele contém. Se você escolheu um grupo de recursos para este exercício, todos os recursos fora do escopo deste 

