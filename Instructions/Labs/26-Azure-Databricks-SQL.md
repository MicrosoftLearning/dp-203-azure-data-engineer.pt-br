---
lab:
  title: Usar um SQL Warehouse no Azure Databricks
  ilt-use: Optional demo
---

SQL é uma linguagem padrão do setor para consultar e manipular dados. Muitos analistas de dados realizam análise de dados usando SQL para consultar tabelas em um banco de dados relacional. O Azure Databricks inclui a funcionalidade SQL que se baseia nas tecnologias Spark e Delta Lake para fornecer uma camada de banco de dados relacional sobre arquivos em um data lake.

Este exercício deve levar aproximadamente **30** minutos para ser concluído.

## Provisionar um workspace do Azure Databricks

> **Dica**: Se você já possui um espaço de trabalho do Azure Databricks *Premium* ou *Avaliação*, pode ignorar esse procedimento e usar o espaço de trabalho já existente.

Este exercício inclui um script para provisionar um novo workspace do Azure Databricks. O script tenta criar um recurso de workspace do Azure Databricks de camada *Premium* em uma região na qual sua assinatura do Azure tenha cota suficiente para os núcleos de computação necessários para este exercício; e pressupõe que sua conta de usuário tenha permissões suficientes na assinatura para criar um recurso de workspace do Azure Databricks. Se o script falhar devido a cota ou permissões insuficientes, você pode tentar [criar um workspace do Azure Databricks interativamente no portal do Azure](https://learn.microsoft.com/azure/databricks/getting-started/#--create-an-azure-databricks-workspace).

1. Em um navegador da web, faça logon no [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`.
2. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure, selecionando um ambiente ***PowerShell*** e criando um armazenamento caso solicitado. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure, conforme mostrado aqui:

    ![Portal do Azure com um painel do Cloud Shell](./images/cloud-shell.png)

    > **Observação**: Se você tiver criado anteriormente um shell de nuvem que usa um ambiente *Bash*, use o menu suspenso no canto superior esquerdo do painel do shell de nuvem para alterá-lo para ***PowerShell***.

3. Observe que você pode redimensionar o Cloud Shell arrastando a barra do separador na parte superior do painel ou usando os ícones **&#8212;** , **&#9723;** e **X** no canto superior direito do painel para minimizar, maximizar e fechar o painel. Para obter mais informações de como usar o Azure Cloud Shell, confira a [documentação do Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. No painel do PowerShell, insira os seguintes comandos para clonar esse repositório:

    ```
    rm -r mslearn-databricks -f
    git clone https://github.com/MicrosoftLearning/mslearn-databricks
    ```

5. Depois que o repositório tiver sido clonado, insira o seguinte comando para executar **setup.ps1** do script, que provisiona um workspace do Azure Databricks em uma região disponível:

    ```
    ./mslearn-databricks/setup.ps1
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
   CREATE DATABASE retail_db;
    ```

4. Use o botão **►Executar (1000)** para executar o código SQL.
5. Quando o código tiver sido executado com êxito, no painel **Navegador de esquema** use o botão Atualizar na parte superior do painel para atualizar a lista. Em seguida, expanda **hive_metastore** e **retail_db** e observe se o banco de dados foi criado, mas não contém tabelas.

Você pode usar o banco de dados **padrão** para suas tabelas, mas ao criar um armazenamento de dados analíticos é melhor criar bancos de dados personalizados para dados específicos.

## Criar uma tabela

1. Baixe o arquivo [`products.csv`](https://raw.githubusercontent.com/MicrosoftLearning/mslearn-databricks/main/data/products.csv) para o computador local, salvando-o como **products.csv**.
1. No portal de workspace do Azure Databricks, na barra lateral, selecione **(+) Novo** e, em seguida, selecione **Dados**.
1. Na página **Adicionar dados** selecione **Criar ou modificar tabela** e carregue o arquivo **products.csv** que você baixou para o seu computador.
1. Na página **Criar ou modificar uma tabela a partir do upload de arquivo** selecione o esquema **retail_db** e defina o nome da tabela como **produtos**. Em seguida, selecione **Criar tabela** no botão inferior esquerdo da página.
1. Quando a tabela tiver sido criada, revise seus detalhes.

A capacidade de criar uma tabela importando dados de um arquivo facilita o preenchimento de um banco de dados. Você também pode usar o Spark SQL para criar tabelas usando código. As tabelas em si são definições de metadados no metastore do hive e os dados que elas contêm são armazenados no formato Delta no armazenamento do DBFS (Sistema de Arquivos do Databricks).

## Criar um painel

1. Na barra lateral, selecione **(+) Novo** e, em seguida, selecione **Painel**.
2. Selecione o nome do novo painel e altere-o para `Retail Dashboard`.
3. Na guia **Dados** selecione **Criar a partir do SQL** e use a seguinte consulta:

    ```sql
   SELECT ProductID, ProductName, Category
   FROM retail_db.products; 
    ```

4. Selecione **Executar** e renomeie o conjunto de dados sem título para `Products and Categories`.
5. Selecione a guia **Tela** e em seguida, selecione **Adicionar uma visualização**.
6. No editor de visualização, defina as seguintes propriedades:
    
    - **Conjunto de Dados**: Produtos e Categorias
    - **Visualização**: barra
    - **X Eixo**: X: COUNT(ProductID)
    - **Y eixo**: categoria

7. Selecione **Publicar** para exibir o painel como os usuários o verão.

Os painéis são uma ótima maneira de compartilhar tabelas de dados e visualizações com usuários corporativos. Você pode agendar os painéis para serem atualizados periodicamente e enviados por e-mail aos assinantes.

## Limpeza

No portal do Azure Databricks, na página **SQL Warehouses**, selecione seu SQL Warehouse e selecione **&#9632; Parar** para desativá-lo.

Se você tiver terminado de explorar o Azure Databricks, poderá excluir os recursos que criou para evitar custos desnecessários do Azure e liberar capacidade em sua assinatura.
