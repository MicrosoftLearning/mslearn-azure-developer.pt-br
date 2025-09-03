---
lab:
  topic: Azure events and messaging
  title: Rotear eventos para um ponto de extremidade personalizado com a Grade de Eventos do Azure
  description: Saiba como usar a Grade de Eventos do Azure para rotear eventos para um ponto de extremidade personalizado.
---

# Rotear eventos para um ponto de extremidade personalizado com a Grade de Eventos do Azure

Neste exercício, você criará um Grade de Eventos do Azure e um ponto de extremidade do aplicativo Web e criará um aplicativo de console .NET que envia eventos personalizados para o tópico grade de eventos. Você aprenderá a configurar assinaturas de eventos, autenticar com a Grade de Eventos e verificar se os eventos foram roteados com êxito para o ponto de extremidade exibindo-os no aplicativo Web.

Tarefas realizadas neste exercício:

* Criar recursos da Grade de Eventos do Azure
* Habilitar um provedor de recursos da Grade de Eventos
* Criar um tópico na Grade de Eventos
* Criar um ponto de extremidade de mensagem
* Assinar o tópico
* Enviar um evento com um aplicativo de console .NET
* Limpar os recursos

Este exercício levará aproximadamente **30** minutos para ser concluído.

## Criar recursos da Grade de Eventos do Azure

Nesta seção do exercício, você criará os recursos necessários no Azure com a CLI do Azure.

1. No navegador, vá par o portal do Azure [https://portal.azure.com](https://portal.azure.com). Faça login com suas credenciais do Azure se for solicitado.

1. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um novo Cloud Shell no portal do Azure, selecionando um ambiente do ***Bash***. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure. Se você for solicitado a selecionar uma conta de armazenamento na qual manter seus arquivos, selecione **Nenhuma conta de armazenamento necessária**, sua assinatura e **Aplicar**.

    > **Observação**: se você já criou uma Cloud Shell que usa um ambiente *PowerShell*, troque-o pelo ***Bash***.

1. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para a versão clássica** (isso é necessário para usar o editor de código).

1. Crie um grupo de recursos para os recursos que este exercício requer. Se você já tiver um grupo de recursos que deseja usar, vá para a próxima etapa. Substitua **myResourceGroup** por um nome que você quer usar para o grupo de recursos. Você pode substituir **eastus** por uma região perto de você, se necessário.

    ```bash
    az group create --name myResourceGroup --location eastus
    ```

1. Muitos comandos exigem nomes exclusivos e usam os mesmos parâmetros. Criar algumas variáveis reduzirá as alterações necessárias para os comandos que criam recursos. Execute os comandos a seguir para criar as variáveis necessárias. Substitua **myResourceGroup** pelo nome que você está usando para este exercício. Se você alterou o local na etapa anterior, faça a mesma alteração na variável **location**.

    ```bash
    let rNum=$RANDOM
    resourceGroup=myResourceGroup
    location=eastus
    topicName="mytopic-evgtopic-${rNum}"
    siteName="evgsite-${rNum}"
    siteURL="https://${siteName}.azurewebsites.net"
    ```

### Habilitar um provedor de recursos da Grade de Eventos

Um Provedor de Recursos do Azure é um serviço que define e gerencia tipos específicos de recursos no Azure. É o que o Azure usa nos bastidores quando você implanta ou gerencia recursos. Registre o provedor de recursos da Grade de Eventos com o comando **az provider register**. 

```bash
az provider register --namespace Microsoft.EventGrid
```

O registro pode demorar alguns minutos. Você pode verificar o status com o comando a seguir.

```bash
az provider show --namespace Microsoft.EventGrid --query "registrationState"
```

> **Observação:** Essa etapa só é necessária em assinaturas que ainda não usaram a Grade de Eventos.

### Criar um tópico na Grade de Eventos

Crie um tópico usando o comando **az eventgrid topic create**. O nome deve ser único porque faz parte da entrada DNS.  

```bash
az eventgrid topic create --name $topicName \
    --location $location \
    --resource-group $resourceGroup
```

### Criar um ponto de extremidade de mensagem

Antes de assinar o tópico personalizado, precisamos criar o ponto de extremidade para a mensagem de evento. Normalmente, o ponto de extremidade executa ações com base nos dados de evento. O script a seguir usa um aplicativo Web pré-criado que exibe as mensagens de evento. A solução implantada inclui um plano do Serviço de Aplicativo, um aplicativo Web do Aplicativo do Serviço de e o código-fonte do GitHub.

1. Execute os comandos a seguir para criar um ponto de extremidade de mensagem. O comando **echo** exibirá a URL do site para o ponto de extremidade.

    ```bash
    az deployment group create \
        --resource-group $resourceGroup \
        --template-uri "https://raw.githubusercontent.com/Azure-Samples/azure-event-grid-viewer/main/azuredeploy.json" \
        --parameters siteName=$siteName hostingPlanName=viewerhost
    
    echo "Your web app URL: ${siteURL}"
    ```

    > **Observação:** esse comando pode demorar um pouco para ser concluído.

1. Abra uma nova guia no navegador e vá até a URL gerada no final do script anterior para garantir que o aplicativo Web esteja em execução. Você verá o site sem nenhuma mensagem exibida no momento.

    > **Dica:** Deixe o navegador em execução, ele é usado para mostrar atualizações.

### Assinar o tópico

Você assina um tópico da Grade de Eventos para indicar à Grade de Eventos quais eventos você deseja acompanhar e para onde enviá-los. 

1. Assine um tópico usando o comando **az eventgrid event-subscription create**. O script a seguir recupera a ID da assinatura da sua conta e a usa para criar a assinatura do evento.

    ```bash
    endpoint="${siteURL}/api/updates"
    topicId=$(az eventgrid topic show --resource-group $resourceGroup \
        --name $topicName --query "id" --output tsv)
    
    az eventgrid event-subscription create \
        --source-resource-id $topicId \
        --name TopicSubscription \
        --endpoint $endpoint
    ```

1. Exiba novamente o seu aplicativo Web e observe que um evento de validação de assinatura foi enviado a ele. Selecione o ícone de olho para expandir os dados de evento. A Grade de Eventos envia o evento de validação de modo que o ponto de extremidade possa verificar se ele deseja receber os dados de evento. O aplicativo Web inclui o código para validar a assinatura.

## Enviar um evento com um aplicativo de console .NET

Agora que os recursos necessários estão implantados no Azure, a próxima etapa é configurar o aplicativo de console. As etapas a seguir são executadas no Cloud Shell.

>**Dica:** redimensione o Cloud Shell para exibir mais informações e código arrastando a borda superior. Você também pode usar os botões minimizar e maximizar para alternar entre o Cloud Shell e a interface do portal principal.

1. Execute os comandos a seguir para criar um diretório para conter o projeto e altere-o para o diretório do projeto.

    ```bash
    mkdir eventgrid
    cd eventgrid
    ```

1. Crie o aplicativo de console .NET.

    ```bash
    dotnet new console
    ```

1. Execute os comandos a seguir para adicionar os pacotes **Azure.Messaging.EventGrid** e **dotenv.net** ao projeto.

    ```bash
    dotnet add package Azure.Messaging.EventGrid
    dotnet add package dotenv.net
    ```

### Configurar o aplicativo de console

Nesta seção, você recupera o ponto de extremidade do tópico e a chave de acesso para que eles possam ser adicionados a um arquivo **.env** para manter esses segredos.

1. Execute os comandos a seguir para recuperar a URL e a chave de acesso para o tópico que você criou antes. Registre esses valores.

    ```bash
    az eventgrid topic show --name $topicName -g $resourceGroup --query "endpoint" --output tsv
    az eventgrid topic key list --name $topicName -g $resourceGroup --query "key1" --output tsv
    ```

1. Execute o comando a seguir para criar o arquivo **.env** para manter os segredos e, em seguida, abra-o no editor de código.

    ```bash
    touch .env
    code .env
    ```

1. Adicione o código a seguir ao arquivo **.env**. Substitua **YOUR_TOPIC_ENDPOINT** e **YOUR_TOPIC_ACCESS_KEY** pelos valores registrados antes.

    ```
    TOPIC_ENDPOINT="YOUR_TOPIC_ENDPOINT"
    TOPIC_ACCESS_KEY="YOUR_TOPIC_ACCESS_KEY"
    ```

1. Pressione **ctrl+s** para salvar o arquivo e **ctrl+q** para sair do editor.

Agora, é hora de substituir o código do modelo no arquivo **Program.cs** usando o editor no Cloud Shell.

### Adicionar o código para o projeto

1. Execute o seguinte comando no Cloud Shell para começar a editar o aplicativo.

    ```bash
    code Program.cs
    ```

1. Substitua qualquer código existente pelo código a seguir. Examine os comentários no código.

    ```csharp
    using dotenv.net; 
    using Azure.Messaging.EventGrid; 
    
    // Load environment variables from .env file
    DotEnv.Load();
    var envVars = DotEnv.Read();
    
    // Start the asynchronous process to send an Event Grid event
    ProcessAsync().GetAwaiter().GetResult();
    
    async Task ProcessAsync()
    {
        // Retrieve Event Grid topic endpoint and access key from environment variables
        var topicEndpoint = envVars["TOPIC_ENDPOINT"];
        var topicKey = envVars["TOPIC_ACCESS_KEY"];
        
        // Check if the required environment variables are set
        if (string.IsNullOrEmpty(topicEndpoint) || string.IsNullOrEmpty(topicKey))
        {
            Console.WriteLine("Please set TOPIC_ENDPOINT and TOPIC_ACCESS_KEY in your .env file.");
            return;
        }
    
        // Create an EventGridPublisherClient to send events to the specified topic
        EventGridPublisherClient client = new EventGridPublisherClient
            (new Uri(topicEndpoint),
            new Azure.AzureKeyCredential(topicKey));
    
        // Create a new EventGridEvent with sample data
        var eventGridEvent = new EventGridEvent(
            subject: "ExampleSubject",
            eventType: "ExampleEventType",
            dataVersion: "1.0",
            data: new { Message = "Hello, Event Grid!" }
        );
    
        // Send the event to Azure Event Grid
        await client.SendEventAsync(eventGridEvent);
        Console.WriteLine("Event sent successfully.");
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

1. Execute o comando a seguir no Cloud Shell para iniciar o aplicativo de console. Você verá a mensagem **Evento enviado com êxito.** quando a mensagem é enviada.

    ```bash
    dotnet run
    ```

1. Exiba seu aplicativo Web para ver o evento que você acabou de enviar. Selecione o ícone de olho para expandir os dados de evento.

## Limpar os recursos

Agora que você concluiu o exercício, exclua os recursos de nuvem que criou para evitar uso desnecessário de recursos.

1. Navegue até o grupo de recursos que você criou e exiba o conteúdo dos recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.

> **CUIDADO:** excluir o grupo de recursos excluirá todos os recursos que ele contém. Se você escolher um grupo de recursos para este exercício, todos os recursos fora do escopo deste exercício também serão excluídos.