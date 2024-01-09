---
lab:
  title: Usar o Delta Lake no Azure Synapse Analytics
  ilt-use: Lab
---

# Usar o Delta Lake com Spark no Azure Synapse Analytics

Delta Lake é um projeto de código aberto para construir uma camada de armazenamento de dados transacionais sobre um Data Lake. O Delta Lake adiciona compatibilidade com semântica relacional em operações de dados em lote e streaming, e permite a criação de uma arquitetura de *Lakehouse* na qual o Apache Spark pode ser usado para processar e consultar dados em tabelas baseadas em arquivos subjacentes no Data Lake.

Este exercício levará aproximadamente **40** minutos para ser concluído.

## Antes de começar

É necessário ter uma [assinatura do Azure](https://azure.microsoft.com/free) com acesso de nível administrativo.

## Provisionar um workspace do Azure Synapse Analytics

Você precisará de um workspace do Azure Synapse Analytics com acesso ao armazenamento do Data Lake e um pool do Apache Spark que possa usar para consultar e processar arquivos no Data Lake.

Neste exercício, você usará uma combinação de um script do PowerShell e um modelo ARM para provisionar um workspace do Azure Synapse Analytics.

1. Entre no [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`.
2. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um novo Cloud Shell no portal do Azure, selecionando um ambiente ***PowerShell*** e criando um armazenamento caso solicitado. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure, conforme mostrado aqui:

    ![Portal do Azure com um painel do Cloud Shell](./images/cloud-shell.png)

    > **Observação**: se você tiver criado anteriormente um Cloud Shell que usa um ambiente *Bash*, use o menu suspenso no canto superior esquerdo do painel do Cloud Shell para alterá-lo para ***PowerShell***.

3. Observe que você pode redimensionar o Cloud Shell arrastando a barra do separador na parte superior do painel ou usando os ícones **&#8212;** , **&#9723;** e **X** no canto superior direito do painel para minimizar, maximizar e fechar o painel. Para obter mais informações de como usar o Azure Cloud Shell, confira a [documentação do Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. No painel do PowerShell, insira os seguintes comandos para clonar este repositório:

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Depois que o repositório tiver sido clonado, digite os seguintes comandos para alterar para a pasta deste exercício e execute o script **setup.ps1** que ele contém:

    ```
    cd dp-203/Allfiles/labs/07
    ./setup.ps1
    ```

6. Se solicitado, escolha qual assinatura deseja usar (isso só acontecerá se você tiver acesso a várias assinaturas do Azure).
7. Quando solicitado, insira uma senha adequada que será definida para seu pool do SQL do Azure Synapse.

    > **Observação**: lembre-se dessa senha!

8. Aguarde a conclusão do script – isso normalmente leva cerca de 10 minutos, mas em alguns casos pode levar mais tempo. Enquanto espera, revise o artigo [O que é Delta Lake](https://docs.microsoft.com/azure/synapse-analytics/spark/apache-spark-what-is-delta-lake) na documentação do Azure Synapse Analytics.

## Criar tabelas Delta

O script provisiona um workspace do Azure Synapse Analytics e uma conta de Armazenamento do Azure para hospedar o Data Lake e, em seguida, carrega um arquivo de dados nele.

### Explorar os dados no Data Lake

1. Depois que o script for concluído, no portal do Azure, vá para o grupo de recursos **dp203-*xxxxxxx*** que ele criou e selecione seu workspace do Synapse.
2. Na página **Visão geral** do seu workspace do Synapse, no cartão **Abrir o Synapse Studio**, selecione **Abrir** para abrir o Synapse Studio em uma nova guia do navegador, fazendo login se solicitado.
3. No lado esquerdo do Synapse Studio, use o ícone **&rsaquo;&rsaquo;** para expandir o menu – o que revela as diferentes páginas do Synapse Studio usadas para gerenciar recursos e executar tarefas de análise de dados.
4. Na página **Dados**, exiba a guia **Vinculado** e verifique se seu workspace inclui um link para sua conta de armazenamento do Azure Data Lake Storage Gen2, que deve ter um nome semelhante à **synapse*xxxxxxx* (Primary - datalake*xxxxxxx*)**.
5. Expanda sua conta de armazenamento e verifique se ela contém um contêiner do sistema de arquivos chamado **arquivos**.
6. Selecione o contêiner de **arquivos** e observe que ele contém uma pasta chamada **produtos**. Esta pasta contém os dados com os quais você trabalhará neste exercício.
7. Abra a pasta **produtos** e observe que ela contém um arquivo chamado **products.csv**.
8. Selecione **products.csv** e, na lista **Novo notebook** na barra de ferramentas, selecione **Carregar para DataFrame**.
9. No painel **Notebook 1** que será exibido, na lista **Anexar a**, selecione o pool do Spark **sparkxxxxxxx** e certifique se de que a **Linguagem** está definida como **PySpark (Python)**.
10. Examine o código na primeira (e única) célula no notebook, que terá esta aparência:

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/products/products.csv', format='csv'
    ## If header exists uncomment line below
    ##, header=True
    )
    display(df.limit(10))
    ```

11. Descompacte a linha *header=True* (porque o arquivo products.csv tem os títulos de coluna na primeira linha), para que seu código tenha esta aparência:

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/products/products.csv', format='csv'
    ## If header exists uncomment line below
    , header=True
    )
    display(df.limit(10))
    ```

12. Use o ícone de **&#9655;** à esquerda da célula de código para executá-lo e aguarde os resultados. Da primeira vez que você executar uma célula em um notebook, o pool do Spark será iniciado. Portanto, qualquer resultado poderá levar cerca de um minuto para ser retornado. Eventualmente, os resultados deverão aparecer abaixo da célula e ser semelhantes a este:

    | ProductID | ProductName | Categoria | ListPrice |
    | -- | -- | -- | -- |
    | 771 | Mountain-100 Silver, 38 | Mountain bikes | 3399.9900 |
    | 772 | Mountain-100 Silver, 42 | Mountain bikes | 3399.9900 |
    | ... | ... | ... | ... |

### Carregar os dados do arquivo em uma tabela Delta

1. Nos resultados retornados pela primeira célula de código, use o botão **+ Código** para adicionar uma nova célula de código. Em seguida, insira o seguinte código na nova célula e execute-o:

    ```Python
    delta_table_path = "/delta/products-delta"
    df.write.format("delta").save(delta_table_path)
    ```

2. Na guia **arquivos**, use o ícone **↑** na barra de ferramentas para retornar à raiz do contêiner de **arquivos** e observe que uma nova pasta chamada **delta** foi criada. Abra esta pasta e a tabela **products-delta** que ela contém, onde você deve ver os arquivos de formato parquet que contém os dados.

3. Retorne à guia **Notebook 1** e adicione outra nova célula de código. Em seguida, na nova célula, adicione o seguinte código e execute-o:

    ```Python
    from delta.tables import *
    from pyspark.sql.functions import *

    # Create a deltaTable object
    deltaTable = DeltaTable.forPath(spark, delta_table_path)

    # Update the table (reduce price of product 771 by 10%)
    deltaTable.update(
        condition = "ProductID == 771",
        set = { "ListPrice": "ListPrice * 0.9" })

    # View the updated data as a dataframe
    deltaTable.toDF().show(10)
    ```

    Os dados são carregados em um objeto **DeltaTable** e atualizados. Você poderá ver a atualização refletida nos resultados da consulta.

4. Adicione outra nova célula de código e execute o seguinte código:

    ```Python
    new_df = spark.read.format("delta").load(delta_table_path)
    new_df.show(10)
    ```

    O código carrega os dados da tabela Delta em um quadro de dados a partir de seu local no Data Lake, verificando se a alteração feita por meio de um objeto **DeltaTable** foi persistente.

5. Modifique o código que você acabou de executar da seguinte maneira, especificando a opção de usar o recurso de *viagem no tempo* do Delta Lake para exibir uma versão anterior dos dados.

    ```Python
    new_df = spark.read.format("delta").option("versionAsOf", 0).load(delta_table_path)
    new_df.show(10)
    ```

    Quando você executa o código modificado, os resultados mostram a versão original dos dados.

6. Adicione outra nova célula de código e execute o seguinte código:

    ```Python
    deltaTable.history(10).show(20, False, True)
    ```

    O histórico das últimas 20 alterações na tabela é mostrado – deve haver duas (a criação original e a atualização que você fez).

## Criar tabelas de catálogo

Até agora, você trabalhou com tabelas Delta carregando dados da pasta que contém os arquivos parquet nos quais a tabela se baseia. Você pode definir *tabelas de catálogo* que encapsulam os dados e fornecem uma entidade de tabela nomeada que você pode referenciar no código SQL. O Spark é compatível com dois tipos de tabelas de catálogo para o Delta Lake:

- Tabelas *externas*, que são definidas pelo caminho para os arquivos parquet que contêm os dados da tabela.
- Tabelas *gerenciadas*, que são definidas no metastore do Hive para o pool do Spark.

### Criar uma tabela externa

1. Em uma nova célula de código, adicione e execute o seguinte código:

    ```Python
    spark.sql("CREATE DATABASE AdventureWorks")
    spark.sql("CREATE TABLE AdventureWorks.ProductsExternal USING DELTA LOCATION '{0}'".format(delta_table_path))
    spark.sql("DESCRIBE EXTENDED AdventureWorks.ProductsExternal").show(truncate=False)
    ```

    Esse código cria um novo banco de dados chamado **AdventureWorks** e, em seguida, cria uma tabela externa chamada **ProductsExternal** nesse banco de dados com base no caminho para os arquivos parquet definidos anteriormente. Em seguida, exibe uma descrição das propriedades da tabela. Observe que a propriedade **Location** é o caminho especificado.

2. Adicione uma nova célula de código e, em seguida, insira e execute o seguinte código:

    ```sql
    %%sql

    USE AdventureWorks;

    SELECT * FROM ProductsExternal;
    ```

    O código usa SQL para alternar o contexto para o banco de dados **AdventureWorks** (que não retorna dados) e, em seguida, consultar a tabela **ProductsExternal** (que retorna um conjunto de resultados contendo os dados de produtos na tabela Delta Lake).

### Criar uma tabela gerenciada

1. Em uma nova célula de código, adicione e execute o seguinte código:

    ```Python
    df.write.format("delta").saveAsTable("AdventureWorks.ProductsManaged")
    spark.sql("DESCRIBE EXTENDED AdventureWorks.ProductsManaged").show(truncate=False)
    ```

    Esse código cria uma tabela gerenciada chamada **ProductsManaged** com base no DataFrame que você carregou originalmente do arquivo **products.csv** (antes de atualizar o preço do produto 771). Você não especifica um caminho para os arquivos parquet usados pela tabela – isso é gerenciado para você no metastore do Hive e mostrado na propriedade **Location** na descrição da tabela (no caminho **files/synapse/workspaces/synapsexxxxxxx/warehouse**).

2. Adicione uma nova célula de código e, em seguida, insira e execute o seguinte código:

    ```sql
    %%sql

    USE AdventureWorks;

    SELECT * FROM ProductsManaged;
    ```

    O código usa SQL para consultar a tabela **ProductsManaged**.

### Comparar tabelas externas e gerenciadas

1. Em uma nova célula de código, adicione e execute o seguinte código:

    ```sql
    %%sql

    USE AdventureWorks;

    SHOW TABLES;
    ```

    Esse código lista as tabelas no banco de dados **AdventureWorks**.

2. Modifique a célula de código da seguinte maneira e execute:

    ```sql
    %%sql

    USE AdventureWorks;

    DROP TABLE IF EXISTS ProductsExternal;
    DROP TABLE IF EXISTS ProductsManaged;
    ```

    Esse código descarta as tabelas do metastore.

3. Retorne à guia **arquivos** e visualize a pasta **files/delta/products-delta**. Observe que os arquivos de dados ainda existem nesse local. Descartar a tabela externa removeu a tabela do metastore, mas deixou os arquivos de dados intactos.
4. Exiba a pasta **files/synapse/workspaces/synapsexxxxxxx/warehouse** e observe que não há nenhuma pasta para os dados da tabela **ProductsManaged**. Soltar uma tabela gerenciada remove a tabela do metastore e também exclui os arquivos de dados da tabela.

### Criar uma tabela usando SQL

1. Adicione uma nova célula de código e, em seguida, insira e execute o seguinte código:

    ```sql
    %%sql

    USE AdventureWorks;

    CREATE TABLE Products
    USING DELTA
    LOCATION '/delta/products-delta';
    ```

2. Adicione uma nova célula de código e, em seguida, insira e execute o seguinte código:

    ```sql
    %%sql

    USE AdventureWorks;

    SELECT * FROM Products;
    ```

    Observe que a nova tabela de catálogo foi criada para a pasta de tabela Delta Lake existente, que reflete as alterações feitas anteriormente.

## Usar tabelas delta para transmitir dados

O Delta Lake dá suporte a dados de streaming. As tabelas delta podem ser um *coletor* ou uma *fonte* para fluxos de dados criados por meio da API de Streaming Estruturado do Spark. Neste exemplo, você usará uma tabela delta como um coletor para alguns dados de streaming em um cenário simulado de IoT (Internet das Coisas).

1. Retorne à guia **Notebook 1** e adicione uma nova célula de código. Em seguida, na nova célula, adicione o seguinte código e execute-o:

    ```python
    from notebookutils import mssparkutils
    from pyspark.sql.types import *
    from pyspark.sql.functions import *

    # Create a folder
    inputPath = '/data/'
    mssparkutils.fs.mkdirs(inputPath)

    # Create a stream that reads data from the folder, using a JSON schema
    jsonSchema = StructType([
    StructField("device", StringType(), False),
    StructField("status", StringType(), False)
    ])
    iotstream = spark.readStream.schema(jsonSchema).option("maxFilesPerTrigger", 1).json(inputPath)

    # Write some event data to the folder
    device_data = '''{"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev2","status":"error"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"error"}
    {"device":"Dev2","status":"ok"}
    {"device":"Dev2","status":"error"}
    {"device":"Dev1","status":"ok"}'''
    mssparkutils.fs.put(inputPath + "data.txt", device_data, True)
    print("Source stream created...")
    ```

    Verifique se a mensagem *Fluxo de origem criado…* está impressa. O código que você acabou de executar criou uma fonte de dados de streaming com base em uma pasta na qual alguns dados foram salvos, representando leituras de dispositivos IoT hipotéticos.

2. Em uma nova célula de código, adicione e execute o seguinte código:

    ```python
    # Write the stream to a delta table
    delta_stream_table_path = '/delta/iotdevicedata'
    checkpointpath = '/delta/checkpoint'
    deltastream = iotstream.writeStream.format("delta").option("checkpointLocation", checkpointpath).start(delta_stream_table_path)
    print("Streaming to delta sink...")
    ```

    Esse código grava os dados do dispositivo de streaming no formato delta.

3. Em uma nova célula de código, adicione e execute o seguinte código:

    ```python
    # Read the data in delta format into a dataframe
    df = spark.read.format("delta").load(delta_stream_table_path)
    display(df)
    ```

    Esse código lê os dados transmitidos no formato delta em um dataframe. Observe que o código para carregar dados de streaming não é diferente daquele usado para carregar dados estáticos de uma pasta delta.

4. Em uma nova célula de código, adicione e execute o seguinte código:

    ```python
    # create a catalog table based on the streaming sink
    spark.sql("CREATE TABLE IotDeviceData USING DELTA LOCATION '{0}'".format(delta_stream_table_path))
    ```

    Esse código cria uma tabela de catálogo chamada **IotDeviceData** (no banco de dados **padrão** ) com base na pasta delta. Novamente, esse código é o mesmo que seria usado para dados que não são de streaming.

5. Em uma nova célula de código, adicione e execute o seguinte código:

    ```sql
    %%sql

    SELECT * FROM IotDeviceData;
    ```

    Esse código consulta a tabela **IotDeviceData**, que contém os dados do dispositivo da fonte de streaming.

6. Em uma nova célula de código, adicione e execute o seguinte código:

    ```python
    # Add more data to the source stream
    more_data = '''{"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"error"}
    {"device":"Dev2","status":"error"}
    {"device":"Dev1","status":"ok"}'''

    mssparkutils.fs.put(inputPath + "more-data.txt", more_data, True)
    ```

    Esse código grava mais dados hipotéticos do dispositivo na fonte de streaming.

7. Em uma nova célula de código, adicione e execute o seguinte código:

    ```sql
    %%sql

    SELECT * FROM IotDeviceData;
    ```

    Esse código consulta a tabela **IotDeviceData** novamente, que agora incluirá os dados extras que foram adicionados à fonte de streaming.

8. Em uma nova célula de código, adicione e execute o seguinte código:

    ```python
    deltastream.stop()
    ```

    Esse código interrompe o fluxo.

## Consultar uma tabela delta a partir de um pool de SQL sem servidor

Além dos pools do Spark, o Azure Synapse Analytics inclui um pool de SQL interno sem servidor. Você pode usar o mecanismo de banco de dados relacional nesse pool para consultar tabelas delta usando SQL.

1. Na guia **arquivos**, navegue até a pasta **arquivos/delta**.
2. Selecione a pasta **products-delta** e, na barra de ferramentas, na lista suspensa **Novo script de SQL**, selecione **Seleionar as 100 primeiras linhas**.
3. No painel **Selecionar as 100 primeiras linhas**, na lista **Tipo do arquivo**, selecione **formato Delta** e, em seguida, selecione **Aplicar**.
4. Revise o código SQL gerado, que deve ter a seguinte aparência:

    ```sql
    -- This is auto-generated code
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/delta/products-delta/',
            FORMAT = 'DELTA'
        ) AS [result]
    ```

5. Use o ícone **▷ Executar** para executar o script e revise os resultados. Eles devem ser semelhantes a este:

    | ProductID | ProductName | Categoria | ListPrice |
    | -- | -- | -- | -- |
    | 771 | Mountain-100 Silver, 38 | Mountain bikes | 3059.991 |
    | 772 | Mountain-100 Silver, 42 | Mountain bikes | 3399.9900 |
    | ... | ... | ... | ... |

    Isso demonstra como você pode usar um pool de SQL sem servidor para consultar arquivos de formato delta que foram criados usando o Spark e usar os resultados para relatórios ou análises.

6. Substitua a consulta pelo código SQL a seguir:

    ```sql
    USE AdventureWorks;

    SELECT * FROM Products;
    ```

7. Execute o código e observe que você também pode usar o pool de SQL sem servidor para consultar dados Delta Lake em tabelas de catálogo definidas no metastore do Spark.

## Excluir recursos do Azure

Se você terminou de explorar Azure Synapse Analytics, exclua os recursos que criou para evitar custos desnecessários do Azure.

1. Feche a guia do navegador do Synapse Studio e retorne ao portal do Azure.
2. No portal do Azure, na **Página Inicial**, selecione **Grupos de recursos**.
3. Selecione o grupo de recursos **dp203-*xxxxxxx*** para o workspace do Synapse Analytics (não o grupo de recursos gerenciado) e verifique se ele contém o workspace do Synapse, a conta de armazenamento e o pool do Spark para seu workspace.
4. Na parte superior da página de **Visão Geral** do grupo de recursos, selecione **Excluir o grupo de recursos**.
5. Digite o nome do grupo de recursos **dp203-*xxxxxxx*** para confirmar que deseja excluí-lo e selecione **Excluir**.

    Após alguns minutos, seu grupo de recursos do workspace do Azure Synapse e o grupo de recursos do workspace gerenciado associado a ele serão excluídos.
