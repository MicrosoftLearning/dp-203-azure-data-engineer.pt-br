---
lab:
  title: Transformar dados usando o Spark no Synapse Analytics
  ilt-use: Lab
---

# Transformar dados usando o Spark no Synapse Analytics

Os *engenheiros* de dados geralmente usam os notebooks do Spark como uma de suas ferramentas preferidas para executar atividades de *ETL (extração, transformação e carregamento)* ou *ETL (extração, carga e transformação)* que transformam dados de um formato ou estrutura para outro.

Neste exercício, você usará um notebook do Spark no Azure Synapse Analytics para transformar dados em arquivos.

Este exercício deve levar aproximadamente **30** minutos para ser concluído.

## Antes de começar

É necessário ter uma [assinatura do Azure](https://azure.microsoft.com/free) com acesso de nível administrativo.

## Provisionar um workspace do Azure Synapse Analytics

Você precisará de um workspace do Azure Synapse Analytics com acesso ao Data Lake Storage e um pool do Spark.

Neste exercício, você usará uma combinação de um script do PowerShell e um modelo ARM para provisionar um workspace do Azure Synapse Analytics.

1. Entre no [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`.
2. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um novo Cloud Shell no portal do Azure, selecionando um ambiente ***PowerShell*** e criando um armazenamento caso solicitado. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure, conforme mostrado aqui:

    ![Portal do Azure com um painel do Cloud Shell](./images/cloud-shell.png)

    > **Observação**: se você tiver criado anteriormente um Cloud Shell que usa um ambiente *Bash*, use o menu suspenso no canto superior esquerdo do painel do Cloud Shell para alterá-la para ***PowerShell***.

3. Observe que você pode redimensionar o Cloud Shell arrastando a barra do separador na parte superior do painel ou usando os ícones **&#8212;** , **&#9723;** e **X** no canto superior direito do painel para minimizar, maximizar e fechar o painel. Para obter mais informações de como usar o Azure Cloud Shell, confira a [documentação do Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. No painel do PowerShell, insira os seguintes comandos para clonar este repositório:

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Depois que o repositório tiver sido clonado, digite os seguintes comandos para alterar para a pasta deste exercício e execute o script **setup.ps1** que ele contém:

    ```
    cd dp-203/Allfiles/labs/06
    ./setup.ps1
    ```

6. Se solicitado, escolha qual assinatura você deseja usar (isso só acontecerá se você tiver acesso a várias assinaturas do Azure).
7. Quando solicitado, insira uma senha adequada a ser definida para o pool do SQL do Azure Synapse.

    > **Observação**: lembre-se desta senha!

8. Aguarde a conclusão do script – isso normalmente leva cerca de 10 minutos, mas em alguns casos pode levar mais tempo. Enquanto você está esperando, revise o artigo [Conceitos principais do Apache Spark no Azure Synapse Analytics](https://learn.microsoft.com/azure/synapse-analytics/spark/apache-spark-concepts) na documentação do Azure Synapse Analytics.

## Usar um notebook do Spark para transformar dados

1. Depois que o script de implantação for concluído, no portal do Azure, vá para o grupo de recursos **dp203-*xxxxxxx*** que ele criou e observe que esse grupo de recursos contém seu workspace Synapse, uma conta de Armazenamento para seu Data Lake e um pool do Apache Spark.
2. Selecione seu workspace do Synapse e, em sua página **Visão geral**, no cartão **Open Synapse Studio**, selecione **Abrir** para abrir o Synapse Studio em uma nova guia do navegador, fazendo login se solicitado.
3. No lado esquerdo do Synapse Studio, use o ícone **&rsaquo;&rsaquo;** para expandir o menu. Isso revela as diferentes páginas do Synapse Studio usadas para gerenciar recursos e executar tarefas de análise de dados.
4. Na página **Gerenciar**, selecione a guia **Pools do Apache Spark** e observe que um pool do Spark com um nome semelhante a **spark*xxxxxxx*** foi provisionado no workspace.
5. Na página **Dados**, exiba a guia **Vinculado** e verifique se seu workspace inclui um link para sua conta de armazenamento do Azure Data Lake Storage Gen2, que deve ter um nome semelhante a **synapse*xxxxxxx* (Primary - datalake*xxxxxxx*)**.
6. Expanda sua conta de armazenamento e verifique se ela contém um contêiner do sistema de arquivos chamado **arquivos (Principal)**.
7. Selecione o contêiner **arquivos** e observe que ele contém pastas chamadas **dados** e **synapse**. A pasta synapse é usada pelo Azure Synapse e a pasta **dados** contém os arquivos de dados que você vai consultar.
8. Abra a pasta **dados** e observe que ela contém arquivos .csv referentes a três anos de dados de vendas.
9. Clique com o botão direito do mouse em qualquer um dos arquivos e selecione **Visualizar** para ver os dados que ele contém. Observe que os arquivos contêm uma linha de cabeçalho; você pode selecionar a opção para exibir cabeçalhos de coluna.
10. Feche a versão preliminar. Em seguida, baixe o **Spark Transform.ipynb** de [https://raw.githubusercontent.com/MicrosoftLearning/dp-203-azure-data-engineer/master/Allfiles/labs/06/notebooks/Spark%20Transform.ipynb](https://raw.githubusercontent.com/MicrosoftLearning/dp-203-azure-data-engineer/master/Allfiles/labs/06/notebooks/Spark%20Transform.ipynb)

    > **Observação**: é melhor copiar este texto usando ***ctrl + a*** seguido de ***ctrl + c*** e colar em uma ferramenta (como o bloco de notas ) usando ***ctrl + v***; em seguida, usando arquivo, salvar como **Spark Transform.ipynb** com tipo de arquivo definido como ***todos os arquivos***.

11. Em seguida, na página **Desenvolver**, expanda **Notebooks**, clique em + Importar opções

    ![Importação do Notebook do Spark](./image/../images/spark-notebook-import.png)
        
12. Selecione o arquivo que você acabou de baixar e salvar como **Spark Transfrom.ipynb**.
13. Conecte o notebook ao seu pool do Spark **spark*xxxxxxx***.
14. Revise as anotações no notebook e execute as células de código.

    > **Observação**: a primeira célula de código levará alguns minutos para ser executada porque o pool do Spark deve ser iniciado. As células subsequentes serão executadas mais rapidamente.

## Excluir recursos do Azure

Se você terminou de explorar Azure Synapse Analytics, exclua os recursos que criou para evitar custos desnecessários do Azure.

1. Feche a guia do navegador do Synapse Studio e retorne ao portal do Azure.
2. No portal do Azure, na **Página Inicial**, selecione **Grupos de recursos**.
3. Selecione o grupo de recursos **dp203-*xxxxxxx*** para o seu workspace do Synapse Analytics (não o grupo de recursos gerenciado) e verifique se ele contém o workspace do Synapse, a conta de armazenamento e o pool do Spark para seu workspace.
4. Na parte superior da página de **Visão Geral** do grupo de recursos, selecione **Excluir o grupo de recursos**.
5. Digite o nome do grupo de recursos **dp203-*xxxxxxx*** para confirmar que deseja excluí-lo e selecione **Excluir**.

    Após alguns minutos, seu grupo de recursos do workspace do Azure Synapse e o grupo de recursos do workspace gerenciado associado a ele serão excluídos.
