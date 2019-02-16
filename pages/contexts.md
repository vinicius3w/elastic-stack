## Entendendo melhor os contextos

> Artigo atualizado para a versão 6.0+ do Elasticsearch por __[Vinicius Garcia](https://github.com/vinicius3w)__

Até agora, aprendemos a inserir dados em nosso Elasticsearch informando o __id__ que o documento irá possuir. Um exemplo disso foi a nossa primeira inserção e o conteúdo do script [funcs.sh](/scripts/funcs.sh), onde passamos o caminho completo que o documento será inserido:

```
curl -XPUT http://localhost:9200/mycompany/funcionarios/1
```

Neste tipo de inserção, informamos o index, o type e o id (1 no exemplo acima) do documento à ser inserido. Isso nos  ajuda no momento de fazer uma busca, por já sabermos o caminho completo do dado. Por ex:

```
curl -XGET http://localhost:9200/mycompany/funcionarios/1
```

A nível de teste não há problemas neste tipo de abordagem, mas não é comum informarmos o id no momento da inserção do dado por 'N' motivos. Para ids, geralmente deixamos o Elasticsearch fazer a criação dinamicamente. Ou seja, você pode inserir dados no contexto que desejar sem precisar se preocupar com um número de id. Vamos criar mais um type para o nosso index "mycompany" chamado "diretores":

```
curl -XPOST http://localhost:9200/diretor/diretores/ -H 'Content-Type: application/json' -d '
{
  "nome": "Roberto Roberts",
  "idade": 40,
  "endereco": "Rua da Chiqueza",
  "hobbies": ["Jogar golf", "Fazer chiquezas"],
  "interesses": ["orquestras", "coisas chiques"]
}'
```

Veja que utilizamos o verbo HTTP __POST__ ao invés de __PUT__. Como vimos muito anteriormente, o verbo __PUT__ fala para o Elasticsearch armazenar o dado em uma URL específica, ou seja, em um caminho completo. Já o __POST__ diz para o Elasticsearch armazenar o dado _ABAIXO_ de uma URL. Leia o diálogo abaixo para entender melhor:

#### PUT VS ELASTICSEARCH

```
PUT: "Oi Elasticsearch."
Elasticsearch: "Fala..."
PUT: "Faz um favor ?"
Elasticsearch: "Diz..."
PUT: "Leva esse documento nesse endereço aqui ó... Bairro: diretor, Rua: diretores no Numero: 1.
Elasticsearch: "Beleza."
```

#### POST VS ELASTICSEARCH

```
POST: "E ai Elasticsearch, tudo bom :) ?"
Elasticsearch: "Lá vem você denovo pra me dar trabalho..."
POST: "Que nada ! Bom.. na verdade, eu gostaria que você enviasse um documento em um endereço pra mim."
Elasticsearch: "Ta... ta... me passa o endereço."
POST: "Então... fica no Bairro: diretor, na Rua: diretores e... hmmmm."
Elasticsearch: "Hmmm o que ? Qual é o número ?"
POST: "Esse é o problema.. eu não tenho o número, mas precisamos entregar isso agora :("
Elasticsearch: "Ah.. que ótimo. Nesse caso, vou entregar em qualquer número que eu escolher !"
```

Tirando o _incrível mau humor_ do Elasticsearch em realizar inserções de dados, é mais ou menos isso que acontece. Se não informarmos o id, o Elasticsearch gera automaticamente um id para o nosso documento. Vamos ver qual o id que ele escolheu para o nosso _fino_ diretor:

```
curl -XGET http://localhost:9200/diretor/diretores/_search?pretty
```

O id que ele escolheu para o meu documento foi "AWDg5HpIZFpbSN2whJNa" e provavelmente, uma outra sequência bizarra de caractes foi escolhida para você.

Outro ponto que talvez você não tenha reparado, é que podemos realizar consultas a nível de ids, types ou index. Por exemplo, se eu não souber em qual type o funcionário "Claudio Silva" está inserido, eu posso realizar uma pesquisa a nível de index. Por exemplo:

```
curl -XGET http://localhost:9200/mycompany/_search?pretty -H 'Content-Type: application/json' -d '
{
  "query": {
    "match_phrase": { "nome": "Claudio Silva"}
  }
}'
```

Mude o valor de "nome" para "Robert Roberts" e sua busca também encontrará o resultado. Isto acontece, pois os types "funcionarios" e "diretores" estão inseridos no mesmo index (mycompany).

> Com as mudanças na versão 6.0+ por conta da remoção do mappings, não vai funcionar exatamente desta forma.

Anterior: [Contagem de Documentos](/pages/counting.md) | Próximo: [Deletando](/pages/delete.md)
