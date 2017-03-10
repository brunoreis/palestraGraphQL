MINHA HISTÓRIA
==============





DISCLAIMER
==========

Não pretendo fazer um duelo do rest com o graphql, pelo contrário, apresentarei apenas minhas opiniões pessoais. 

Ambos são válidos, porém, para os tipos de aplicações que eu desenvolvo, GraphQL se apresentou como alternativa superior.

Vamos focar em GraphQL para entender o máximo possível neste pouco tempo. Dúvidas sobre GraphQL são muito bem vindas. Comparações com REST, talvez seja melhor conversarmos depois dado o tempo da palestra. 


HISTÓRICO
=========
Definido no ano 2000 by Roy Fielding, um dos autores do HTTP, em uma tese de doutorado.

"Do ponto de vista da evolução das tecnologias de software, é algo antigo."

GraphQL se originou no Facebook em 2012. Foi usado em larga escala por anos até ter seus padrões publicados em 2015.


O QUE É GRAPHQL
==================

GraphQL is 
- Uma linguagem de consulta para sua api 
- and a server-side runtime for executing queries by using a type system you define for your data. 

** GraphQL isn't tied to any specific database or storage engine and is instead backed by your existing code and data. **

A GraphQL service is 
- created by defining types and fields on those types, 
- then providing functions for each field on each type. 

A received query is 
  - first checked to ensure it only refers to the types and fields defined
  - then runs the provided functions to produce a result.

TEMOS A DEFINIÇÃO DE TIPOS E CAMPOS 
	- JÁ É UMA BELA VALIDAÇÃO
TEMOS A DEFINIÇÃO DE FUNÇÕES PARA CADA CAMPO 
	- ORGANIZA OS REQUESTS / TEM A "FUNÇÃO" DOS CONTROLLERS

GRAPHQL E REST
==============

Tradicionalmente se usava páginas monolíticas. Toda página era renderizada e mandada para o cliente/navegador.

Ao longo do tempo surgiram aplicações em javascript. Então o servidor manda a aplicação ao cliente, que por sua vez, consome dados. 

REST estabelece algumas restrições para garantir o bom uso do HTTp, mas não é um padrão bem definido. Por conta disso, há geralmente um grande acoplamento entre clientes e serviços específicos. 

GraphQL é protocolo aberto, mais claramente definido. É um protocolo de comunicação de alto nível, não específico para "Graph DBs" como pode parecer. É robusto e usado em bilhões de requests diários no FB.

Destacam-se também o SOAP, e o Falcor, mas as escolhas principais hoje no mercado são entre GraphQL e REST. 

Como GraphQL impõe certas convenções, isso torna os sistemas que o usam mais interoperáveis. Além disso, essas convenções e funcionalidades agregada, facilita muito a vida do desenvolvedor, que já encontra diversas fundações prontas para desenvolver. 


O QUE É UMA BOA API
===================

Rápida 
------
É bom que a API facilite fazer o mínimo possível de requests.
E que, a cada request, sejam enviados sómente os dados necessários.

Imutável (ou minimamente mutável) 
---------------------------------
Quanto mais consistente é uma API, é mais seguro que um ou mais clientes a usem. Mudar o mínimo as operações ao longo do tempo, não quebrar por adicionar novos campos e outros requisitos similares são muito bem vindos. 

Boas de Usar
------------
Boa documentação, facilidade de compreensão e uso, simplicidade, clareza são todos requisitos fundamentais que impactam em todo o projeto. 

PORQUE GRAPHQL É UMA BOA API
============================

Rápida 
------
Navega pela estrutura de relacionamento entre os dados, trazendo tudo o que for preciso. Se for o caso, busca mais de uma estrutura de dados em uma mesma query. Defino quais campos quer de cada objeto, de modo que campos desnecessários não são trafegados. 

Imutável (ou minimamente mutável) 
---------------------------------
Como uma query pode se expandir, adicionando campos e critérios/argumentos, e o cliente pode definir quais campos quer consultar. Alterações na API, uma vez ela formada, são mínimas ao longo do tempo. Quem tem usado há tempos diz que não precisou estabelecer uma v2.  

Boa de Usar
-----------
Assim que for definido o schema, que é requisito para o servidor funcionar, já temos disponível uma documentação da API. Esta pode ser usada e explorada usando belas ferramentas que já fornecem dicas(hints), graças à característica do GraphQL de permitir introspecção. 

Além disso, temos a facilidade de customizar as queries no cliente, que dá uma liberdade muito maior para o desenvolvedor do front, que pode adicionar e remover campos e relacionamentos das queries de acordo com as demandas que surgirem na evolução do software. 


VANTAGENS
=========
- self-checks embedded on the ground level of your backend architecture
- reusable API for different client versions and devices, i.e. no more need in maintaining "/v1" and "/v2"
- a complete new level of distinguishing of the backend and frontend logic
- easily generated documentation and incredibly intuitive way to explore created API
- once your architecture is complete – most client-based changes does not require backend modifications

PRIMEIRO EXEMPLO
==================

GraphQL is about asking for specific fields on objects

Exemplo:
{
  hero {
    name
  }
}
--- result --- 
{
  "data": {
    "hero": {
      "name": "R2-D2"
    }
  }
}

- query has exactly the same shape as the result

CAMPOS QUE SÃO OBJETOS
======================

navegando no grafo

thread{
	id
	messages{
		id
		text
		sentAt
	}
}
-------
{thread:{
	34
	messages:[
		{
			id:1,
			text:"Olá",
			sentAt:'2005-12-22'
		},
		{
			id:2,
			text:"Tudo bem?",
			sentAt:'2005-12-22'
		},
	]
}}

Como é feita a distinção entre um item e uma lista? Pelo schema. 

Soluções em rest: 
	1 - criar um endpoint específico que manda as mensagens aninhadas
	2 - fazer vários requests

Com GraphQL uma query apenas é especificada no schema e ela pode ser chamada por diferentes queries no cliente. 


ARGUMENTOS
==========

especifique cada campo

thread(id:2){
	id
	messages(first:20){
		id
		text(truncate:30)
		sentAt(locale:["pt","BR"])
	}
}

Cada campo pode receber seus argumentos. No rest os argumentos são específicos de um request. Essa abordagem do GraphQL repõe completamente múltiplos requests. 

ALIASES
=======

thread(id:2){
	id,

	firstMessages: messages(first:5){
		id
		text
		sentAt
	},

	lastMessages: messages(last:5){
		id
		text
		sentAt
	}
}

Você pode renomear um campo, para usá-lo mais de uma vez em uma mesma query. 

FRAGMENTOS
==========

para não ficar repetindo campos em uma mesma query ou mesmo em queries diferentes, use fragmentos

thread(id:2){
	id,
	firstMessages: messages(first:5){
		...messageFields
	},
	lastMessages: messages(last:5){
		...messageFields
	}
}

fragment messageFields on Message {
  id
  text
  sentAt
}


PASSANDO VARIÁVEIS
==================

query oneThread($id: ID) {
	thread(id:$id) {
		...
	}
}
Variables:
{
  "id": 2
}

A primeira linha tem a definição. Onde $id é o nome e "ID" é o tipo. Depois disso a query repõe id na segunda linha pelo valor passado. 

As queries nunca devem ser montadas concatenando strings, sempre com a passagem de variávies. Isso facilita muito uma série de operações e otimizações tanto nos clientes como nos servidores. 

Tipos complexos podem ser passados, de acordo com os tipos especificados no servidor.

OBS: variáveis exigem uma sintaxe que tem um nome da operação (oneThread nesse caso). Nomes de operação também são recomendados para sistemas em produção. Para debug e outras funcionalidades. 

DIRETIVAS
=========

@include(if: Boolean) Inclui o campo se for true
@skip(if: Boolean) Pula o campo se for true

thread ($withMessages: Boolean!) {
	thread{
		id
		messages @include(if: $withMessages){
			id
			text
			sentAt
		}
	}
}

Variáveis:
{
	withMessages:true
}


MUTATIONS
=========
Mutations são queries. 
Específicas, dentro de um "namespace raiz" específico. 
E, por convenção, são as únicas que devem alterar dados no servidor. 

As mutations retornam dados. São tipadas tal qual as queries, com um tipo de retorno. E também permitem "diferentes queries" no cliente especificando o que retornar. 

Podem passar diversos campos, fazendo assim com que um mesmo request execute mais de uma operação. 

Ao passar mais de um campo, nas queries eles são executados em paralelo. Nas mutations, executam de forma serial.


TÓPICOS AVANÇADOS
=================
Inline Fragments - especificando campos diferentes dependento do tipo, para uma query em um "union" ou "interface"



=============
APOLLO CLIENT
=============

Apollo is 
 - an incrementally-adoptable data stack 
 - that manages the flow of data between clients and backends. 
 - based on GraphQL, it gives you a principled, unified, and scalable API for developing modern apps on top of services.

Mostrar ao menos como funciona:
http://dev.apollodata.com/core/how-it-works.html
https://dev-blog.apollodata.com/the-concepts-of-graphql-bc68bd819be3#.jpqm96sp2

OUTROS
======
https://www.reindex.io/docs/

Exemplos legais de uso: 
=======================

A query que busca os status das mensagens pode ser chamada de um jeito, fazendo 4 queries, ou de outro, fazendo apenas uma. 

 - isso pode ser feito com a mesma query definida no servidor
   - ela pode ser chamada de duas formas no cliente
 - ao se chamar um campo apenas, apenas um resolver é chamado
   - instalar algum debug para monitorar as queries
 - o resultado é guardado no cache do apollo (pois tem tipo e id)
 - ao fazer a segunda query o resultado é atualizado onde for pertinente

 mostrar exemplo de outro resultado atualizado com refetchQueries

====================
IMPLEMENTAÇÃO EM PHP
====================

youshido
 - Full compatibility with the RFC Specification for GraphQL
 - Agile object oriented structure to architect your GraphQL Schema
 - Intuitive Type system that allows you to build your project much faster and stay consistent
 - Build-in validation for the GraphQL Schema you develop
 - Well documented classes with a lot of examples
 - Automatically created endpoint /graphql to handle requests



