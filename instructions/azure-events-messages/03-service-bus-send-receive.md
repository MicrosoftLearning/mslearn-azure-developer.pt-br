---
lab:
  topic: Azure events and messaging
  title: Enviar e receber mensagens do Barramento de Serviços do Azure
  description: Saiba como enviar e enviar mensagens Barramento de Serviço do Azure com o SDK do .NET Azure.Messaging.ServiceBus.
---

# Enviar e receber mensagens do Barramento de Serviços do Azure

Neste exercício, você criará e configurará Barramento de Serviço do Azure e criará um aplicativo .NET para enviar e receber mensagens usando o SDK do **Azure.Messaging.ServiceBus**. Você aprenderá a provisionar um namespace e uma fila do Barramento de Serviço, atribuir permissões e interagir com mensagens programaticamente. 

Tarefas realizadas neste exercício:

* Criar recursos do Barramento de Serviço do Azure
* Atribuir uma função ao nome de usuário do Microsoft Entra
* Criar um aplicativo de console .NET para enviar e receber mensagens
* Limpar os recursos

Este exercício levará aproximadamente **30** minutos para ser concluído.

## Criar recursos de Hubs de Eventos do Azure

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
    namespaceName=svcbusns$RANDOM
    ```

1. Você precisará do nome atribuído ao namespace mais adiante neste exercício. Execute o comando e a saída do registro a seguir.

    ```
    echo $namespaceName
    ```

### Criar um namespace e uma fila do Barramento de Serviço do Azure

1. Crie um namespace de mensagens do Barramento de Serviço. O comando a seguir cria um namespace usando a variável criada anteriormente. A operação leva alguns minutos para ser concluída.

    ```bash
    az servicebus namespace create \
        --resource-group $resourceGroup \
        --name $namespaceName \
        --location $location
    ```

1. Agora que um namespace foi criado, você precisa criar uma fila para armazenar as mensagens. Execute o comando a seguir para criar uma fila chamada **myqueue**.

    ```bash
    az servicebus queue create --resource-group $resourceGroup \
        --namespace-name $namespaceName \
        --name myqueue
    ```

### Atribuir uma função ao nome de usuário do Microsoft Entra

Para permitir que seu aplicativo envie e receba mensagens, atribua o usuário Microsoft Entra à função **Proprietário de Dados do Barramento de Serviço do Azure** no nível do namespace do Barramento de Serviço. Isso dá à sua conta de usuário permissão para gerenciar e acessar filas e tópicos usando o RBAC do Azure. Execute as etapas a seguir no Cloud Shell.

1. Execute o comando a seguir para recuperar o **userPrincipalName** da sua conta. Isso representa a quem a função será atribuída.

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. Execute o comando a seguir para recuperar a ID de recurso do namespace do Barramento de Serviço. A ID do recurso define o escopo da atribuição de função para um namespace específico.

    ```
    resourceID=$(az servicebus namespace show --name $namespaceName \
        --resource-group $resourceGroup \
        --query id --output tsv)
    ```
1. Execute o comando a seguir para criar e atribuir a função de **Proprietário de Dados do Barramento de Serviço do Azure**.

    ```
    az role assignment create --assignee $userPrincipal \
        --role "Azure Service Bus Data Owner" \
        --scope $resourceID
    ```

## Criar um aplicativo de console .NET para enviar e receber mensagens

Agora que os recursos necessários estão implantados no Azure, a próxima etapa é configurar o aplicativo de console. As etapas a seguir são executadas no Cloud Shell.

>**Dica:** redimensione o Cloud Shell para exibir mais informações e código arrastando a borda superior. Você também pode usar os botões minimizar e maximizar para alternar entre o Cloud Shell e a interface do portal principal.

1. Execute os comandos a seguir para criar um diretório para conter o projeto e altere-o para o diretório do projeto.

    ```
    mkdir svcbus
    cd svcbus
    ```

1. Crie o aplicativo de console .NET.

    ```
    dotnet new console
    ```

1. Execute os comandos a seguir para adicionar os pacotes **Azure.Messaging.ServiceBus** e **Azure.Identity** ao projeto.

    ```
    dotnet add package Azure.Messaging.ServiceBus
    dotnet add package Azure.Identity
    ```

### Adicionar o código inicial para o projeto

1. Execute o seguinte comando no Cloud Shell para começar a editar o aplicativo.

    ```
    code Program.cs
    ```

1. Substitua qualquer conteúdo pelo código a seguir. Examine os comentários no código e substitua **<YOUR-NAMESPACE>** pelo namespace do Barramento de Serviço que você registrou antes.

    ```csharp
    using Azure.Messaging.ServiceBus;
    using Azure.Identity;
    using System.Timers;
    
    
    // TODO: Replace <YOUR-NAMESPACE> with your Service Bus namespace
    string svcbusNameSpace = "<YOUR-NAMESPACE>.servicebus.windows.net";
    string queueName = "myQueue";
    
    
    // ADD CODE TO CREATE A SERVICE BUS CLIENT
    
    
    
    // ADD CODE TO SEND MESSAGES TO THE QUEUE
    
    
    
    // ADD CODE TO PROCESS MESSAGES FROM THE QUEUE
    
    
    
    // Dispose client after use
    await client.DisposeAsync();
    ```

1. Pressione **ctrl+s** para salvar as alterações.

### Adicionar código para enviar mensagens à fila

Agora é hora de adicionar código para criar o cliente do Barramento de Serviço e enviar um lote de mensagens para a fila.

1. Localize o comentário **// ADICIONAR CÓDIGO PARA CRIAR UM CLIENTE DO BARRAMENTO DE SERVIÇO** e adicione o código a seguir diretamente após o comentário. Examine o código e os comentários.

    ```csharp
    // Create a DefaultAzureCredentialOptions object to configure the DefaultAzureCredential
    DefaultAzureCredentialOptions options = new()
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true
    };
    
    // Create a Service Bus client using the namespace and DefaultAzureCredential
    // The DefaultAzureCredential will use the Azure CLI credentials, so ensure you are logged in
    ServiceBusClient client = new(svcbusNameSpace, new DefaultAzureCredential(options));
    ```

1. Localize o comentário **// ADICIONAR CÓDIGO PARA ENVIAR MENSAGENS PARA A FILA** e adicione o código a seguir diretamente após o comentário. Examine o código e os comentários.

    ```csharp
    // Create a sender for the specified queue
    ServiceBusSender sender = client.CreateSender(queueName);
    
    // create a batch 
    using ServiceBusMessageBatch messageBatch = await sender.CreateMessageBatchAsync();
    
    // number of messages to be sent to the queue
    const int numOfMessages = 3;
    
    for (int i = 1; i <= numOfMessages; i++)
    {
        // try adding a message to the batch
        if (!messageBatch.TryAddMessage(new ServiceBusMessage($"Message {i}")))
        {
            // if it is too large for the batch
            throw new Exception($"The message {i} is too large to fit in the batch.");
        }
    }
    
    try
    {
        // Use the producer client to send the batch of messages to the Service Bus queue
        await sender.SendMessagesAsync(messageBatch);
        Console.WriteLine($"A batch of {numOfMessages} messages has been published to the queue.");
    }
    finally
    {
        // Calling DisposeAsync on client types is required to ensure that network
        // resources and other unmanaged objects are properly cleaned up.
        await sender.DisposeAsync();
    }
    
    Console.WriteLine("Press any key to continue");
    Console.ReadKey();
    ```

1. Pressione **ctrl+s** para salvar o arquivo e continue com o exercício.

### Adicionar código para processar mensagens na fila

1. Localize o comentário **// ADICIONAR CÓDIGO PARA PROCESSAR MENSAGENS DA FILA** e adicione o código a seguir diretamente após o comentário. Examine o código e os comentários.

    ```csharp
    // Create a processor that we can use to process the messages in the queue
    ServiceBusProcessor processor = client.CreateProcessor(queueName, new ServiceBusProcessorOptions());
    
    // Idle timeout in milliseconds, the idle timer will stop the processor if there are no more 
    // messages in the queue to process
    const int idleTimeoutMs = 3000;
    System.Timers.Timer idleTimer = new(idleTimeoutMs);
    idleTimer.Elapsed += async (s, e) =>
    {
        Console.WriteLine($"No messages received for {idleTimeoutMs / 1000} seconds. Stopping processor...");
        await processor.StopProcessingAsync();
    };
    
    try
    {
        // add handler to process messages
        processor.ProcessMessageAsync += MessageHandler;
    
        // add handler to process any errors
        processor.ProcessErrorAsync += ErrorHandler;
    
        // start processing 
        idleTimer.Start();
        await processor.StartProcessingAsync();
    
        Console.WriteLine($"Processor started. Will stop after {idleTimeoutMs / 1000} seconds of inactivity.");
        // Wait for the processor to stop
        while (processor.IsProcessing)
        {
            await Task.Delay(500);
        }
        idleTimer.Stop();
        Console.WriteLine("Stopped receiving messages");
    }
    finally
    {
        // Dispose processor after use
        await processor.DisposeAsync();
    }
    
    // handle received messages
    async Task MessageHandler(ProcessMessageEventArgs args)
    {
        string body = args.Message.Body.ToString();
        Console.WriteLine($"Received: {body}");
    
        // Reset the idle timer on each message
        idleTimer.Stop();
        idleTimer.Start();
    
        // complete the message. message is deleted from the queue. 
        await args.CompleteMessageAsync(args.Message);
    }
    
    // handle any errors when receiving messages
    Task ErrorHandler(ProcessErrorEventArgs args)
    {
        Console.WriteLine(args.Exception.ToString());
        return Task.CompletedTask;
    }
    ```

1. Pressione **ctrl+s** para salvar o arquivo e **ctrl+q** para sair do editor.

## Execute o aplicativo e entre no Azure.

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para entrar no Azure.

    ```
    az login
    ```

    **<font color="red">Você deve entrar no Azure, mesmo que a sessão do Cloud Shell já esteja autenticada.</font>**

    > **Observação**: na maioria dos cenários, apenas usar *az login* será suficiente. No entanto, se você tiver assinaturas em vários locatários, talvez seja necessário especificar o locatário usando o parâmetro *--tenant* . Confira [Entrar no Azure interativamente usando a CLI do Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obter detalhes.

1. Execute o comando a seguir para iniciar o aplicativo de console. O aplicativo pausará vários estágios e solicitará que você pressione uma tecla para continuar. Com isso, você consegue exibir as mensagens no portal do Azure.

    ```
    dotnet run
    ```

    

1. Na página portal do Azure, navegue até o namespace do Barramento de Serviço que você criou. 

1. Selecione **myqueue** na parte inferior da janela **Visão geral**.

1. Selecione **Service Bus Explorer** no painel de navegação esquerdo.

1. Selecione **Pré-visualizar desde o início**, e as três mensagens deverão aparecer após alguns segundos.

1. No Cloud Shell, pressione qualquer tecla para continuar e o aplicativo processará as três mensagens. 
 
1. Retorne ao portal depois que o aplicativo concluir o processamento das mensagens. Selecione **Pré-visualizar do início** novamente e observe que não há mensagens na fila.

## Limpar os recursos

Agora que você concluiu o exercício, exclua os recursos de nuvem que criou para evitar uso desnecessário de recursos.

1. Navegue até o grupo de recursos que você criou e exiba o conteúdo dos recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.

> **CUIDADO:** excluir o grupo de recursos excluirá todos os recursos que ele contém. Se você escolheu um grupo de recursos para este exercício, todos os recursos fora do escopo deste 

