---
lab:
  topic: Azure container services
  title: Criar e executar uma imagem de contêiner com Tarefas do Registro de Contêiner do Azure
  description: Saiba como usar comandos da CLI do Azure para criar e executar imagens de contêiner com Tarefas do Registro de Contêiner do Azure.
---

# Criar e executar uma imagem de contêiner com Tarefas do Registro de Contêiner do Azure

Neste exercício, você criará uma imagem de contêiner do código do aplicativo e a enviará por push para Registro de Contêiner do Azure usando a CLI do Azure. Você aprenderá a preparar seu aplicativo para contêineres, criar uma instância do ACR e armazenar sua imagem de contêiner no Azure.

Tarefas realizadas neste exercício:

* Criar um recurso do Registro de Contêiner do Azure
* Criar e transmitir uma imagem a partir de um Dockerfile
* Verifique os resultados
* Executar a imagem no Registro de Contêiner do Azure

Este exercício levará aproximadamente **20** minutos para ser concluído.

## Criar um recurso do Registro de Contêiner do Azure

1. No navegador, vá par o portal do Azure [https://portal.azure.com](https://portal.azure.com). Faça login com suas credenciais do Azure se for solicitado.

1. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um novo Cloud Shell no portal do Azure, selecionando um ambiente do ***Bash***. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure. Se você for solicitado a selecionar uma conta de armazenamento na qual manter seus arquivos, selecione **Nenhuma conta de armazenamento necessária**, sua assinatura e **Aplicar**.

    > **Observação**: se você já criou uma Cloud Shell que usa um ambiente *PowerShell*, troque-o pelo ***Bash***.

1. Crie um grupo de recursos para os recursos que este exercício requer. Substitua **myResourceGroup** por um nome que você quer usar para o grupo de recursos. Você pode substituir **eastus** por uma região perto de você, se necessário. Se você já tiver um grupo de recursos que deseja usar, vá para a próxima etapa.

    ```
    az group create --location eastus --name myResourceGroup
    ```

1. Execute o comando a seguir para criar um registro de contêiner básico. O nome do registro deve ser exclusivo no Azure e conter de 5 a 50 caracteres alfanuméricos. Substitua **myResourceGroup** pelo nome que você usou antes e **myContainerRegistry** por um valor exclusivo.

    ```bash
    az acr create --resource-group myResourceGroup \
        --name myContainerRegistry --sku Basic
    ```

    > **Observação:** O comando cria um registro *Básico*, uma opção com otimização de custo para os desenvolvedores que estão aprendendo a usar o Registro de Contêiner do Azure.

## Criar e transmitir uma imagem a partir de um Dockerfile

Em seguida, você cria e envia uma imagem por push com base em um Dockerfile.

1. Execute o comando a seguir para criar o Dockerfile. O Dockerfile contém uma única linha que faz referência à imagem *hello-world* hospedada no Registro de Contêiner da Microsoft.

    ```bash
    echo FROM mcr.microsoft.com/hello-world > Dockerfile
    ```

1. Execute o comando **az acr build** a seguir, que compila a imagem e, depois que a imagem é compilada com êxito, a envia por push para o registro. Substitua **myContainerRegistry** pelo nome que você criou antes.

    ```bash
    az acr build --image sample/hello-world:v1  \
        --registry myContainerRegistry \
        --file Dockerfile .
    ```

    A seguir está um exemplo abreviado da saída do comando anterior mostrando as últimas linhas com os resultados finais. No campo do *repositório*, você pode ver que a imagem *sample/hello-word* está listada.

    ```
    - image:
        registry: myContainerRegistry.azurecr.io
        repository: sample/hello-world
        tag: v1
        digest: sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a
      runtime-dependency:
        registry: mcr.microsoft.com
        repository: hello-world
        tag: latest
        digest: sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a
      git: {}
    
    
    Run ID: cf1 was successful after 11s
    ```

## Verifique os resultados

1. Execute o comando a seguir para listar os repositórios no registro. Substitua **myContainerRegistry** pelo nome que você criou antes.

    ```bash
    az acr repository list --name myContainerRegistry --output table
    ```

    Saída:

    ```
    Result
    ----------------
    sample/hello-world
    ```

1. Execute o comando a seguir para listar as marcas no repositório **sample/hello-world**. Substitua **myContainerRegistry** pelo nome que você usou antes.

    ```bash
    az acr repository show-tags --name myContainerRegistry \
        --repository sample/hello-world --output table
    ```

    Saída:

    ```
    Result
    --------
    v1
    ```

## Executar a imagem no ACR

1. Execute a imagem de contêiner *sample/hello-world:v1* do registro de contêiner com o comando **az acr run**. O exemplo a seguir usa **$Registry** para especificar o Registro em que você executa o comando. Substitua **myContainerRegistry** pelo nome que você usou antes.

    ```bash
    az acr run --registry myContainerRegistry \
        --cmd '$Registry/sample/hello-world:v1' /dev/null
    ```

    O parâmetro **cmd** neste exemplo executa o contêiner em sua configuração padrão, mas o **cmd** dá suporte a outros parâmetros **docker run** ou até mesmo a outros comandos do **docker**. 

    O seguinte exemplo de saída é abreviado:

    ```
    Packing source code into tar to upload...
    Uploading archived source code from '/tmp/run_archive_ebf74da7fcb04683867b129e2ccad5e1.tar.gz'...
    Sending context (1.855 KiB) to registry: mycontainerre...
    Queued a run with ID: cab
    Waiting for an agent...
    2019/03/19 19:01:53 Using acb_vol_60e9a538-b466-475f-9565-80c5b93eaa15 as the home volume
    2019/03/19 19:01:53 Creating Docker network: acb_default_network, driver: 'bridge'
    2019/03/19 19:01:53 Successfully set up Docker network: acb_default_network
    2019/03/19 19:01:53 Setting up Docker configuration...
    2019/03/19 19:01:54 Successfully set up Docker configuration
    2019/03/19 19:01:54 Logging in to registry: mycontainerregistry008.azurecr.io
    2019/03/19 19:01:55 Successfully logged into mycontainerregistry008.azurecr.io
    2019/03/19 19:01:55 Executing step ID: acb_step_0. Working directory: '', Network: 'acb_default_network'
    2019/03/19 19:01:55 Launching container with name: acb_step_0
    
    Hello from Docker!
    This message shows that your installation appears to be working correctly.
    
    2019/03/19 19:01:56 Successfully executed container: acb_step_0
    2019/03/19 19:01:56 Step ID: acb_step_0 marked as successful (elapsed time in seconds: 0.843801)
    
    Run ID: cab was successful after 6s
    ```

## Limpar os recursos

Agora que você concluiu o exercício, exclua os recursos de nuvem que criou para evitar uso desnecessário de recursos.

1. Navegue até o grupo de recursos que você criou e exiba o conteúdo dos recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.

> **CUIDADO:** excluir o grupo de recursos excluirá todos os recursos que ele contém. Se você escolher um grupo de recursos para este exercício, todos os recursos fora do escopo deste exercício também serão excluídos.
