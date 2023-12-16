---
lab:
  title: Usar o Delta Lake no Azure Databricks
  ilt-use: Optional demo
---

# Usar o Delta Lake no Azure Databricks

O Delta Lake é um projeto de código aberto para criar uma camada de armazenamento de dados transacionais para o Spark sobre um data lake. O Delta Lake adiciona suporte à semântica relacional em operações de dados em lote e streaming e permite a criação de uma arquitetura de *Lakehouse* na qual o Apache Spark pode ser usado para processar e consultar dados em tabelas baseadas em arquivos subjacentes em um data lake.

Este exercício levará aproximadamente **40** minutos para ser concluído.

## Antes de começar

É necessário ter uma [assinatura do Azure](https://azure.microsoft.com/free) com acesso de nível administrativo.

## Provisionar um workspace do Azure Databricks

Neste exercício, você usará um script para provisionar um novo workspace do Azure Databricks.

> **Dica**: se você já tiver um workspace do Azure Databricks *Padrão* ou de *Avaliação*, pule este procedimento.

1. Em um navegador da web, faça logon no [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`.
2. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure, selecionando um ambiente ***PowerShell*** e criando um armazenamento caso solicitado. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure, conforme mostrado aqui:

    ![Portal do Azure com um painel do Cloud Shell](./images/cloud-shell.png)

    > **Observação**: se você tiver criado anteriormente um Cloud Shell que usa um ambiente *Bash*, use o menu suspenso no canto superior esquerdo do painel do Cloud Shell para alterá-lo para ***PowerShell***.

3. Observe que você pode redimensionar o Cloud Shell arrastando a barra do separador na parte superior do painel ou usando os ícones **&#8212;** , **&#9723;** e **X** no canto superior direito do painel para minimizar, maximizar e fechar o painel. Para obter mais informações de como usar o Azure Cloud Shell, confira a [documentação do Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. No painel do PowerShell, insira os seguintes comandos para clonar esse repositório:

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Depois que o repositório tiver sido clonado, digite os seguintes comandos para alterar para a pasta deste laboratório e execute o script **setup.ps1** que ele contém:

    ```
    cd dp-203/Allfiles/labs/25
    ./setup.ps1
    ```

6. Se solicitado, escolha qual assinatura deseja usar (isso só acontecerá se você tiver acesso a várias assinaturas do Azure).

7. Aguarde a conclusão do script. Isso normalmente leva cerca de 5 minutos, mas em alguns casos pode demorar mais. Enquanto espera, revise o artigo [Introdução às tecnologias delta](https://learn.microsoft.com/azure/databricks/introduction/delta-comparison) na documentação do Azure Databricks.

## Criar um cluster

O Azure Databricks é uma plataforma de processamento distribuído que usa *clusters* do Apache Spark para processar dados em paralelo em vários nós. Cada cluster consiste em um nó de driver para coordenar o trabalho e nós de trabalho para executar tarefas de processamento.

> **Dica**: se você já tiver um cluster com uma versão de runtime 13.3 LTS em seu workspace do Azure Databricks, use-o para concluir este exercício e pule este procedimento.

1. No portal do Azure, navegue até o grupo de recursos **dp203-*xxxxxxx*** que foi criado pelo script (ou o grupo de recursos que contém seu workspace do Azure Databricks já existente)
1. Selecione seu recurso do Serviço do Azure Databricks (chamado **databricks*xxxxxxx*** se você usou o script de instalação para criá-lo).
1. Na página **Visão geral** do seu workspace, use o botão **Iniciar workspace** para abrir seu workspace do Azure Databricks em uma nova guia do navegador, fazendo o logon se solicitado.

    > **Dica**: ao usar o portal do workspace do Databricks, várias dicas e notificações podem ser exibidas. Dispense-as e siga as instruções fornecidas para concluir as tarefas neste exercício.

1. Veja o portal do workspace do Azure Databricks e observe que a barra lateral à esquerda contém ícones para as várias tarefas executáveis.

1. Selecione a tarefa **(+) Novo** e, em seguida, selecione **Cluster**.
1. Na página **Novo cluster**, crie um novo cluster com as seguintes configurações:
    - **Nome do cluster**: cluster do *nome do usuário* (o nome padrão do cluster)
    - **Modo de cluster**: nó único
    - **Modo de acesso**: usuário único (*com sua conta de usuário selecionada*)
    - **Versão do runtime do Databricks**: 13.3 LTS (Spark 3.4.1, Scala 2.12)
    - **Usar aceleração de Photon**: selecionado
    - **Tipo de nó**: Standard_DS3_v2
    - **Terminar após** *30* **minutos de inatividade**

1. Aguarde a criação do cluster. Isso pode levar um ou dois minutos.

> **Observação**: se o cluster não for iniciado, sua assinatura pode não ter cota suficiente na região onde seu workspace do Azure Databricks está provisionado. Veja [O limite de núcleos da CPU impede a criação do cluster](https://docs.microsoft.com/azure/databricks/kb/clusters/azure-core-limit) para obter mais detalhes. Se isso acontecer, você pode tentar excluir o workspace e criar um novo em uma região diferente. Você pode especificar uma região como um parâmetro para o script de instalação da seguinte maneira: `./setup.ps1 eastus`

## Explore o delta lake usando um notebook

Neste exercício, você usará código em um notebook para explorar o delta lake no Azure Databricks.

1. Na barra lateral à esquerda do portal do workspace do Azure Databricks para seu workspace, selecione **Workspace**. Em seguida, selecione a pasta **⌂ Página inicial**.
1. Na parte superior da página, no menu **⋮** ao lado do seu nome de usuário, selecione **Importar**. Em seguida, na caixa de diálogo **Importar**, selecione **URL** e importe o notebook de `https://github.com/MicrosoftLearning/dp-203-azure-data-engineer/raw/master/Allfiles/labs/25/Delta-Lake.ipynb`
1. Conecte o notebook ao cluster e siga as instruções nele contidas, executando as células que ele contém para explorar a funcionalidade do delta lake.

## Excluir recursos do Azure Databricks

Agora que você terminou de explorar o Delta Lake no Azure Databricks, deve excluir os recursos criados para evitar custos desnecessários do Azure e liberar capacidade em sua assinatura.

1. Feche a guia do navegador do workspace do Azure Databricks e volte para o portal do Azure.
2. No portal do Azure, na **Página Inicial**, selecione **Grupos de recursos**.
3. Selecione o grupo de recursos **dp203-*xxxxxxx*** (não o grupo de recursos gerenciados) e verifique se ele contém seu workspace do Azure Databricks.
4. Na parte superior da página de **Visão Geral** do grupo de recursos, selecione **Excluir o grupo de recursos**.
5. Digite o nome do grupo de recursos **dp203-*xxxxxxx*** para confirmar que deseja excluí-lo e selecione **Excluir**.

    Após alguns minutos, seu grupo de recursos e os grupos de recursos gerenciados do workspace associado a ele serão excluídos.
