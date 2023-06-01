# **Projeto** | Pipeline de Dados do Telegram

---

# **Tópicos**

<ol type="1">
  <li>Contexto;</li>
  <li>Telegram;</li>
  <li>Bot API;</li>
  <li>Dados;</li>
  <li>Ingestão;</li>
  <li>ETL;</li>
  <li>Apresentação;</li>
  <li>Conclusão.</li>
</ol>

---


## 1\. Contexto

> O principal objetivo desse projeto é a criação de um pipeline de dados baseado em nuvem.
> O pipeline de dados foi projetado para receber, processar e armazenar mensagens de texto provenientes de um grupo específico no aplicativo de mensagens **Telegram**. Um bot foi adicionado ao grupo, criado dentro da própria plataforma, e responsável por coletar as mensagens e enviá-las via API para a infraestrutura em nuvem da Amazon Web Services (AWS). Lá, as mensagens são processadas, armazenadas, tratadas e analisadas.

### **1.1. Chatbot** 

> Um **chatbot** é um tipo de software que interage com usuários através de conversas automatizadas em plataformas de mensagens. Uma aplicação comum de **chatbots** é o seu uso no atendimento ao cliente, onde, de maneira geral, ajudam clientes a resolver problemas ou esclarecer dúvidas recorrentes antes mesmo que um atendente humano seja acionado.

### **1.2. Telegram** 

> **Telegram** é uma plataforma de mensagens instantâneas **freeware** (distribuído gratuitamente) e, em sua maioria, open source. É muito popular entre desenvolvedores por ser pioneiro na implantação da funcionalidade de criação de chatbots, que, por sua vez, permitem a criação de diversas automações.

### **1.3. Arquitetura** 

Uma atividade analítica de interesse é a de realizar a análise exploratória de dados enviadas a um **chatbot** para responder perguntas como:

1. Qual o horário que os usuários mais acionam o *bot*?
1. Qual o problema ou dúvida mais frequente?
1. O *bot* está conseguindo resolver os problemas ou esclarecer as dúvidas?
1. Etc.

> Portanto, vamos construir um *pipeline* de dados que ingira, processe, armazene e exponha mensagens de um grupo do **Telegram** para que profissionais de dados possam realizar análises. A arquitetura proposta é dividida em duas: `transacional`, no **Telegram**, onde os dados são produzidos, e `analítica`, na Amazon Web Services (AWS), onde os dados são analisados.

![modulo_43-4.png](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/0015%20-%20Projeto%20final.png?raw=true)

 - **Telegram**

> O `Telegram` representa a fonte de dados transacionais. Mensagens enviadas por usuários em um grupo são capturadas por um *bot* e redirecionadas via *webhook* do *backend* do aplicativo para um *endpoint* (endereço *web* que aceita requisições HTTP) exposto pelo `AWS API Gateway`. As mensagens trafegam no corpo ou *payload* da requisição.

 - **AWS | Ingestão**

> Uma requisição HTTP com o conteúdo da mensagem em seu *payload* é recebida pelo `AWS API Gateway` que, por sua vez, as redireciona para o `AWS Lambda`, servindo assim como seu gatilho. Já o `AWS Lambda` recebe o *payload* da requisição em seu parâmetro *event*, salva o conteúdo em um arquivo no formato JSON (original, mesmo que o *payload*) e o armazena no `AWS S3` particionado por dia.

 - **AWS | ETL**

> Uma vez ao dia, o `AWS Event Bridge` aciona o `AWS Lambda` que processa todas as mensagens do dia anterior (atraso de um dia ou D-1), denormaliza o dado semi-estruturado típico de arquivos no formato JSON, salva o conteúdo processado em um arquivo no formato Apache Parquet e o armazena no `AWS S3` particionado por dia.

- **AWS | Apresentação**

> Por fim, uma tabela do `AWS Athena` é apontada para o *bucket* do `AWS S3` que armazena o dado processado: denormalizado, particionado e orientado a coluna. Profissionais de dados podem então executar consultas analíticas (agregações, ordenações, etc.) na tabela utilizando o SQL para a extração de *insights*.

---

## 2\. Telegram.

> Nesta etapa do projeto criei o bot **ProjetoFinalEBAC_bot** no Telegram, e criei o grupo **ProjetoFinalEBAC Group**, onde adicionei o bot, e foram enviadas as mensagens para que posteriormente sejam realizadas as análises.

- **Obs:** O bot precisa estar como um dos adiministradores do grupo, para que possa capturar as mensagens enviadas. A opção de adicionar o bot a outros grupos também foi desabilitada, para que não haja nenhuma interferencia na condução do projeto.

2.1. Criando o *bot*.

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Crie%20um%20bot%20-%20Projeto%20final.png?raw=true)

---

2.2. Criando o grupo e adicionando o *bot*.

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Crie%20um%20grupo%20e%20adicione%20o%20bot%20-%20Projeto%20final.png?raw=true)

---

2.3. Tornando o *bot* administrador do grupo.

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Torne%20o%20bot%20administrador%20do%20grupo%20-%20Projeto%20final.png?raw=true)

---

2.4. Desabilitando a opção de adicionar o *bot* a novos grupos.

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Desabilite%20a%20op%C3%A7%C3%A3o%20de%20adicionar%20o%20bot%20a%20novos%20grupos%20-%20Projeto%20final.png?raw=true)

---

2.5. Mensagens enviadas ao grupo (text, imagem, arquivos, video, áudio, etc.) para serem consumidas pela API de *bots* do **Telegram**.

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Envie%20diversos%20tipos%20de%20mensagens%20no%20grupo%20-%20Projeto%20final.png?raw=true)

---

## 3\. Bot API

> As mensagens captadas por um *bot* podem ser acessadas via API. A única informação necessária é o `token` de acesso fornecido pelo `BotFather` na criação do *bot*. Para isso, vamos utilizar o pacote nativo do Python **getpass** para ingerir o token, e não deixá-lo exposto no Google colab.

- **Nota:** A documentação completa da API pode ser encontrada neste [link](https://core.telegram.org/bots/api)

![imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/0001%20-%20Projeto%20final.png?raw=true)

> A `url` base é comum a todos os métodos da API.

![imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/0002%20-%20Projeto%20final.png?raw=true)

- **getMe**

> O método `getMe` retorna informações sobre o *bot*.

![](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/0003%20-%20Projeto%20final.png?raw=true)

 - **getUpdates**

O método `getUpdates` retorna as mensagens captadas pelo *bot*.

![](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/0004%20-%20Projeto%20final.png?raw=true)

## 4\. Dados

> Antes de avançar para etapa analítica, vamos trabalhar na manipulação dos dados de mensagens do **Telegram**. 

### **4.1. Mensagem** 

> Uma mensagem recuperada via API é um dado semi-estruturado no formato JSON com algumas chaves mandatórias e diversas chaves opcionais, estas últimas presentes (ou não) dependendo do tipo da mensagem. Por exemplo, mensagens de texto apresentam a chave `text` enquanto mensagens de áudio apresentam a chave `audio`. Neste projeto vamos focar em mensagens do tipo texto, ou seja, vamos ingerir as chaves mandatórias e a chave `text`.

[texto do link](https://)- **Nota**: A lista completa das chaves disponíveis pode ser encontrada na documentação neste [link](https://core.telegram.org/bots/api#message).

> Exemplo:

![](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/0005%20-%20Projeto%20final.png?raw=true)

> Descrição:

| chave | tipo valor | opcional | descrição |
| -- | -- | -- | -- |
| updated_id | int | não | id da mensagem enviada ao **bot** |
| message_id | int | não | id da mensagem enviada ao grupo |
| from_id | int | sim | id do usuário que enviou a mensagem |
| from_is_bot | bool | sim | se o usuário que enviou a mensagem é um **bot** |
| from_first_name | str | sim | primeiro nome do usário que enviou a mensagem |
| chat_id | int | não | id do *chat* em que a mensagem foi enviada |
| chat_type | str | não | tipo do *chat*: private, group, supergroup ou channel |
| date | int | não | data de envio da mensagem no formato unix |
| text | str | sim | texto da mensagem |

### **4.2. Wrangling** 

> Vamos denormalizar o conteúdo da mensagem semi-estruturado no formato JSON utilizando apenas Python nativo, ou seja, sem o auxílio de pacotes, como Pandas.

Para começar, vamos carregar o arquivo `telegram.json` utilizando o pacote nativo `json`.

![](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/0006%20-%20Projeto%20final.png?raw=true)

> Vamos então utilizar um laço de repetição para varrer todas as chaves do arquivo e selecionar apenas as de interesse. Caso a mensagem não possua a chave `text`, ela será criada com o valor igual a `None`. Além disso, vamos adicionar duas chaves de tempo para indicar o momento em que o dado foi processado: `context_date` e `context_timestamp`.

![](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/0007%20-%20Projeto%20final.png?raw=true)

> Por fim, vamos utilizar o pacote Python PyArrow para criar uma tabela com os dados processados que, posteriormente, pode ser facilmente persistida em um arquivo no formato Apache Parquet.

![](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/0008%20-%20Projeto%20final.png?raw=true)

*texto em itálico*## 5\. Ingestão

> A etapa de ingestão é responsável, como seu próprio nome já diz, pela ingestão dos dados transacionais em ambientes analíticos. De maneira geral, o dado ingerido é persistido no formato mais próximo ao original, ou seja, nenhuma transformação é realizada em seu conteúdo ou estrutura (schema). Como exemplo, dados de uma API web que segue o formato REST (representational state transfer) são entregues, logo, persistidos no formato JSON.

- Persistir os dados em seu formato original trás muitas vantagens, como a possibilidade de reprocessamento.

Pode ser conduzida de duas formas:

- **Batch:** blocos de dados são ingeridos em uma frequência bem definida, geralmente na escala de horas ou dias;
- **Streaming:** dados são ingeridos conforme são produzidos e disponibilizados.

> No projeto, as mensagens capturadas pelo bot podem ser ingeridas através da API web de bots do **Telegram**, portanto são fornecidos no formato JSON. Como o Telegram retem mensagens por apenas 24h em seus servidores, a ingestão via **streaming** é a mais indicada. Para que seja possível esse tipo de ingestão, vamos utilizar um webhook (gancho web), ou seja, vamos redirecionar as mensagens automaticamente para outra API web.

> Sendo assim, precisamos de um serviço da AWS que forneça um API web para receber os dados direcionados, o `AWS API Gateway`. Dentre suas diversas funcionalidades, o AWS API Gateway permite o direcionamento do dado recebido para outros serviços da AWS. Logo, vamos conectá-lo ao `AWS Lambda`, que por sua vez, irá armazenar o dado em seu formato original (JSON) em um Bucket do AWS S3.

- Sistemas que reagem a eventos são conhecidos como `event-driven`.

Portanto, precisamos: 

- Criar um bucket no **AWS S3**;
- Criar uma função no **AWS Lambda**;
- Criar uma API web no **AWS API Gateway**;
- Configurar o **webhook** da API de bots do **Telegram**.

### **5.1. AWS S3** 

> Na etapa de **ingestão**, o AWS S3 tem a função de passivamente armazenar as mensagens captadas pelo bot do Telegram no seu formato original (JSON). Para tanto, basta a criação de um bucket. Como padrão, vamos adicionar o sufixo **-raw** ao seu nome (vamos seguir esse padrão para todos os serviços dessa camada).

- **Nota**: Um `data lake` é o nome dado a um repositório de um grande volume de dados. É organizado em zonas que armazenam réplicas dos dados em diferentes níveis de processamento. A nomenclatura das zonas varia, contudo, as mais comuns são: raw e enriched ou bronze, silver e gold. 

*texto em itálico*> **Bucket no AWS S3:**

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/bucket%20s3%20-%20Projeto%20final.png?raw=true)

---

### **5.2. AWS Lambda** 

> Na etapa de ingestão, o `AWS Lambda` tem a função de ativamente persistir as mensagens captadas pelo bot do **Telegram** em um bucket do `AWS S3`. Para tanto vamos criar uma função que opera da seguinte forma:

- Recebe a mensagem no parâmetro `event`;
- Verifica se a mensagem tem origem no grupo do **Telegram** correto;
- Persiste a mensagem no formato JSON no bucket do `AWS S3`;
- Retorna uma mensagem de sucesso (código de retorno HTTP igual a 200) a API de bots do **Telegram**.


**Nota**: No **Telegram** restringimos a opção de adicionar o bot a grupos, contudo, ainda é possível iniciar uma conversa em um chat privado.

> **Função Lambda:**

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Fun%C3%A7%C3%A3o%20lambda%20criada%20-%20Projeto%20final.png?raw=true)

---

> **O código da função:**

![](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/0009%20-%20Projeto%20final.png?raw=true)

> Para que a função funcione corretamente, algumas configurações precisam ser realizadas.

- **Variáveis de ambiente**

Note que o código exige a configuração de duas variáveis de ambiente: **AWS_S3_BUCKET** com o nome do bucket do **AWS_S3** e **TELEGRAM_CHAT_ID** com o id do chat do grupo do **Telegram**. Para adicionar variáveis de ambiente em uma função do **AWS Lambda**, basta acessar configurações -> variáveis de ambiente no console da função.

**Nota**: Variáveis de ambiente são excelentes formas de armazenar informações sensíveis.

> **Preenchendo as variáveis de ambiente na função Lambda criada na AWS:**

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Preencher%20vari%C3%A1veis%20de%20ambiente%20-%20Projeto%20final.png?raw=true)

---

- **Permissão**

> Por fim, precisamos adicionar a permissão de escrita no bucket do `AWS S3` para a função do `AWS Lambda` no `AWS IAM`.

> **Política anexada:**

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Permiss%C3%A3o%20anexada%20-%20Projeto%20final.png?raw=true)

---

> **Teste da função Lambda:**

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Teste%20fun%C3%A7%C3%A3o%20Lambda%20-%20Projeto%20final.png?raw=true)

---

> **Dado armazenado no bucket do `AWS S3` após a execução da função `Lambda`:**

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Dado%20armazenado%20ap%C3%B3s%20a%20execu%C3%A7%C3%A3o%20do%20c%C3%B3digo%20-%20Projeto%20final.png?raw=true)

---

### **5.3. AWS API Gateway** 

> Na etapa de **ingestão**, o `AWS API Gateway` tem a função de receber as mensagens captadas pelo bot do **Telegram**, enviadas via webhook, e iniciar uma função no `AWS Lambda`, passando o conteúdo da mensagem no seu parâmetro event. Para tanto vamos criar uma API, e configurá-la como gatilho da função do `AWS Lambda`.

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Gateway%20-%20Projeto%20final.png?raw=true)

---

> Podemos testar a integração com `AWS Lambda` através da ferramenta de testes do serviço. Por fim, vamos fazer a implantação da API e obter o seu endereço web.

- **Teste:**

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Teste%20Gateway%20-%20Projeto%20final.png?raw=true)

---

> **Implantação da API (deploy):**

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/URL%20Gateway%20-%20Projeto%20final.png?raw=true)

---

> **Ingetar a url do deploy:**

![](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/0010%20-%20Projeto%20final.png?raw=true)

### **5.4. Telegram** 

> Vamos configurar o webhook para redirecionar as mensagens para a url do `AWS API Gateway`.

- **setWebhook**

> O método `setWebhook` configura o redirecionamento das mensagens captadas pelo bot para o endereço web do parâmetro url.

**Nota**: os métodos `getUpdates` e `setWebhook` são mutuamente exclusivos, ou seja, enquanto o webhook estiver ativo, o método `getUpdates` não funcionará. Para desativar o webhook, basta utilizar o método `deleteWebhook`.

![](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/0011%20-%20Projeto%20final.png?raw=true)

- **getWebhookInfo**

> O método `getWebhookInfo` retorna as informações sobre o webhook configurado.

![](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/0012%20-%20Projeto%20final.png?raw=true)

## 6\. ETL (Extraction, transformation and load)

> A etapa de extração, transformação e carregamento é a etapa responsável pela manipulação dos dados extraídos na etapa transacional, ou seja, persistidos em camadas cruas (ou raw) de sistemas analíticos. O dado cru armazenado passa por um processo recorrente onde ele é limpo, duplicado e persistido com técnicas de particionamento, orientado a coluna e compressão. Ao final, o dado está pronto para ser analisado por profissionais da área.

> Uma vez ao dia, o `AWS Event Brigde` aciona o `AWS Lambda` que, por sua vez, processa todas as mensagens geradas no dia anterior (D-1), denormaliza o dado semi-estruturado típico de arquivos JSON, salva o conteúdo processado em um arquivo no formato Apache Parquet, e o armazena no `AWS S3` particionado por dia.

> **Bucket criado para receber o arquivo processado no formato `Apache Parquet`:**

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Bucket%20enriched%20-%20Projeto%20final.png?raw=true)

---

> O código abaixo é executado diariamente pelo `AWS Event Brigde`, em horário pré-estabelecido, para compactar as diversas mensagens que chegam no grupo (do dia anterior), no formato JSON armazenadas no bucket de dados cru (raw), em um único arquivo no formato Parquet. Após o processamento, as mensagens são armazenadas no bucket de dados enriquecidos (enriched).

![](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/0013%20-%20Projeto%20final.png?raw=true)

> Junto ao código acima, temos que adicionar um laço de repetição para varrer todas as chaves do arquivo e selecionar apenas a **text**, que é a única de interesse deste projeto. Caso a mensagem não possua a chave text, ela será criada com o valor **None**.

![](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/0014%20-%20Projeto%20final.png?raw=true)

> **Função Lambda criada na AWS (enriched):**

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Fun%C3%A7%C3%A3o%20Lambda%20Enriched%20-%20Projeto%20final.png?raw=true)

---

**Nota**: O procedimento para as configurações de **Variáveis de ambiente** e **Permissão** foram feitas da mesma forma como foi mostrado na função anterior (datalake-projetofinalebac-raw).

- **Camadas**

> Por fim, note que o código da função utiliza o pacote Python PyArrow. Contudo, o ambiente padrão do `AWS Lambda` possui poucos pacotes externos instalados, como o pacote Python boto3, logo PyArrow não será encontrado e a execução da função falhará. Existem algumas formas de adicionar pacotes externos no ambiente de execução do `AWS Lambda`, um deles é a criação de camadas ou layers, onde podemos fazer o upload dos pacotes Python direto na plataforma, ou através de um bucket do `AWS S3`. Vamos utilizar a última opção, criando um bucket no `AWS S3`, fazendo o upload do pacote Python do PyArrow  ([link](https://github.com/aws/aws-sdk-pandas/releases)), criando layer e conectando na função.

> **Bucket do S3 criado, e arquivo carregado:**

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Bucket%20layer%20com%20arquivo%20carregado%20-%20Projeto%20final.png?raw=true)

---

> **Versão da camada PyArrow criada:**

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Vers%C3%A3o%20da%20camada%20pyarrow%20criada%20-%20Projeto%20final.png?raw=true)

---

> **Função executada com sucesso:**

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Fun%C3%A7%C3%A3o%20executada%20com%20sucesso%20-%20Projeto%20final.png?raw=true)

---

> **Arquivos carregados no bucket do S3 (enriched) no formato Parquet:**

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Arquivos%20carregados%20no%20formato%20Parquet%20-%20Projeto%20final.png?raw=true)

---

### **6.1. AWS Event Bridge** 

> Na etapa de **ETL**, o `AWS Event Bridge` tem a função de ativar diariamente a função de **ETL** do `AWS Lambda`, funcionando assim como um scheduler.

- **Nota**: Atente-se ao fato de que a função processa as mensagens do dia anterior (D-1).

*texto em itálico*> **Regra criada:**

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Regra%20criada%20-%20Projeto%20final.png?raw=true)

---

## 7\. Apresentação

> A etapa de **apresentação** é responsável por entregar o dado aos usuários (analistas, cientistas, etc) e sistemas (dashboards, motores de consultas. etc), idealmente através de uma interface de fácil uso, como o SQL, logo, essa é a única etapa que a maioria dos usuários terá acesso. Além disso, é importante que as ferramentas da etapa entreguem dados armazenados em camadas refinadas, pois assim as consultas são mais baratas e os dados mais consistentes.

### **7.1. AWS Athena**

> Na etapa de **apresentação**, o `AWS Athena` tem a função de entregar os dados através de uma interface SQL para os usuários do sistema analítico. Para criar a interface, basta criar uma tabela externa sobre o dado armazenado na camada mais refinada da arquitetura, a camada enriquecida.

> **Criando a tabela `data_pipeline_demo_enriched` no `AWS Athena`:**

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Criando%20a%20tabela%20no%20Athena%20-%20Projeto%20final.png?raw=true)

---

> **Adicionando as partições disponíveis:**

**Nota**: Toda vez que uma nova partição é adicionada ao repositório de dados, é necessário informar ao `AWS Athena` para que ela esteja disponível via SQL.

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Carregando%20as%20parti%C3%A7%C3%B5es%20-%20Projeto%20final.png?raw=true)

---

> **Selecionando as 10 primeiras linhas da tabela:**

`Query`

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Query%2010%20linhas%20-%20Projeto%20final.png?raw=true)

---

`Resultado`

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Resultado%20query%2010%20linhas%20-%20Projeto%20final.png?raw=true)

---

### **7.2. Analytics**

> Com o dado disponível, os usuários podem executar consultas analíticas para extrair insights dos dados.

- `Quantidade de mensagens por dia`

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Quantidade%20de%20mensagens%20por%20dia%20-%20Projeto%20final.png?raw=true)

- `Visualização`

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Gr%C3%A1fico%20quantidade%20mensagens%20por%20dia%20-%20Projeto%20final.png?raw=true)

---

- `Quantidade de mensagens por usuário por dia`

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Quantidade%20de%20mensagens%20por%20usu%C3%A1rio%20por%20dia%20-%20Projeto%20final.png?raw=true)

- `Visualização`

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Gr%C3%A1fico%20quantidade%20mensagens%20por%20usu%C3%A1rio%20por%20dia%20-%20Projeto%20final.png?raw=true)

**Nota**: Como o grupo foi criado apenas para fins didáticos, não adicionei ninguém além do bot, por isso o resultado retornou somente 1 usuário na consulta. 

---

- `Média do tamanho das mensagens por dia`

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/Tamanho%20m%C3%A9dio%20de%20mensagens%20por%20dia%20-%20Projeto%20final.png?raw=true)

- `Visualização`

![Imagem](https://github.com/jaquessonoliveira/modulo38-ebac/blob/main/gr%C3%A1fico%20tamanho%20m%C3%A9dio%20de%20mensagens%20por%20dia%20-%20Projeto%20final.png?raw=true)

---

## 8\. Conclusão

> A utilização de `chatbots` em conjunto com a `análise de dados` extraídos através de um pipeline traz benefícios significativos para empresas de todos os tamanhos. Os `chatbots` desempenham um papel fundamental ao facilitar a interação dos clientes com a empresa, permitindo que eles resolvam problemas e tirem dúvidas diretamente do seu smartphone ou computador, eliminando a necessidade de visitar uma loja física ou entrar em contato com um funcionário.

> Além disso, os dados obtidos a partir dessas interações são extremamente valiosos para os tomadores de decisão. Através de análises personalizadas, os gestores podem obter insights que vão desde o comportamento dos clientes até feedbacks e esclarecimentos sobre os produtos oferecidos. Essas informações fornecem uma visão aprofundada do mercado e permitem que a empresa tome decisões mais assertivas e estratégicas.

> Dessa forma, a combinação entre `chatbots` e `análise de dados` é uma poderosa ferramenta para melhorar a experiência do cliente, otimizar processos internos e impulsionar o crescimento das empresas em todos os setores.
