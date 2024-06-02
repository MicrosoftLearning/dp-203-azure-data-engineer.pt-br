---
lab:
  title: Analisar dados em um data lake com o Spark
  module: 'Model, query, and explore data in Azure Synapse'
---

# Analisar dados em um data lake com o Spark

O Apache Spark é um mecanismo de código aberto para processamento de dados distribuído e é amplamente usado para explorar, processar e analisar grandes volumes de dados no data lake storage. O Spark está disponível como uma opção de processamento em vários produtos de plataforma de dados, incluindo o Azure HDInsight, o Azure Databricks, o Azure Synapse Analytics e o Microsoft o Azure Cloud. Um dos benefícios do Spark é o suporte a uma ampla variedade de linguagens de programação, incluindo Java, Scala, Python e SQL, tornando o Spark uma solução muito flexível para cargas de trabalho de processamento de dados, incluindo limpeza e processamento de dados, análise estatística e machine learning, análise e visualização de dados.

Este laboratório levará aproximadamente **45** minutos para ser concluído.

## Antes de começar

É necessário ter uma [assinatura do Azure](https://azure.microsoft.com/free) com acesso de nível administrativo.

## Provisionar um workspace do Azure Synapse Analytics

Você precisará de um workspace do Azure Synapse Analytics com acesso ao armazenamento do Data Lake e um pool do Apache Spark que possa usar para consultar e processar arquivos no Data Lake.

Neste exercício, você usará uma combinação de um script do PowerShell e um modelo ARM para provisionar um workspace do Azure Synapse Analytics.

1. Entre no [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`.
2. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure, selecionando um ambiente ***PowerShell*** e criando armazenamento caso solicitado. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure, conforme mostrado aqui:

    ![Portal do Azure com um painel do Cloud Shell](./images/cloud-shell.png)

    > **Observação**: se você tiver criado anteriormente um cloud shell que usa um ambiente *Bash*, use o menu suspenso no canto superior esquerdo do painel do cloud shell para alterá-lo para ***PowerShell***.

3. Observe que você pode redimensionar o Cloud Shell arrastando a barra do separador na parte superior do painel ou usando os ícones **&#8212;** , **&#9723;** e **X** no canto superior direito do painel para minimizar, maximizar e fechar o painel. Para obter mais informações de como usar o Azure Cloud Shell, confira a [documentação do Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. No painel do PowerShell, insira os seguintes comandos para clonar esse repositório:

    ```
    rm -r dp203 -f
    git clone https://github.com/MicrosoftLearning/DP-203-Azure-Data-Engineer dp203
    ```

5. Depois que o repositório tiver sido clonado, digite os seguintes comandos para alterar para a pasta deste laboratório e execute o script **setup.ps1** que ele contém:

    ```
    cd dp203/Allfiles/02
    ./setup.ps1
    ```

6. Se solicitado, escolha qual assinatura você deseja usar (isso só acontecerá se você tiver acesso a várias assinaturas do Azure).
7. Quando solicitado, insira uma senha adequada a ser definida para seu pool de SQL do Azure Synapse.

    > **Observação**: Memorize a senha.

8. Aguarde a conclusão do script – isso normalmente leva cerca de 10 minutos, mas em alguns casos pode demorar mais. Enquanto espera, revise o artigo [Apache Spark no Azure Synapse Analytics](https://docs.microsoft.com/azure/synapse-analytics/spark/apache-spark-overview) na documentação do Azure Synapse Analytics.

## Consultar dados em arquivos

O script provisiona um espaço de trabalho do Azure Synapse Analytics e uma conta de Armazenamento do Azure para hospedar o data lake e, em seguida, carrega alguns arquivos de dados no data lake.

### Exibir arquivos no data lake

1. Depois que o script for concluído, no portal do Azure, vá para o grupo de recursos **dp500-*xxxxxxx*** que ele criou e selecione seu workspace Sinapse.
2. Na página **Visão geral** do seu workspace Synapse, no cartão **Abrir Synapse Studio**, selecione **Abrir** para abrir o Synapse Studio em uma nova guia do navegador, fazendo login se solicitado.
3. No lado esquerdo do Synapse Studio, use o ícone **&rsaquo;&rsaquo;** para expandir o menu, o que revela as diferentes páginas do Synapse Studio usadas para gerenciar recursos e executar tarefas de análise de dados.
4. Na página **Gerenciar**, selecione a guia **Pools do Apache Spark** e observe que um pool do Spark com um nome semelhante ao **spark*xxxxxxx*** foi provisionado no workspace. Posteriormente, você usará esse pool do Spark para carregar e analisar dados de arquivos no armazenamento do data lake para o workspace.
5. Na página **Dados**, exiba a guia **Vinculado** e verifique se seu workspace inclui um link para sua conta de armazenamento do Azure Data Lake Storage Gen2, que deve ter um nome semelhante à **sinapse*xxxxxxx* (Principal – datalake*xxxxxxx*)**.
6. Expanda sua conta de armazenamento e verifique se ela contém um contêiner do sistema de arquivos chamado **arquivos**.
7. Selecione o contêiner de **arquivos** e observe que ele contém pastas chamadas **vendas** e **sinapse**. A pasta **sinapse** é usada pelo Azure Synapse e a pasta **vendas** contém os arquivos de dados que você vai consultar.
8. Abra a pasta de **vendas** e a pasta **pedidos** contida nela e observe que a pasta **pedidos** contém arquivos de .csv para três anos de dados de vendas.
9. Clique com o botão direito do mouse em qualquer um dos arquivos e selecione **Visualizar** para ver os dados que ele contém. Observe que os arquivos não contêm uma linha de cabeçalho, portanto, você pode desmarcar a opção para exibir cabeçalhos de coluna.

### Use o Spark para explorar dados

1. Selecione qualquer um dos arquivos na pasta **pedidos** e, na lista **Novo bloco de anotações** na barra de ferramentas, selecione **Carregar para DataFrame**. Um dataframe é uma estrutura no Spark que representa um conjunto de dados tabular.
2. Na nova guia **Bloco de Anotações 1** que é aberta, na lista **Anexar a**, selecione seu pool do Spark (**spark*xxxxxxx***). Em seguida, use o botão **▷ Executar tudo** para executar todas as células do notebook (atualmente há apenas uma!).

    Como esta é a primeira vez que você executa qualquer código Spark nesta sessão, o pool do Spark precisa ser iniciado. Isso significa que a primeira execução na sessão pode levar alguns minutos. As execuções seguintes serão mais rápidas.

3. Enquanto aguarda a inicialização da sessão do Spark, revise o código gerado; que se parece com isto:

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/sales/orders/2019.csv', format='csv'
    ## If header exists uncomment line below
    ##, header=True
    )
    display(df.limit(10))
    ```

4. Quando o código terminar de ser executado, revise a saída abaixo da célula no bloco de anotações. Ele mostra as primeiras dez linhas no arquivo selecionado, com nomes de coluna automáticos no formato **_c0**, _c1 **, ****_c2** e assim por diante.
5. Modifique o código para que a função **spark.read.load** leia dados de <u>todos</u> os arquivos CSV na pasta e a função de **exibição** mostre as primeiras 100 linhas. Seu código deve ter esta aparência (com *datalakexxxxxxx* correspondente ao nome do seu armazenamento de data lake):

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/sales/orders/*.csv', format='csv'
    )
    display(df.limit(100))
    ```

6. Use o botão **&#9655;** à esquerda da célula de código para executar apenas essa célula e revisar os resultados.

    O dataframe agora inclui dados de todos os arquivos, mas os nomes das colunas não são úteis. O Spark usa uma abordagem de “esquema em leitura” para tentar determinar os tipos de dados apropriados para as colunas com base nos dados que elas contêm e, se uma linha de cabeçalho estiver presente em um arquivo de texto, ela poderá ser usada para identificar os nomes das colunas (especificando um parâmetro **header=True** na **função **load). Como alternativa, você pode definir um esquema explícito para o dataframe.

7. Modifique o código da seguinte forma (substituindo *datalakexxxxxxx*), para definir um esquema explícito para o dataframe que inclui os nomes de coluna e tipos de dados. Execute novamente o código na célula.

    ```Python
    %%pyspark
    from pyspark.sql.types import *
    from pyspark.sql.functions import *

    orderSchema = StructType([
        StructField("SalesOrderNumber", StringType()),
        StructField("SalesOrderLineNumber", IntegerType()),
        StructField("OrderDate", DateType()),
        StructField("CustomerName", StringType()),
        StructField("Email", StringType()),
        StructField("Item", StringType()),
        StructField("Quantity", IntegerType()),
        StructField("UnitPrice", FloatType()),
        StructField("Tax", FloatType())
        ])

    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/sales/orders/*.csv', format='csv', schema=orderSchema)
    display(df.limit(100))
    ```

8. Nos resultados, use botão **+ Código** para adicionar uma nova célula de código ao notebook. Em seguida, na nova célula, adicione o seguinte código para exibir o esquema do dataframe:

    ```Python
    df.printSchema()
    ```

9. Execute a nova célula e verifique se o esquema de dataframe corresponde ao **orderSchema** que você definiu. A função **printSchema** pode ser útil ao usar um dataframe com um esquema inferido automaticamente.

## Análise de dados em um dataframe

O objeto **de dataframe** no Spark é semelhante a um dataframe do Pandas em Python e inclui uma ampla variedade de funções que você pode usar para Manipular filtrar, agrupar e processar os dados que ele contém.

### Filtrar um dataframe

1. Adicione uma nova célula de código ao notebook e insira o seguinte código nela:

    ```Python
    customers = df['CustomerName', 'Email']
    print(customers.count())
    print(customers.distinct().count())
    display(customers.distinct())
    ```

2. Execute a nova célula de código e analise os resultados. Observe os seguintes detalhes:
    - Quando você executa uma operação em um dataframe, o resultado é um novo dataframe (nesse caso, um dataframe **customers** é criado pela seleção de um subconjunto específico de colunas do dataframe **df**)
    - Os dataframes fornecem funções como **count** e **distinct** que podem ser usadas para resumir e filtrar os dados que eles contêm.
    - A sintaxe `dataframe['Field1', 'Field2', ...]` é uma forma abreviada de definir um subconjunto de colunas. Você também pode usar o método **select**, para que a primeira linha do código acima possa ser escrita como `customers = df.select("CustomerName", "Email")`

3. Modifique o código da seguinte maneira:

    ```Python
    customers = df.select("CustomerName", "Email").where(df['Item']=='Road-250 Red, 52')
    print(customers.count())
    print(customers.distinct().count())
    display(customers.distinct())
    ```

4. Execute o código modificado para visualizar os clientes que compraram o produto *Road-250 Red, 52*. Observe que você pode "encadear" várias funções para que a saída de uma função se torne a entrada da próxima. Nesse caso, o dataframe criado pelo método **select** é o dataframe de origem do método **where** usado para aplicar os critérios de filtragem.

### Agregar e agrupar dados em um dataframe

1. Adicione uma nova célula de código ao notebook e insira o seguinte código nela:

    ```Python
    productSales = df.select("Item", "Quantity").groupBy("Item").sum()
    display(productSales)
    ```

2. Execute a célula de código que você adicionou e observe que os resultados mostram a soma das quantidades de pedidos agrupadas por produto. O método **groupBy** agrupa as linhas por *Item*, e a função de agregação de **soma** seguinte é aplicada a todas as colunas numéricas restantes (nesse caso, *Quantidade*)

3. Adicione outra nova célula de código ao notebook e insira o seguinte código nela:

    ```Python
    yearlySales = df.select(year("OrderDate").alias("Year")).groupBy("Year").count().orderBy("Year")
    display(yearlySales)
    ```

4. Execute a célula de código que você adicionou e observe que os resultados mostram o número de pedidos de vendas por ano. Observe que o método **select** inclui uma função **year** do  SQL para extrair o componente de ano do campo *OrderDate* e, em seguida, um método de **alias** é usado para atribuir um nome de coluna ao valor de ano extraído. Em seguida, os dados são agrupados pela coluna *Year* derivada, e a contagem de linhas em cada grupo é calculada antes de finalmente o método **orderBy** ser usado para classificar o dataframe resultante.

## Consultar dados usando Spark SQL

Como você viu, os métodos nativos do objeto de dataframe permitem que você consulte e analise os dados com bastante eficiência. No entanto, muitos analistas de dados se sentem mais à vontade em trabalhar com a sintaxe SQL. O Spark SQL é uma API de linguagem SQL no Spark que você pode usar para executar instruções SQL ou até mesmo persistir dados em tabelas relacionais.

### Usar o Spark SQL no código PySpark

A linguagem padrão nos blocos de anotações do Azure Synapse Studio é PySpark, que é um tempo de execução Python baseado no Spark. Nesse tempo de execução, você pode usar a biblioteca **spark.sql** para incorporar a sintaxe do Spark SQL em seu código Python e trabalhar com construções SQL, como tabelas e exibições.

1. Adicione uma nova célula de código ao notebook e insira o seguinte código nela:

    ```Python
    df.createOrReplaceTempView("salesorders")

    spark_df = spark.sql("SELECT * FROM salesorders")
    display(spark_df)
    ```

2. Execute a célula e analise os resultados. Observe que:
    - O código mantém os dados no **dataframe df** como uma exibição temporária chamada **salesorders**. O Spark SQL oferece suporte ao uso de exibições temporárias ou tabelas persistentes como fontes para consultas SQL.
    - O método **spark.sql** é usado para executar uma consulta SQL no modo de exibição **salesorders**.
    - Os resultados da consulta estão no formato de um DataFrame do Pandas.

### Executar um código SQL em uma célula

Embora seja útil a inserção de instruções SQL em uma célula que contém um código PySpark, os analistas de dados costumam desejar apenas trabalhar diretamente no SQL.

1. Adicione uma nova célula de código ao notebook e insira o seguinte código nela:

    ```sql
    %%sql
    SELECT YEAR(OrderDate) AS OrderYear,
           SUM((UnitPrice * Quantity) + Tax) AS GrossRevenue
    FROM salesorders
    GROUP BY YEAR(OrderDate)
    ORDER BY OrderYear;
    ```

2. Execute a célula e analise os resultados. Observe que:
    - A linha `%%sql` no início da célula (chamada *magic*) indica que o runtime da linguagem Spark SQL deve ser usado para executar o código nessa célula em vez do PySpark.
    - O código SQL referencia a exibição **salesorders** que você criou antes usando PySpark.
    - A saída da consulta SQL é exibida automaticamente como o resultado abaixo da célula.

> **Observação**: para obter mais informações sobre o Spark SQL e os dataframes, confira a [documentação do Spark SQL](https://spark.apache.org/docs/2.2.0/sql-programming-guide.html).

## Visualizar os dados com o Spark

Como o provérbio diz, uma imagem vale mil palavras, e um gráfico geralmente é melhor do que mil linhas de dados. Embora os notebooks do Azure Synapse Analytics incluam uma exibição de gráfico interna para dados exibidos em um dataframe ou em uma consulta Spark SQL, ele não foi projetado para gráficos abrangentes. No entanto, você pode usar bibliotecas de elementos gráficos do Python, como a **matplotlib** e a **seaborn**, para criar gráficos com base em dados em dataframes.

### Exibir os resultados como um gráfico

1. Adicione uma nova célula de código ao notebook e insira o seguinte código nela:

    ```sql
    %%sql
    SELECT * FROM salesorders
    ```

2. Execute o código e observe que ele retorna os dados da exibição **salesorders** que você já criou.
3. Na seção de resultados abaixo da célula, altere a opção **Exibir** de **Tabela** para **Gráfico**.
4. Use o botão **Exibir opções** no canto superior direito do gráfico para exibir o painel de opções do gráfico. Em seguida, defina as opções da seguinte maneira e selecione **Aplicar**:
    - **Tipo de gráfico**: Gráfico de barras
    - **Chave**: Item
    - **Valores**: Quantidade
    - **Grupo de Séries**: *deixe em branco*
    - **Agregação**: Soma
    - **Empilhado**: *Não selecionado*

5. Verifique se o gráfico é parecido com este:

    ![Um gráfico de barras de produtos pela quantidade total de pedidos](./images/notebook-chart.png)

### Introdução à **matplotlib**

1. Adicione uma nova célula de código ao notebook e insira o seguinte código nela:

    ```Python
    sqlQuery = "SELECT CAST(YEAR(OrderDate) AS CHAR(4)) AS OrderYear, \
                    SUM((UnitPrice * Quantity) + Tax) AS GrossRevenue \
                FROM salesorders \
                GROUP BY CAST(YEAR(OrderDate) AS CHAR(4)) \
                ORDER BY OrderYear"
    df_spark = spark.sql(sqlQuery)
    df_spark.show()
    ```

2. Execute o código e observe se ele retorna um dataframe do Spark que contém a receita anual.

    Para visualizar os dados como um gráfico, começaremos usando a biblioteca **matplotlib** do Python. Essa biblioteca é a biblioteca de plotagem principal na qual muitas outras se baseiam e fornece muita flexibilidade na criação de gráficos.

3. Adicione uma nova célula de código ao notebook e adicione o seguinte código a ele:

    ```Python
    from matplotlib import pyplot as plt

    # matplotlib requires a Pandas dataframe, not a Spark one
    df_sales = df_spark.toPandas()

    # Create a bar plot of revenue by year
    plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'])

    # Display the plot
    plt.show()
    ```

4. Execute a célula e analise os resultados, que consistem em um gráfico de colunas com a receita bruta total de cada ano. Observe os seguintes recursos do código usado para produzir este gráfico:
    - A biblioteca **matplotlib** exige um dataframe do *Pandas*, ou seja, você precisa converter o dataframe do *Spark* retornado pela consulta Spark SQL nesse formato.
    - No núcleo da biblioteca **matplotlib** está o objeto **pyplot**. Essa é a base para a maioria das funcionalidades de plotagem.
    - As configurações padrão resultam em um gráfico utilizável, mas há um escopo considerável para personalizá-lo

5. Modifique o código para plotar o gráfico da seguinte maneira:

    ```Python
    # Clear the plot area
    plt.clf()

    # Create a bar plot of revenue by year
    plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')

    # Customize the chart
    plt.title('Revenue by Year')
    plt.xlabel('Year')
    plt.ylabel('Revenue')
    plt.grid(color='#95a5a6', linestyle='--', linewidth=2, axis='y', alpha=0.7)
    plt.xticks(rotation=45)

    # Show the figure
    plt.show()
    ```

6. Execute novamente a célula de código e veja os resultados. O gráfico agora inclui um pouco mais de informações.

    Tecnicamente, um gráfico está contido com uma **Figura**. Nos exemplos anteriores, a figura foi criada implicitamente, mas você pode criá-la de modo explícito.

7. Modifique o código para plotar o gráfico da seguinte maneira:

    ```Python
    # Clear the plot area
    plt.clf()

    # Create a Figure
    fig = plt.figure(figsize=(8,3))

    # Create a bar plot of revenue by year
    plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')

    # Customize the chart
    plt.title('Revenue by Year')
    plt.xlabel('Year')
    plt.ylabel('Revenue')
    plt.grid(color='#95a5a6', linestyle='--', linewidth=2, axis='y', alpha=0.7)
    plt.xticks(rotation=45)

    # Show the figure
    plt.show()
    ```

8. Execute novamente a célula de código e veja os resultados. A figura determina a forma e o tamanho do gráfico.

    Uma figura pode conter vários subgráficos, cada um em um *eixo* próprio.

9. Modifique o código para plotar o gráfico da seguinte maneira:

    ```Python
    # Clear the plot area
    plt.clf()

    # Create a figure for 2 subplots (1 row, 2 columns)
    fig, ax = plt.subplots(1, 2, figsize = (10,4))

    # Create a bar plot of revenue by year on the first axis
    ax[0].bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')
    ax[0].set_title('Revenue by Year')

    # Create a pie chart of yearly order counts on the second axis
    yearly_counts = df_sales['OrderYear'].value_counts()
    ax[1].pie(yearly_counts)
    ax[1].set_title('Orders per Year')
    ax[1].legend(yearly_counts.keys().tolist())

    # Add a title to the Figure
    fig.suptitle('Sales Data')

    # Show the figure
    plt.show()
    ```

10. Execute novamente a célula de código e veja os resultados. A figura contém os subgráficos especificados no código.

> **Observação**: para saber mais sobre a plotagem com a matplotlib, confira a [documentação da matplotlib](https://matplotlib.org/).

### Usar a biblioteca **seaborn**

Embora a **matplotlib** permita que você crie gráficos complexos de vários tipos, ele pode exigir um código complexo para obter os melhores resultados. Por esse motivo, ao longo dos anos, muitas bibliotecas foram criadas na base na matplotlib para abstrair a complexidade e aprimorar as funcionalidades. Uma dessas bibliotecas é a **seaborn**.

1. Adicione uma nova célula de código ao notebook e insira o seguinte código nela:

    ```Python
    import seaborn as sns

    # Clear the plot area
    plt.clf()

    # Create a bar chart
    ax = sns.barplot(x="OrderYear", y="GrossRevenue", data=df_sales)
    plt.show()
    ```

2. Execute o código e observe que ele exibe um gráfico de barras usando a biblioteca seaborn.
3. Adicione uma nova célula de código ao notebook e insira o seguinte código nela:

    ```Python
    # Clear the plot area
    plt.clf()

    # Set the visual theme for seaborn
    sns.set_theme(style="whitegrid")

    # Create a bar chart
    ax = sns.barplot(x="OrderYear", y="GrossRevenue", data=df_sales)
    plt.show()
    ```

4. Execute o código e observe que a seaborn permite que você defina um tema de cor consistente para seus gráficos.

5. Adicione uma nova célula de código ao notebook e insira o seguinte código nela:

    ```Python
    # Clear the plot area
    plt.clf()

    # Create a bar chart
    ax = sns.lineplot(x="OrderYear", y="GrossRevenue", data=df_sales)
    plt.show()
    ```

6. Execute o código para ver a receita anual como um gráfico de linhas.

> **Observação**: para saber mais sobre como fazer uma plotagem com a seaborn, confira a [documentação da seaborn](https://seaborn.pydata.org/index.html).

## Excluir recursos do Azure

Se você terminou de explorar Azure Synapse Analytics, exclua os recursos que criou para evitar custos desnecessários do Azure.

1. Feche a guia do navegador do Synapse Studio e retorne ao portal do Azure.
2. No portal do Azure, na **Página Inicial**, selecione **Grupos de recursos**.
3. Selecione o grupo de recursos **dp500-*xxxxxxx*** para o workspace do Synapse Analytics (não o grupo de recursos gerenciado) e verifique se ele contém o workspace do Synapse, a conta de armazenamento e o pool do Spark para seu workspace.
4. Na parte superior da página de **Visão Geral** do grupo de recursos, selecione **Excluir o grupo de recursos**.
5. Digite o nome do grupo de recursos **dp500-*xxxxxxx*** para confirmar que deseja excluí-lo e selecione **Excluir**.

    Após alguns minutos, seu grupo de recursos do workspace do Azure Synapse e o grupo de recursos do workspace gerenciado associado a ele serão excluídos.
