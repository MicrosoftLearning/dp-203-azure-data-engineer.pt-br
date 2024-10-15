---
lab:
  title: Explorar o Azure Synapse Analytics
  ilt-use: Lab
---

# Explorar o Azure Synapse Analytics

O Azure Synapse Analytics fornece uma plataforma única e consolidada de análise de dados para análise de dados de ponta a ponta. Neste exercício, você explorará várias maneiras de ingerir e explorar dados. Este exercício foi concebido como uma visão geral de alto nível dos vários recursos principais do Azure Synapse Analytics. Outros exercícios estão disponíveis para explorar capacidades específicas com mais detalhes.

Este exercício deve levar aproximadamente **60** minutos para ser concluído.

## Antes de começar

É necessário ter uma [assinatura do Azure](https://azure.microsoft.com/free) com acesso de nível administrativo.

## Provisionar um workspace do Azure Synapse Analytics

Um *workspace* do Azure Synapse Analytics fornece um ponto central para gerenciar dados e tempos de execução de processamento de dados. Você pode provisionar um workspace usando a interface interativa no portal do Azure ou pode implantar um workspace e recursos nele usando um script ou modelo. Na maioria dos cenários de produção, é melhor automatizar o provisionamento com scripts e modelos para que você possa incorporar a implantação de recursos em um processo de desenvolvimento e operações (*DevOps*) repetível.

Neste exercício, você usará uma combinação de um script do PowerShell e um modelo ARM para provisionar o Azure Synapse Analytics.

1. Em um navegador da web, entre no [portal da Azure](https://portal.azure.com) em `https://portal.azure.com`.
2. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure, selecionando um ambiente ***PowerShell*** e criando um armazenamento caso solicitado. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure, conforme mostrado aqui:

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
    cd dp-203/Allfiles/labs/01
    ./setup.ps1
    ```

6. Se solicitado, escolha qual assinatura você deseja usar (isso só acontecerá se você tiver acesso a várias assinaturas do Azure).
7. Quando solicitado, insira uma senha adequada a ser definida para seu pool de SQL do Azure Synapse.

    > **Observação**: Memorize a senha. Além disso, a senha não pode conter todo ou parte do nome de login.

8. Aguarde a conclusão do script - isso normalmente leva cerca de 20 minutos, mas em alguns casos pode levar mais tempo. Enquanto espera, revise o artigo [O que é o Azure Synapse Analytics?](https://docs.microsoft.com/azure/synapse-analytics/overview-what-is) na documentação do Azure Synapse Analytics.

## Explorar o Synapse Studio

O *Synapse Studio* é um portal baseado na Web no qual você pode gerenciar e trabalhar com os recursos em seu espaço de trabalho do Azure Synapse Analytics.

1. Quando o script de instalação terminar de ser executado, no portal do Azure, vá para o grupo de recursos **dp203-*xxxxxxx*** que foi criado e observe que esse grupo de recursos contém o espaço de trabalho do Synapse, uma conta de armazenamento do seu data lake, um pool do Apache Spark e um pool SQL dedicado.
2. Selecione o seu workspace do Synapse e a página de **Visão Geral** dele, no cartão do **Open Synapse Studio**, selecione **Abrir** para abrir o Synapse Studio em uma nova guia do navegador. O Synapse Studio é uma interface baseada na Web que pode ser usada para trabalhar com o seu workspace do Synapse Analytics.
3. No lado esquerdo do Synapse Studio, use o ícone **&rsaquo;&rsaquo;** para expandir o menu, isso revela as diferentes páginas no Synapse Studio que você usará para gerenciar recursos e executar tarefas de análise de dados, como mostrado aqui:

    ![Imagem mostrando o menu expandido do Synapse Studio para gerenciar recursos e executar tarefas de análise de dados](./images/synapse-studio.png)

4. Exiba a página **Dados** e observe que há duas guias contendo fontes de dados:
    - Uma guia **Workspace** que contém bancos de dados definidos no espaço de trabalho (incluindo bancos de dados SQL dedicados e bancos de dados do Data Explorer)
    - Uma guia **Vinculada** que contém fontes de dados vinculadas ao espaço de trabalho, incluindo o armazenamento do Azure Data Lake.

5. Exiba a página **Desenvolver**, que está vazia no momento. É aqui que você pode definir scripts e outros ativos usados para desenvolver soluções de processamento de dados.
6. Exiba a página **Integrar**, que também está vazia. Você usa esta página para gerenciar a ingestão de dados e ativos de integração; como pipelines para transferir e transformar dados entre fontes de dados.
7. Exiba a página **Monitor**. É aqui que você pode observar os trabalhos de processamento de dados enquanto eles são executados e visualizar seu histórico.
8. Exiba a página **Gerenciar**. É aqui que você gerencia os pools, tempos de execução e outros ativos usados em seu workspace do Azure Synapse. Exiba cada uma das guias na seção **Pools do Google Analytics** e observe que seu workspace inclui os seguintes pools:
    - **Pools de SQL**:
        - **Interno**: um pool SQL *sem servidor* que você pode usar sob demanda para explorar ou processar dados em um data lake usando comandos SQL.
        - **sql*xxxxxxx***: Um pool SQL *dedicado* que hospeda um banco de dados de data warehouse relacional.
    - **Pools do Apache Spark**:
        - **spark*xxxxxxx***: que você pode usar sob demanda para explorar ou processar dados em um data lake usando linguagens de programação como Scala ou Python.

## Ingerir dados com um pipeline

Uma das principais tarefas que você pode executar com o Azure Synapse Analytics é definir *pipelines* que transferem (e, se necessário, transformam) dados de uma ampla gama de fontes no seu workspace para análise.

### Usar a tarefa Copiar Dados para criar um pipeline

1. No Synapse Studio, na **Página inicial**, selecione **Ingerir** para abrir a ferramenta **Copiar dados**
2. Na ferramenta Copiar Dados, na etapa **Propriedades**, verifique se **Tarefa de cópia interna** e **Executar agora** estão selecionados e clique em **Avançar >**.
3. Na etapa de **Origem**, na subetapa de **Conjunto de dados**, selecione as seguintes configurações:
    - **Tipo de fonte**: Todas
    - **Conexão**: *Crie uma nova conexão e, no painel de **Serviço vinculado** exibido, na guia **Protocolo genérico**, selecione **HTTP**. Então crie uma conexão com um arquivo de dados usando as seguintes configurações:*
        - **Nome**: Produtos
        - **Descrição**: Lista de produtos via HTTP
        - **Conectar por meio de runtime de integração**: AutoResolveIntegrationRuntime
        - **URL Base**: `https://raw.githubusercontent.com/MicrosoftLearning/dp-203-azure-data-engineer/master/Allfiles/labs/01/adventureworks/products.csv`
        - **Validação de Certificado do Servidor**: Habilitar
        - **Tipo de autenticação**: Anônimo
4. Depois de criar a conexão, na página **Armazenamento de dados de origem**, verifique se as seguintes configurações estão selecionadas e selecione **Avançar >**:
    - **URL Relativa**: *Deixar em branco*
    - **Método de solicitação**: GET
    - **Cabeçalhos adicionais**: *Deixar em branco*
    - **Cópia binária**: <u>Des</u>marcada
    - **Tempo limite de solicitação**: *Deixar em branco*
    - **Máximo de conexões simultâneas**: *Deixar em branco*
5. Na etapa **Origem**, na subetapa **Configuração**, selecione **Visualizar dados** para ver uma prévia dos dados do produto que seu pipeline ingerirá e, em seguida, feche a prévia.
6. Depois de visualizar os dados, na página **Configurações de formato de arquivo**, verifique se as seguintes configurações estão selecionadas e selecione **Avançar >**:
    - **Formato de arquivo**: DelimitedText
    - **Delimitador de colunas**: Vírgula (,)
    - **Delimitador de linha**: Alimentação de linha (\n)
    - **Primeira linha como cabeçalho**: Selecionada
    - **Tipo de compactação**: Nenhum
7. Na etapa **Destino**, na subetapa **Conjunto de dados**, selecione as seguintes configurações:
    - **Tipo de destino**: Azure Data Lake Storage Gen 2
    - **Conexão**: *selecione a conexão existente com o data lake store (gerado para você no momento da criação do workspace).*
8. Depois de selecionar a conexão, na etapa **Destino/conjunto de dados**, verifique se as seguintes configurações estão selecionadas e, em seguida, clique em **Avançar >** :
    - **Caminho da pasta**: files/product_data
    - **Nome do arquivo**: products.csv
    - **Comportamento da cópia**: Nenhum
    - **Máximo de conexões simultâneas**: *Deixar em branco*
    - **Tamanho do bloco (MB)**: *Deixar em branco*
9. Na etapa **Destino**, na subetapa **Configuração**, na página **Configurações de formato do arquivo**, verifique se as propriedades a seguir estão selecionadas. Em seguida, selecione **Avançar >**:
    - **Formato de arquivo**: DelimitedText
    - **Delimitador de colunas**: Vírgula (,)
    - **Delimitador de linha**: Alimentação de linha (\n)
    - **Adicionar cabeçalho ao arquivo**: Selecionado
    - **Tipo de compactação**: Nenhum
    - **Máximo de linhas por arquivo**: *Deixar em branco*
    - **Prefixo do nome do arquivo**: *Deixar em branco*
10. Na etapa **Configurações**, insira estas configurações e clique em **Avançar >**:
    - **Nome da tarefa**: Copiar produtos
    - **Descrição da tarefa** Copiar dados dos produtos
    - **Tolerância a falhas**: *Deixar em branco*
    - **Habilitar registro em log**: <u>Des</u>marcado
    - **Habilitar processo de preparo**: <u>Des</u>marcado
11. Na etapa **Revisar e concluir**, na subetapa **Revisar**, leia o resumo e clique em **Avançar >**.
12. Na etapa de **Implantar**, aguarde a implantação do pipeline e clique em **Concluir**.
13. No Synapse Studio, selecione a página **Monitorar** e, na guia **Execuções de pipeline**, aguarde até que o pipeline **Copiar produtos** seja concluído com um status de **Sucesso** (é possível usar o botão **&#8635; Atualizar** na página Execuções de pipeline para atualizar o status).
14. Exiba a página **Integrar** e verifique se ela agora contém um pipeline chamado **Copiar produtos**.

### Exibir os dados ingeridos

1. Na página **Dados**, selecione a guia **Vinculado** e expanda a hierarquia de contêiner **datalake synapse*xxxxxxx* (Primário)** até ver o armazenamento de arquivos **arquivos** para o seu workspace Synapse. Em seguida, selecione o armazenamento de arquivos para verificar se uma pasta chamada **product_data** contendo um arquivo chamado **products.csv** foi copiado para esse local, como mostrado aqui:

    ![Imagem mostrando a hierarquia do Azure Data Lake Storage expandida do Synapse Studio com o armazenamento de arquivos do workspace do Synapse](./images/product_files.png)

2. Clique com o botão direito do mouse no arquivo de dados **products.csv** e selecione **Visualizar** para exibir os dados ingeridos. Em seguida, feche a visualização.

## Usar o pool de SQL sem servidor para analisar dados

Agora que você já ingeriu alguns dados no seu workspace, use o Synapse Analytics para consultá-los e analisá-los. Uma das maneiras mais comuns de consultar dados é usar o SQL e, no Synapse Analytics, você pode usar um pool de SQL sem servidor para executar o código de SQL com relação aos dados em um data lake.

1. No Synapse Studio, clique com o botão direito do mouse no arquivo **products.csv** no armazenamento de arquivos do seu workspace do Synapse, aponte para **Novo script de SQL** e escolha **Selecionar as 100 linhas superiores**.
2. No painel **Script SQL 1** que será aberto, revise o código de SQL que foi gerado, que deve ser semelhante a este:

    ```SQL
    -- This is auto-generated code
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/product_data/products.csv',
            FORMAT = 'CSV',
            PARSER_VERSION='2.0'
        ) AS [result]
    ```

    Esse código abre um conjunto de linhas do arquivo de texto que você importou e recupera as primeiras 100 linhas de dados.

3. Na lista **Conectar-se a**, verifique se **Interno** está selecionado — isso representa o Pool de SQL interno que foi criado com o seu workspace.
4. Na barra de ferramentas, use o botão **&#9655; Executar** para executar o código SQL e examine os resultados, que devem ser semelhantes a este:

    | C1 | C2 | C3 | C4 |
    | -- | -- | -- | -- |
    | ProductID | ProductName | Categoria | ListPrice |
    | 771 | Mountain-100 Silver, 38 | Mountain bikes | 3399.9900 |
    | 772 | Mountain-100 Silver, 42 | Mountain bikes | 3399.9900 |
    | ... | ... | ... | ... |

5. Observe que os resultados consistem em quatro colunas chamadas C1, C2, C3 e C4; e que a primeira linha nos resultados contém os nomes dos campos de dados. Para corrigir esse problema, adicione um parâmetro HEADER_ROW = TRUE à função OPENROWSET conforme mostrado aqui (substituindo *datalakexxxxxxx* pelo nome da sua conta de armazenamento do data lake) e execute novamente a consulta:

    ```SQL
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/product_data/products.csv',
            FORMAT = 'CSV',
            PARSER_VERSION='2.0',
            HEADER_ROW = TRUE
        ) AS [result]
    ```

    Agora, os resultados terão a seguinte aparência:

    | ProductID | ProductName | Categoria | ListPrice |
    | -- | -- | -- | -- |
    | 771 | Mountain-100 Silver, 38 | Mountain bikes | 3399.9900 |
    | 772 | Mountain-100 Silver, 42 | Mountain bikes | 3399.9900 |
    | ... | ... | ... | ... |

6. Modifique a consulta da seguinte maneira (substituindo *datalakexxxxxxx* pelo nome da conta de armazenamento do data lake):

    ```SQL
    SELECT
        Category, COUNT(*) AS ProductCount
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/product_data/products.csv',
            FORMAT = 'CSV',
            PARSER_VERSION='2.0',
            HEADER_ROW = TRUE
        ) AS [result]
    GROUP BY Category;
    ```

7. Execute a consulta modificada, que retornará um conjunto de resultados contendo o número de produtos em cada categoria, desta forma:

    | Categoria | ProductCount |
    | -- | -- |
    | Bib Shorts | 3 |
    | Racks de bicicleta | 1 |
    | ... | ... |

8. No painel **Propriedades** do **Script SQL 1**, altere o **Nome** para **Contar Produtos por Categoria**. Em seguida, na barra de ferramentas, selecione **Publicar** para salvar o script.

9. Feche o painel do script **Contar Produtos por Categoria**.

10. No Synapse Studio, selecione a página **Desenvolver** e observe que seu script SQL **Contar Produtos por Categoria** publicado foi salvo nela.

11. Selecione o script SQL **Contar Produtos por Categoria** para abri-lo novamente. Em seguida, verifique se o script está conectado ao pool de SQL **Interno** e execute-o para recuperar as contagens de produtos.

12. No painel de **Resultados**, selecione o modo de exibição de **Gráfico** e selecione as seguintes configurações para o gráfico:
    - **Tipo de gráfico**: Coluna
    - **Coluna de categoria**: Categoria
    - **Colunas de legenda (série)**: ProductCount
    - **Posição da legenda**: Inferior central
    - **Rótulo de legenda (série)**: *Deixar em branco*
    - **Valor mínimo da legenda (série)**: *Deixar em branco*
    - **Máximo da legenda (série)**: *Deixar em branco*
    - **Rótulo da categoria**: *Deixar em branco*

    O gráfico resultante deve ser semelhante a este:

    ![Imagem mostrando a exibição de gráfico de contagem de produtos](./images/column-chart.png)

## Usar um pool do Spark para analisar os dados

Embora a linguagem SQL seja comum para a consulta de conjuntos de dados estruturados, muitos analistas de data consideram linguagens como Python úteis para explorar e preparar dados para análise. No Azure Synapse Analytics, você pode executar o código Python (e outros) em um *pool do Spark*; que usa um mecanismo de processamento de dados distribuído com base em Apache Spark.

1. no Synapse Studio, se a guia **arquivos** que você abriu anteriormente contendo o arquivo **products.csv** não estiver mais aberta, na página **Dados** , procure a pasta **product_data**. Em seguida, clique com o botão direito do mouse em **products.csv**, aponte para **Novo notebook** e selecione **Carregar no DataFrame**.
2. No painel **Notebook 1** que será exibido, na lista **Anexar a**, selecione o pool do Spark **sparkxxxxxxx** e verifique se a **Linguagem** está configurada como **PySpark (Python)**.
3. Examine o código na primeira (e única) célula no notebook, que terá esta aparência:

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/product_data/products.csv', format='csv'
    ## If header exists uncomment line below
    ##, header=True
    )
    display(df.limit(10))
    ```

4. Use o ícone de **&#9655;** à esquerda da célula de código para executá-lo e aguarde os resultados. Da primeira vez que você executar uma célula em um notebook, o pool do Spark será iniciado. Portanto, qualquer resultado poderá levar cerca de um minuto para ser retornado.
5. Eventualmente, os resultados deverão aparecer abaixo da célula e ser semelhantes a este:

    | _c0_ | _c1_ | _c2_ | _c3_ |
    | -- | -- | -- | -- |
    | ProductID | ProductName | Categoria | ListPrice |
    | 771 | Mountain-100 Silver, 38 | Mountain bikes | 3399.9900 |
    | 772 | Mountain-100 Silver, 42 | Mountain bikes | 3399.9900 |
    | ... | ... | ... | ... |

6. Descompacte a linha *header=True* (porque o arquivo products.csv tem os títulos de coluna na primeira linha), para que seu código tenha esta aparência:

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/product_data/products.csv', format='csv'
    ## If header exists uncomment line below
    , header=True
    )
    display(df.limit(10))
    ```

7. Execute novamente a célula e verifique se o resultado terá esta aparência:

    | ProductID | ProductName | Categoria | ListPrice |
    | -- | -- | -- | -- |
    | 771 | Mountain-100 Silver, 38 | Mountain bikes | 3399.9900 |
    | 772 | Mountain-100 Silver, 42 | Mountain bikes | 3399.9900 |
    | ... | ... | ... | ... |

    Observe que executar a célula novamente leva menos tempo, porque o pool do Spark já foi iniciado.

8. Nos resultados, use o ícone **&#65291; Código** para adicionar uma nova célula de código ao notebook.
9. Na nova célula de código vazia, adicione o seguinte código:

    ```Python
    df_counts = df.groupby(df.Category).count()
    display(df_counts)
    ```

10. Execute a nova célula de código clicando no ícone **&#9655;** e revise os resultados, que devem ser semelhantes a este:

    | Categoria | count |
    | -- | -- |
    | Fones de ouvido | 3 |
    | Rodas | 14 |
    | ... | ... |

11. Na saída dos resultados da célula, selecione o modo de exibição de **Gráfico.** O gráfico resultante deve ser semelhante a este:

    ![Imagem mostrando a exibição do gráfico de contagem de categorias](./images/bar-chart.png)

12. Se ainda não estiver visível, exiba a página **Propriedades** selecionando o botão **Propriedades** (que se parece com **<sub>*</sub>**) na extremidade direita da barra de ferramentas. Em seguida, no painel **Propriedades**, altere o nome do notebook para **Explorar produtos** e use o botão **Publicar** na barra de ferramentas para salvá-lo.

13. Feche o painel do notebook e pare a sessão do Spark quando solicitado. Em seguida, exiba a página **Desenvolver** para verificar se o bloco de anotações foi salvo.

## Usar um pool de SQL dedicado para consultar um data warehouse

Até agora você viu algumas técnicas para explorar e processar dados baseados em arquivos em um data lake. Em muitos casos, uma solução de análise corporativa usa um data lake para armazenar e preparar dados não estruturados que podem ser carregados em um data warehouse relacional para dar suporte a cargas de trabalho de BI (business intelligence). No Azure Synapse Analytics, esses data warehouses podem ser implementados em um pool de SQL dedicado.

1. No Synapse Studio, na página **Gerenciar**, na seção **pools de SQL**, selecione a linha de pool de SQL dedicado **sql*xxxxxxx*** e, em seguida, use o ícone **▷** para retomar.
2. Aguarde até que o pool de SQL seja iniciado. Isso pode levar alguns minutos. Use o botão **↻ Atualizar** para verificar seu status periodicamente. O status será exibido como **Online** quando estiver pronto.
3. Quando o pool de SQL for iniciado, selecione a página **Dados** e, na guia **Workspace**, expanda **Bancos de dados SQL** e verifique se **sql*xxxxxxx*** está listado (use o ícone **↻** no canto superior esquerdo da página para atualizar a exibição, se necessário).
4. Expanda o banco de dados **sql*xxxxxxx*** e a pasta **Tabelas** e, em seguida, no menu **...** para a tabela **FactInternetSales**, aponte para **Novo script de SQL** e selecione **Selecionar as 100 PRIMEIRAS LINHAS**.
5. Revise os resultados da consulta, que mostram as primeiras 100 transações de vendas na tabela. Esses dados foram carregados no banco de dados pelo script de instalação e são armazenados permanentemente no banco de dados associado ao pool de SQL dedicado.
6. Substitua a consulta SQL pelo código a seguir:

    ```sql
    SELECT d.CalendarYear, d.MonthNumberOfYear, d.EnglishMonthName,
           p.EnglishProductName AS Product, SUM(o.OrderQuantity) AS UnitsSold
    FROM dbo.FactInternetSales AS o
    JOIN dbo.DimDate AS d ON o.OrderDateKey = d.DateKey
    JOIN dbo.DimProduct AS p ON o.ProductKey = p.ProductKey
    GROUP BY d.CalendarYear, d.MonthNumberOfYear, d.EnglishMonthName, p.EnglishProductName
    ORDER BY d.MonthNumberOfYear
    ```

7. Use o botão **▷ Executar** para executar a consulta modificada, que retorna a quantidade de cada produto vendido por ano e mês.
8. Se ainda não estiver visível, mostre a página **Propriedades** selecionando o botão **Propriedades** (que se parece com **<sub>*</sub>**) na extremidade direita da barra de ferramentas. Em seguida, no painel **Propriedades**, altere o nome da consulta para **Agregar vendas de produtos** e use o botão **Publicar** na barra de ferramentas para salvá-lo.

9. Feche o painel de consulta e exiba a página **Desenvolver** para verificar se o script SQL foi salvo.

10. Na página **Gerenciar**, selecione a linha do pool de SQL dedicado **sql*xxxxxxx*** e use o ícone ❚ ❚ para pausá-lo.

<!--- ## Explore data with a Data Explorer pool

Azure Synapse Data Explorer provides a runtime that you can use to store and query data by using Kusto Query Language (KQL). Kusto is optimized for data that includes a time series component, such as realtime data from log files or IoT devices.

### Create a Data Explorer database and ingest data into a table

1. In Synapse Studio, on the **Manage** page, in the **Data Explorer pools** section, select the **adx*xxxxxxx*** pool row and then use its **&#9655;** icon to resume it.
2. Wait for the pool to start. It can take some time. Use the **&#8635; Refresh** button to check its status periodically. The status will show as **online** when it is ready.
3. When the Data Explorer pool has started, view the **Data** page; and on the **Workspace** tab, expand **Data Explorer Databases** and verify that **adx*xxxxxxx*** is listed (use **&#8635;** icon at the top-left of the page to refresh the view if necessary)
4. In the **Data** pane, use the **&#65291;** icon to create a new **Data Explorer database** in the **adx*xxxxxxx*** pool with the name **sales-data**.
5. In Synapse Studio, wait for the database to be created (a notification will be displayed).
6. Switch to the **Develop** page, and in the **+** menu, add a KQL script. Then, when the script pane opens, in the **Connect to** list, select your **adx*xxxxxxx*** pool, and in the **Database** list, select **sales-data**.
7. In the new script, add the following code:

    ```kusto
    .create table sales (
        SalesOrderNumber: string,
        SalesOrderLineItem: int,
        OrderDate: datetime,
        CustomerName: string,
        EmailAddress: string,
        Item: string,
        Quantity: int,
        UnitPrice: real,
        TaxAmount: real)
    ```

8. On the toolbar, use the **&#9655; Run** button to run the selected code, which creates a table named **sales** in the **sales-data** database you created previously.
9. After the code has run successfully, replace it with the following code, which loads data into the table:

    ```kusto
    .ingest into table sales 'https://raw.githubusercontent.com/microsoftlearning/dp-203-azure-data-engineer/master/Allfiles/labs/01/files/sales.csv' 
    with (ignoreFirstRecord = true)
    ```

10. Run the new code to ingest the data.

> **Note**: In this example, you imported a very small amount of batch data from a file, which is fine for the purposes of this exercise. In reality, you can use Data Explorer to analyze much larger volumes of data; including realtime data from a streaming source such as Azure Event Hubs.

### Use Kusto query language to query the table

1. Switch back to the **Data** page and in the **...** menu for the **sales-data** database, select **Refresh**.
2. Expand the **sales-data** database's **Tables** folder. Then in the **...** menu for the **sales** table, select **New KQL script** > **Take 1000 rows**.
3. Review the generated query and its results. The query should contain the following code:

    ```kusto
    sales
    | take 1000
    ```

    The results of the query contain the first 1000 rows of data.

4. Modify the query as follows:

    ```kusto
    sales
    | where Item == 'Road-250 Black, 48'
    ```

5. Use the **&#9655; Run** button to run the query. Then review the results, which should contain only the rows for sales orders for the *Road-250 Black, 48* product.

6. Modify the query as follows:

    ```kusto
    sales
    | where Item == 'Road-250 Black, 48'
    | where datetime_part('year', OrderDate) > 2020
    ```

7. Run the query and review the results, which should contain only sales orders for *Road-250 Black, 48* made after 2020.

8. Modify the query as follows:

    ```kusto
    sales
    | where OrderDate between (datetime(2020-01-01 00:00:00) .. datetime(2020-12-31 23:59:59))
    | summarize TotalNetRevenue = sum(UnitPrice) by Item
    | sort by Item asc
    ```

9. Run the query and review the results, which should contain the total net revenue for each product between January 1st and December 31st 2020 in ascending order of product name.

10. If it is not already visible, show the **Properties** page by selecting the **Properties** button (which looks similar to **&#128463;<sub>*</sub>**) on the right end of the toolbar. Then in the **Properties** pane, change the query name to **Explore sales data** and use the **Publish** button on the toolbar to save it.

11. Close the query pane, and then view the **Develop** page to verify that the KQL script has been saved.

12. On the **Manage** page, select the **adx*xxxxxxx*** Data Explorer pool row and use its &#10074;&#10074; icon to pause it. --->

## Excluir recursos do Azure

Agora que você terminou de explorar Azure Synapse Analytics, exclua os recursos que criou para evitar custos desnecessários do Azure.

1. Feche a guia do navegador do Synapse Studio e retorne ao portal do Azure.
2. No portal do Azure, na **Página Inicial**, selecione **Grupos de recursos**.
3. Selecione o grupo de recursos **dp203-*xxxxxxx*** para o espaço de trabalho do Synapse Analytics (não o grupo de recursos gerenciado) e verifique que ele contém o espaço de trabalho do Synapse, a conta de armazenamento, o pool de SQL e o pool do Spark para seu espaço de trabalho.
4. Na parte superior da página de **Visão Geral** do grupo de recursos, selecione **Excluir o grupo de recursos**.
5. Digite o nome do grupo de recursos **dp203-*xxxxxxx*** para confirmar que deseja excluí-lo e selecione **Excluir**.

    Após alguns minutos, o grupo de recursos de seu workspace do Azure Synapse e o grupo de recursos do workspace gerenciado associado a ele serão excluídos.
