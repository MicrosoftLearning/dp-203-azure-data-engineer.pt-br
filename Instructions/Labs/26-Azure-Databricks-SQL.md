---
lab:
  title: Usar um SQL Warehouse no Azure Databricks
  ilt-use: Optional demo
---

# Usar um SQL Warehouse no Azure Databricks

SQL é uma linguagem padrão do setor para consultar e manipular dados. Muitos analistas de dados realizam análise de dados usando SQL para consultar tabelas em um banco de dados relacional. O Azure Databricks inclui a funcionalidade SQL que se baseia nas tecnologias Spark e Delta Lake para fornecer uma camada de banco de dados relacional sobre arquivos em um data lake.

Este exercício deve levar aproximadamente **30** minutos para ser concluído.

## Antes de começar

Você precisará de uma [assinatura do Azure](https://azure.microsoft.com/free) na qual tenha acesso de nível administrativo e cota suficiente em pelo menos uma região para provisionar um SQL Warehouse do Azure Databricks.

## Provisionar um workspace do Azure Databricks

Neste exercício, você precisará de um workspace do Azure Databricks de nível premium.

> **Dica**: se você já tiver um workspace do Azure Databricks *Premium* ou de *Avaliação*, pule este procedimento.

1. Em um navegador da web, faça logon no [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`.
2. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure, selecionando um ambiente ***PowerShell*** e criando um armazenamento caso solicitado. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure, conforme mostrado aqui:

    ![Portal do Azure com um painel do Cloud Shell](./images/cloud-shell.png)

    > **Observação**: se você tiver criado anteriormente um cloud shell que usa um ambiente *Bash*, use o menu suspenso no canto superior esquerdo do painel do cloud shell para alterá-lo para ***PowerShell***.

3. Observe que você pode redimensionar o Cloud Shell arrastando a barra do separador na parte superior do painel ou usando os ícones **&#8212;** , **&#9723;** e **X** no canto superior direito do painel para minimizar, maximizar e fechar o painel. Para obter mais informações de como usar o Azure Cloud Shell, confira a [documentação do Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. No painel do PowerShell, insira os seguintes comandos para clonar esse repositório:

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Depois que o repositório tiver sido clonado, digite os seguintes comandos para alterar para a pasta deste laboratório e execute o script **setup.ps1** que ele contém:

    ```
    cd dp-203/Allfiles/labs/26
    ./setup.ps1
    ```

6. Se solicitado, escolha qual assinatura você deseja usar (isso só acontecerá se você tiver acesso a várias assinaturas do Azure).

7. Aguarde a conclusão do script - isso normalmente leva cerca de 5 minutos, mas em alguns casos pode levar mais tempo. Enquanto você espera, revise o artigo [O que é data warehousing no Azure Databricks?](https://learn.microsoft.com/azure/databricks/sql/) na documentação do Azure Databricks.

## Exibir e iniciar um SQL Warehouse

1. Quando o recurso de workspace do Azure Databricks tiver sido implantado, vá para ele no portal do Azure.
1. Na página **Visão geral** do seu workspace do Azure Databricks, use o botão **Iniciar workspace** para abrir seu workspace do Azure Databricks em uma nova guia do navegador, fazendo logon se solicitado.

    > **Dica**: ao usar o portal do workspace do Databricks, várias dicas e notificações podem ser exibidas. Dispense-as e siga as instruções fornecidas para concluir as tarefas neste exercício.

1. Veja o portal do workspace do Azure Databricks e observe que a barra lateral no lado esquerdo contém os nomes das categorias de tarefas.
1. Na barra lateral, em **SQL**, selecione **SQL warehouses**.
1. Observe que o workspace já inclui um SQL Warehouse chamado **Warehouse Inicial**.
1. No menu **Ações** (**⁝**) do SQL Warehouse, selecione **Editar**. Em seguida, defina a propriedade **Tamanho do cluster** como **2X-Small** e salve as alterações.
1. Use o botão **Iniciar** para iniciar o SQL Warehouse (o que pode levar um ou dois minutos).

> **Observação**: se o SQL Warehouse não for iniciado, sua assinatura pode ter cota insuficiente na região em que seu workspace do Azure Databricks está provisionado. Para obter detalhes, confira [Cota de vCPU do Azure necessária](https://docs.microsoft.com/azure/databricks/sql/admin/sql-endpoints#required-azure-vcpu-quota). Se isso acontecer, você pode tentar solicitar um aumento de cota, conforme detalhado na mensagem de erro, quando o depósito falhar ao iniciar. Como alternativa, tente excluir seu workspace e criar um novo em uma região diferente. Você pode especificar uma região como um parâmetro para o script de instalação da seguinte maneira: `./setup.ps1 eastus`

## Criar um esquema de banco de dados

1. Quando o SQL Warehouse estiver *em execução*, selecione **Editor SQL** na barra lateral.
2. No painel **Navegador de esquema**, observe que o catálogo *hive_metastore* contém um banco de dados chamado **padrão**.
3. No painel **Nova consulta**, insira o seguinte código SQL:

    ```sql
    CREATE SCHEMA adventureworks;
    ```

4. Use o botão **►Executar (1000)** para executar o código SQL.
5. Quando o código tiver sido executado com êxito, no painel **Navegador de esquema**, use o botão Atualizar na parte inferior do painel para atualizar a lista. Em seguida, expanda **hive_metastore** e **adventureworks** e observe se o banco de dados foi criado, mas não contém tabelas.

Você pode usar o banco de dados **padrão** para suas tabelas, mas ao criar um armazenamento de dados analíticos é melhor criar bancos de dados personalizados para dados específicos.

## Criar uma tabela

1. Baixe o arquivo [**products.csv**](https://raw.githubusercontent.com/MicrosoftLearning/dp-203-azure-data-engineer/master/Allfiles/labs/26/data/products.csv) para o computador local, salvando-o como **products.csv**.
1. No portal do workspace do Azure Databricks, na barra lateral, selecione **(+) Novo** e, em seguida, selecione **Upload de Arquivos** e carregue o arquivo **products.csv** que você baixou para o seu computador.
1. Na página **Carregar dados**, selecione o esquema **adventureworks** e defina o nome da tabela como **produtos**. Em seguida, selecione **Criar tabela** no canto inferior esquerdo da página.
1. Quando a tabela tiver sido criada, revise seus detalhes.

A capacidade de criar uma tabela importando dados de um arquivo facilita o preenchimento de um banco de dados. Você também pode usar o Spark SQL para criar tabelas usando código. As tabelas em si são definições de metadados no metastore do hive e os dados que elas contêm são armazenados no formato Delta no armazenamento do DBFS (Sistema de Arquivos do Databricks).

## Criar uma consulta

1. Na barra lateral, selecione **(+) Novo** e, em seguida, selecione **Consulta**.
2. No painel **Navegador de esquema**, expanda **hive_metastore** e **adventureworks** e verifique se a tabela de **produtos** está listada.
3. No painel **Nova consulta**, insira o seguinte código SQL:

    ```sql
    SELECT ProductID, ProductName, Category
    FROM adventureworks.products; 
    ```

4. Use o botão **►Executar (1000)** para executar o código SQL.
5. Quando a consulta for concluída, revise a tabela de resultados.
6. Use o botão **Salvar** no canto superior direito do editor de consultas para salvar a consulta como **Produtos e Categorias**.

Salvar uma consulta facilita a recuperação dos mesmos dados novamente posteriormente.

## Criar um painel

1. Na barra lateral, selecione **(+) Novo** e, em seguida, selecione **Painel**.
2. Na caixa de diálogo **Novo painel**, digite o nome **Produtos da Adventure Works** e selecione **Salvar**.
3. No painel **Produtos da Adventure Works**, na lista suspensa **Adicionar**, selecione **Visualização**.
4. Na caixa de diálogo **Adicionar widget de visualização**, selecione a consulta **Produtos e categorias**. Em seguida, selecione **Criar nova visualização**, defina o título como **Produtos por categoria** e selecione **Criar visualização**.
5. No editor de visualização, defina as seguintes propriedades:
    - **Tipo de visualização**: barra
    - **Gráfico horizontal**: selecionado
    - **Coluna Y**: Categoria
    - **Colunas X**: ID do produto (product ID) : contagem
    - **Agrupar por**: *deixar em branco*
    - **Empilhamento**: desativado
    - **Normalizar valores para porcentagem**: <u>não</u> selecionado
    - **Valores ausentes e NULL**: Não são exibidos no gráfico

6. Salve a visualização e visualize-a no painel.
7. Selecione **Edição concluída** para exibir o painel como os usuários o verão.

Os painéis são uma ótima maneira de compartilhar tabelas de dados e visualizações com usuários corporativos. Você pode agendar os painéis para serem atualizados periodicamente e enviados por e-mail aos assinantes.

## Excluir recursos do Azure Databricks

Agora que você terminou de explorar os SQL Warehouses no Azure Databricks, deve excluir os recursos criados para evitar custos desnecessários do Azure e liberar capacidade em sua assinatura.

1. Feche a guia do navegador do workspace do Azure Databricks e volte para o portal do Azure.
2. No portal do Azure, na **Página Inicial**, selecione **Grupos de recursos**.
3. Selecione o grupo de recursos que contém seu workspace do Azure Databricks (não o grupo de recursos gerenciados).
4. Na parte superior da página de **Visão Geral** do grupo de recursos, selecione **Excluir o grupo de recursos**.
5. Digite o nome do grupo de recursos para confirmar que deseja excluí-lo e selecione **Excluir**.

    Após alguns minutos, seu grupo de recursos e o grupo de recursos do workspace gerenciado associado a ele serão excluídos.
