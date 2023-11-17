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

# Configurando e Implementando o AWS Kinesis Data Stream
* Vá no painel de serviços do AWS e pesquise por Kinesis, e então crie um serviço do tipo Kinesis Data Stream
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/b5181036-35e3-4716-b92e-bf69bd98a693)
* Existe pouca opção de criação, então optei por uso mínimo
* Após criar, o serviço fica disponivel como na imagem abaixo
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/ffbbef8b-64ac-4506-8dfe-fa56f2271f77)


# Criando Aplicação Python Que Simula Um Site
* Para criar essa aplicação, usaremos a SDK boto3 da AWS. Esse framework é preparado para rodar dentro do próprio serviço da amazon, podendo ser no CLI, no SHELL ou no Cloud9.
* Para o nosso projeto, vamos usar o Cloud9. Para isso, vá até o serviço de aplicações da AWS e procure por Cloud9. Crie um novo serviço e use um sistema de computação EC2 que esteja habilitado para o nível grátis. Use a minima quantidade de recursos possiveis.
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/94bb948d-ef5f-491e-b909-209d2e60e267)
* A imagem acima mostra o ambiente já criado, e onde clicar caso queira criar outro. Depois de criado, basta clicar em "em aberto" que é o link da IDE
* Com a IDE aberta, você instala a biblioteca na IDE através do bash no canto inferior usando o comando pip do python e depois testa com uso no bucket S3. A imagem abaixo mostra o processo
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/21d94e95-41c8-4035-adbf-b2a8e33ba2a8)
* No meu caso deu certo, ele executou o e printou os bucketes disponíveis
* Agora vamos acessar dados de dentro desses buckets. Isso é interessante porque vamos recuperar os dados do nosso bucket e vamos enviar em streaming para o Kinesis
* O código abaixo está mostrando a execução dos comandos de acesso ao bucket, com a adição do comando split('/n') podemos facilmente gerar uma matriz de strings e depois transformalos em um json que vamos enviar para o kinesis
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/b1582093-8be0-4493-af8e-34ad8a267929)
* Abaixo está o código modificado para poder transformar cada registro em um vetor. A função decode('utf-8') serve para transformar o tipo byte em string e a função split('\r\n') serve para poder quebrar de maneira correta cada registro
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/27aa5113-8498-403c-acdc-b551bd0de258)
* Agora vamos criar a função de inserção para o kimnesis englobando os recuperados do s3
* Primeiro vamos importar as bibliotecas necessárias, entre elas a biblioteca csv e json. A biblioteca csv vai ler os dados em forma de dicionário através do método DictReader() e a biblioteca json vai devolver os dados em json para o kinesis
* Abaixo está o código e o response, esse código é do projectpro, mas na documentação obtemos quase os mesmos códigos, entretanto, aqui eles fazem tratamento dos dados csv
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/20cf0000-14a9-473f-a240-7233b16d31a8)
* Primeiro usamos a biblioteca boto3, nela, podemos acessar os clientes s3 e kinesis. A partir disso, recuperamos os dados do S3 a partir do comando "response_nov = client_s3.get_object(Bucket={},Key={})". Bucket é o nome do bucket em que está os dados e Key é o nome do arquivo. O método get_object recupera qualquer tipo de arquivo do bucket.
* O código "response_nov = response_nov['Body'].read().decode("utf-8").split('\r\n')" realiza a leitura do arquivo, a resposta da requisção chega em formato JSON, e no campo Body vem os dados em forma de byte, para convertelos para string usamos decode('utf-8'). o método read() é usado para ler os dados em binário e split('\r\n') é usado para quebrar a string unica em vários vetores contendo as informações de uma só linha
* Depois criamos o for iterativo onde vamos percorrer as linhas e enviar as informações. Como o arquivo possuia cabeçalho usei o método da biblioteca csv no "for x in csv.DictReader(response):"
* Dentro do looping fazemos a transformação dos dados, aqui é importante entender o que a linha "json_load['txn_timestamp'] = datetime.now().isoformat()" está fazendo, ela salva o horário de envio da informação e a linha "response = client_kinesis.put_record(StreamName='{},Data=json.dumps(json_load, indent=4),PartitionKey=str(json_load['category_id']))" é quem de fato faz a postagem dos dados para o kinesis
* É importante resssaltar o uso da "PartitionKey" de 'category_id'. Isso é importante porque conseguimos melhorar a divisão dos nossos dados
* Podemos ver os dados chegando através do painel de obtenção de dados, como abaixo:
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/6759cbd6-ff9d-48de-a159-52fba19b8651)

# Próximo Passo
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/26890c1b-150f-4e35-a98d-deb3ed69269f)
* Os próximos passos estão relacionados a construção do processamento dos dados em streaming
* Para isso, vamos criar um banco de dados no AWS Glue utilizando o arquivo que usamos para simular os dados em stream
* Depois, vamos construir mais um canal de stream no kinesis para poder empurrar os dados do flink para ele
* Por fim, vamos construir um caderno flink que vai ler os dados do stream 1 e empurrar os dados para o stream 2
* O AWS Glue vai funcionar como registro de schema para o flink, basicamente, é ele quem vai nortear o SQL do Flink a partir do schema que ele puxar dos nossos dados em CSV guardados no S3

# Criação do AWS Glue
* Para criar um novo banco de dados no AWS Glue, entre no serviço, vá em banco de dados e aperte em criar um novo
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/9b4a5a7c-0722-416b-8daf-bb40927609f5)
* No nivel gratuito temos bastante espaço para poder utilizar o serviço
* No painel de criação, coloque o nome do seu banco de dados e a URI da sua pasta S3 onde você quer salvar o banco
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/08c031e4-78d3-418f-b1dc-f748755cb919)
* Agora vamos adicionar a tabela usando o crawler, isso significa que o glue vai analisar o arquivo e vai definir um schema pra ele
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/f611c55a-9114-40e0-a3b0-d54bfecde347)
* As configurações são pessoais, a única que deve ser realmente vista é a onfiguração que aponta para onde estão os dados. O que montamos nessa config é o crawler, então a missão dele é pesquisar dentro do bucket S3 em busca dos dados. Portanto, coloque apenas o caminho do bucket, não do dado em si
* Depois de construido, vá em tables e acesse a sua tabela AWS Glue
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/339f0fc7-468a-4b91-9e45-2650940f4736)
* Dentro da tabela, acesse os dados através do Athena
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/5434eeff-8ccf-4c47-b9e0-e0442718b3ac)
* Dentro do Athen, aperte em configurações e gerenciar, para indicar um local onde as consultas vão ficar salval. Vou salvar na mesma pasta do bucket S3 que estou usando em todo o projeto
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/5c1b812b-587c-4274-86d1-0b9fa6ce1ad9)
* Agora podemos executar a query padrão e ver os resultados
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/508d4af7-1526-47d4-a406-53c4c899df1d)
* Pronto, nosso banco de dados AWS Glue foi criado

# Criação do Canal De Saída de Stream 2
* Apenas siga os passos da seção Configurando e Implementando o AWS Kinesis Data Stream

# Criação e Integração de Aplicação com Apache Flink
* Criar Stream Aplication no Kinesis -> Ler os dados Stream com a Estrutura da tabela do Glue e passar pro outro stream-> Salvar como Aplicação -> Fazer o Deploy da Aplicação -> Executar a aplicação
* Primeiro, vamos no fluxo de dados do kinesis e clique e "Apache Flink Gerenciado"
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/a3bb8f27-029d-492f-be5d-474eb7691630)
* Depois, selecione studio notebook
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/48fbbb69-a63e-4911-9136-604eb560b020)
* Selecione criação personalizada, coloque o nome e aperte em avançar
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/04948fc0-3173-4aa9-80d9-0474078cb8e4)
* Na seção de permissões, deixe a permissão padrão e em baixo coloque o banco de dados AWS Glue que criamos anteriormente
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/ccaa39c0-1be1-4e29-9218-5463b6bc2611)
* Em fontes incluidas, na fonte destino, coloque o primeiro fluxo kinesis que criei e na fonte de destino, coloque o segundo fluxo kinesis
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/ac3d6892-076e-4bcd-8523-4bd409aa94e9)
* Na próxima página, deixe tudo como padrão e coloque o bucket s3 que criamos anteriormente como destino de configurações
* Agora podemos acessar nosso caderno zepelin e acompanhar o stream de dados em tempo real por lá, aperte em executar e abra o zepelin
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/4f902b5a-d762-4fe3-a098-f36596a26609)
* Crie um novo notebook, ele deve ter essa cara
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/b2b46c98-6044-4294-a21e-9f6a2af786fb)
* É nele onde vamos codar a nossa aplicação
* Para acessar os dados usando o flink, precisamos construir uma tabela que será salva no AWS Glue, podemos construir diversos tipos de comandos SQL e usar em stream nessas tabelas, que possuem taxa de atualização em real time. Primeiro, vamos construir a tabela que vai ler os dados.
* ```
  %flink.ssql
  /*Primeiro, drop um schema caso ele exista*/
  DROP TABLE IF EXISTS kinesis_pipeline_table_1;
  /*Aqui, vou usar o schema dos dados que chegam para a aplicação no formato json*/
  CREATE TABLE kinesis_pipeline_table_1 (
    event_time VARCHAR(30), 
    event_type VARCHAR(30), 
    product_id BIGINT, 
    category_id BIGINT, 
    category_code VARCHAR(30), 
    brand VARCHAR(30), 
    price DOUBLE, 
    user_id BIGINT, 
    user_session VARCHAR(30), 
    txn_timestamp VARCHAR(30)
    )
    PARTITIONED BY (category_id)
    WITH (
        'connector' = 'kinesis',
        'stream' = '{kinesis_stream}',
        'aws.region' = 'us-east-2',
        'format' = 'json'
        );
  ```
* O comando '%flink.ssql' é fundamental para que o zepelin identifique que é um comando fliker, se não colocar os comandos SQL não funciona
* Criamos a tabela e inserimos o SCHEMA como um SGBD normal, inclusive, utilizando os mesmos comandos
* O comando PARTITIONED BY (category_id) é uma boa prática do flink
* O comando WITH indica os conectores, nesse caso, conector kinesis, em stream coloquei o meu fluxo de dados de entrada, o formato de leitura e a região da minha conta da AWS
* Agora vou criar mais uma célula e colocar um comando simples para poder ler o banco de dados
* ```
  %flink.ssql
  SELECT * FROM kinesis_pipeline_table_1;
  ```
* Vamos ver lá no AWS Glue se o banco foi criado
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/0bd244c4-5124-4de7-bf1e-6a89add3f392)
* Pronto, o banco foi criado da maneira correta no Glue
* Agora vamos ver se o flink está lendo corretamente o stream do kinesis. Execute a aplicação do Cloud9 para simular acessos no site e depois execute o comando de select
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/4c9c055b-7b99-4e28-abeb-40811a04863d)
* No meu caso, funcionou, agora vamos para a segunda parte, que é construir o input para o segundo fluxo de dados do kinesis
* Os dados que eu vou repassar para o segundo fluxo de dados é uma contagem de ações por usuário. Para isso, vou testar a seguinte query
* ```
  SELECT user_id,event_type, count(event_type) FROM kinesis_pipeline_table_1 GROUP BY user_id,event_type;
  ```
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/1b8903e0-f485-46e9-9468-add29e622c0f)
* Ok, está funcionando, agora, vou adicionar uma tabela nova antes da query, essa tabela vai corresponder ao segundo fluxo de dados
* ```
  %flink.ssql
  DROP TABLE IF EXISTS kinesis_pipeline_table_2;
  /*Aqui, vou usar o schema dos dados que chegam para a aplicação no formato json*/
  CREATE TABLE kinesis_pipeline_table_2 (
    user_id BIGINT, 
    event_type VARCHAR(30), 
    qtd_event BIGINT
    )
    WITH (
        'connector' = 'kinesis',
        'stream' = 'data_ecomerce_2',
        'aws.region' = 'us-east-2',
        'format' = 'json'
        );
  ```
* Vamos confirmar se a tabela foi criada no AWS Glue
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/c77b97a5-91ea-478f-8e9c-4ffb3f0ec796)
* Agora, vou inserir no insert into o comando de group by para inserir os dados no segundo fluxo do kinesis
* Segundo a [documentação](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/table/sql/queries/window-agg/) precisamos definir janelas de tempo para que possamgos fazer agregamentos em gropuby e envia-los por stream para o kinesis, então vamos fazer uma alteração na tabela 1 e adicionar como datetime a coluna 'txn_timestamp'
* O flink sql não tem suporte para alteração de colunas, então temos que apagar as colunas diretamente no glue, atera-las e executar o script novamente
* A nova tabela para o primeiro fluxo de dados foi alterada. Temos 3 diferenças, a primeira é a mudança em do tipo da coluna 'txn_timestamp' e a criação de uma nova coluna WATERMARK como fala a [documentação](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/table/sql/queries/window-agg/) e a terceira mudança é a inserção do parâmetro 'json.timestamp-format.standard' = 'ISO-8601' para que possa ser lido normalmente nossos dados na forma como construimos na seção de aplicação python com AWS Cloud9
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/6875b010-670f-4563-8126-2b2b205c3447)
* Executo o aplicativo python de simulação e vemos o coportamento da nova tabela
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/c22278cf-2b76-48c8-8244-67d653fe45de)
* Parece que está ok. Agora vamos criar a aplicação para o segundo fluxo de dados em janelas de 1 minuto
* Para executar o comando em uma janela de tempo, utilizei a seguinte query
* ```
  %flink.ssql
  SELECT user_id,event_type, count(event_type) as qtd FROM kinesis_pipeline_table_1 GROUP BY TUMBLE(txn_timestamp,INTERVAL '1'  
  MINUTE),user_id,event_type;
  ```
* Após executar com a aplicação de simulação pronta, temos o seguinte resultado
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/3ddd38b6-5ecd-45c8-a7e0-8bed1f1533e3)
* Ok, agora é so adicionar o INSERT e pronto
* ```
  INSERT INTO kinesis_pipeline_table_2 
  SELECT user_id,event_type, count(event_type) as qtd FROM kinesis_pipeline_table_1 GROUP BY TUMBLE(txn_timestamp,INTERVAL '1'   
  MINUTE),user_id,event_type;
  ```
* Agora, executamos a aplicação e vamos ver se os dados chegam no segundo fluxo de dados do kinesis
* Deixei um tempo a aplicação funcionando
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/c008c373-b7c4-41f5-ab1f-adb753b7f20e)
* E os dados estão chegando para o segundo fluxo de dados
* Estou enviando poucos dados porque tenho medo de cobranças, mas na foto abaixo já da pra visualizar os dados
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/bab9a8ad-d8a6-4d9f-9c01-6eaa98c28c00)
* Agora vamos por todas as células juntas em apenas uma célula e criar a build da aplicação
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/653527cd-2f4b-425f-ac3d-8abe9649ad1b)
* Depois, gere uma aplicação flink
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/8eb4dc8a-ed82-4332-840b-1e802e745f08)
* Pronto, aplicação flink criada, Agora é só executar a aplicação e o pipeline vai funcionar corretamente, sem necessarimanete executar diretamente no zepelin
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/873ca838-50a3-4eb2-a622-d3830d67a7f6)

# Construindo Sistema de Alerta com AWS SNS
* Em algumas situações, podemos ser atacados por bots ou algo do tipo
* Para essas situações, vamos construir um sistema de alerta que vai enviar um Email para uma conta cadastrada quando isso acontecer
* Para isso, vá até o SNS e crie um tópico, os clientes vão consumir esse tópico
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/d7830b0f-5bc2-41a2-9d73-723902ae29da)
* Crie como padrão
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/d5cef026-41f9-4374-903f-b96b26ac6c46)
* Deixe todos as config minimas e padrão e crie o tópico
* Depois de criado, insira um consumidor criando assinatura
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/f2f938f0-bb5d-4e6b-8809-862c00d553a2)
* Coloque para enviar um email-json e coloque seu email
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/ba4cc66c-2a9f-42c4-a882-8d2b221f5701)
* Para confirmar a assinatura, você entra no seu email, copia o url e confirma a assinatura com o URL enviado

# Implementando Banco de Dados DynamoDB
* Vamos agora criar o banco de dados que vai persistir os processamentos do segundo fluxo de dados do kinesis utilizando o AWS Lambda
* Para isso, vá para o serviço DynamoDB e crie uma nova tabela
* Agora dé um nome e uma chave primária
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/bdb044be-3563-4caf-8945-3166c021b409)
* No restante, mantenha a configuração padrão mínima
 
# Implementando AWS Lambda 
* Por motivos de custo, continuar utilizando essa arquitetura é muito caro para um projeto simples, então vou fazer uma leve modificação aqui. Em vez de processar os dados utilizando o flink (que me custou 4 dolares) vou fazer passar apenas o primeiro fluxo de dados
* Aqui é importante salientar uma coisa, os fluxos de dados estão funcionando por padrão, então, bastaria implementar no AWS Lambda o gatilho para o fluxo de dados processados, em vez do primeiro fluxo de dados. Nesse caso, só estou mostrando minha expertise, mas não posso me dar o luxo de gastar muito com isso
* Para criar o AWS Lambda, você entra no recurso



  
  
  
   
