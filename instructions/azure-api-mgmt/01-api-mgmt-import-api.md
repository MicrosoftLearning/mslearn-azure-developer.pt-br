---
lab:
  topic: Azure API Management
  title: Importar e configurar uma API com o Gerenciamento de API do Azure
  description: 'Saiba como importar, publicar e testar uma API em conformidade com a especificação de OpenAPI.'
---

# Importar e configurar uma API com o Gerenciamento de API do Azure

Neste exercício, você criará uma instância do Gerenciamento de API do Azure, importará uma API de back-end da especificação OpenAPI, definirá as configurações de API, incluindo a URL do serviço Web e os requisitos de assinatura, e testará as operações de API para verificar se funcionam corretamente.

Tarefas realizadas neste exercício:

* Criar uma instância de APIM (Gerenciamento de API) do Azure
* Importar uma API
* Definir as configurações de back-end
* Testar a API

Este exercício levará aproximadamente **20** minutos para ser concluído.

## Criar uma instância de Gerenciamento de API

Nesta seção do exercício, você cria um grupo de recursos e uma conta de Armazenamento do Azure. Você também registra o ponto de extremidade e a chave de acesso da conta.

1. No navegador, vá par o portal do Azure [https://portal.azure.com](https://portal.azure.com). Faça login com suas credenciais do Azure se for solicitado.

1. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um novo Cloud Shell no portal do Azure, selecionando um ambiente do ***Bash***. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure. Se você for solicitado a selecionar uma conta de armazenamento na qual manter seus arquivos, selecione **Nenhuma conta de armazenamento necessária**, sua assinatura e **Aplicar**.

    > **Observação**: se você já criou uma Cloud Shell que usa um ambiente *PowerShell*, troque-o pelo ***Bash***.

1. Crie um grupo de recursos para os recursos que este exercício requer. Substitua **myResourceGroup** por um nome que você quer usar para o grupo de recursos. Você pode substituir **eastus2** por uma região perto de você, se necessário. Se você já tiver um grupo de recursos que deseja usar, vá para a próxima etapa.

    ```azurecli
    az group create --location eastus2 --name myResourceGroup
    ```

1. Crie algumas variáveis para os comandos da CLI usarem. Isso reduz a quantidade de digitação. Substitua **myLocation** pelo valor escolhido antes. O nome da APIM precisa ser um nome globalmente exclusivo, e o script a seguir gera uma cadeia de caracteres aleatória. Substitua **myEmail** por um endereço de email que você possa acessar.

    ```bash
    myApiName=import-apim-$RANDOM
    myLocation=myLocation
    myEmail=myEmail
    ```

1. Crie uma instância APIM. O comando **az apim create** é usado para criar a instância. Substitua **myResourceGroup** pelo valor escolhido antes.

    ```bash
    az apim create -n $myApiName \
        --location $myLocation \
        --publisher-email $myEmail  \
        --resource-group myResourceGroup \
        --publisher-name Import-API-Exercise \
        --sku-name Consumption 
    ```
    > **Observação:** A operação deve ser concluída em cerca de cinco minutos. 

## Importar uma API de back-end

Esta seção mostra como importar e publicar uma API de back-end da especificação OpenAPI.

1. No portal do Azure, pesquise e selecione **Serviços de Gerenciamento de API**.

1. Na tela **serviços de Gerenciamento de API**, selecione a instância de Gerenciamento de API criada.

1. No painel de navegação do **Serviço de gerenciamento de API**, selecione **> APIs** e então **APIs**.

    ![Captura de tela da seção APIs do painel de navegação.](./media/select-apis-navigation-pane.png)


1. Selecione **OpenAPI** na seção **Criar com base na definição** e defina a alternância **Básico/Completo** como **Completo** no pop-up exibido.

    ![Captura de tela da caixa de diálogo da OpenAPI. Os campos são detalhados na tabela a seguir.](./media/create-api.png)

    Use os valores da tabela abaixo para preencher o formulário. Você pode deixar o valor padrão para os campos não mencionados.

    | Configuração | Valor | Descrição |
    |--|--|--|
    | **Especificação de OpenAPI** | `https://bigconference.azurewebsites.net/` | Faz referência ao serviço que implementa a API, as solicitações são encaminhadas para esse endereço. A maioria das informações necessárias no formulário é preenchida automaticamente depois que você insere esse valor. |
    | **Esquema de URL** | Selecione **HTTPS**. | Define o nível de segurança do protocolo HTTP aceito pela API. |

1. Selecione **Criar**.

## Definir as configurações da API

A *Big Conference API* foi criada. Agora é hora de definir as configurações da API. 

1. Selecione **Configurações** no menu.

1. Insira `https://bigconference.azurewebsites.net/` o campo **URL do serviço Web**.

1. Desmarque a caixa de seleção **Assinatura necessária**.

1. Selecione **Salvar**.

## Testar a API

Agora que a API foi importada e configurada, é hora de testar a API.

1. Selecione **Testar** na barra de menus. Isso exibirá todas as operações disponíveis na API.

1. Pesquise e selecione a operação **Speakers_Get**. 

1. Selecione **Enviar**. Talvez seja necessário rolar para baixo na página para exibir a resposta HTTP.

    O back-end responde com **200 OK** e alguns dados.

## Limpar os recursos

Agora que você concluiu o exercício, exclua os recursos de nuvem que criou para evitar uso desnecessário de recursos.

1. No navegador, vá par o portal do Azure [https://portal.azure.com](https://portal.azure.com). Faça login com suas credenciais do Azure se for solicitado.
1. Navegue até o grupo de recursos que você criou e exiba o conteúdo dos recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.

> **CUIDADO:** excluir o grupo de recursos excluirá todos os recursos que ele contém. Se você escolher um grupo de recursos para este exercício, todos os recursos fora do escopo deste exercício também serão excluídos.
