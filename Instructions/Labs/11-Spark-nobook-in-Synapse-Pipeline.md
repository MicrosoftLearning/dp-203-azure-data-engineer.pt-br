---
lab:
  title: Use um notebook Apache Spark em um pipeline
  ilt-use: Lab
---

# Use um notebook Apache Spark em um pipeline

Neste exercício, criaremos um pipeline do Azure Synapse Analytics que inclui uma atividade para executar um notebook Apache Spark.

Este exercício deve levar aproximadamente **30** minutos para ser concluído.

## Antes de começar

É necessário ter uma [assinatura do Azure](https://azure.microsoft.com/free) com acesso de nível administrativo.

## Provisionar um workspace do Azure Synapse Analytics

Você precisará de um espaço de trabalho do Azure Synapse Analytics com acesso ao armazenamento de data lake e a um pool Spark.

Neste exercício, você usará uma combinação de um script do PowerShell e um modelo ARM para provisionar um espaço de trabalho do Azure Synapse Analytics.

1. Entre no [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`.
2. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um novo Cloud Shell no portal do Azure, selecionando um ***PowerShell*** ambiente e criação de armazenamento, se solicitado. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure, conforme mostrado aqui:

    ![Portal do Azure com um painel do Cloud Shell](./images/cloud-shell.png)

    > **Observação**: se você já criou um Cloud Shell que usa um ambiente *Bash*, use o menu suspenso no canto superior esquerdo do painel do Cloud Shell para alterá-lo para ***PowerShell***.

3. Observe que o Cloud Shell pode ser redimensionado arrastando a barra separadora na parte superior do painel ou usando os ícones—, **◻** e **X** no canto superior direito do painel tpanenimize, maximize e feche o painel. Para obter mais informações de como usar o Azure Cloud Shell, confira a [documentação do Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. No painel PowerShell, insira os seguintes comandos para clonar este repositório:

    ```powershell
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Após a clonagem do repositório, insira os seguintes comandos para mudar para a pasta deste exercício e execute o script **setup.ps1** que ele contém:

    ```powershell
    cd dp-203/Allfiles/labs/11
    ./setup.ps1
    ```
    
6. Se solicitado, escolha qual assinatura você deseja usar (isso só acontecerá se você tiver acesso a várias assinaturas do Azure).
7. Quando solicitado, insira uma senha adequada a ser definida para seu pool SQL do Azure Synapse.

    > **Observação**: Certifique-se de memorizar essa senha!

8. Aguarde a conclusão do script. Isso normalmente leva cerca de 10 minutos, mas em alguns casos pode demorar mais. Enquanto espera, revise o artigo [Azure Synapse Pipelines](https://learn.microsoft.com/en-us/azure/data-factory/concepts-data-flow-performance-pipelines) na documentação do Azure Synapse Analytics.

## Executar um bloco de anotações do Spark interativamente

Antes de automatizar um processo de transformação de dados com um notebook, pode ser útil executar o notebook de forma interativa para entender melhor o processo que você automatizará posteriormente.

1. Após a conclusão do script, no portal do Azure, vá para o grupo de recursos dp203-xxxxxxx que ele criou e selecione seu Workspace do Synapse.
2. Na página **Visão Geral** do seu Synapse Workspace, no cartão **Abrir o Synapse Studio**, selecione **Abrir** para abrir o Synapse Studio em uma nova guia do navegador; faça login se solicitado.
3. No lado esquerdo do Synapse Studio, use o ícone ›› para expandir o menu - isso revela as diferentes páginas do Synapse Studio.
4. Na página **Dados**, visualize a guia Vinculado e verifique se seu espaço de trabalho inclui um link para sua conta de armazenamento do Azure Data Lake Storage Gen2, que deve ter um nome semelhante a **synapsexxxxxx (Primário - datalakexxxxxxx) **.
5. Expanda sua conta de armazenamento e verifique se ela contém um contêiner do sistema de arquivos chamado **arquivos (primário)**.
6. Selecione o contêiner de arquivos e observe que ele contém uma pasta chamada **dados**, que contém os arquivos de dados que você irá transformar.
7. Abra a pasta **dados**** e visualize os arquivos CSV que ela contém. Clique com o botão direito em qualquer um dos arquivos e selecione **Visualizar** para ver uma amostra dos dados. Feche a visualização quando terminar.
8. No Synapse Studio, na página **Desenvolver**, expanda **Notebooks** e abra o notebook **Spark Transform**.
9. Revise o código que o notebook contém, observando que ele:
    - Define uma variável para definir um nome de pasta exclusivo.
    - Carrega os dados do pedido de vendas CSV da pasta **/dados**.
    - Transforma os dados ao distribuir o nome do cliente em vários campos.
    - Salva os dados transformados no formato Parquet na pasta com nome exclusivo.
10. Na barra de ferramentas do notebook, anexe o notebook ao seu pool do **spark*xxxxxxx*** Spark e, em seguida, use o botão **▷ Executar Tudo** para executar todas as células de código no notebook.

    > **Observação**: Se você constatar que o notebook não foi carregado durante o script de execução, deverá baixá-lo do GitHub [Allfiles/labs/11/notebooks](https://github.com/MicrosoftLearning/dp-203-azure-data-engineer/tree/master/Allfiles/labs/11/notebooks) O arquivo chamado Spark Transform.ipynb e carregá-lo no Synapse.
    
    A sessão do Spark pode levar alguns minutos para iniciar antes que as células de código possam ser executadas.

11. Após a execução de todas as células do notebook, anote o nome da pasta na qual os dados transformados foram salvos.
12. Alterne para a guia **arquivos** (que ainda deve estar aberta) e visualize a pasta raiz **arquivos**. Se necessário, no menu **Mais**, selecione **Atualizar** para ver a nova pasta. Em seguida, abra-o para verificar se ele contém arquivos Parquet.
13. Retorne à pasta raiz **arquivos**, selecione a pasta com nome exclusivo gerada pelo notebook e no menu **Novo Script SQL**, selecione **Selecionar as 100 melhores linhas**.
14. No painel **Selecionar as 100 melhores linhas**, defina o tipo de arquivo como **Formato Parquet** e aplique a alteração.
15. No novo painel SQL Script que é aberto, use o botão **▷ Executar** para executar o código thrunQL e verificar se ele retorna os dados de vendas transformados.

## Executar o bloco de anotações em um pipeline

Agora que você entende o processo de transformação, está pronto para automatizá-lo, encapsulando o notebook em um pipeline.

### Criar uma célula de parâmetro

1. No Synapse Studio, retorne à guia **Spark Transform** que contém o bloco de notas e, na barra de ferramentas, no menu **...** na extremidade direita, selecione **Limpar saída**.
2. Selecione a primeira célula de código (que contém o código para definir a variável **folderName**).
3. Na barra de ferramentas pop-up no canto superior direito da célula de código, no menu **...**, selecione **\[@] Alternar célula de parâmetro**. Verifique se a palavra **parâmetros** aparece na parte inferior direita da célula.
4. Na barra de ferramentas, use o botão **Publicar** para salvar as alterações.

### Criar um pipeline

1. No Synapse Studio, selecione a página **Integrar**. Em seguida, no menu **+** selecione **Pipeline** para criar um novo pipeline.
2. No painel **Propriedades** do seu novo pipeline, altere o nome dele de **Pipeline1** para **Transformar Dados de Vendas**. Em seguida, use o botão **Propriedades** acima do painel **Propriedades** para ocultá-lo.
3. No painel **Atividades**, expanda **Synapse**; e arraste uma atividade **Notebook** para a superfície de design do pipeline, conforme mostrado aqui:

    ![Captura de tela de um pipeline com uma atividade do Notebook.](images/notebook-pipeline.png)

4. Na guia **Geral** da atividade Notebook, altere seu nome para **Executar Spark Transform**.
5. Na guia **Configurações** da atividade Notebook, defina as seguintes propriedades:
    - **Notebook**: Selecione o bloco de notas **Spark Transform**.
    - **Parâmetros base**: Expanda esta seção e defina um parâmetro com as seguintes configurações:
        - **Nome**: folderName
        - **Tipo:** string
        - **Valor**: Selecione **Adicionar conteúdo dinâmico** e defina o valor do parâmetro como a variável de sistema *ID de execução do pipeline* (`@pipeline().RunId`)
    - **Spark pool**: selecione o pool **spark*xxxxxxx***.
    - **Tamanho do executor**: Selecione **Small (4 vCores, 28GB Memory)**.

    Seu painel de pipeline deve ser semelhante a este:

    ![Captura de tela de um pipeline contendo uma atividade do Notebook com configurações.](images/notebook-pipeline-settings.png)

### Publicar e executar um pipeline

1. Use o botão **Publicar tudo** para publicar o pipeline (e quaisquer outros ativos não salvos).
2. Na parte superior do painel do designer de pipeline, no menu **Adicionar acionador**, selecione **Acionar agora**. Em seguida, selecione **OK** para confirmar que deseja executar o pipeline.

    **Observação**: Você também pode criar um acionador para executar o pipeline em um horário agendado ou em resposta a um evento específico.

3. Quando o pipeline começar a ser executado, na página **Monitor**, visualize a guia **Execução de pipeline** e revise o status do pipeline **Transformar Dados de Vendas**.
4. Selecione o pipeline **Transformar Dados de Vendas** para visualizar seus detalhes e observe o ID de execução do pipeline no painel **Execuções de atividades**.

    O pipeline pode levar cinco minutos ou mais para ser concluído. Você pode usar o botão **↻ Atualizar** na barra de ferramentas para verificar seu status.

5. Quando a execução do pipeline for bem-sucedida, na página **Dados**, navegue até o contêiner de armazenamento **arquivos** e verifique se uma nova pasta nomeada para o ID de execução do pipeline foi criada e se ele contém arquivos Parquet para os dados de vendas transformados.
   
## Excluir recursos do Azure

Se você terminou de explorar Azure Synapse Analytics, exclua os recursos que criou para evitar custos desnecessários do Azure.

1. Feche a guia do navegador do Synapse Studio e retorne ao portal do Azure.
2. No portal do Azure, na **Página Inicial**, selecione **Grupos de recursos**.
3. Selecione o grupo de recursos **dp203-*xxxxxxx*** para o seu workspace do Synapse Analytics (não o grupo de recursos gerenciados) e verifique se ele contém o workspace do Synapse, a conta de armazenamento e o pool do Spark para o seu workspace.
4. Na parte superior da página de **Visão Geral** do grupo de recursos, selecione **Excluir o grupo de recursos**.
5. Insira o nome do grupo de recursos **dp203-*xxxxxxx*** para confirmar que deseja excluí-lo e selecione **Excluir**.

    Após alguns minutos, seu grupo de recursos do workspace do Azure Synapse e o grupo de recursos do workspace gerenciado associado a ele serão excluídos.
