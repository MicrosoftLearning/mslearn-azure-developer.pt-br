---
lab:
  topic: Application Insights
  title: Monitorar um aplicativo com autoinstrução
  description: 'Saiba como monitorar um aplicativo no Application Insights sem modificar o código configurando a autoinstrução '
---

# Monitorar um aplicativo com autoinstrução

Neste exercício, crie um aplicativo Web Serviço de Aplicativo do Azure com o Application Insights habilitado, configure a autoinstrução sem modificar o código, crie e implante um aplicativo Blazor e exiba as métricas do aplicativo e os dados de erro no Application Insights. Implementar o monitoramento e a observabilidade abrangentes de aplicativos, sem precisar fazer alterações em seu código, simplifica as implantações e migrações.

Tarefas realizadas neste exercício:

* Criar um recurso de aplicativo Web com Application Insights habilitado
* Configure a instrumentação para o aplicativo Web.
* Crie um aplicativo Blazor e implante-o no recurso de aplicativo Web.
* Exibir a atividade do aplicativo no Application Insights
* Limpar os recursos

Este exercício levará aproximadamente **20** minutos para ser concluído.

## Criar recursos no Azure

1. No navegador, vá par o portal do Azure [https://portal.azure.com](https://portal.azure.com). Faça login com suas credenciais do Azure se for solicitado.
1. Selecione **+ Criar um recurso** localizado no cabeçalho **Serviços do Azure** próximo à parte superior da página inicial. 
1. Na barra de pesquisa **Pesquisar no Marketplace**, insira *aplicativo Web* e pressione **Enter** para realizar a pesquisa.
1. No bloco Aplicativo Web, selecione a lista suspensa **Criar** e, em seguida, selecione **Aplicativo Web**.

    ![Captura de tela do bloco Aplicativo Web.](./media/create-web-app-tile.png)

Selecionar **Criar** faz abrir um modelo com algumas guias para serem preenchidas com informações sobre a sua implantação. As etapas a seguir orientam você sobre as alterações a serem feitas nas guias relevantes.

1. Preencha a guia **Básico** com as informações da seguinte tabela:

    | Configuração | Ação |
    |--|--|
    | **Assinatura** | Manter o valor padrão. |
    | **Grupo de recursos** | Selecione Criar novo, pressione Enter`rg-WebApp` e, em seguida, selecione OK. Você também pode selecionar um grupo de recursos existente, se preferir. |
    | **Nome** | Insira um nome exclusivo, por exemplo, **YOUR-INITIALS-monitorapp**. Substitua **YOUR-INITIALS** pelas suas iniciais ou algum outro valor. O nome precisa ser exclusivo, por isso pode exigir algumas alterações. |
    | Controle deslizante na configuração **Nome** | Selecione o controle deslizante para desativá-lo. Esse controle deslizante só aparece em algumas configurações do Azure. |
    | **Publicar** | Selecione a opção **Código**. |
    | **Pilha de runtime** | Selecione **.NET 8 (LTS)** no menu suspenso. |
    | **Sistema operacional** | Selecione **Windows**. |
    | **Região** | Mantenha a seleção padrão ou escolha uma região perto de você. |
    | **Plano do Windows** | Mantenha a seleção padrão. |
    | **Plano de preços** | Selecione o menu suspenso e escolha o plano **F1 gratuito**. |

1. Selecione ou navegue até a guia **Monitor + seguro** e insira as informações na tabela a seguir:

    | Configuração | Ação |
    |--|--|
    | **Habilitar o Application Insights** | Selecione **Sim**. |
    | **Application Insights** | Selecione **Criar** e uma caixa de diálogo será exibida. Insira `autoinstrument-insights` no campo **Nome** da caixa de diálogo. Em seguida, selecione **OK** para aceitar o nome. |
    | **Workspace** | Insira `Workspace` se o campo ainda não estiver preenchido e bloqueado. |

1. Selecione **Examinar + criar** e examinar os detalhes da sua implantação. Em seguida, selecione **Criar** para criar os recursos.

Levará alguns minutos para que a implantação seja concluída. Ao terminar, selecione o botão **Ir para o recurso**.

### Definir configurações de instrumentação

Para habilitar o monitoramento sem alterações no código, configure a instrumentação para seu aplicativo no nível de serviço.

1. No menu de navegação à esquerda, expanda **Monitoramento** e selecione **Application Insights**.

1. Localize a seção **Instrumentar seu aplicativo** e selecione **.NET Core**.

1. Selecione **Recomendado** na seção **Nível de coleção**.

1. Selecione **Aplicar** e confirme as alterações.

1. No menu de navegação à esquerda, selecione **Visão geral**.

## Criar e implantar um aplicativo Blazor

Nesta seção do exercício, você criará um aplicativo Blazor no Cloud Shell e o implantará no aplicativo Web que você criou. Todas as etapas nesta seção são executadas no Cloud Shell.

1. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um novo Cloud Shell no portal do Azure, selecionando um ambiente do ***Bash***. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure. Se você for solicitado a selecionar uma conta de armazenamento na qual manter seus arquivos, selecione **Nenhuma conta de armazenamento necessária**, sua assinatura e **Aplicar**.

    > **Observação**: se você já criou uma Cloud Shell que usa um ambiente *PowerShell*, troque-o pelo ***Bash***.

1. Execute os comandos a seguir para criar um diretório para o aplicativo Blazor e alterar para o diretório.

    ```
    mkdir blazor
    cd blazor
    ```

1. Execute o comando a seguir para criar um aplicativo Blazor na pasta.

    ```
    dotnet new blazor
    ```

1. Execute o comando a seguir para compilar o aplicativo para garantir que não houve problemas na criação.

    ```
    dotnet build
    ```

### Implantar o aplicativo no Serviço de Aplicativo

Para implantar o aplicativo, primeiro você precisa publicá-lo com o comando **dotnet publish** e criar um arquivo *.zip* para implantação.

1. Execute o comando a seguir para publicar o aplicativo em um diretório de *publicação*.

    ```
    dotnet publish -c Release -o ./publish
    ```

1. Execute os comandos a seguir para criar um arquivo *.zip* do aplicativo publicado. O arquivo *.zip* será localizado no diretório raiz do aplicativo.

    ```
    cd publish
    zip -r ../app.zip .
    cd ..
    ```

1. Execute o comando a seguir para implantar o aplicativo no Serviço de Aplicativo. Substitua **YOUR-WEB-APP-NAME** e **YOUR-RESOURCE-GROUP** pelos valores usados ao criar os recursos do Serviço de Aplicativo antes no exercício.

    ```
    az webapp deploy --name YOUR-WEB-APP-NAME \
        --resource-group YOUR-RESOURCE-GROUP \
        --src-path ./app.zip
    ```

1. Quando a implantação for concluída, selecione o link no campo de **Domínio padrão** na seção **Essentials** para abrir o aplicativo em uma nova guia no navegador.

Agora é hora de exibir algumas métricas básicas do aplicativo Application Insights. Não feche esta guia, você a usará no restante do exercício.

## Exibir métricas no Application Insights

Retorne a guia com o Portal do Azure e navegue até o Application Insights que você criou antes. A guia **Visão geral** exibe alguns gráficos básicos:

* Solicitações com falhas
* Tempo de resposta do servidor
* Solicitações de servidor
* Disponibilidade

Nesta seção, você executará algumas ações no aplicativo Web e retornará a esta página para ver a atividade. O relatório de atividade está atrasado, portanto, pode levar alguns minutos para que apareça nos gráficos.

Execute as etapas a seguir no aplicativo Web.

1. Navegue entre as opções de navegação **Página Inicial**, **+ Contador** e **Clima** no menu do aplicativo Web.

1. Atualize a página da Web várias vezes para gerar o **Tempo de resposta do servidor** e os dados de **Solicitações do servidor**.

1. Para criar alguns erros, selecione o botão **Página inicial** e acrescente a URL com **/failures**. Essa rota não existe no aplicativo Web e gerará um erro. Atualize a página várias vezes para gerar dados de erro.

1. Retorne à guia em que o Application Insights está em execução e aguarde um ou dois minutos para que as informações apareçam nos gráficos. 

1. Na navegação à esquerda, expanda a seção **Investigar** e selecione **Falhas**. Ele exibe a contagem de solicitações com falha junto com informações mais detalhadas sobre os códigos de resposta para as falhas.

Explore outras opções de relatório para ter uma ideia de quais outros tipos de informações estão disponíveis. 

## Limpar os recursos

Agora que você concluiu o exercício, exclua os recursos de nuvem que criou para evitar uso desnecessário de recursos.

1. Navegue até o grupo de recursos que você criou e exiba o conteúdo dos recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.

> **CUIDADO:** excluir o grupo de recursos excluirá todos os recursos que ele contém. Se você escolher um grupo de recursos para este exercício, todos os recursos fora do escopo deste exercício também serão excluídos.
