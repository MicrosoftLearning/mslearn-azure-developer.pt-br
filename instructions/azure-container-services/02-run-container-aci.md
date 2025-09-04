---
lab:
  topic: Azure container services
  title: Implantar um contêiner para Instâncias de Contêiner do Azure usando comandos da CLI do Azure
  description: Saiba como usar comandos da CLI do Azure para implantar um contêiner em Instâncias de Contêiner do Azure.
---

# Implantar um contêiner para Instâncias de Contêiner do Azure usando comandos da CLI do Azure

Neste exercício, você implantará e executará um contêiner em ACI (Instâncias de Contêiner do Azure) usando a CLI do Azure. Você aprenderá a criar um grupo de contêineres, especificar configurações de contêiner e verificar se o aplicativo em contêineres está em execução na nuvem.

Tarefas realizadas neste exercício:

* Criar recursos da Instância de Contêiner do Azure no Azure
* Criar e implantar um contêiner
* Verificar se o contêiner está em execução

Este exercício levará aproximadamente **15** minutos para ser concluído.

## Criar um grupo de recursos

1. No navegador, vá par o portal do Azure [https://portal.azure.com](https://portal.azure.com). Faça login com suas credenciais do Azure se for solicitado.

1. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um novo Cloud Shell no portal do Azure, selecionando um ambiente do ***Bash***. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure. Se você for solicitado a selecionar uma conta de armazenamento na qual manter seus arquivos, selecione **Nenhuma conta de armazenamento necessária**, sua assinatura e **Aplicar**.

    > **Observação**: se você já criou uma Cloud Shell que usa um ambiente *PowerShell*, troque-o pelo ***Bash***.

1. Crie um grupo de recursos para os recursos que este exercício requer. Substitua **myResourceGroup** por um nome que você quer usar para o grupo de recursos. Você pode substituir **eastus** por uma região perto de você, se necessário. Se você já tiver um grupo de recursos que deseja usar, vá para a próxima etapa.

    ```
    az group create --location eastus --name myResourceGroup
    ```

## Criar e implantar um contêiner

Crie um contêiner fornecendo um nome, uma imagem do Docker e um grupo de recursos do Azure para o comando **az container create**. Você exporá o contêiner à Internet especificando um rótulo de nome DNS.

1. Execute o comando a seguir para criar um nome DNS usado para expor seu contêiner à Internet. Seu nome DNS deve ser exclusivo, execute este comando do Cloud Shell para criar uma variável contendo um nome exclusivo.

    ```bash
    DNS_NAME_LABEL=aci-example-$RANDOM
    ```

1. Execute o comando a seguir para criar uma instância de contêiner. Substitua **myResourceGroup** e **myLocation** pelos valores usados antes. Leva alguns minutos para a operação ser concluída.

    ```bash
    az container create --resource-group myResourceGroup \
        --name mycontainer \
        --image mcr.microsoft.com/azuredocs/aci-helloworld \
        --ports 80 \
        --dns-name-label $DNS_NAME_LABEL --location myLocation \
        --os-type Linux \
        --cpu 1 \
        --memory 1.5 
    ```

    No comando anterior, **$DNS_NAME_LABEL** especifica seu nome DNS. O nome da imagem, **mcr.microsoft.com/azuredocs/aci-helloworld**, refere-se a uma imagem do Docker que executa um aplicativo Node.js Web básico.

Vá para a próxima seção após a conclusão do comando **az container create**.

## Verificar se o contêiner está em execução

Você pode verificar o status de compilação dos contêineres com o comando **az container show**. 

1. Execute o comando a seguir para verificar o status de provisionamento do contêiner que você criou. Substitua **myResourceGroup** pelo valor usado antes.

    ```bash
    az container show --resource-group myResourceGroup \
        --name mycontainer \
        --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" \
        --out table 
    ```

    Você verá o FQDN (nome de domínio totalmente qualificado) do contêiner e o estado de provisionamento dele. Veja um exemplo.

    ```
    FQDN                                    ProvisioningState
    --------------------------------------  -------------------
    aci-wt.eastus.azurecontainer.io         Succeeded
    ```

    > **Observação:** Se o contêiner estiver com o estado **Criando**, aguarde alguns minutos e execute o comando novamente até ver o estado **Com êxito**.

1. Use um navegador para acessar o FQDN do contêiner a fim de vê-lo em execução. Você pode receber um aviso de que o site não é seguro.

## Limpar os recursos

Agora que você concluiu o exercício, exclua os recursos de nuvem que criou para evitar uso desnecessário de recursos.

1. Navegue até o grupo de recursos que você criou e exiba o conteúdo dos recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.

> **CUIDADO:** excluir o grupo de recursos excluirá todos os recursos que ele contém. Se você escolher um grupo de recursos para este exercício, todos os recursos fora do escopo deste exercício também serão excluídos.
