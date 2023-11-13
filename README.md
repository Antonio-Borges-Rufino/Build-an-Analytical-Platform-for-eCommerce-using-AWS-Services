# Descrição do Projeto
* Referencia: O projeto aqui implementado é do site projectpro.io onde estou estudando projetos de engenharia de dados profissionais. Créditos para a equipe.
* Neste projeto, usaremos um conjunto de dados de comércio eletrônico para simular os registros de compras do usuário, visualizações de produtos, histórico de carrinho e jornada do usuário na plataforma online para criar dois pipelines analíticos, Lote e Tempo Real. O processamento em lote envolverá ingestão de dados, arquitetura Lake House, processamento e visualização usando Amazon Kinesis, Glue, S3 e QuickSight para obter insights sobre o seguinte:
  * Visitantes únicos por dia
  * Durante um determinado tempo, os usuários adicionam produtos aos carrinhos, mas não os compram
  * Principais categorias por hora ou dia da semana (ou seja, para promover descontos com base em tendências)
  * Para saber quais marcas precisam de mais marketing
# Diagrama de Arquitetura
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/148a4423-b257-4d3d-993e-1d3aa0fe4e9a)
# Informação sobre os dados
* Os dados usados nesse projeto são o [eCommerce behavior data from multi category store](https://www.kaggle.com/datasets/mkechinov/ecommerce-behavior-data-from-multi-category-store/) do kaggle
* Os dados possuem 9 colunas referente ao acesso em um site. Basicamente, o que os dados fazem, é disponibilizar o que uma pessoa faz em um acesso em uma loja de compras, por exemplo: "As 9 horas o ID_CONTA X acessou o produto XY que é um smarthone e colocou o produto no carrinho na seção Z"
* As colunas são detalhadas a seguir
  * event_time: Hora em que o evento aconteceu
  * event_type: Apenas um tipo dos eventos (view,cart,remove_from_cart,purchase)
  * product_id: Id do produto
  * category_id: Id da categoria do produto
  * category_code: Nome da categoria
  * brand: Sequencia do nome da marca em letra minuscula
  * price: Preço do produto
  * user_id: Id do usuário (estático)
  * user_session: Id da sessão (Pode variar para o mesmo usuario, cada sessão é um id único)
* Usaremos esses dados para simular como se fosse uma pessoa usando o site e então vamos fazer o pipeline acima descrito
  
  
  
   
