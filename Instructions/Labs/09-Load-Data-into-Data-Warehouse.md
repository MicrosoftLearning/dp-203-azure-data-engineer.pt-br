---
lab:
  title: Carregar dados em um data warehouse relacional
  ilt-use: Lab
---

# Carregar dados em um data warehouse relacional

Neste exercício, você carregará dados em um pool de SQL dedicado.

Este exercício deve levar aproximadamente **30** minutos para ser concluído.

## Antes de começar

É necessário ter uma [assinatura do Azure](https://azure.microsoft.com/free) com acesso de nível administrativo.

## Provisionar um workspace do Azure Synapse Analytics

Você precisará de um workspace do Azure Synapse Analytics com acesso ao armazenamento Data Lake e um pool de SQL dedicado hospedando um data warehouse.

Neste exercício, você usará uma combinação de um script do PowerShell e um modelo ARM para provisionar um workspace do Azure Synapse Analytics.

1. Entre no [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`.
2. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um novo Cloud Shell no portal do Azure, selecionando um ambiente ***PowerShell*** e criando um armazenamento caso solicitado. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure, conforme mostrado aqui:

    ![Portal do Azure com um painel do Cloud Shell](./images/cloud-shell.png)

    > **Observação**: se você tiver criado anteriormente um Cloud Shell que usa um ambiente *Bash*, use o menu suspenso no canto superior esquerdo do painel do Cloud Shell para alterá-lo para ***PowerShell***.

3. Você pode redimensionar o Cloud Shell arrastando a barra do separador na parte superior do painel ou usando os ícones — , **&#9723;** e **X** no canto superior direito do painel para minimizar, maximizar e fechar o painel. Para obter mais informações de como usar o Azure Cloud Shell, confira a [documentação do Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. No painel do PowerShell, insira os seguintes comandos para clonar este repositório:

    ```powershell
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Depois que o repositório tiver sido clonado, digite os seguintes comandos para alterar para a pasta deste exercício e execute o script **setup.ps1** que ele contém:

    ```powershell
    cd dp-203/Allfiles/labs/09
    ./setup.ps1
    ```

6. Se solicitado, escolha qual assinatura deseja usar (essa opção só acontecerá se você tiver acesso a várias assinaturas do Azure).
7. Quando solicitado, insira uma senha adequada a ser definida para seu pool de SQL do Azure Synapse.

    > **Observação**: lembre-se da senha!

8. Aguarde a conclusão do script – normalmente isso leva cerca de 10 minutos, mas em alguns casos pode levar mais tempo. Enquanto espera, revise o artigo [Estratégias de carregamento de dados para pool de SQL dedicado no Azure Synapse Analytics](https://learn.microsoft.com/azure/synapse-analytics/sql-data-warehouse/design-elt-data-loading) na documentação do Azure Synapse Analytics.

## Preparar-se para carregar dados

1. Depois que o script for concluído, no portal do Azure, vá para o grupo de recursos **dp203-*xxxxxxx*** que ele criou e selecione seu workspace do Synapse.
2. Na página**Visão geral** do seu workspace do Synapse, no cartão **Abrir Synapse Studio**, selecione **Abrir** para abrir o Synapse Studio em uma nova guia do navegador, entrando caso necessário.
3. No lado esquerdo do Synapse Studio, use o ícone ›› para expandir o menu, o que revela as diferentes páginas do Synapse Studio usadas para gerenciar recursos e executar tarefas de análise de dados.
4. Na página **Gerenciar**, na guia **Pools de SQL**, selecione a linha do pool de SQL dedicado **sql*xxxxxxx***, que hospeda o data warehouse para este exercício, e use o ícone **▷** para iniciá-lo, confirmando que você deseja retomá-lo quando solicitado.

    A retomada de um pool pode levar alguns minutos. Você pode usar o botão **↻ Atualizar** para verificar seu status periodicamente. O status será exibido como **Online** quando estiver pronto. Enquanto você está esperando, prossiga com as etapas abaixo para exibir os arquivos de dados que você carregará.

5. Na página **Dados**, exiba a guia **Vinculado** e verifique se seu workspace inclui um link para sua conta de armazenamento do Azure Data Lake Storage Gen2, que deve ter um nome semelhante a **synapsexxxxxxx (Primary - datalakexxxxxxx)**.
6. Expanda sua conta de armazenamento e verifique se ela contém um contêiner do sistema de arquivos chamado **arquivos (principal).**
7. Selecione o contêiner de arquivos e observe que ele contém uma pasta chamada **dados**. Essa pasta contém os arquivos de dados que você carregará no data warehouse.
8. Abra a pasta **dados** e observe que ela contém arquivos .csv de dados de clientes e produtos.
9. Clique com o botão direito do mouse em qualquer um dos arquivos e selecione **Visualizar** para ver os dados que ele contém. Observe que os arquivos contêm uma linha de cabeçalho; portanto, você pode selecionar a opção para exibir cabeçalhos de coluna.
10. Retorne à página **Gerenciar** e verifique se o pool de SQL dedicado está online.

## Carregar tabelas de data warehouse

Vejamos algumas abordagens baseadas em SQL para carregar dados no Data Warehouse.

1. Na página **Dados**, selecione a guia **workspace**.
2. Expanda **Banco de dados SQL** e selecione seu banco de dados **sql*xxxxxxx***. Em seguida, em seu menu <bpt ctype="x-unknown" id="1" rid="1"><bpt xmlns="urn:oasis:names:tc:xliff:document:1.2" id="p1">**</bpt></bpt>...<ept id="2" rid="1"><ept xmlns="urn:oasis:names:tc:xliff:document:1.2" id="p1">**</ept></ept>, selecione <bpt ctype="x-unknown" id="3" rid="2"><bpt xmlns="urn:oasis:names:tc:xliff:document:1.2" id="p2">**</bpt></bpt>Novo script SQL <ept id="4" rid="2"><ept xmlns="urn:oasis:names:tc:xliff:document:1.2" id="p2">**</ept></ept><ph ctype="x-unknown" id="5"><ph xmlns="urn:oasis:names:tc:xliff:document:1.2" id="ph1"> > 
</ph></ph><bpt ctype="x-unknown" id="6" rid="3"><bpt xmlns="urn:oasis:names:tc:xliff:document:1.2" id="p3">**</bpt></bpt>Script vazio<ept id="7" rid="3"><ept xmlns="urn:oasis:names:tc:xliff:document:1.2" id="p3">**</ept></ept>.

Agora você tem uma página SQL em branco, que está conectada à instância para os exercícios a seguir. Você usará esse script para explorar várias técnicas SQL que podem ser usadas para carregar dados.

### Carregar dados de um Data Lake usando a instrução COPY

1. Em seu script SQL, digite o seguinte código na janela.

    ```sql
    SELECT COUNT(1) 
    FROM dbo.StageProduct
    ```

2. Na barra de ferramentas, use o botão **&#9655; Executar** para executar o código SQL e confirmar se há **0** linhas atualmente na tabela **StageProduct**.
3. Substitua o código pela seguinte instrução COPY (alterando **datalake*xxxxxx*** para o nome do data lake):

    ```sql
    COPY INTO dbo.StageProduct
        (ProductID, ProductName, ProductCategory, Color, Size, ListPrice, Discontinued)
    FROM 'https://datalakexxxxxx.blob.core.windows.net/files/data/Product.csv'
    WITH
    (
        FILE_TYPE = 'CSV',
        MAXERRORS = 0,
        IDENTITY_INSERT = 'OFF',
        FIRSTROW = 2 --Skip header row
    );


    SELECT COUNT(1) 
    FROM dbo.StageProduct
    ```

4. Execute o script e analise os resultados. 11 linhas deveriam ter sido carregadas na tabela **StageProduct**.

    Agora vamos usar a mesma técnica para carregar outra tabela, desta vez registrando quaisquer erros que possam ocorrer.

5. Substitua o código SQL no painel de script pelo código a seguir, alterando **datalake*xxxxxx*** para o nome do data lake em ```FROM``` e cláusulas ```ERRORFILE```:

    ```sql
    COPY INTO dbo.StageCustomer
    (GeographyKey, CustomerAlternateKey, Title, FirstName, MiddleName, LastName, NameStyle, BirthDate, 
    MaritalStatus, Suffix, Gender, EmailAddress, YearlyIncome, TotalChildren, NumberChildrenAtHome, EnglishEducation, 
    SpanishEducation, FrenchEducation, EnglishOccupation, SpanishOccupation, FrenchOccupation, HouseOwnerFlag, 
    NumberCarsOwned, AddressLine1, AddressLine2, Phone, DateFirstPurchase, CommuteDistance)
    FROM 'https://datalakexxxxxx.dfs.core.windows.net/files/data/Customer.csv'
    WITH
    (
    FILE_TYPE = 'CSV'
    ,MAXERRORS = 5
    ,FIRSTROW = 2 -- skip header row
    ,ERRORFILE = 'https://datalakexxxxxx.dfs.core.windows.net/files/'
    );
    ```

6. Execute o script e revise a mensagem resultante. O arquivo de origem contém uma linha com dados inválidos, portanto, uma linha é rejeitada. O código acima especifica um máximo de **5** erros, portanto, um único erro não deve ter impedido que as linhas válidas fossem carregadas. Você pode exibir as linhas que *foram* carregadas executando a consulta a seguir.

    ```sql
    SELECT *
    FROM dbo.StageCustomer
    ```

7. Na guia **arquivos**, veja a pasta raiz do seu data lake e verifique se foi criada uma nova pasta chamada **_rejectedrows** (caso não veja essa pasta, no menu **Mais**, selecione **Atualizar** para atualizar a visualização).
8. Abra a pasta **_rejectedrows** e a subpasta específica de data e hora que ela contém, e observe que os arquivos com nomes semelhantes a ***QID123_1_2*.Error.Txt** e ***QID123_1_2*.Row.Txt** foram criados. Você pode clicar com o botão direito do mouse em cada um desses arquivos e selecionar **Visualizar** para ver detalhes do erro e da linha que foi rejeitada.

    O uso de tabelas de preparo permite validar ou transformar dados antes de movê-los ou usá-los para anexar ou upsert em qualquer tabela de dimensão existente. A instrução COPY fornece uma técnica simples, mas de alto desempenho, que você pode usar para carregar facilmente dados de arquivos em um data lake em tabelas de preparo e, como você viu, identificar e redirecionar linhas inválidas.

### Usar uma instrução CREATE TABLE AS (CTAS)

1. Retorne ao painel de script e substitua o código que ele contém pelo seguinte código:

    ```sql
    CREATE TABLE dbo.DimProduct
    WITH
    (
        DISTRIBUTION = HASH(ProductAltKey),
        CLUSTERED COLUMNSTORE INDEX
    )
    AS
    SELECT ROW_NUMBER() OVER(ORDER BY ProductID) AS ProductKey,
        ProductID AS ProductAltKey,
        ProductName,
        ProductCategory,
        Color,
        Size,
        ListPrice,
        Discontinued
    FROM dbo.StageProduct;
    ```

2. Execute o script, o que criará uma nova tabela chamada **DimProduct** a partir dos dados do produto em estágios que usam **ProductAltKey** como sua chave de distribuição de hash e tem um índice columnstore clusterizado.
4. Execute a consulta a seguir para exibir o conteúdo da nova tabela **DimProduct**:

    ```sql
    SELECT ProductKey,
        ProductAltKey,
        ProductName,
        ProductCategory,
        Color,
        Size,
        ListPrice,
        Discontinued
    FROM dbo.DimProduct;
    ```

    A expressão CREATE TABLE AS SELECT (CTAS) tem vários usos, que incluem:

    - Redistribuir a chave de hash de uma tabela para alinhar com outras tabelas para melhorar o desempenho da consulta.
    - Atribuir uma chave substituta a uma tabela de preparo com base em valores existentes após a execução de uma análise delta.
    - Criação rápida de tabelas agregadas para fins de relatório.

### Combinar instruções INSERT e UPDATE para carregar uma tabela de dimensões que muda lentamente

A tabela **DimCustomer** oferece suporte a SCDs (dimensões que mudam lentamente) do tipo 1 e do tipo 2, em que as alterações do tipo 1 resultam em uma atualização in-loco para uma linha existente e as alterações do tipo 2 resultam em uma nova linha para indicar a versão mais recente de uma instância de entidade de dimensão específica. O carregamento dessa tabela requer uma combinação de instruções INSERT (para carregar novos clientes) e instruções UPDATE (para aplicar alterações de tipo 1 ou tipo 2).

1. No painel da consulta, substitua o código SQL existente no arquivo pelo seguinte código:

    ```sql
    INSERT INTO dbo.DimCustomer ([GeographyKey],[CustomerAlternateKey],[Title],[FirstName],[MiddleName],[LastName],[NameStyle],[BirthDate],[MaritalStatus],
    [Suffix],[Gender],[EmailAddress],[YearlyIncome],[TotalChildren],[NumberChildrenAtHome],[EnglishEducation],[SpanishEducation],[FrenchEducation],
    [EnglishOccupation],[SpanishOccupation],[FrenchOccupation],[HouseOwnerFlag],[NumberCarsOwned],[AddressLine1],[AddressLine2],[Phone],
    [DateFirstPurchase],[CommuteDistance])
    SELECT *
    FROM dbo.StageCustomer AS stg
    WHERE NOT EXISTS
        (SELECT * FROM dbo.DimCustomer AS dim
        WHERE dim.CustomerAlternateKey = stg.CustomerAlternateKey);

    -- Type 1 updates (change name, email, or phone in place)
    UPDATE dbo.DimCustomer
    SET LastName = stg.LastName,
        EmailAddress = stg.EmailAddress,
        Phone = stg.Phone
    FROM DimCustomer dim inner join StageCustomer stg
    ON dim.CustomerAlternateKey = stg.CustomerAlternateKey
    WHERE dim.LastName <> stg.LastName OR dim.EmailAddress <> stg.EmailAddress OR dim.Phone <> stg.Phone

    -- Type 2 updates (address changes triggers new entry)
    INSERT INTO dbo.DimCustomer
    SELECT stg.GeographyKey,stg.CustomerAlternateKey,stg.Title,stg.FirstName,stg.MiddleName,stg.LastName,stg.NameStyle,stg.BirthDate,stg.MaritalStatus,
    stg.Suffix,stg.Gender,stg.EmailAddress,stg.YearlyIncome,stg.TotalChildren,stg.NumberChildrenAtHome,stg.EnglishEducation,stg.SpanishEducation,stg.FrenchEducation,
    stg.EnglishOccupation,stg.SpanishOccupation,stg.FrenchOccupation,stg.HouseOwnerFlag,stg.NumberCarsOwned,stg.AddressLine1,stg.AddressLine2,stg.Phone,
    stg.DateFirstPurchase,stg.CommuteDistance
    FROM dbo.StageCustomer AS stg
    JOIN dbo.DimCustomer AS dim
    ON stg.CustomerAlternateKey = dim.CustomerAlternateKey
    AND stg.AddressLine1 <> dim.AddressLine1;
    ```

2. Execute o script e analise a saída.

## Executar a otimização pós-carregamento

Depois de carregar novos dados no data warehouse, é recomendado recompilar os índices da tabela e atualizar as estatísticas nas colunas comumente consultadas.

1. Substitua o código no painel de script pelo seguinte código:

    ```sql
    ALTER INDEX ALL ON dbo.DimProduct REBUILD;
    ```

2. Execute o script para reconstruir os índices na tabela **DimProduct**.
3. Substitua o código no painel de script pelo seguinte código:

    ```sql
    CREATE STATISTICS customergeo_stats
    ON dbo.DimCustomer (GeographyKey);
    ```

4. Execute o script para criar ou atualizar estatísticas na coluna **GeographyKey** da tabela **DimCustomer**.

## Excluir recursos do Azure

Se você terminou de explorar Azure Synapse Analytics, exclua os recursos que criou para evitar custos desnecessários do Azure.

1. Feche a guia do navegador do Synapse Studio e retorne ao portal do Azure.
2. No portal do Azure, na **Página Inicial**, selecione **Grupos de recursos**.
3. Selecione o grupo de recursos **dp203-*xxxxxxx*** para o workspace do Synapse Analytics (não o grupo de recursos gerenciado) e verifique se ele contém o workspace do Synapse, a conta de armazenamento e o pool do Spark para seu workspace.
4. Na parte superior da página de **Visão Geral** do grupo de recursos, selecione **Excluir o grupo de recursos**.
5. Digite o nome do grupo de recursos **dp203-*xxxxxxx*** para confirmar que deseja excluí-lo e selecione **Excluir**.

    Após alguns minutos, seu grupo de recursos do workspace do Azure Synapse e o grupo de recursos do workspace gerenciado associado a ele serão excluídos.
