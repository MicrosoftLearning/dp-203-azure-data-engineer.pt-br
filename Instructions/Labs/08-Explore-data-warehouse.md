---
lab:
  title: Explorar um data warehouse relacional
  ilt-use: Suggested demo
---

# Explorar um data warehouse relacional

O Azure Synapse Analytics baseia-se num conjunto escalável de capacidades para suportar o armazenamento de dados empresariais; incluindo análise de dados baseada em arquivos em um data lake, bem como data warehouses relacionais em grande escala e os pipelines de transferência e transformação de dados usados​para carregá-los. Neste laboratório, você explorará como usar um pool do SQL dedicado no Azure Synapse Analytics para armazenar e consultar dados em um data warehouse relacional.

Este laboratório levará aproximadamente **45** minutos para ser concluído.

## Antes de começar

É necessário ter uma [assinatura do Azure](https://azure.microsoft.com/free) com acesso de nível administrativo.

## Provisionar um workspace do Azure Synapse Analytics

Um *workspace* do Azure Synapse Analytics fornece um ponto central para gerenciar dados e tempos de execução de processamento de dados. Você pode provisionar um workspace usando a interface interativa no portal do Azure ou pode implantar um workspace e recursos nele usando um script ou modelo. Na maioria dos cenários de produção, é melhor automatizar o provisionamento com scripts e modelos para que você possa incorporar a implantação de recursos em um processo de desenvolvimento e operações (*DevOps*) repetível.

Neste exercício, você usará uma combinação de um script do PowerShell e um modelo ARM para provisionar o Azure Synapse Analytics.

1. Entre no [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`.
2. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure, selecionando um ambiente ***PowerShell*** e criando armazenamento caso solicitado. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure, conforme mostrado aqui:

    ![Portal do Azure com um painel do Cloud Shell](./images/cloud-shell.png)

    > **Observação**: se você tiver criado anteriormente um cloud shell que usa um ambiente *Bash*, use o menu suspenso no canto superior esquerdo do painel do cloud shell para alterá-lo para ***PowerShell***.

3. Observe que você pode redimensionar o Cloud Shell arrastando a barra do separador na parte superior do painel ou usando os ícones **&#8212;** , **&#9723;** e **X** no canto superior direito do painel para minimizar, maximizar e fechar o painel. Para obter mais informações de como usar o Azure Cloud Shell, confira a [documentação do Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. No painel do PowerShell, insira os seguintes comandos para clonar esse repositório:

    ```
    rm -r dp500 -f
    git clone https://github.com/MicrosoftLearning/DP-500-Azure-Data-Analyst dp500
    ```

5. Depois que o repositório tiver sido clonado, digite os seguintes comandos para alterar para a pasta deste laboratório e execute o script **setup.ps1** que ele contém:

    ```
    cd dp500/Allfiles/03
    ./setup.ps1
    ```

6. Se solicitado, escolha qual assinatura você deseja usar (isso só acontecerá se você tiver acesso a várias assinaturas do Azure).
7. Quando solicitado, insira uma senha adequada a ser definida para seu pool de SQL do Azure Synapse.

    > **Obsrevação**: lembre-se dessa senha.

8. Aguarde a conclusão do script – isso normalmente leva cerca de 15 minutos, mas em alguns casos pode demorar mais. Enquanto espera, revise o artigo [O que é o pool de SQL dedicado no Azure Synapse Analytics?](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is) na documentação do Azure Synapse Analytics.

## Explore o esquema do data warehouse

Neste laboratório, o data warehouse está hospedado em um pool de SQL dedicado no Azure Synapse Analytics.

### Comece o pool de SQL dedicado

1. Depois que o script for concluído, no portal do Azure, vá para o grupo de recursos **dp500-*xxxxxxx*** que ele criou e selecione seu workspace Sinapse.
2. Na página **Visão geral** do seu workspace Synapse, no cartão **Abrir Synapse Studio**, selecione **Abrir** para abrir o Synapse Studio em uma nova guia do navegador, fazendo login se solicitado.
3. No lado esquerdo do Synapse Studio, use o ícone **&rsaquo;&rsaquo;** para expandir o menu, o que revela as diferentes páginas do Synapse Studio usadas para gerenciar recursos e executar tarefas de análise de dados.
4. Na página **Gerenciar**, certifique-se de que a guia **Pools de SQL** esteja selecionada e, em seguida, selecione o pool de SQL dedicado **sql*xxxxxxx*** e use seu ícone **&#9655;** para iniciá-lo; confirmando que deseja retomá-lo quando solicitado.
5. Aguarde até que o pool de SQL seja retomado. Isso pode levar alguns minutos. Use o botão **↻ Atualizar** para verificar seu status periodicamente. O status será exibido como **Online** quando estiver pronto.

### Exibir as tabelas no banco de dados

1. No Synapse Studio, selecione a página **Dados** e verifique se a guia **Workspace** está selecionada e contém uma categoria de **banco de dados SQL**.
2. Expanda o **banco de dados SQL**, o pool **sql*xxxxxxx*** e sua pasta **Tabelas** para ver as tabelas no banco de dados.

    Um data warehouse relacional é normalmente baseado em um esquema que consiste tabelas de *fatos* e *dimensões*. As tabelas são otimizadas para consultas analíticas nas quais as métricas numéricas nas tabelas de fatos são agregadas por atributos das entidades representadas pelas tabelas de dimensão – por exemplo, permitindo agregar receitas de vendas pela Internet por produto, cliente, data e assim por diante.
    
3. Expanda a tabela **dbo.FactInternetSales** e a pasta **Colunas** para ver as colunas nesta tabela. Observe que muitas das colunas são *chaves* que fazem referência a linhas nas tabelas de dimensão. Outros são valores numéricos (*medidas*) para análise.
    
    As chaves são usadas para relacionar uma tabela de fatos a uma ou mais tabelas de dimensão, geralmente em um esquema em *estrelas*, no qual a tabela de fatos está diretamente relacionada a cada tabela de dimensões (formando uma "estrela" de várias pontas com a tabela de fatos no centro).

4. Visualize as colunas da tabela **dbo.DimPromotion** e observe que ela possui um **PromotionKey** exclusivo que identifica exclusivamente cada linha da tabela. Também tem uma **AlternateKey**.

    Normalmente, os dados em um data warehouse foram importados de uma ou mais fontes transacionais. A chave *alternativa* reflete o identificador de negócios para a instância dessa entidade na origem, mas uma chave *substituta *numérica e exclusiva geralmente é gerada para identificar exclusivamente cada linha na tabela de dimensão do data warehouse. Um dos benefícios dessa abordagem é que ela permite que o data warehouse contenha várias instâncias da mesma entidade em diferentes momentos (por exemplo, registros para o mesmo cliente refletindo seu endereço no momento em que um pedido foi feito).

5. Visualize as colunas de **dbo.DimProduct** e observe que ela contém uma coluna **ProductSubcategoryKey**, que faz referência à tabela **dbo.DimProductSubcategory**, que por sua vez contém uma coluna **ProductCategoryKey** que faz referência à tabela **dbo.DimProductCategory**.

    Em alguns casos, as dimensões são parcialmente normalizadas em várias tabelas relacionadas para permitir diferentes níveis de granularidade – como produtos que podem ser agrupados em subcategorias e categorias. Isso resulta em uma estrela simples sendo estendida para um esquema de *snowflake*, no qual a tabela de fatos central está relacionada a uma tabela de dimensões, que é relacionada a tabelas de dimensões posteriores.

6. Exiba as colunas da tabela **dbo.DimDate** e observe que ela contém várias colunas que refletem diferentes atributos temporais de uma data – incluindo o dia da semana, dia do mês, mês, ano, nome do dia, nome do mês e assim por diante.

    As dimensões de tempo em um data warehouse geralmente são implementadas como uma tabela de dimensões que contém uma linha para cada uma das menores unidades temporais de granularidade (geralmente chamada *grão* da dimensão) pela qual você deseja agregar as medidas nas tabelas de fatos. Nesse caso, o grão mais baixo no qual as medidas podem ser agregadas é uma data individual, e a tabela contém uma linha para cada data da primeira à última data referenciada nos dados. Os atributos na tabela **DimDate** permitem que os analistas agreguem medidas com base em qualquer chave de data na tabela de fatos, usando um conjunto consistente de atributos temporais (por exemplo, exibindo pedidos por mês com base na data do pedido). A tabela **FactInternetSales** contém três chaves relacionadas à tabela **DimDate**: **OrderDateKey**, **DueDateKey** e **ShipDateKey**.

## Consultar as tabelas do data warehouse

Agora que você explorou alguns dos aspectos mais importantes do esquema de data warehouse, está pronto para consultar as tabelas e recuperar alguns dados.

### Consultar tabelas de fatos e tabelas de dimensões

Os valores numéricos em um data warehouse relacional são armazenados em tabelas de fato com tabelas de dimensão relacionadas que você pode usar para agregar os dados em vários atributos. Este design significa que a maioria das consultas de um data warehouse relacional envolve a agregação e o agrupamento de dados (por meio de funções de agregação e cláusulas GROUP BY) entre tabelas relacionadas (usando as cláusulas JOIN).

1. Na página **Dados**, selecione o pool de SQL **sql*xxxxxxx*** e no menu **...**, selecione **Novo script de SQL** > **Script Vazio**.
2. Quando uma nova guia **Script de SQL 1** for aberta, em seu painel **Propriedades**, altere o nome do script para **Examinar Vendas pela Internet** e altere as **Configurações de resultado por consulta** para retornar todas as linhas. Em seguida, use o botão **Publicar** na barra de ferramentas para salvar o script e use o botão **Propriedades** (que é semelhante a **&#128463;.**) na extremidade direita da barra de ferramentas para fecharo painel **Propriedades** de modo que você possa ver o painel do script.
3. No script vazio, adicione o seguinte código:

    ```sql
    SELECT  d.CalendarYear AS Year,
            SUM(i.SalesAmount) AS InternetSalesAmount
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    GROUP BY d.CalendarYear
    ORDER BY Year;
    ```

4. Use o botão **Executar &#9655;** para executar o script, examinar resultados, que devem mostrar as vendas totais pela Internet de cada ano. Esta consulta une a tabela de fatos para vendas pela Internet a uma tabela de dimensão de tempo baseada na data do pedido e agrega a medida do valor das vendas na tabela de fatos pelo atributo mês calendário da tabela de dimensões.

5. Modifique a consulta da seguinte forma para adicionar o atributo mês da dimensão de tempo e, em seguida, execute a consulta modificada.

    ```sql
    SELECT  d.CalendarYear AS Year,
            d.MonthNumberOfYear AS Month,
            SUM(i.SalesAmount) AS InternetSalesAmount
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    GROUP BY d.CalendarYear, d.MonthNumberOfYear
    ORDER BY Year, Month;
    ```

    Observe que os atributos na dimensão temporal permitem agregar as medidas na tabela de fatos em vários níveis hierárquicos, nesse caso, ano e mês. Esse é um padrão comum em data warehouses.

6. Modifique a consulta da seguinte forma para remover o mês e adicionar uma segunda dimensão à agregação e, em seguida, execute-a para exibir os resultados (que mostram os totais anuais de vendas pela Internet para cada região):

    ```sql
    SELECT  d.CalendarYear AS Year,
            g.EnglishCountryRegionName AS Region,
            SUM(i.SalesAmount) AS InternetSalesAmount
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
    JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
    GROUP BY d.CalendarYear, g.EnglishCountryRegionName
    ORDER BY Year, Region;
    ```

    Observe que a geografia é uma dimensão de *snowflake* que está relacionada à tabela de fatos de vendas pela Internet através da dimensão cliente. Portanto, você precisa de duas junções na consulta para agregar as vendas pela Internet por geografia.

7. Modifique e execute novamente a consulta para adicionar outra dimensão de snowflake e agregar as vendas regionais anuais por categoria de produto:

    ```sql
    SELECT  d.CalendarYear AS Year,
            pc.EnglishProductCategoryName AS ProductCategory,
            g.EnglishCountryRegionName AS Region,
            SUM(i.SalesAmount) AS InternetSalesAmount
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
    JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
    JOIN DimProduct AS p ON i.ProductKey = p.ProductKey
    JOIN DimProductSubcategory AS ps ON p.ProductSubcategoryKey = ps.ProductSubcategoryKey
    JOIN DimProductCategory AS pc ON ps.ProductCategoryKey = pc.ProductCategoryKey
    GROUP BY d.CalendarYear, pc.EnglishProductCategoryName, g.EnglishCountryRegionName
    ORDER BY Year, ProductCategory, Region;
    ```

    Desta vez, a dimensão de floco de neve para categoria de produto requer três junções para refletir a relação hierárquica entre produtos, subcategorias e categorias.

8. Publique o script para salvar.

### Usar as funções de classificação

Outro requisito comum ao analisar grandes volumes de dados é agrupar os dados por partições e determinar a *classificação* de cada entidade na partição com base em uma métrica específica.

1. Na consulta existente, adicione o seguinte SQL para recuperar valores de vendas para 2022 em partições com base no nome do país/região:

    ```sql
    SELECT  g.EnglishCountryRegionName AS Region,
            ROW_NUMBER() OVER(PARTITION BY g.EnglishCountryRegionName
                              ORDER BY i.SalesAmount ASC) AS RowNumber,
            i.SalesOrderNumber AS OrderNo,
            i.SalesOrderLineNumber AS LineItem,
            i.SalesAmount AS SalesAmount,
            SUM(i.SalesAmount) OVER(PARTITION BY g.EnglishCountryRegionName) AS RegionTotal,
            AVG(i.SalesAmount) OVER(PARTITION BY g.EnglishCountryRegionName) AS RegionAverage
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
    JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
    WHERE d.CalendarYear = 2022
    ORDER BY Region;
    ```

2. Selecione apenas o novo código de consulta e use o botão **Executar &#9655;** para executá-lo. Em seguida, analise os resultados, que devem ser semelhantes à tabela a seguir:

    | Região | RowNumber | OrderNo | LineItem | SalesAmount | RegionTotal | RegionAverage |
    |--|--|--|--|--|--|--|
    |Austrália|1|SO73943|2|2.2900|2172278,7900|375,8918|
    |Austrália|2|SO74100|4|2.2900|2172278,7900|375,8918|
    |...|...|...|...|...|...|...|
    |Austrália|5779|SO64284|1|2443,3500|2172278,7900|375,8918|
    |Canadá|1|SO66332|2|2.2900|563177,1000|157,8411|
    |Canadá|2|SO68234|2|2.2900|563177,1000|157,8411|
    |...|...|...|...|...|...|...|
    |Canadá|3568|SO70911|1|2443,3500|563177,1000|157,8411|
    |França|1|SO68226|3|2.2900|816259,4300|315,4016|
    |França|2|SO63460|2|2.2900|816259,4300|315,4016|
    |...|...|...|...|...|...|...|
    |França|2588|SO69100|1|2443,3500|816259,4300|315,4016|
    |Alemanha|1|SO70829|3|2.2900|922368,2100|352,4525|
    |Alemanha|2|SO71651|2|2.2900|922368,2100|352,4525|
    |...|...|...|...|...|...|...|
    |Alemanha|2617|SO67908|1|2443,3500|922368,2100|352,4525|
    |Reino Unido|1|SO66124|3|2.2900|1051560,1000|341,7484|
    |Reino Unido|2|SO67823|3|2.2900|1051560,1000|341,7484|
    |...|...|...|...|...|...|...|
    |Reino Unido|3077|SO71568|1|2443,3500|1051560,1000|341,7484|
    |Estados Unidos|1|SO74796|2|2.2900|2905011,1600|289,0270|
    |Estados Unidos|2|SO65114|2|2.2900|2905011,1600|289,0270|
    |...|...|...|...|...|...|...|
    |Estados Unidos|10051|SO66863|1|2443,3500|2905011,1600|289,0270|

    Observe os seguintes fatos sobre esses resultados:

    - Há uma linha para cada item de linha da ordem do cliente.
    - As linhas são organizadas em divisórias com base na geografia onde a venda foi feita.
    - As linhas dentro de cada partição geográfica são numeradas em ordem de quantidade de vendas (da menor para a maior).
    - Para cada linha, o valor de vendas do item de linha, bem como o total regional e os valores médios de vendas são incluídos.

3. Nas consultas existentes, adicione o seguinte código para aplicar funções de janela em uma consulta GROUP BY e classifique as cidades em cada região com base no valor total das vendas:

    ```sql
    SELECT  g.EnglishCountryRegionName AS Region,
            g.City,
            SUM(i.SalesAmount) AS CityTotal,
            SUM(SUM(i.SalesAmount)) OVER(PARTITION BY g.EnglishCountryRegionName) AS RegionTotal,
            RANK() OVER(PARTITION BY g.EnglishCountryRegionName
                        ORDER BY SUM(i.SalesAmount) DESC) AS RegionalRank
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
    JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
    GROUP BY g.EnglishCountryRegionName, g.City
    ORDER BY Region;
    ```

4. Selecione apenas o novo código de consulta e use o botão **Executar &#9655;** para executá-lo. Execute o seguinte e observe os resultados.
    - Os resultados incluem uma linha para cada cidade, agrupada por região.
    - O total de vendas (soma dos valores de vendas individuais) é calculado para cada cidade
    - O total de vendas regionais (a soma dos valores de vendas para cada cidade da região) é calculado com base na partição regional.
    - A classificação para cada cidade dentro de sua partição regional é calculada ordenando o valor total de vendas por cidade em ordem decrescente.

5. Publique o script atualizado para salvar as alterações.

> **Dica**: ROW_NUMBER e RANK são exemplos de funções de classificação disponíveis no Transact-SQL. Para saber mais sobre essas funções, confira a referência [Funções de Classificação](https://docs.microsoft.com/sql/t-sql/functions/ranking-functions-transact-sql) na documentação da linguagem Transact-SQL.

### Recuperar uma contagem aproximada

Ao explorar grandes volumes de dados, as consultas podem levar tempo e recursos significativos para serem executadas. Muitas vezes, a análise de dados não requer valores absolutamente precisos – uma comparação de valores aproximados pode ser suficiente.

1. Nas consultas existentes, adicione o seguinte código para recuperar o número de ordens de venda para cada ano civil:

    ```sql
    SELECT d.CalendarYear AS CalendarYear,
        COUNT(DISTINCT i.SalesOrderNumber) AS Orders
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    GROUP BY d.CalendarYear
    ORDER BY CalendarYear;
    ```

2. Selecione apenas o novo código de consulta e use o botão **Executar &#9655;** para executá-lo. Em seguida, revise a saída retornada:
    - Na guia **Resultados** da consulta, exiba as contagens de pedidos para cada ano.
    - Na guia **Mensagens**, exiba o tempo total de execução da consulta.
3. Modifique a consulta da seguinte maneira para retornar uma contagem aproximada para cada ano. Em seguida, executar novamente a consulta.

    ```sql
    SELECT d.CalendarYear AS CalendarYear,
        APPROX_COUNT_DISTINCT(i.SalesOrderNumber) AS Orders
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    GROUP BY d.CalendarYear
    ORDER BY CalendarYear;
    ```

4. Examine a saída retornada:
    - Na guia **Resultados** da consulta, exiba as contagens de pedidos para cada ano. Elas devem estar dentro de 2% das contagens reais recuperadas pela consulta anterior.
    - Na guia **Mensagens**, exiba o tempo total de execução da consulta. Ele deve ser mais curto do que para a consulta anterior.

5. Publique o script para salvar as alterações.

> **Dica**: confira a documentação da função [APPROX_COUNT_DISTINCT](https://docs.microsoft.com/sql/t-sql/functions/approx-count-distinct-transact-sql) para obter mais detalhes.

## Desafio – Analisar as vendas do revendedor

1. Crie um novo script vazio para o pool de SQL **sql*xxxxxxx**** e salve-o com o nome **Analisar as vendas do revendedor**.
2. Crie consultas SQL no script para localizar as seguintes informações com base na tabela de fatos **FactResellerSales** e nas tabelas de dimensão às quais ela está relacionada:
    - A quantidade total de itens vendidos por ano fiscal e trimestre.
    - A quantidade total de itens vendidos por ano fiscal, trimestre e região do território de vendas associada ao funcionário que fez a venda.
    - A quantidade total de itens vendidos por ano fiscal, trimestre e região de território de vendas por categoria de produto.
    - A classificação de cada território de vendas por ano fiscal com base no valor total de vendas do ano.
    - O número aproximado do pedido de venda por ano em cada território de vendas.

    > **Dica**: Compare suas consultas com as do script **Solução** na página **Desenvolver** no Synapse Studio.

3. Experimente consultas para explorar o restante das tabelas no esquema de data warehouse de acordo com a sua conveniência.
4. Quando terminar, na página **Gerenciar**, pause o pool de SQL dedicado **sql*xxxxxxx***.

## Excluir recursos do Azure

Se você terminou de explorar Azure Synapse Analytics, exclua os recursos que criou para evitar custos desnecessários do Azure.

1. Feche a guia do navegador do Synapse Studio e retorne ao portal do Azure.
2. No portal do Azure, na **Página Inicial**, selecione **Grupos de recursos**.
3. Selecione o grupo de recursos **dp500-*xxxxxxx*** para o workspace do Synapse Analytics (não o grupo de recursos gerenciado) e verifique se ele contém o workspace do Synapse, a conta de armazenamento e o pool do SQL dedicado para seu workspace.
4. Na parte superior da página de **Visão Geral** do grupo de recursos, selecione **Excluir o grupo de recursos**.
5. Digite o nome do grupo de recursos **dp500-*xxxxxxx*** para confirmar que deseja excluí-lo e selecione **Excluir**.

    Após alguns minutos, seu grupo de recursos do workspace do Azure Synapse e o grupo de recursos do workspace gerenciado associado a ele serão excluídos.
