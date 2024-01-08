---
lab:
  title: Criar um relatório em tempo real com o Azure Stream Analytics e o Microsoft Power BI
  ilt-use: Suggested demo
---

# Criar um relatório em tempo real com o Azure Stream Analytics e o Microsoft Power BI

As soluções de análise de dados geralmente incluem um requisito para ingerir e processar *fluxos* de dados. O processamento de fluxo difere do processamento em lote porque os fluxos são geralmente *ilimitados* – em outras palavras, eles são fontes contínuas de dados que devem ser processados perpetuamente, em vez de em intervalos fixos.

O Azure Stream Analytics fornece um serviço de nuvem que você pode usar para definir uma *consulta* que opera em um fluxo de dados de uma fonte de streaming, como Hubs de Eventos do Azure ou um Hub IoT do Azure. Você pode usar uma consulta do Azure Stream Analytics para processar um fluxo de dados e enviar os resultados diretamente para o Microsoft Power BI para visualização em tempo real.

Neste exercício, você usará o Azure Stream Analytics para processar um fluxo de dados de ordem de venda, da maneira como pode ser gerado a partir de um aplicativo de varejo online. Os dados do pedido serão enviados para os Hubs de Eventos do Azure, de onde seu trabalho do Azure Stream Analytics lerá e resumirá os dados antes de enviá-los ao Power BI, onde você visualizará os dados em um relatório.

Este exercício deve levar aproximadamente **45** minutos para ser concluído.

## Antes de começar

É necessário ter uma [assinatura do Azure](https://azure.microsoft.com/free) com acesso de nível administrativo.

Você também precisará ter acesso ao serviço do Microsoft Power BI. Sua escola ou sua organização já pode fornecer isso ou você pode [se inscrever no serviço do Power BI como um indivíduo](https://learn.microsoft.com/power-bi/fundamentals/service-self-service-signup-for-power-bi).

## Provisionar recursos do Azure

Neste exercício, você precisará de um workspace do Azure Synapse Analytics com acesso ao Data Lake Storage e um pool de SQL dedicado. Você também precisará de um namespace dos Hubs de Eventos do Azure para o qual os dados da ordem de streaming podem ser enviados.

Você usará uma combinação de um script do PowerShell e um modelo ARM para provisionar esses recursos.

1. Entre no [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`.
2. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure, selecionando um ambiente ***PowerShell*** e criando armazenamento caso solicitado. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure, conforme mostrado aqui:

    ![Uma captura de tela do portal do Azure com um painel do Cloud Shell.](./images/cloud-shell.png)

    > **Observação**: se você tiver criado anteriormente um Cloud Shell que usa um ambiente *Bash*, use o menu suspenso no canto superior esquerdo do painel do Cloud Shell para alterá-lo para ***PowerShell***.

3. Observe que você pode redimensionar o Cloud Shell arrastando a barra do separador na parte superior do painel ou usando os ícones **&#8212;** , **&#9723;** e **X** no canto superior direito do painel para minimizar, maximizar e fechar o painel. Para obter mais informações de como usar o Azure Cloud Shell, confira a [documentação do Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. No painel do PowerShell, insira os seguintes comandos para clonar o repositório que contém este exercício:

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Depois que o repositório tiver sido clonado, digite os seguintes comandos para alterar para a pasta deste exercício e execute o script **setup.ps1** que ele contém:

    ```
    cd dp-203/Allfiles/labs/19
    ./setup.ps1
    ```

6. Se solicitado, escolha qual assinatura você deseja usar (isso só acontecerá se você tiver acesso a várias assinaturas do Azure).

7. Enquanto aguarda a conclusão do script, continue com a próxima tarefa.

## Criar um workspace do Power BI

No serviço do Power BI, você organiza conjuntos de dados, relatórios e outros recursos em *workspaces*. Cada usuário do Power BI tem um workspace padrão chamado **Meu Workspace**, que você pode usar neste exercício, mas geralmente é uma boa prática criar um workspace para cada uma das soluções de relatório que você deseja gerenciar.

1. Entre no serviço do Power BI em [https://app.powerbi.com/](https://app.powerbi.com/) usando suas credenciais de serviço do Power BI.
2. Na barra de menus à esquerda, selecione **Workspaces** (o ícone é semelhante a &#128455;).
3. Crie um novo workspace com um nome significativo (por exemplo, *mslearn-streaming*), selecionando o modo de licenciamento **Pro**.

    > **Observação**: se você estiver usando uma conta de avaliação, talvez seja necessário habilitar recursos de avaliação adicionais.

4. Ao exibir seu workspace, observe seu GUID (identificador global exclusivo) na URL da página (que deve ser semelhante a `https://app.powerbi.com/groups/<GUID>/list`). Você precisará desse GUID posteriormente.

## Uso do Azure Stream Analytics para processar dados de streaming

Um trabalho do Azure Stream Analytics define uma consulta perpétua que opera no streaming de dados de uma ou mais entradas e envia os resultados para uma ou mais saídas.

### Criar um trabalho de Stream Analytics

1. Volte para a guia do navegador que contém o portal do Azure e, quando o script terminar, observe a região onde seu grupo de recursos **dp203-*xxxxxxx*** foi provisionado.
2. Na **Página Inicial** do portal do Azure, selecione **+ Criar um recurso** e pesquise por `Stream Analytics job`. Em seguida, crie um **trabalho do Stream Analytics** com as seguintes propriedades:
    - **Assinatura:** sua assinatura do Azure
    - **Grupo de recursos**: selecione o grupo de recursos existente **dp203-*xxxxxxx***.
    - **Nome**: `stream-orders`
    - **Região**: selecione a região onde seu workspace do Synapse Analytics está provisionado.
    - **Ambiente de hospedagem**: nuvem
    - **Unidades de streaming**: 1
3. Aguarde a conclusão da implantação e acesse o recurso de trabalho do Stream Analytics implantado.

### Crie uma entrada para o fluxo de dados de eventos

1. Na página de visão geral de **ordens de fluxo**, selecione a página **Entradas** e use o menu **Adicionar entrada** para adicionar uma entrada do  **Hub de Eventos** com as seguintes propriedades:
    - **Alias de entrada**:`orders`
    - **Selecione o Hub de Eventos em suas assinaturas**: selecionado
    - **Assinatura:** sua assinatura do Azure
    - **Namespace do Hub de Eventos**: selecione o namespace **events*xxxxxxx*** dos Hubs de Eventos
    - **Nome do Hub de Eventos**: selecione o hub de eventos **eventhub*xxxxxxx*** existente.
    - **Grupo de consumidores do Hub de Eventos**: selecione o grupo de consumidores **$Default ** existente
    - **Modo de autenticação**: Criar identidade gerenciada atribuída pelo sistema
    - **Chave de partição**: *Deixe em branco*
    - **Formato de serialização do evento**: JSON
    - **Codificação**: UTF-8
2. Salve a entrada e aguarde enquanto ela é criada. Você verá várias notificações. Aguarde uma notificação de **teste de conexão bem-sucedida**.

### Criar uma saída para o Workspace do Power BI

1. Exiba a página **Saídas** do trabalho do Stream Analytics de **ordens de fluxo**. Em seguida, use o menu **Adicionar saída** para adicionar uma saída do **Power BI** com as seguintes propriedades:
    - **Alias de saída**: `powerbi-dataset`
    - **Selecionar configurações do Power BI manualmente**: selecionado
    - **Espaço de trabalho de grupo**: *o GUID do seu espaço de trabalho*
    - **Modo de autenticação**: *selecione***Token de usuário***e use o botão ***Autorizar***na parte inferior para entrar em sua conta do Power BI*
    - **Nome do conjunto de dados**: `realtime-data`
    - **Nome da tabela**: `orders`

2. Salve a saída e aguarde enquanto ela é criada. Você verá várias notificações. Aguarde uma notificação de **teste de conexão bem-sucedida**.

### Crie uma consulta para resumir o fluxo de eventos

1. Exiba a página de **Consulta** para o trabalho do Stream Analytics de **ordens de fluxo**.
2. Modifique a consulta padrão conforme o seguinte:

    ```
    SELECT
        DateAdd(second,-5,System.TimeStamp) AS StartTime,
        System.TimeStamp AS EndTime,
        ProductID,
        SUM(Quantity) AS Orders
    INTO
        [powerbi-dataset]
    FROM
        [orders] TIMESTAMP BY EventEnqueuedUtcTime
    GROUP BY ProductID, TumblingWindow(second, 5)
    HAVING COUNT(*) > 1
    ```

    Observe que essa consulta usa o **System.Timestamp** (com base no campo **EventEnqueuedUtcTime**) para definir o início e o fim de cada janela *em cascata* de 5 segundos (sequencial não sobreposta) na qual a quantidade total para cada ID de produto é calculada.

3. Salvar a consulta.

### Execute o trabalho de streaming para processar dados de pedidos

1. Exiba a página **Visão geral** do trabalho do Stream Analytics de **ordens de fluxo** e, na guia **Propriedades**, examine **Entradas**, **Consulta**, **Saídas** e **Funções** do trabalho. Se o número de **Entradas** e de **Saídas** for 0, use o botão **↻ Atualizar** na página **Visão geral** para exibir a entrada de **pedidos** e saída de **conjunto de dados powerbi**.
2. Selecione o botão **▷ Iniciar** e inicie o trabalho de streaming agora. Aguarde uma notificação de que o trabalho de streaming foi iniciado com êxito.
3. Reabra o painel do Cloud Shell e execute o seguinte comando para enviar 100 pedidos.

    ```
    node ~/dp-203/Allfiles/labs/19/orderclient
    ```

4. Enquanto o aplicativo cliente de pedido estiver em execução, alterne para a guia do navegador do aplicativo Power BI e exiba seu espaço de trabalho.
5. Atualize a página do aplicativo Power BI até ver o conjunto de **dados em tempo real** em seu espaço de trabalho. Esse conjunto de dados é gerado pelo trabalho do Azure Stream Analytics.

## Visualize os dados de streaming no Power BI

Agora que você tem um conjunto de dados para os dados de pedidos de streaming, já pode criar um painel do Power BI que os represente visualmente.

1. Retorne à guia do navegador do PowerBI.

2. No menu suspenso **+ Novo** para seu espaço de trabalho, selecione **Painel** e crie um novo painel chamado **Acompanhamento de Pedidos**.

3. No painel **Acompanhamento de Pedidos**, selecione o menu **✏️ Editar** e, em seguida, selecione **+ Adicionar um bloco**. Em seguida, no painel **Adicionar um bloco**, selecione **Dados de Streaming Personalizados** e selecione **Avançar**:

4. No painel **Adicionar um bloco de dados de streaming personalizado**, em **Seus conjuntos de dados**, selecione o conjunto de dados **em tempo real** e, em seguida, selecione **Avançar**.

5. Altere o tipo de visualização padrão para **Gráfico de linhas**. Em seguida, configure as seguintes propriedades e selecione **Avançar**:
    - **Eixo**: EndTime
    - **Valor**: pedidos
    - **Janela de tempo para exibir**: 1 Minuto

6. No painel **Detalhes do bloco**, defina o **Título** como **Contagem de Pedidos em Tempo Real** e selecione **Aplicar**.

7. Volte para a guia do navegador que contém o portal do Azure e, se necessário, reabra o painel do cloud shell. Em seguida, execute novamente o seguinte comando para enviar outros 100 pedidos.

    ```
    node ~/dp-203/Allfiles/labs/19/orderclient
    ```

8. Enquanto o script de envio de pedidos estiver em execução, volte para a guia do navegador que contém o painel do Power BI **Acompanhamento de Pedidos** e observe que a visualização é atualizada para refletir os novos dados de pedido à medida que são processados pelo trabalho do Stream Analytics (que ainda deve estar em execução).

    ![Uma captura de tela de um relatório do Power BI mostrando um fluxo em tempo real de dados de pedidos.](./images/powerbi-line-chart.png)

    Você pode executar novamente o script **orderclient** e observar os dados que estão sendo capturados no painel em tempo real.

## Excluir recursos

Se você terminou de explorar o Azure Synapse Analytics e o Power BI, exclua os recursos que criou para evitar custos desnecessários do Azure.

1. Feche a guia do navegador que contém o relatório do Power BI. Em seguida, no painel **Workspaces**, no menu **⋮** do seu workspace, selecione **Configurações do Workspace** e exclua o workspace.
2. Retorne à guia do navegador que contém o Portal do Azure, feche o painel do cloud shell e use o botão **&#128454; Parar** para interromper o trabalho do Stream Analytics. Aguarde uma notificação de que o trabalho do Stream Analytics foi interrompido com êxito.
3. No portal do Azure, na **Página Inicial**, selecione **Grupos de recursos**.
4. Selecione o grupo de recursos **dp203-*xxxxxxx*** que contém seus recursos do Hub de Eventos do Azure e do Stream Analytics.
5. Na parte superior da página de **Visão Geral** do grupo de recursos, selecione **Excluir o grupo de recursos**.
6. Digite o nome do grupo de recursos **dp203-*xxxxxxx*** para confirmar que deseja excluí-lo e selecione **Excluir**.

    Após alguns minutos, os recursos criados neste exercício serão excluídos.
