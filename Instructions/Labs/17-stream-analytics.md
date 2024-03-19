---
lab:
  title: Introdução ao Azure Stream Analytics
  ilt-use: Suggested demo
---

# Introdução ao Azure Stream Analytics

Neste exercício, você vai provisionar um trabalho do Azure Stream Analytics em sua assinatura do Azure e usá-lo para consultar e resumir um fluxo de dados de eventos em tempo real e armazenar os resultados no Azure Storage.

Este exercício deve levar aproximadamente **15** minutos para ser concluído.

## Antes de começar

É necessário ter uma [assinatura do Azure](https://azure.microsoft.com/free) com acesso de nível administrativo.

## Provisionar recursos do Azure

Neste exercício, você capturará um fluxo de dados de transações de vendas simuladas, processá-los e armazenar os resultados em um contêiner de blob no Armazenamento do Azure. Você precisará de um namespace dos Hubs de Eventos do Azure para o qual os dados de streaming podem ser enviados e uma conta de Armazenamento do Azure na qual os resultados do processamento de fluxo serão armazenados.

Você usará uma combinação de um script do PowerShell e um modelo ARM para provisionar esses recursos.

1. Entre no [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`.
2. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure, selecionando um ambiente ***PowerShell*** e criando armazenamento caso solicitado. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure, conforme mostrado aqui:

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
    cd dp-203/Allfiles/labs/17
    ./setup.ps1
    ```

6. Se solicitado, escolha qual assinatura você deseja usar (isso só acontecerá se você tiver acesso a várias assinaturas do Azure).
7. Aguarde a conclusão do script – isso normalmente leva cerca de 5 minutos, mas em alguns casos pode levar mais tempo. Enquanto espera, revise o artigo [Bem-vindo ao Azure Stream Analytics](https://learn.microsoft.com/azure/stream-analytics/stream-analytics-introduction) na documentação do Azure Stream Analytics.

## Exibir a fonte de dados de streaming

Antes de criar um trabalho do Azure Stream Analytics para processar dados em tempo real, vamos dar uma olhada no fluxo de dados que ele precisará consultar.

1. Quando o script de instalação terminar de ser executado, redimensione ou minimize o painel do cloud shell para que você possa ver o portal do Azure (você retornará ao cloud shell mais tarde). Em seguida, no portal do Azure, vá para o grupo de recursos **dp203-*xxxxxxx*** que ele criou e observe que esse grupo de recursos contém uma conta de Armazenamento do Azure e um namespace de Hubs de Eventos.

    Observe o **Local** onde os recursos foram provisionados - posteriormente, você criará um trabalho do Azure Stream Analytics no mesmo local.

2. Reabra o painel do cloud shell e insira o seguinte comando para executar um aplicativo cliente que envia 100 ordens simuladas para os Hubs de Eventos do Azure:

    ```
    node ~/dp-203/Allfiles/labs/17/orderclient
    ```

3. Observe os dados da ordem de vendas à medida que são enviados - cada pedido consiste em uma ID do produto e uma quantidade. O aplicativo terminará após o envio de 1000 pedidos, o que leva cerca de um minuto.

## Criar um trabalho do Azure Stream Analytics

Agora você está pronto para criar um trabalho do Azure Stream Analytics para processar os dados da transação de vendas à medida que eles chegam ao hub de eventos.

1. No portal do Azure, na página **dp203-*xxxxxxx***, selecione **+ Criar** e procure `Stream Analytics job`. Em seguida, crie um **trabalho do Stream Analytics** com as seguintes propriedades:
    - **Noções básicas**:
        - **Assinatura:** sua assinatura do Azure
        - **Grupo de recursos**: selecione o grupo de recursos existente **dp203-*xxxxxxx***.
        - **Nome**: `process-orders`
        - **Região**: selecione a região onde seus outros recursos do Azure estão provisionados.
        - **Ambiente de hospedagem**: nuvem
        - **Unidades de streaming**: 1
    - **Armazenamento**:
        - **Adicionar conta de armazenamento**: Não selecionado
    - **Tags:**
        - *Nenhuma*
2. Aguarde a conclusão da implantação e acesse o recurso de trabalho do Stream Analytics implantado.

## Criar uma entrada para o streaming de eventos

Seu trabalho do Azure Stream Analytics deve obter dados de entrada do hub de eventos onde as ordens de venda são registradas.

1. Na página de visão geral **process-orders**, selecione **Adicionar entrada**. Em seguida, na página **Entradas**, use o menu **Adicionar entrada de fluxo** para adicionar uma entrada **Hub de Eventos** com as seguintes propriedades:
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

## Criar uma saída para o repositório de blob

Você armazenará os dados agregados da ordem de venda no formato JSON em um contêiner do Azure Storage Blob.

1. Exiba a página **Saídas** para o trabalho do Stream Analytics **process-orders**. Em seguida, use o menu **Adicionar** para adicionar uma saída do **Armazenamento de Blobs/ADLS Gen2** com as seguintes propriedades:
    - **Alias de saída**: `blobstore`
    - **Selecione Selecionar Armazenamento de Blobs/ADLS Gen2 em suas assinaturas de suas assinaturas**: selecionado
    - **Assinatura:** sua assinatura do Azure
    - **Conta de armazenamento**: selecione a conta de armazenamento **store*xxxxxxx***
    - **Contêiner**: selecione o contêiner de **dados** existentes
    - **Modo de autenticação**: Identidade Gerenciada: Sistema atribuído
    - **Formato de serialização do evento**: JSON
    - **Formato**: Linha separada
    - **Codificação**: UTF-8
    - **Modo de gravação**: anexar à medida que os resultados chegam
    - **Padrão de caminho**: `{date}`
    - **Formato de data**: AAAA/MM/DD
    - **Formato de hora**: *não aplicável*
    - **Mínimo de linhas**: 20
    - **Tempo máximo**: 0 horas, 1 minuto, 0 segundos
2. Salve a saída e aguarde enquanto ela é criada. Você verá várias notificações. Aguarde uma notificação de **teste de conexão bem-sucedida**.

## Criar uma consulta

Agora que você definiu uma entrada e uma saída para seu trabalho do Azure Stream Analytics, pode usar uma consulta para selecionar, filtrar e agregar dados da entrada e enviar os resultados para a saída.

1. Exiba a página **Consulta** para o trabalho do Stream Analytics **process-orders**. Em seguida, aguarde alguns instantes até que a visualização de entrada seja exibida (com base nos eventos de ordem do cliente capturados anteriormente no hub de eventos).
2. Observe que os dados de entrada incluem os campos **ProductID** e **Quantidade** nas mensagens enviadas pelo aplicativo cliente, bem como campos adicionais de Hubs de Eventos – incluindo o campo **EventProcessedUtcTime** que indica quando o evento foi adicionado ao hub de eventos.
3. Modifique a consulta padrão conforme o seguinte:

    ```
    SELECT
        DateAdd(second,-10,System.TimeStamp) AS StartTime,
        System.TimeStamp AS EndTime,
        ProductID,
        SUM(Quantity) AS Orders
    INTO
        [blobstore]
    FROM
        [orders] TIMESTAMP BY EventProcessedUtcTime
    GROUP BY ProductID, TumblingWindow(second, 10)
    HAVING COUNT(*) > 1
    ```

    Observe que essa consulta usa o **System.Timestamp** (com base no campo **EventProcessedUtcTime**) para definir o início e o fim de cada janela em *cascata* de 10 segundos (sequencial não sobreposta) na qual a quantidade total para cada ID de produto é calculada.

4. Use o botão **▷ Testar consulta** para validar a consulta e garantir que o status **Resultados do teste** indique **Êxito** (mesmo que nenhuma linha seja retornada).
5. Salvar a consulta.

## Executar o trabalho de streaming

Ok, agora você está pronto para executar o trabalho e processar alguns dados de ordem de venda em tempo real.

1. Exiba a página **Visão geral** para o trabalho do Stream Analytics **process-orders** e, na guia **Propriedades**, revise as **Entradas**, a **Consulta**, as **Saídas** e as **Funções** para o trabalho. Se o número de **Entradas** e **Saídas** for 0, use o botão **↻Atualizar** na página **Visão geral** para exibir a entrada de **pedidos** e a saída **blobstore**.
2. Selecione o botão **▷ Iniciar** e inicie o trabalho de streaming agora. Aguarde uma notificação de que o trabalho de streaming foi iniciado com êxito.
3. Reabra o painel do Cloud Shell, reconectando-se caso necessário e, em seguida, execute novamente o seguinte comando para enviar outros 1000 pedidos.

    ```
    node ~/dp-203/Allfiles/labs/17/orderclient
    ```

4. Enquanto o aplicativo estiver em execução, no portal do Azure, retorne à página do grupo de recursos **dp203-*xxxxxxx*** e selecione a conta de armazenamento **store*xxxxxxxxxxxx***.
6. No painel à esquerda do painel da conta de armazenamento, selecione a guia **Contêineres**.
7. Abra o contêiner de **dados** e use o botão **&#8635; Atualizar** para atualizar o modo de exibição até ver uma pasta com o nome do ano atual.
8. No contêiner de **dados**, navegue pela hierarquia de pastas, que inclui uma pasta para o ano atual, com subpastas para mês e dia.
9. Na pasta da hora, observe o arquivo que foi criado, que deve ter um nome semelhante a **0_xxxxxxxxxxxxxxxx.json**.
10. No menu **...** do arquivo (à direita dos detalhes do arquivo), selecione **Exibir/editar** e revise o conteúdo do arquivo, que deve consistir em um registro JSON para cada período de 10 segundos, mostrando o número de pedidos processados de cada ID de produto, assim:

    ```
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":6,"Orders":13.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":8,"Orders":15.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":5,"Orders":15.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":1,"Orders":16.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":3,"Orders":10.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":2,"Orders":25.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":7,"Orders":13.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":4,"Orders":12.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":10,"Orders":19.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":9,"Orders":8.0}
    {"StartTime":"2022-11-23T18:16:35.0000000Z","EndTime":"2022-11-23T18:16:45.0000000Z","ProductID":6,"Orders":41.0}
    {"StartTime":"2022-11-23T18:16:35.0000000Z","EndTime":"2022-11-23T18:16:45.0000000Z","ProductID":8,"Orders":29.0}
    ...
    ```

11. No painel do Azure Cloud Shell, aguarde até que o aplicativo cliente do pedido seja concluído.
12. De volta ao portal do Azure, atualize o arquivo mais uma vez para ver o conjunto completo de resultados que foram produzidos.
13. Retorne ao grupo de recursos **dp203-*xxxxxxx*** e abra novamente o trabalho **process-orders** do Stream Analytics.
14. Na parte superior da página de trabalho do Stream Analytics, use o botão **&#11036; Parar** para interromper o trabalho, confirmando quando solicitado.

## Excluir recursos do Azure

Se você terminou de explorar o Azure Stream Analytics, exclua os recursos que criou para evitar custos desnecessários do Azure.

1. No portal do Azure, na **Página Inicial**, selecione **Grupos de recursos**.
2. Selecione o grupo de recursos **dp203-*xxxxxxx*** que contém seus recursos de Armazenamento do Azure, Hubs de Eventos e Stream Analytics.
3. Na parte superior da página de **Visão Geral** do grupo de recursos, selecione **Excluir o grupo de recursos**.
4. Digite o nome do grupo de recursos **dp203-*xxxxxxx*** para confirmar que deseja excluí-lo e selecione **Excluir**.

    Após alguns minutos, os recursos criados neste exercício serão excluídos.
