---
lab:
  topic: Azure container services
  title: Implantar um contêiner nos Aplicativos de Contêiner do Azure com a CLI do Azure
  description: Saiba como usar comandos da CLI do Azure para criar um ambiente seguro de Aplicativos de Contêiner do Azure e implantar um contêiner.
---

# Implantar um contêiner nos Aplicativos de Contêiner do Azure com a CLI do Azure

Neste exercício, você implantará um aplicativo em contêiner nos Aplicativos de Contêiner do Azure usando a CLI do Azure. Você aprenderá a criar um ambiente de aplicativo de contêiner, implantar seu contêiner e verificar se o aplicativo está em execução no Azure.

Tarefas realizadas neste exercício:

* Criar recursos no Azure
* Criar um ambiente de Aplicativos de Contêiner do Azure
* Implantar um aplicativo de contêiner no ambiente

Este exercício levará aproximadamente **15** minutos para ser concluído.

## Criar um grupo de recursos e preparar o ambiente do Azure

1. No navegador, vá par o portal do Azure [https://portal.azure.com](https://portal.azure.com). Faça login com suas credenciais do Azure se for solicitado.

1. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um novo Cloud Shell no portal do Azure, selecionando um ambiente do ***Bash***. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure. Se você for solicitado a selecionar uma conta de armazenamento na qual manter seus arquivos, selecione **Nenhuma conta de armazenamento necessária**, sua assinatura e **Aplicar**.

    > **Observação**: se você já criou uma Cloud Shell que usa um ambiente *PowerShell*, troque-o pelo ***Bash***.

1. Crie um grupo de recursos para os recursos que este exercício requer. Substitua **myResourceGroup** por um nome que você quer usar para o grupo de recursos. Você pode substituir **eastus** por uma região perto de você, se necessário. Se você já tiver um grupo de recursos que deseja usar, vá para a próxima etapa.

    ```azurecli
    az group create --location eastus --name myResourceGroup
    ```

1. Execute o comando a seguir para garantir que você tenha a versão mais recente da extensão de Aplicativos de Contêiner do Azure para a CLI instalada.

    ```azurecli
    az extension add --name containerapp --upgrade
    ```

### Registrar namespaces

Há dois namespaces que precisam ser registrados para os Aplicativos de Contêiner do Azure, e você garante que eles estejam registrados nas etapas a seguir. Cada registro pode levar alguns minutos para ser concluído se ainda não estiverem configurados em sua assinatura. 

1. Registre o namespace **Microsoft.App**. 

    ```bash
    az provider register --namespace Microsoft.App
    ```

1. Registre o provedor **Microsoft.OperationalInsights** para o workspace do Log Analytics do Azure Monitor se você não o tiver usado antes.

    ```bash
    az provider register --namespace Microsoft.OperationalInsights
    ```

## Criar um ambiente de Aplicativos de Contêiner do Azure

Um ambiente em aplicativos de contêiner do Azure cria um marco de delimitação seguro em um grupo de aplicativos de contêiner. Os Aplicativos de Contêiner implantados no mesmo ambiente são implantados na mesma rede virtual e registram logs no mesmo workspace do Log Analytics.

1. Crie um ambiente com o comando **az containerapp env create**. Substitua **myResourceGroup** e **myLocation** pelos valores usados antes. Leva alguns minutos para a operação ser concluída.

    ```bash
    az containerapp env create \
        --name my-container-env \
        --resource-group myResourceGroup \
        --location myLocation
    ```

## Implantar um aplicativo de contêiner no ambiente

Depois que o ambiente do aplicativo de contêiner concluir a implantação, você poderá implantar uma imagem de contêiner em seu ambiente.

1. Implante uma imagem de contêiner de aplicativo de exemplo com o comando **containerapp create**. Substitua **myResourceGroup** pelo valor usado antes.

    ```bash
    az containerapp create \
        --name my-container-app \
        --resource-group myResourceGroup \
        --environment my-container-env \
        --image mcr.microsoft.com/azuredocs/containerapps-helloworld:latest \
        --target-port 80 \
        --ingress 'external' \
        --query properties.configuration.ingress.fqdn
    ```

    Ao definir **--ingress** como **externo**, você disponibiliza o aplicativo de contêiner para solicitações públicas. O comando retorna um link para acessar seu aplicativo.

    ```
    Container app created. Access your app at <url>
    ```

Para verificar a implantação, selecione a URL retornada pelo comando **az containerapp create** para verificar se o aplicativo de contêiner está em execução.

## Limpar os recursos

Agora que você concluiu o exercício, exclua os recursos de nuvem que criou para evitar uso desnecessário de recursos.

1. No navegador, vá par o portal do Azure [https://portal.azure.com](https://portal.azure.com). Faça login com suas credenciais do Azure se for solicitado.
1. Navegue até o grupo de recursos que você criou e exiba o conteúdo dos recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.

> **CUIDADO:** excluir o grupo de recursos excluirá todos os recursos que ele contém. Se você escolher um grupo de recursos para este exercício, todos os recursos fora do escopo deste exercício também serão excluídos.
