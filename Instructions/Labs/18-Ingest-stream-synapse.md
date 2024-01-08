---
lab:
  title: Ingerir dados de tempo real com o Azure Stream Analytics e o Azure Synapse Analytics
  ilt-use: Lab
---

# Ingerir dados de tempo real com o Azure Stream Analytics e o Azure Synapse Analytics

As soluções de análise de dados geralmente incluem um requisito para ingerir e processar *fluxos* de dados. O processamento de fluxo difere do processamento em lote porque os fluxos são geralmente *ilimitados* – em outras palavras, eles são fontes contínuas de dados que devem ser processados perpetuamente, em vez de em intervalos fixos.

O Azure Stream Analytics fornece um serviço de nuvem que você pode usar para definir uma *consulta* que opera em um fluxo de dados de uma fonte de streaming, como Hubs de Eventos do Azure ou um Hub IoT do Azure. Você pode usar uma consulta do Azure Stream Analytics para ingerir o fluxo de dados diretamente em um armazenamento de dados para análise adicional ou para filtrar, agregar e resumir os dados com base em janelas temporais.

Neste exercício, você usará o Azure Stream Analytics para processar um fluxo de dados de ordem de venda, da maneira como pode ser gerado a partir de um aplicativo de varejo online. Os dados do pedido serão enviados para os Hubs de Eventos do Azure, onde seus trabalhos do Azure Stream Analytics lerão os dados e os ingerirão no Azure Synapse Analytics.

Este exercício deve levar aproximadamente **45** minutos para ser concluído.

## Antes de começar

É necessário ter uma [assinatura do Azure](https://azure.microsoft.com/free) com acesso de nível administrativo.

## Provisionar recursos do Azure

Neste exercício, você precisará de um workspace do Azure Synapse Analytics com acesso ao Data Lake Storage e um pool de SQL dedicado. Você também precisará de um namespace dos Hubs de Eventos do Azure para o qual os dados da ordem de streaming podem ser enviados.

Você usará uma combinação de um script do PowerShell e um modelo ARM para provisionar esses recursos.

1. Entre no [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`.
2. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure, selecionando um ambiente ***PowerShell*** e criando um armazenamento caso solicitado. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure, conforme mostrado aqui:

    ![Portal do Azure com um painel do Cloud Shell](./images/cloud-shell.png)

    > **Observação**: se você tiver criado anteriormente um cloud shell que usa um ambiente *Bash*, use o menu suspenso no canto superior esquerdo do painel do cloud shell para alterá-lo para ***PowerShell***.

3. Observe que você pode redimensionar o Cloud Shell arrastando a barra do separador na parte superior do painel ou usando os ícones **&#8212;** , **&#9723;** e **X** no canto superior direito do painel para minimizar, maximizar e fechar o painel. Para obter mais informações de como usar o Azure Cloud Shell, confira a [documentação do Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. No painel do PowerShell, insira os seguintes comandos para clonar o repositório que contém este exercício:

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Depois que o repositório tiver sido clonado, digite os seguintes comandos para alterar para a pasta deste exercício e execute o script **setup.ps1** que ele contém:

    ```
    cd dp-203/Allfiles/labs/18
    ./setup.ps1
    ```

6. Se solicitado, escolha qual assinatura você deseja usar (isso só acontecerá se você tiver acesso a várias assinaturas do Azure).
7. Quando solicitado, insira uma senha adequada a ser definida para seu pool de SQL do Azure Synapse.

    > **Obsrevação**: lembre-se dessa senha.

8. Aguarde a conclusão do script – isso normalmente leva cerca de 15 minutos, mas em alguns casos pode levar mais tempo. Enquanto espera, revise o artigo [Bem-vindo ao Azure Stream Analytics](https://learn.microsoft.com/azure/stream-analytics/stream-analytics-introduction) na documentação do Azure Stream Analytics.

## Ingerir dados de streaming em um pool de SQL dedicado

Vamos começar ingerindo um fluxo de dados diretamente em uma tabela em um pool de SQL dedicado do Azure Synapse Analytics.

### Exibir a fonte de streaming e a tabela de banco de dados

1. Quando o script de instalação terminar de ser executado, minimize o painel do cloud shell (você retornará a ele mais tarde). Em seguida, no portal do Azure, vá para o grupo de recursos **dp203-*xxxxxxx*** que ele criou e observe que esse grupo de recursos contém um workspace do Azure Synapse, uma conta de armazenamento para seu data lake, um pool de SQL dedicado e um namespace de Hubs de Eventos.
2. Selecione o seu workspace do Synapse e a página de **Visão Geral** dele, no cartão do **Open Synapse Studio**, selecione **Abrir** para abrir o Synapse Studio em uma nova guia do navegador. O Synapse Studio é uma interface baseada na Web que pode ser usada para trabalhar com o seu workspace do Synapse Analytics.
3. No lado esquerdo do Synapse Studio, use o ícone **&rsaquo;&rsaquo;** para expandir o menu, o que revela as diferentes páginas do Synapse Studio usadas para gerenciar recursos e executar tarefas de análise de dados.
4. Na página **Gerenciar**, na seção **pools de SQL**, selecione a linha dedicada do pool de SQL **sql*xxxxxxx*** e use seu ícone **▷** para retomá-la.
5. Enquanto aguarda o início do pool de SQL, volte para a guia do navegador que contém o portal do Azure e reabra o painel do cloud shell.
6. No painel do cloud shell, insira o seguinte comando para executar um aplicativo cliente que envia 100 ordens simuladas para os Hubs de Eventos do Azure:

    ```
    node ~/dp-203/Allfiles/labs/18/orderclient
    ```

7. Observe os dados do pedido à medida que são enviados - cada pedido consiste em uma ID do produto e uma quantidade.
8. Depois que o aplicativo cliente do pedido for concluído, minimize o painel do cloud shell e volte para a guia do navegador Synapse Studio.
9. No Synapse Studio, na página **Gerenciar**, verifique se o seu pool de SQL tem um status de **Online** e, em seguida, mude para a página de **dados** e, no painel **Workspace**, expanda **banco de dados SQL**, seu pool de SQL **sql*xxxxxxx*** e **Tabelas** para ver a tabela **dbo.FactOrder**.
10. No menu **...** para a tabela **dbo.FactOrder**, selecione **Novo script de SQL script** > **Selecionar as 100 primeiras linhas** e revise os resultados. Observe que a tabela inclui colunas para **OrderDateTime**, **ProductID** e **Quantidade**, mas atualmente não há linhas de dados.

### Criar um trabalho do Azure Stream Analytics para ingerir dados de pedidos

1. Volte para a guia do navegador que contém o portal do Azure e observe a região onde seu grupo de recursos **dp203-*xxxxxxx*** foi provisionado - você criará seu trabalho do Stream Analytics na <u>mesma região</u>.
2. Na página **Inicial**, selecione **+ Criar um recurso** e pesquise `Stream Analytics job`. Em seguida, crie um **trabalho do Stream Analytics** com as seguintes propriedades:
    - **Noções básicas**:
        - **Assinatura:** sua assinatura do Azure
        - **Grupo de recursos**: selecione o grupo de recursos existente **dp203-*xxxxxxx***.
        - **Nome**: `ingest-orders`
        - **Região**: selecione a <u>mesma região</u> em que o workspace do Synapse Analytics está provisionado.
        - **Ambiente de hospedagem**: nuvem
        - **Unidades de streaming**: 1
    - **Armazenamento**:
        - **Adicionar conta de armazenamento**: selecionado
        - **Assinatura:** sua assinatura do Azure
        - **Contas de armazenamento**: selecione a conta de armazenamento **datalake*xxxxxxx***
        - **Modo de autenticação**: cadeia de conexão
        - **Proteger dados privados na conta de armazenamento**: selecionado
    - **Marcas:**
        - *Nenhuma*
3. Aguarde a conclusão da implantação e acesse o recurso de trabalho do Stream Analytics implantado.

### Crie uma entrada para o fluxo de dados de eventos

1. Na página de visão geral **ingest-orders**, selecione a página **Entradas**. Use o menu **Adicionar entrada** para adicionar uma entrada **Hub de Eventos** com as seguintes propriedades:
    - **Alias de entrada**:`orders`
    - **Selecione o Hub de Eventos em suas assinaturas**: selecionado
    - **Assinatura:** sua assinatura do Azure
    - **Namespace do Hub de Eventos**: selecione o namespace **events*xxxxxxx*** dos Hubs de Eventos
    - **Nome do Hub de Eventos**: selecione o hub de eventos **eventhub*xxxxxxx*** existente.
    - **Grupo de consumidores do Hub de Eventos:** selecione **Usar existente** e, em seguida, selecione o grupo de consumidores **$Default**
    - **Modo de autenticação**: Criar identidade gerenciada atribuída pelo sistema
    - **Chave de partição**: *Deixe em branco*
    - **Formato de serialização do evento**: JSON
    - **Codificação**: UTF-8
2. Salve a entrada e aguarde enquanto ela é criada. Você verá várias notificações. Aguarde uma notificação de **teste de conexão bem-sucedida**.

### Criar uma saída para a tabela SQL

1. Exiba a página **Saídas** para o trabalho do Stream Analytics **ingest-orders**. Em seguida, use o menu **Adicionar saída** para adicionar uma saída do **Azure Synapse Analytics** com as seguintes propriedades:
    - **Alias de saída**: `FactOrder`
    - **Selecione Azure Synapse Analytics em suas assinaturas**: Selecionado
    - **Assinatura:** sua assinatura do Azure
    - **Banco de dados**: selecione o banco de dados **sql*xxxxxxx* (synapse*xxxxxxx *)**
    - **Modo de autenticação**: autenticação do SQL Server
    - **Nome de usuário**: SQLUser
    - **Senha**: *a senha especificada para o Pool do SQL ao executar o script de instalação*
    - **Tabela **: `FactOrder`
2. Salve a saída e aguarde enquanto ela é criada. Você verá várias notificações. Aguarde uma notificação de **teste de conexão bem-sucedida**.

### Criar uma consulta para ingerir o fluxo de eventos

1. Exiba a página **Consulta** para o trabalho do Stream Analytics **ingest-orders**. Em seguida, aguarde alguns instantes até que a visualização de entrada seja exibida (com base nos eventos de ordem do cliente capturados anteriormente no hub de eventos).
2. Observe que os dados de entrada incluem os campos **ProductID** e **Quantidade** nas mensagens enviadas pelo aplicativo cliente, bem como campos adicionais de Hubs de Eventos – incluindo o campo **EventProcessedUtcTime** que indica quando o evento foi adicionado ao hub de eventos.
3. Modifique a consulta padrão conforme o seguinte:

    ```
    SELECT
        EventProcessedUtcTime AS OrderDateTime,
        ProductID,
        Quantity
    INTO
        [FactOrder]
    FROM
        [orders]
    ```

    Observe que essa consulta pega campos da entrada (hub de eventos) e os grava diretamente na saída (tabela SQL).

4. Salvar a consulta.

### Execute o trabalho de streaming para ingerir dados do pedido

1. Exiba a página **Visão geral** para o trabalho do Stream Analytics **ingest-orders** e, na guia **Propriedades**, revise as **Entradas**, a **Consulta**, as **Saídas** e as **Funções** para o trabalho. Se o número de **Entradas** e **Saídas** for 0, use o botão **↻ Atualizar** na página **Visão geral** para exibir a entrada de **pedidos** e a saída **FactTable**.
2. Selecione o botão **▷ Iniciar** e inicie o trabalho de streaming agora. Aguarde uma notificação de que o trabalho de streaming foi iniciado com êxito.
3. Reabra o painel do cloud shell e execute novamente o seguinte comando para enviar outros 100 pedidos.

    ```
    node ~/dp-203/Allfiles/labs/18/orderclient
    ```

4. Enquanto o aplicativo cliente de pedido estiver em execução, alterne para a guia do navegador Synapse Studio e exiba a consulta executada anteriormente para selecionar as 100 primeiras linhas da tabela **dbo.FactOrder**.
5. Use o botão **▷ Executar** para executar novamente a consulta e verificar se a tabela agora contém dados de ordem do fluxo de eventos (se não, aguarde um minuto e execute novamente a consulta). O trabalho do Stream Analytics enviará todos os novos dados de eventos para a tabela, desde que o trabalho esteja em execução e os eventos de ordem estejam sendo enviados para o hub de eventos.
6. Na página **Gerenciar**, pause o pool de SQL dedicado **sql*xxxxxxx*** (para evitar cobranças desncessárias do Azure).
7. Retorne à guia do navegador que contém o Portal do Azure e minimize o painel do cloud shell. Em seguida, use o botão ** Parar** para interromper o trabalho do Stream Analytics e aguarde a notificação de que o trabalho do Stream Analytics foi interrompido com êxito.

## Resumir dados de streaming em um data lake

Até agora, você viu como usar um trabalho do Stream Analytics para ingerir mensagens de uma fonte de streaming em uma tabela SQL. Agora vamos explorar como usar o Azure Stream Analytics para agregar dados em janelas temporais - neste caso, para calcular a quantidade total de cada produto vendido a cada 5 segundos. Também exploraremos como usar um tipo diferente de saída para o trabalho gravando os resultados no formato CSV em um repositório de blob de data lake.

### Criar um trabalho do Azure Stream Analytics para agregar dados de pedidos

1. No portal do Azure, na página **Inicial**, selecione **+ Criar um recurso** e pesquise por `Stream Analytics job`. Em seguida, crie um **trabalho do Stream Analytics** com as seguintes propriedades:
    - **Noções básicas**:
        - **Assinatura:** sua assinatura do Azure
        - **Grupo de recursos**: selecione o grupo de recursos existente **dp203-*xxxxxxx***.
        - **Nome**: `aggregate-orders`
        - **Região**: selecione a <u>mesma região</u> em que o workspace do Synapse Analytics está provisionado.
        - **Ambiente de hospedagem**: nuvem
        - **Unidades de streaming**: 1
    - **Armazenamento**:
        - **Adicionar conta de armazenamento**: selecionado
        - **Assinatura:** sua assinatura do Azure
        - **Contas de armazenamento**: selecione a conta de armazenamento **datalake*xxxxxxx***
        - **Modo de autenticação**: cadeia de conexão
        - **Proteger dados privados na conta de armazenamento**: selecionado
    - **Marcas:**
        - *Nenhuma*

2. Aguarde a conclusão da implantação e acesse o recurso de trabalho do Stream Analytics implantado.

### Criar uma entrada para os dados brutos do pedido

1. Na página de visão geral **aggregate-orders**, selecione a página de **Entradas**. Use o menu **Adicionar entrada** para adicionar uma entrada **Hub de Eventos** com as seguintes propriedades:
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

### Criar uma saída para o repositório do data lake

1. Exiba a página **Saídas** para o trabalho do Stream Analytics **aggregate-orders**. Em seguida, use o menu **Adicionar saída** para adicionar uma saída **Repositório de blob/ADLS Gen2** com as propriedades a seguir:
    - **Alias de saída**: `datalake`
    - **Selecione Selecionar Armazenamento de Blobs/ADLS Gen2 em suas assinaturas de suas assinaturas**: selecionado
    - **Assinatura:** sua assinatura do Azure
    - **Conta de armazenamento**: selecione a conta de armazenamento **datalake*xxxxxxx***
    - **Contêiner**: Selecione **Usar existente** e, na lista, selecione o contêiner **arquivos**
    - **Modo de autenticação**: cadeia de conexão
    - **Formato de serialização de eventos**: CSV - Vírgula (,)
    - **Codificação**: UTF-8
    - **Modo de gravação**: Anexar, à medida que os resultados chegam
    - **Padrão de caminho**: `{date}`
    - **Formato de data**: AAAA/MM/DD
    - **Formato de hora**: *não aplicável*
    - **Mínimo de linhas**: 20
    - **Tempo máximo**: 0 horas, 1 minuto, 0 segundos
2. Salve a saída e aguarde enquanto ela é criada. Você verá várias notificações. Aguarde uma notificação de **teste de conexão bem-sucedida**.

### Criar uma consulta para agregar os dados do evento

1. Exiba a página **Consulta** para o trabalho do Stream Analytics **aggregate-orders**.
2. Modifique a consulta padrão conforme o seguinte:

    ```
    SELECT
        DateAdd(second,-5,System.TimeStamp) AS StartTime,
        System.TimeStamp AS EndTime,
        ProductID,
        SUM(Quantity) AS Orders
    INTO
        [datalake]
    FROM
        [orders] TIMESTAMP BY EventProcessedUtcTime
    GROUP BY ProductID, TumblingWindow(second, 5)
    HAVING COUNT(*) > 1
    ```

    Observe que essa consulta usa o **System.Timestamp** (com base no campo **EventProcessedUtcTime**) para definir o início e o término de cada janela *em cascata* de 5 segundos (sequencial sem sobreposição) em que a quantidade total para o qual cada ID de produto é calculado.

3. Salvar a consulta.

### Executar o trabalho de streaming para agregar dados de pedidos

1. Exiba a página **Visão geral** para o trabalho do Stream Analytics **aggregate-orders** e, na guia **Propriedades**, revise as **Entradas**, a **Consulta**, as **Saídas** e as **Funções** para o trabalho. Se o número de **Entradas** e **Saídas** for 0, use o botão **↻ Atualizar** na página **Visão geral** para exibir a entrada de **pedidos** e a saída **datalake**.
2. Selecione o botão **▷ Iniciar** e inicie o trabalho de streaming agora. Aguarde uma notificação de que o trabalho de streaming foi iniciado com êxito.
3. Reabra o painel do cloud shell e execute novamente o seguinte comando para enviar outros 100 pedidos:

    ```
    node ~/dp-203/Allfiles/labs/18/orderclient
    ```

4. Quando o aplicativo de pedido terminar, minimize o painel do cloud shell. Em seguida, alterne para a guia do navegador do Synapse Studio e, na página **Dados**, na guia **Vinculado**, expanda **Azure Data Lake Storage Gen2** > **synapse*xxxxxxx* (principal - datalake*xxxxxxx *)** e selecione o contêiner **arquivos (Principal)**.
5. Se o contêiner de **arquivos** estiver vazio, aguarde um minuto ou mais e use o botão **↻ Atualizar** para atualizar o modo de exibição. Eventualmente, uma pasta nomeada para o ano atual deve ser exibida. Este, por sua vez, contém pastas para o mês e dia.
6. Selecione a pasta para o ano e, no menu **Novo script de SQL**, selecione **Selecionar as 100 primeiras linhas**. Em seguida, defina o **Tipo de arquivos** para **Formato de texto** e aplique as configurações.
7. No painel de consulta que é aberto, modifique a consulta para adicionar um parâmetro `HEADER_ROW = TRUE`, conforme mostrado aqui:

    ```sql
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/2023/**',
            FORMAT = 'CSV',
            PARSER_VERSION = '2.0',
            HEADER_ROW = TRUE
        ) AS [result]
    ```

8. Use o botão **▷ Executar** para executar a consulta SQL e exibir os resultados, que mostram a quantidade de cada produto solicitado em períodos de cinco segundos.
9. Retorne à guia do navegador que contém o Portal do Azure e use o botão ** Parar** para interromper o trabalho do Stream Analytics e aguardar a notificação de que o trabalho do Stream Analytics foi interrompido com êxito.

## Excluir recursos do Azure

Se você terminou de explorar o Azure Stream Analytics, exclua os recursos que criou para evitar custos desnecessários do Azure.

1. Feche a guia do navegador do Azure Synapse Studio e retorne ao portal do Azure.
2. No portal do Azure, na **Página Inicial**, selecione **Grupos de recursos**.
3. Selecione o grupo de recursos **dp203-*xxxxxxx*** que contém seus recursos do Azure Synapse, Hubs de Eventos e Stream Analytics (não o grupo de recursos gerenciados).
4. Na parte superior da página de **Visão Geral** do grupo de recursos, selecione **Excluir o grupo de recursos**.
5. Digite o nome do grupo de recursos **dp203-*xxxxxxx*** para confirmar que deseja excluí-lo e selecione **Excluir**.

    Após alguns minutos, os recursos criados neste exercício serão excluídos.
