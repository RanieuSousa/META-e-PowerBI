# Integração da API da Meta com Power BI

## **Visão Geral**
Este projeto oferece um script simples em M (linguagem do Power Query) para extrair dados da API de Anúncios da Meta e carregá-los no Power BI. Com essa integração, você pode obter métricas importantes, como **alcance**, **impressões**, **cliques**, **frequência**, **gastos** e o desempenho dos anúncios ao longo do tempo. Uma vez que os dados estão no Power BI, você pode cruzá-los com outros datasets, construir relatórios e gerar insights valiosos.

## **Como Usar**

Siga estes passos para recuperar seus dados de anúncios da Meta e carregá-los no Power BI:

### **Passo 1: Abrir o Editor do Power Query**
1. Abra o Power BI Desktop.
2. Na aba **Início**, clique em **Transformar Dados** para abrir o Editor do Power Query.

### **Passo 2: Criar uma Nova Consulta em Branco**
1. No Editor do Power Query, clique em **Nova Fonte** e selecione **Consulta em Branco**.
2. No painel à esquerda, clique com o botão direito na nova consulta, renomeie-a para algo como `DadosMetaAds`.

### **Passo 3: Colar o Código M**
1. Clique com o botão direito na nova consulta e escolha **Editor Avançado**.
2. No editor, apague o código de exemplo e cole o código M fornecido abaixo:
      
      ```m
      let
          // Função para buscar os dados da API
          GetPage = (url as text) =>
          let
              Fonte = Json.Document(Web.Contents(url)),
              Dados = Fonte[data],
              ProximaPagina = try Fonte[paging][next] otherwise null,
              Resultado = [Dados = Dados, ProximaPagina = ProximaPagina]
          in
              Resultado,
          
          // URL inicial da API (primeira página)
          InitialUrl = "https://graph.facebook.com/v20.0/act_seuid/insights?access_token=<seutoken>&fields=ad_id,ad_name,reach,impressions,clicks,frequency,spend,date_start,date_stop&level=ad&time_increment=1",
          
          // Função para iterar sobre as páginas e acumular os dados
          GetAllPages = (url as text) =>
          let
              Pagina = GetPage(url),
              DadosPagina = Pagina[Dados],
              ProximaPagina = Pagina[ProximaPagina],
              
              // Se houver uma próxima página, continuar a buscar os dados
              TodosOsDados = if ProximaPagina <> null then
                  List.Combine({DadosPagina, @GetAllPages(ProximaPagina)})
              else
                  DadosPagina
          in
              TodosOsDados,
          
          // Obter todos os dados chamando a função
          TodasAsPaginas = GetAllPages(InitialUrl),
          
          // Converter os dados para uma tabela
          DadosTabela = Table.FromList(TodasAsPaginas, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
          
          // Expandir os registros da tabela
          DadosExpandido = Table.ExpandRecordColumn(DadosTabela, "Column1", {"ad_id", "ad_name", "reach", "impressions", "clicks", "frequency", "spend", "date_start", "date_stop"}, {"data.ad_id", "data.ad_name", "data.reach", "data.impressions", "data.clicks", "data.frequency", "data.spend", "data.date_start", "data.date_stop"}),
      
          // Alterar os tipos das colunas
          TiposAlterados = Table.TransformColumnTypes(
              DadosExpandido,
              {
                  {"data.ad_id", Int64.Type}, 
                  {"data.ad_name", type text}, 
                  {"data.reach", Int64.Type}, 
                  {"data.impressions", Int64.Type}, 
                  {"data.clicks", Int64.Type}, 
                  {"data.frequency", type number}, 
                  {"data.spend", type number}, 
                  {"data.date_start", type date}, 
                  {"data.date_stop", type date}
              }
          )
      in
          TiposAlterados

### **Passo 4: Substitua os Parâmetros **

Substitua <seuid> pelo seu ID de conta de anúncios Meta.
Substitua <seutoken> pelo seu token de acesso da API da Meta.

### **Passo 5: Carregar os Dados **

Após colar o código e substituir os parâmetros, clique em Concluído.
Se os dados forem recuperados com sucesso, clique em Fechar & Carregar no Power Query Editor.
Os dados de anúncios agora estarão disponíveis no Power BI e prontos para análise.
