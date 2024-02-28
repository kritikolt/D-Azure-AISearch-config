# Configurando um Recurso de Serviços de IA do Azure

Este README fornece um guia passo a passo para criar um recurso de Serviços de IA do Azure para enriquecer os dados em um armazenamento com insights gerados por IA.

## Passo 1: Provisionar um Recurso de Serviços de IA do Azure

1. Retorne à página inicial do portal Azure.
2. Clique no botão **＋Create a resource** e pesquise por "Azure AI services".
3. Selecione **create**. Isso o levará a uma página para criar um recurso de Serviços de IA do Azure.
4. Configure o recurso com as seguintes configurações:
   - **Subscription**: Sua assinatura do Azure.
   - **Resource group**: O mesmo grupo de recursos do seu recurso de Azure AI Search.
   - **Region**: A mesma localização do seu recurso de Azure AI Search.
   - **Name**: Um nome exclusivo.
   - **Pricing tier**: Standard S0.
   - Marque a caixa reconhecendo os termos.
5. Selecione **Review + create**. Após ver a resposta "Validation Passed", selecione **create**.
6. Aguarde a implantação ser concluída e depois visualize os detalhes da implantação.

## Passo 2: Criar uma Conta de Armazenamento

1. Retorne à página inicial do portal Azure e selecione o botão **+ Create a resource**.
2. Pesquise por "storage account" e crie um recurso de Conta de Armazenamento com as seguintes configurações:
   - **Subscription**: Sua assinatura do Azure.
   - **Resource group**: O mesmo grupo de recursos dos seus recursos de Azure AI Search e Serviços de IA do Azure.
   - **Storage account name**: Um nome exclusivo.
   - **Location**: Escolha qualquer local disponível.
   - **Performance**: Padrão.
   - **Redundancy**: Armazenamento com redundância local (LRS).
3. Clique em **Review** e depois em **Create**. Aguarde a implantação ser concluída e então acesse o recurso implantado.

## Passo 3: Carregar Documentos no Armazenamento do Azure

1. Na conta de armazenamento criada, no painel de menu à esquerda, selecione **Configuration ** (em Settings).
2. Altere a configuração para "Allow Blob anonymous access" para "Enabled" e depois selecione **Save**.
3. No painel de menu à esquerda, selecione **Containers**.
4. Selecione **+ Container**. Uma janela será aberta à direita.
5. Insira as seguintes configurações e clique em **Create**:
   - **Name**: coffee-reviews
   - **Public access level**: Container (acesso de leitura anônimo para contêineres e blobs)
6. Em uma nova aba do navegador, baixe os arquivos de revisões de café zipados de [aqui](https://aka.ms/mslearn-coffee-reviews) e extraia os arquivos para a pasta de revisões.
7. No portal Azure, selecione seu contêiner coffee-reviews.
8. No contêiner, selecione **Upload**.
9. No painel de upload, selecione **Select a file**.
10. Na janela do Explorador, selecione todos os arquivos na pasta de revisões, abra-os e selecione **Upload**.
11. Após o upload ser concluído, você pode fechar o painel de upload. Seus documentos agora estão no contêiner de armazenamento coffee-reviews.

## Passo 4: Indexar os Documentos

1. No portal do Azure, acesse o recurso Azure AI Search.

2. Na página de **Overview**, selecione "Import data".

3. Na página "Connect to your data", na lista de Origem de Dados, selecione "Azure Blob Storage".

   - **Data Source:** Azure Blob Storage
   - **Data source name:** coffee-customer-data
   - **Data to extract:** Conteúdo e metadados
   - **Parsing mode:** Padrão
   - **Connection string:** Selecione "Choose an existing connection" e escolha sua conta de armazenamento e o container coffee-reviews.
   - **Managed identity authentication:** Nenhum
   - **Container name:** Este campo será preenchido automaticamente após a escolha da conexão existente.

4. Selecione "Next: Add cognitive skills (Opcional)".

   - No campo "Enable OCR and merge all text into merged_content field", marque a caixa de seleção.
   - **Skillset name:** coffee-skillset
   - **Source data field:** merged_content
   - **Enrichment granularity level:** Páginas (chunks de 5000 caracteres)
   - Não selecione "Enable incremental enrichment".

5. Em "Save enrichments to a knowledge store", siga as instruções:

   - Selecione "Azure blob projections: Document".
   - Se aparecer um aviso solicitando a String de Conexão da Conta de Armazenamento, selecione "Choose an existing connection", escolha a conta de armazenamento e crie um novo container chamado "knowledge-store" com nível de privacidade "Private".

6. Selecione "Next: Customize target index".

   - **Index name:** coffee-index
   - Deixe o Sugeridor em branco e o Modo de Busca será autopreenchido.
   - Revise as configurações padrão dos campos do índice e selecione "filterable" para todos os campos já selecionados por padrão.

7. Selecione "Next: Create an indexer".

   - **Indexer name:** coffee-indexer
   - Deixe o Agendamento como "Once".
   - No menu de Opções Avançadas, certifique-se de que a opção "Base-64 Encode Keys" esteja selecionada.

8. Clique em "Submit" para criar a fonte de dados, conjunto de habilidades, índice e indexador. O indexador será executado automaticamente, realizando a indexação dos documentos conforme o pipeline configurado.

9. Volte para a página do recurso Azure AI Search. No painel esquerdo, em "Search Management", selecione "Indexers". Selecione o indexador recém-criado "coffee-indexer". Aguarde até que o Status indique sucesso.

10. Se desejar, selecione o nome do indexador para ver mais detalhes.

# passo 5: Consulta ao Índice

1. Acesse a página Visão Geral do seu serviço de Pesquisa Azure.

2. Selecione "Search explorer" no topo da tela.

3. Verifique se o índice selecionado é o coffee-index que você criou. Abaixo do índice selecionado, altere a visualização para a visualização JSON.

4. No campo do editor de consulta JSON, copie e cole:

   ```json
   {
       "search": "*",
       "count": true
   }
   ```

5. Selecione "Search". A consulta de pesquisa retorna todos os documentos no índice de pesquisa, incluindo uma contagem de todos os documentos no campo "@odata.count".

6. No campo do editor de consulta JSON, copie e cole:

    ```json
    {
      "search": "locations:'Chicago'",
      "count": true
    }
    ```

7. Selecione "Search". A consulta busca todos os documentos no índice e filtra as avaliações com uma localização em Chicago.

8. No campo do editor de consulta JSON, copie e cole:

    ```json
    {
      "search": "sentiment:'negative'",
      "count": true
    }
    ```

9. Selecione "Search". A consulta busca todos os documentos no índice e filtra as avaliações com sentimento negativo.

10. Observe como os resultados são classificados por @search.score, que indica a pontuação de relevância atribuída pelo mecanismo de pesquisa para mostrar o quão bem os resultados correspondem à consulta fornecida.

11. Para entender possíveis problemas com avaliações negativas, explore as frases-chave associadas a elas para identificar possíveis causas.

## Passo 6: Revisão do Repositório de Conhecimento

1. No portal do Azure, navegue de volta para sua conta de armazenamento do Azure.

2. No painel de menu à esquerda, selecione "Containers". Selecione o container "knowledge-store".

3. Selecione um dos itens e, em seguida, clique no arquivo objectprojection.json.

4. Selecione "Edit" para visualizar o JSON produzido para um dos documentos do seu armazenamento de dados do Azure.

5. Selecione o breadcrumb do blob de armazenamento no canto superior esquerdo da tela para retornar aos Containers de armazenamento.

6. Nos Containers, selecione o container "coffee-skillset-image-projection". Selecione qualquer um dos itens.

7. Selecione qualquer um dos arquivos .jpg. Selecione "Edit" para visualizar a imagem armazenada do documento. Observe como todas as imagens dos documentos são armazenadas dessa maneira.

8. Selecione o breadcrumb do blob de armazenamento no canto superior esquerdo da tela para retornar aos Containers de armazenamento.

9. Selecione "Storage browser" no painel à esquerda e selecione "Tables". Há uma tabela para cada entidade no índice. Selecione a tabela "coffeeSkillsetKeyPhrases".

10. Analise as frases-chave que o repositório de conhecimento foi capaz de capturar a partir do conteúdo das avaliações. Muitos dos campos são chaves, então você pode vincular as tabelas como em um banco de dados relacional. O último campo mostra as frases-chave que foram extraídas pelo conjunto de habilidades.
