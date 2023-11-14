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
# Instalar comando por CLI da AWS
* Vou seguir o projeto e vou usar o CLI da AWS
* Em vez de instalar na minha máquina, vou usar o AWS CloudShell, que é mais prático
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/8be6aca4-0ffc-4cc4-aee7-7d2affa1a34b)

# Criar o bucket S3
* Crie o bucket usando CLI como seguinte comando (substitua {bucket-name} pelo nome do bucket):
* ```
  aws s3 mb s3://{bucket-name}
  ```
* Podemos ver se o bucket foi criado acessando o recurso S3 e indo em bucket. No meu caso, consegui criar um bucket com acesso privado, como mostra a imagem abaixo (apaguei parte das informações)
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/5e26e951-e06e-4cd8-a981-e578a43a49ae)
* Agora vou adicionar uma tag para o bucket
* ```
  aws s3api put-bucket-tagging --bucket {bucket-name} --tagging 'TagSet=[{Key={key-tag},Value={value-tag}}]'
  ```
* Essa tag rastreia as modificações feitas pelo usuário que está consumindo o bucket
* Agora podemos enviar nossos dados para o bucket. Se eu tivesse optado pela escolha de usar o CLI do windowns, poderia fazer isso diretamente do meu pc, mas com o CloudShell é necessário ou enviar o arquivo para o CloudShell (máximo 1gb) ou enviar direto para o bucket no painel (Máximo de 5gb, que é o nivel gratuito)
* Portanto, vou enviar direto pelo painel mesmo, inclusive, vou diminuir a quantidade de dados que tenho atualmente para poder caber no nível de teste gratuito do bucket S3, como mostrado na imagem abaixo
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/7b9b8633-b6eb-4174-a86d-eb96870073ad)
* Agora é só enviar os dados (no meu caso, utilizei só as as primeiras 7 milhões de linha do arquivo) para o bucket
* 


  
  
  
   
