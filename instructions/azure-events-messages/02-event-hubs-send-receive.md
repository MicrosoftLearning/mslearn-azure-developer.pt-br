---
lab:
  topic: Azure events and messaging
  title: Enviar e recuperar eventos de Hubs de Eventos do Azure
  description: Saiba como enviar e recuperar eventos dos Hubs de Eventos do Azure com o SDK do .NET Azure.Messaging.EventHubs.
---

# Enviar e recuperar eventos de Hubs de Eventos do Azure

Neste exercício, você cria recursos dos Hubs de Eventos do Azure e criar um aplicativo de console .NET para enviar e receber eventos usando o SDK do **Azure.Messaging.EventHubs**. Você aprenderá a provisionar recursos de nuvem, interagir com os Hubs de Eventos e limpar seu ambiente quando terminar.

Tarefas realizadas neste exercício:

* Criar um grupo de recursos
* Criar recursos de Hubs de Eventos do Azure
* Criar um aplicativo de console do .NET para enviar e recuperar eventos
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
    namespaceName=eventhubsns$RANDOM
    ```

### Criar um namespace e um hub de eventos dos Hubs de Eventos do Azure

Um namespace dos Hubs de Eventos do Azure é um contêiner lógico para recursos do hub de eventos no Azure. Ele fornece um contêiner de escopo exclusivo em que você pode criar um ou mais hubs de eventos, que são usados para ingerir, processar e armazenar grandes volumes de dados de eventos. As instruções a seguir são executadas no Cloud Shell. 

1. Execute o comando a seguir para criar um namespace dos Hubs de Eventos.

    ```
    az eventhubs namespace create --name $namespaceName --resource-group $resourceGroup -l $location
    ```

1. Execute o comando a seguir para criar um hub de eventos chamado **myEventHub** no namespace dos Hubs de Eventos. 

    ```
    az eventhubs eventhub create --name myEventHub --resource-group $resourceGroup \
      --namespace-name $namespaceName
    ```

### Atribuir uma função ao nome de usuário do Microsoft Entra

Para permitir que seu aplicativo envie e receba mensagens, atribua o usuário Microsoft Entra à função **Proprietário de Dados dos Hubs de Eventos do Azure** no nível do namespace dos Hubs de Eventos. Isso dá à sua conta de usuário permissão para gerenciar e acessar filas e tópicos usando o RBAC do Azure. Execute as etapas a seguir no Cloud Shell.

1. Execute o comando a seguir para recuperar o **userPrincipalName** da sua conta. Isso representa a quem a função será atribuída.

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. Execute o comando a seguir para recuperar a ID de recurso do namespace dos Hubs de Eventos. A ID do recurso define o escopo da atribuição de função para um namespace específico.

    ```
    resourceID=$(az eventhubs namespace show --resource-group $resourceGroup \
        --name $namespaceName --query id --output tsv)
    ```
1. Execute o comando a seguir para criar e atribuir a função **Proprietário de Dados de Hubs de Eventos do Azure**, o que dá permissão para enviar e recuperar eventos.

    ```
    az role assignment create --assignee $userPrincipal \
        --role "Azure Event Hubs Data Owner" \
        --scope $resourceID
    ```

## Enviar e recuperar eventos com um aplicativo de console do .NET

Agora que os recursos necessários estão implantados no Azure, a próxima etapa é configurar o aplicativo de console. As etapas a seguir são executadas no Cloud Shell.

>**Dica:** redimensione o Cloud Shell para exibir mais informações e código arrastando a borda superior. Você também pode usar os botões minimizar e maximizar para alternar entre o Cloud Shell e a interface do portal principal.

1. Execute os comandos a seguir para criar um diretório para conter o projeto e altere-o para o diretório do projeto.

    ```
    mkdir eventhubs
    cd eventhubs
    ```

1. Crie o aplicativo de console .NET.

    ```
    dotnet new console
    ```

1. Execute os comandos a seguir para adicionar os pacotes **Azure.Messaging.EventHubs** e **Azure.Identity** ao projeto.

    ```
    dotnet add package Azure.Messaging.EventHubs
    dotnet add package Azure.Identity
    ```

Agora, é hora de substituir o código do modelo no arquivo **Program.cs** usando o editor no Cloud Shell.

### Adicionar o código inicial para o projeto

1. Execute o seguinte comando no Cloud Shell para começar a editar o aplicativo.

    ```
    code Program.cs
    ```

1. Substitua qualquer conteúdo pelo código a seguir. Examine os comentários no código e substitua **YOUR_EVENT_HUB_NAMESPACE** pelo namespace do hub de eventos.

    ```csharp
    using Azure.Messaging.EventHubs;
    using Azure.Messaging.EventHubs.Producer;
    using Azure.Messaging.EventHubs.Consumer;
    using Azure.Identity;
    using System.Text;
    
    // TO-DO: Replace YOUR_EVENT_HUB_NAMESPACE with your actual Event Hub namespace
    string namespaceURL = "YOUR_EVENT_HUB_NAMESPACE.servicebus.windows.net";
    string eventHubName = "myEventHub"; 
    
    // Create a DefaultAzureCredentialOptions object to exclude certain credentials
    DefaultAzureCredentialOptions options = new()
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true
    };
    
    // Number of events to be sent to the event hub
    int numOfEvents = 3;
    
    // CREATE A PRODUCER CLIENT AND SEND EVENTS
    
    
    
    // CREATE A CONSUMER CLIENT AND RECEIVE EVENTS
    
    
    ```

1. Pressione **ctrl+s** para salvar as alterações.

### Adicionar código para concluir o aplicativo

Nesta seção, você adiciona código para criar os clientes do produtor e do consumidor para enviar e receber eventos.

1. Localize o comentário **// CRIAR UM CLIENTE PRODUTOR E ENVIAR EVENTOS** e adicione o código a seguir diretamente após o comentário. Examine os comentários no código.

    ```csharp
    // Create a producer client to send events to the event hub
    EventHubProducerClient producerClient = new EventHubProducerClient(
        namespaceURL,
        eventHubName,
        new DefaultAzureCredential(options));
    
    // Create a batch of events 
    using EventDataBatch eventBatch = await producerClient.CreateBatchAsync();
    
    
    // Adding a random number to the event body and sending the events. 
    var random = new Random();
    for (int i = 1; i <= numOfEvents; i++)
    {
        int randomNumber = random.Next(1, 101); // 1 to 100 inclusive
        string eventBody = $"Event {randomNumber}";
        if (!eventBatch.TryAdd(new EventData(Encoding.UTF8.GetBytes(eventBody))))
        {
            // if it is too large for the batch
            throw new Exception($"Event {i} is too large for the batch and cannot be sent.");
        }
    }
    
    try
    {
        // Use the producer client to send the batch of events to the event hub
        await producerClient.SendAsync(eventBatch);
    
        Console.WriteLine($"A batch of {numOfEvents} events has been published.");
        Console.WriteLine("Press Enter to retrieve and print the events...");
        Console.ReadLine();
    }
    finally
    {
        await producerClient.DisposeAsync();
    }
    ```

1. Pressione **ctrl+s** para salvar as alterações.

1. Localize o comentário **// CRIAR UM CLIENTE CONSUMIDOR E RECUPERAR EVENTOS** e adicione o código a seguir diretamente após o comentário. Examine os comentários no código.

    ```csharp
    // Create an EventHubConsumerClient
    await using var consumerClient = new EventHubConsumerClient(
        EventHubConsumerClient.DefaultConsumerGroupName,
        namespaceURL,
        eventHubName,
        new DefaultAzureCredential(options));
    
    Console.Clear();
    Console.WriteLine("Retrieving all events from the hub...");
    
    // Get total number of events in the hub by summing (last - first + 1) for all partitions
    // This count is used to determine when to stop reading events
    long totalEventCount = 0;
    string[] partitionIds = await consumerClient.GetPartitionIdsAsync();
    foreach (var partitionId in partitionIds)
    {
        PartitionProperties properties = await consumerClient.GetPartitionPropertiesAsync(partitionId);
        if (!properties.IsEmpty && properties.LastEnqueuedSequenceNumber >= properties.BeginningSequenceNumber)
        {
            totalEventCount += (properties.LastEnqueuedSequenceNumber - properties.BeginningSequenceNumber + 1);
        }
    }
    
    // Start retrieving events from the event hub and print to the console
    int retrievedCount = 0;
    await foreach (PartitionEvent partitionEvent in consumerClient.ReadEventsAsync(startReadingAtEarliestEvent: true))
    {
        if (partitionEvent.Data != null)
        {
            string body = Encoding.UTF8.GetString(partitionEvent.Data.Body.ToArray());
            Console.WriteLine($"Retrieved event: {body}");
            retrievedCount++;
            if (retrievedCount >= totalEventCount)
            {
                Console.WriteLine("Done retrieving events. Press Enter to exit...");
                Console.ReadLine();
                return;
            }
        }
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

1. Inicie o aplicativo executando o seguinte comando:

    ```
    dotnet run
    ```

    Após alguns segundos, você deverá ver uma saída semelhante ao seguinte exemplo:
    
    ```
    A batch of 3 events has been published.
    Press Enter to retrieve and print the events...
    
    Retrieving all events from the hub...
    Retrieved event: Event 4
    Retrieved event: Event 96
    Retrieved event: Event 74
    Done retrieving events. Press Enter to exit...
    ```

O aplicativo sempre envia três eventos para o hub, mas recupera todos os eventos no hub. Se você executar o aplicativo várias vezes, será recuperado um número cada vez maior de eventos. Os números aleatórios usados para a criação de eventos ajudam a identificar eventos diferentes.

## Limpar os recursos

Agora que você concluiu o exercício, exclua os recursos de nuvem que criou para evitar uso desnecessário de recursos.

1. No navegador, vá par o portal do Azure [https://portal.azure.com](https://portal.azure.com). Faça login com suas credenciais do Azure se for solicitado.
1. Navegue até o grupo de recursos que você criou e exiba o conteúdo dos recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.

> **CUIDADO:** excluir o grupo de recursos excluirá todos os recursos que ele contém. Se você escolher um grupo de recursos para este exercício, todos os recursos fora do escopo deste exercício também serão excluídos. 
