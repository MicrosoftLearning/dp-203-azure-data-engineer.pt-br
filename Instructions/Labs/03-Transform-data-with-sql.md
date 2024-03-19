---
lab:
  title: Transformar dados usando um pool de SQL sem servidor
  ilt-use: Lab
---

# Transformar arquivos usando um pool de SQL sem servidor

Os *analistas* de dados geralmente usam SQL para consultar dados para análise e relatórios. Os *engenheiros* de dados também podem fazer uso do SQL para manipular e transformar dados, muitas vezes como parte de um pipeline de ingestão de dados ou processo de extração, transformação e carregamento (ETL).

Neste exercício, você usará um pool de SQL sem servidor no Azure Synapse Analytics para transformar dados em arquivos.

Este exercício deve levar aproximadamente **30** minutos para ser concluído.

## Antes de começar

É necessário ter uma [assinatura do Azure](https://azure.microsoft.com/free) com acesso de nível administrativo.

## Provisionar um workspace do Azure Synapse Analytics

Você precisará de um espaço de trabalho do Azure Synapse Analytics com acesso ao armazenamento de data lake. Você pode usar o pool do SQL interno sem servidor para consultar arquivos no data lake.

Neste exercício, você usará uma combinação de um script do PowerShell e um modelo ARM para provisionar um espaço de trabalho do Azure Synapse Analytics.

1. Entre no [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`.
2. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure, selecionando um ambiente ***PowerShell*** e criando armazenamento caso solicitado. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure, conforme mostrado aqui:

    ![Portal do Azure com um painel do Cloud Shell](./images/cloud-shell.png)

    > **Observação**: se você tiver criado anteriormente um cloud shell que usa um ambiente *Bash*, use o menu suspenso no canto superior esquerdo do painel do cloud shell para alterá-lo para ***PowerShell***.

3. Observe que você pode redimensionar o Cloud Shell arrastando a barra do separador na parte superior do painel ou usando os ícones **&#8212;** , **&#9723;** e **X** no canto superior direito do painel para minimizar, maximizar e fechar o painel. Para obter mais informações de como usar o Azure Cloud Shell, confira a [documentação do Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. No painel do PowerShell, insira os seguintes comandos para clonar esse repositório:

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Depois que o repositório tiver sido clonado, digite os seguintes comandos para alterar para a pasta deste exercício e execute o script **setup.ps1** que ele contém:

    ```
    cd dp-203/Allfiles/labs/03
    ./setup.ps1
    ```

6. Se solicitado, escolha qual assinatura você deseja usar (isso só acontecerá se você tiver acesso a várias assinaturas do Azure).
7. Quando solicitado, insira uma senha adequada a ser definida para seu pool de SQL do Azure Synapse.

    > **Observação**: Memorize a senha.

8. Aguarde a conclusão do script - isso normalmente leva cerca de 10 minutos, mas em alguns casos pode levar mais tempo. Enquanto espera, revise o artigo [CETAS com Synapse SQL](https://docs.microsoft.com/azure/synapse-analytics/sql/develop-tables-cetas) na documentação do Azure Synapse Analytics.

## Consultar dados em arquivos

O script provisiona um espaço de trabalho do Azure Synapse Analytics e uma conta de Armazenamento do Azure para hospedar o data lake e, em seguida, carrega alguns arquivos de dados no data lake.

### Exibir arquivos no data lake

1. Depois que o script for concluído, no portal do Azure, vá para o Azure Storage **dp203-*xxxxxxx*** que ele criou e selecione seu espaço de trabalho Synapse.
2. Na página **Visão geral** do seu workspace do Synapse, no cartão **Abrir o Synapse Studio**, selecione **Abrir** para abrir o Synapse Studio em uma nova guia do navegador, fazendo login se solicitado.
3. No lado esquerdo do Synapse Studio, use o ícone **&rsaquo;&rsaquo;** para expandir o menu – o que revela as diferentes páginas do Synapse Studio usadas para gerenciar recursos e executar tarefas de análise de dados.
4. Na página **Dados**, exiba a guia **Vinculado** e verifique se seu workspace inclui um link para sua conta de armazenamento do Azure Data Lake Storage Gen2, que deve ter um nome semelhante à **synapse*xxxxxxx* (Primary - datalake*xxxxxxx*)**.
5. Expanda sua conta de armazenamento e verifique se ela contém um contêiner do sistema de arquivos chamado **arquivos**.
6. Selecione o contêiner de **arquivos** e observe que ele contém uma pasta chamada **vendas**. Esta pasta contém os arquivos de dados que você vai consultar.
7. Abra a pasta de **vendas** e a pasta **csv** que ela contém e observe que essa pasta contém arquivos .csv para três anos de dados de vendas.
8. Clique com o botão direito do mouse em qualquer um dos arquivos e selecione **Visualizar** para ver os dados que ele contém. Observe que os arquivos contêm uma linha de cabeçalho.
9. Feche a visualização e use o botão **↑** para navegar de volta para a pasta de **vendas**.

### Usar o SQL para consultar arquivos CSV

1. Selecione a pasta **csv** e, na lista **Novo script SQL** na barra de ferramentas, selecione **Selecionar as 100 PRIMEIRAS LINHAS**.
2. Na lista **Tipo de arquivo**, selecione **Formato de texto** e aplique as configurações para abrir um novo script SQL que consulta os dados na pasta.
3. No painel **Propriedades** do **SQL Script 1** criado, altere o nome para **Arquivos CSV de Vendas de Consulta** e altere as configurações de resultado para mostrar **Todas as linhas**. Em seguida, na barra de ferramentas, selecione **Publicar** para salvar o script e use o botão **Propriedades** (que se parece com **<sub>*</sub>**) na extremidade direita da barra de ferramentas para ocultar o painel **Propriedades**.
4. Revise o código de SQL que foi gerado, que deve ser semelhante a este:

    ```SQL
    -- This is auto-generated code
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/sales/csv/**',
            FORMAT = 'CSV',
            PARSER_VERSION='2.0'
        ) AS [result]
    ```

    Esse código usa o OPENROWSET para ler dados dos arquivos CSV na pasta de vendas e recupera as primeiras 100 linhas de dados.

5. Nesse caso, os arquivos de dados incluem os nomes das colunas na primeira linha; Portanto, modifique a consulta para adicionar um parâmetro `HEADER_ROW = TRUE` à cláusula `OPENROWSET`, como mostrado aqui (não se esqueça de adicionar uma vírgula após o parâmetro anterior):

    ```SQL
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/sales/csv/**',
            FORMAT = 'CSV',
            PARSER_VERSION='2.0',
            HEADER_ROW = TRUE
        ) AS [result]
    ```

6. Na lista **Conectar-se a**, verifique se **Interno** está selecionado — isso representa o Pool de SQL interno que foi criado com o seu workspace. Na barra de ferramentas, use o botão **▷ Executar** para executar o código SQL e examine os resultados, que devem ser semelhantes a este:

    | SalesOrderNumber | SalesOrderLineNumber | OrderDate | CustomerName | EmailAddress | Item | Quantidade | UnitPrice | TaxAmount |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | SO43701 | 1 | 2019-07-01 | Christy Zhu | christy12@adventure-works.com |Mountain-100 Silver, 44 | 1 | 3399.99 | 271.9992 |
    | ... | ... | ... | ... | ... | ... | ... | ... | ... |

7. Publique as alterações no script e feche o painel de script.

## Transformar dados usando instruções CREATE EXTERNAL TABLE AS SELECT (CETAS)

Uma maneira simples de usar o SQL para transformar dados em um arquivo e persistir os resultados em outro arquivo é usar uma instrução CREATE EXTERNAL TABLE AS SELECT (CETAS). Essa instrução cria uma tabela com base nas solicitações de uma consulta, mas os dados da tabela são armazenados como arquivos em um data lake. Os dados transformados podem então ser consultados por meio da tabela externa ou acessados diretamente no sistema de arquivos (por exemplo, para inclusão em um processo downstream a fim de carregar os dados transformados em um data warehouse).

### Criar uma fonte de dados externa e um formato de arquivo

Ao definir uma fonte de dados externa em um banco de dados, você pode usá-la para fazer referência ao local do data lake onde deseja armazenar arquivos para tabelas externas. Um formato de arquivo externo permite que você defina o formato para esses arquivos - por exemplo, Parquet ou CSV. Para usar esses objetos a fim de trabalhar com tabelas externas, você precisa criá-los em um banco de dados diferente do banco de dados **mestre** padrão.

1. No Synapse Studio, na página **Desenvolver**, no menu ****+, selecione **Script SQL**.
2. No novo painel de script, adicione o seguinte código (substituindo *datalakexxxxxxx* pelo nome da sua conta de armazenamento do data lake) para criar um novo banco de dados e adicionar uma fonte de dados externa a ele.

    ```sql
    -- Database for sales data
    CREATE DATABASE Sales
      COLLATE Latin1_General_100_BIN2_UTF8;
    GO;
    
    Use Sales;
    GO;
    
    -- External data is in the Files container in the data lake
    CREATE EXTERNAL DATA SOURCE sales_data WITH (
        LOCATION = 'https://datalakexxxxxxx.dfs.core.windows.net/files/'
    );
    GO;
    
    -- Format for table files
    CREATE EXTERNAL FILE FORMAT ParquetFormat
        WITH (
                FORMAT_TYPE = PARQUET,
                DATA_COMPRESSION = 'org.apache.hadoop.io.compress.SnappyCodec'
            );
    GO;
    ```

3. Modifique as propriedades do script para alterar seu nome para **Create Sales DB** e publique.
4. Verifique se o script está conectado ao pool SQL **interno** e ao banco de dados **mestre** e execute-o.
5. Volte para a página **Dados** e use o botão **↻** no canto superior direito do Synapse Studio para atualizar a página. Em seguida, exiba a guia **Espaço de trabalho** no painel **Dados**, onde uma lista **banco de dados SQL** agora é exibida. Expanda essa lista para verificar se o banco de dados **Vendas** foi criado.
6. Expanda o banco de dados **Vendas**, sua pasta **Recursos Externos** e a pasta **Fontes de dados externas** para ver a fonte de dados externa **sales_data** que você criou.

### Criar uma tabela externa

1. No Synapse Studio, na página **Desenvolver**, no menu ****+, selecione **Script SQL**.
2. No novo painel de script, adicione o seguinte código para recuperar e agregar dados dos arquivos de vendas CSV usando a fonte de dados externa - observando que o caminho **BULK** é relativo ao local da pasta no qual a fonte de dados está definida:

    ```sql
    USE Sales;
    GO;
    
    SELECT Item AS Product,
           SUM(Quantity) AS ItemsSold,
           ROUND(SUM(UnitPrice) - SUM(TaxAmount), 2) AS NetRevenue
    FROM
        OPENROWSET(
            BULK 'sales/csv/*.csv',
            DATA_SOURCE = 'sales_data',
            FORMAT = 'CSV',
            PARSER_VERSION = '2.0',
            HEADER_ROW = TRUE
        ) AS orders
    GROUP BY Item;
    ```

3. Execute o script. Os resultados deverão ter a seguinte aparência:

    | Product | ItemsSold | NetRevenue |
    | -- | -- | -- |
    | AWC Logo Cap | 1063 | 8791.86 |
    | ... | ... | ... |

4. Modifique o código SQL para salvar os resultados da consulta em uma tabela externa, da seguinte forma:

    ```sql
    CREATE EXTERNAL TABLE ProductSalesTotals
        WITH (
            LOCATION = 'sales/productsales/',
            DATA_SOURCE = sales_data,
            FILE_FORMAT = ParquetFormat
        )
    AS
    SELECT Item AS Product,
        SUM(Quantity) AS ItemsSold,
        ROUND(SUM(UnitPrice) - SUM(TaxAmount), 2) AS NetRevenue
    FROM
        OPENROWSET(
            BULK 'sales/csv/*.csv',
            DATA_SOURCE = 'sales_data',
            FORMAT = 'CSV',
            PARSER_VERSION = '2.0',
            HEADER_ROW = TRUE
        ) AS orders
    GROUP BY Item;
    ```

5. Execute o script. Desta vez, não há saída, mas o código deve ter criado uma tabela externa com base nos resultados da consulta.
6. Nomeie o script **Create ProductSalesTotals table** e publique-o.
7. Na página **dados**, na guia **Espaço de trabalho**, consulte o conteúdo da pasta **Tabelas externas** do banco de dados SQL **Vendas** para verificar se uma nova tabela chamada **ProductSalesTotals** foi criada.
8. No menu **...** para a tabela **ProductSalesTotals**, selecione **Novo script SQL** > **Selecionar as 100 PRIMEIRAS LINHAS**. Em seguida, execute o script resultante e verifique se ele retorna os dados agregados de vendas do produto.
9. Na guia **arquivos** que contém o sistema de arquivos do seu data lake, consulte o conteúdo da pasta **vendas** (atualizando a visualização, se necessário) e verifique se uma nova pasta **productsales** foi criada.
10. Na pasta **productsales**, observe que um ou mais arquivos com nomes semelhantes a ABC123DE----.parquet foram criados. Esses arquivos contêm os dados agregados de vendas do produto. Para provar isso, selecione um dos arquivos e use o menu **Novo script SQL** > **Selecione 100 linhas prinicipais** para consultar diretamente.

## Encapsular transformações de dados em um procedimento armazenado

Se você precisar transformar dados com frequência, use um procedimento armazenado para encapsular uma instrução CETAS.

1. No Synapse Studio, na página **Desenvolver**, no menu ****+, selecione **Script SQL**.
2. No novo painel de script, adicione o seguinte código para criar um procedimento armazenado no banco de dados **Vendas** que agrega vendas por ano e salva os resultados em uma tabela externa:

    ```sql
    USE Sales;
    GO;
    CREATE PROCEDURE sp_GetYearlySales
    AS
    BEGIN
        -- drop existing table
        IF EXISTS (
                SELECT * FROM sys.external_tables
                WHERE name = 'YearlySalesTotals'
            )
            DROP EXTERNAL TABLE YearlySalesTotals
        -- create external table
        CREATE EXTERNAL TABLE YearlySalesTotals
        WITH (
                LOCATION = 'sales/yearlysales/',
                DATA_SOURCE = sales_data,
                FILE_FORMAT = ParquetFormat
            )
        AS
        SELECT YEAR(OrderDate) AS CalendarYear,
                SUM(Quantity) AS ItemsSold,
                ROUND(SUM(UnitPrice) - SUM(TaxAmount), 2) AS NetRevenue
        FROM
            OPENROWSET(
                BULK 'sales/csv/*.csv',
                DATA_SOURCE = 'sales_data',
                FORMAT = 'CSV',
                PARSER_VERSION = '2.0',
                HEADER_ROW = TRUE
            ) AS orders
        GROUP BY YEAR(OrderDate)
    END
    ```

3. Execute o script para criar o procedimento armazenado.
4. Sob o código que você acabou de executar, adicione o seguinte código para chamar o procedimento armazenado:

    ```sql
    EXEC sp_GetYearlySales;
    ```

5. Selecione apenas a instrução `EXEC sp_GetYearlySales;` que você acabou de adicionar e use o botão **▷ Executar** para executá-lo.
6. Na guia **arquivos** que contém o sistema de arquivos para seu data lake, exiba o conteúdo da pasta **vendas** (atualizando a exibição, se necessário) e verifique se uma nova pasta **yearlysales** foi criada.
7. Na pasta **yearlysales**, observe que um arquivo parquet contendo os dados de vendas anuais agregados foi criado.
8. Volte para o script SQL e execute novamente a instrução `EXEC sp_GetYearlySales;` e observe que ocorre um erro.

    Mesmo que o script descarte a tabela externa, a pasta que contém os dados não é excluída. Para executar novamente o procedimento armazenado (por exemplo, como parte de um pipeline de transformação de dados agendado), exclua os dados antigos.

9. Volte para a guia **arquivos** e exiba a pasta **vendas**. Em seguida, selecione a pasta **yearlysales** e exclua-a.
10. Volte para o script SQL e execute novamente a instrução `EXEC sp_GetYearlySales;`. Desta vez, a operação é bem-sucedida e um novo arquivo de dados é gerado.

## Excluir recursos do Azure

Se você terminou de explorar Azure Synapse Analytics, exclua os recursos que criou para evitar custos desnecessários do Azure.

1. Feche a guia do navegador do Synapse Studio e retorne ao portal do Azure.
2. No portal do Azure, na **Página Inicial**, selecione **Grupos de recursos**.
3. Selecione o grupo de recursos **dp203-*xxxxxxx*** para o workspace do Synapse Analytics (não o grupo de recursos gerenciado) e verifique se ele contém o workspace do Synapse e a conta de armazenamento para seu workspace.
4. Na parte superior da página de **Visão Geral** do grupo de recursos, selecione **Excluir o grupo de recursos**.
5. Digite o nome do grupo de recursos **dp203-*xxxxxxx*** para confirmar que deseja excluí-lo e selecione **Excluir**.

    Após alguns minutos, o grupo de recursos de seu workspace do Azure Synapse e o grupo de recursos do workspace gerenciado associado a ele serão excluídos.
