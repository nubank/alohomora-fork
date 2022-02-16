# Componentes NuInvest (lado Responsys)

Os componentes de e-mail são blocos reutilizáveis e dinâmicos. No fluxo Figma -> Alohomora -> Responsys, de criação de templates e campanhas, temos dois arquivos para cada componente, por assim. Temos o código do componente escrito em JS/React, do lado do Alohomora, e do lado do Responsys temos o código em HTML, CSS e RPL.

Todos os arquivos presentes nesse repositório contam com alguma quantidade de RPL, seja para receber dados, definir propriedades condicionalmente, ou qualquer outra funcionalidade que envolva lógica. Nem todos os arquivos são componentes. Os arquivos dentro da pasta *base* são arquivos base de configuração, dentro da pasta *helpers* estão os arquivos auxiliares, como a macro de links, por exemplo. E na pasta *mails* estão alguns blocos e e-mails completos montados ao redor dos componentes que já existem, ao passar do tempo.

# RPL (Responsys Personalization Language)

O Responsys possui uma linguagem própria de template, chamada de RPL. A sintaxe é parecida com HTML, pois o RPL é usado junto ao HTML, em arquivos .html, porém com diretivas e funções similares à qualquer linguagem de programação, como loops, condicionais, etc.

## Conceitos básicos

Os tipos de variável suportados são string, number, boolean, sequence e hash. 

**string:** "Some string"
**number:** 23
**boolean:** true | false
**sequence:** [ "Some string", 23, true ]
**hash:** { "id": 23, "color": "red", "isActive": false }

Sequence é como um array ou lista, e hash é um objeto (estilo JSON). Uma string sempre precisa estar entre aspas, sejam simples ou duplas, porém preferencialmente duplas para manter um padrão de código. O mesmo vale para o nome de propriedades dentro de um hash.

<hr>

Geralmente as tags abrem com ``<``, e são seguidas de ``#``. Quando elas se autofecham, igual a tag ``<img />`` do html, nós fechamos apenas com um ``>``, sem necessidade da barra. Quando existe uma tag de fechamento, ela é similar à tag de abertura, porém com uma barra antes do ``#``. Exemplo:

*com tag de fechamento*
``<#if></#if>``
*sem tag de fechamento*
``<#assign var = "hello">``

## Variáveis

Para utilizar uma variável, você pode usar ``${variable}``, englobando o nome dela dentro de um símbolo de dólar e chaves. Por exemplo:

``<h1>${title}</h1>``

Isso pode ser usado com textos também, interpolando os valores, ``<h1>Boas vindas, ${nome}</h1>``. Você pode concatenar hash, ``<#assign newHash = {"Joe":23, "Fred":25} + {"Joe":30, "Julia":18}>``, sequences ``<#assign newSeq = ["Joe", "Fred"] + ["Julia", "Kate"]>``.

Ao usar variáveis dentro de diretivas, não use o ``${}``, caso contrário dará erro.

| Do | Don't |
|----|------|
| ``<#list myVar></#list>``  | ``<#list ${myVar}></#list>``     |

## Diretivas básicas

**Declarar variáveis**

*assign* (assina uma variável a um valor)
```<#assign variableName = string|number|boolean|sequence|hash> ```

*local* (assina uma variável a um valor, visível apenas em um escopo, como dentro de uma macro)
```<#local variableName = string|number|boolean|sequence|hash> ```

*global* (assina uma variável a um valor, no escopo global)
```<#global variableName = string|number|boolean|sequence|hash> ```

**Loop em uma sequence**

```html
<#assign names = ['João', 'Marcos', 'Bob']>

<#list names as name>
	${name}
</#list>

<!-- João, Marcos, Bob -->
```
Você pode criar uma espécie de *for* também:
```html
<#list 1..10 as item>
	${item}
</#list>

<!-- 1, 2, 3, ..., 10 -->
```
Existem variáveis especiais para uso dentro um loop.

**_index**: index do item atual
**_has_next**: booleano para checar se o item atual é o último da sequence
```html
<#list 1..10 as item>
	${item_has_next}
	${item_index}
</#list>

<!-- 0, 1, ..., 9 -->
<!-- true, true, ..., false -->
```
**Condicionais**

Para condicionais, sempre se usa o básico, que seria ``<#if>``, ``<#elseif>`` e ``<#else>``. A estrutura de uma condicional completa é a seguinte:

```html
<#if condition>
	<!-- do something -->
<#elseif condition>
	<!-- do something else if -->
<#else>
	<!-- do something else -->
</#if>
```
Aonde condition pode ser um booleano, uma comparação, uma expressão aritmética, etc. Você pode omitir o ``<#elseif>`` e o ``<#else>`` quiser, mas nunca a tag de fechamento ``</#if>``. Você pode usar quantos ``<#elseif>`` quiser. As comparações sempre devem usar dois sinais de igual ``==``, nunca três (como em JS, por exemplo, para *type checking*).

**Incluir arquivos externos**

Temos duas diretivas, ``<#include>`` e ``<#import>``. Enquanto o include pega o conteúdo do arquivo e substitui aonde a diretiva se encontra, o import só cria um novo escopo e lê o conteúdo do arquivo em si. Provavelmente você sempre usará ``<#include>``, pelo menos na arquitetura atual do fluxo.

```js
<#include "cms://contentlibrary/components/helpers/parser.htm">
```

## Macros

Macros são diretivas criadas pelo usuário. Pense como se fossem funções, que aceitam parâmetros, e dentro dela se pode fazer o que desejar, desde printar algum código, até incluir arquivos.

Para se criar uma macro, se usa a diretiva ``<#macro></#macro>``. A estrutura é a seguinte:

```html
<#macro macroName param1 param2 param3>
	<h1>${param1} ${param2} ${param3}</h1>
</#macro>
```
E para se usar:

```html
<@macroName param1="Hello" param2="World" param3=23></@macroName>
<!-- Hello World 23 -->
```
Se não for passar nada dentro da chamada da macro, podemos fazer ela com autofechamento. Como é uma diretiva criada pelo usuário, não segue a regra de não se ter ``/`` no final, ela é necessária.
```html
<@macroName param1="Hello" param2="World" param3=23 />
```
É importante entender que tudo o que for "printável" dentro de uma macro, será printado aonde a chamada da macro for feita. Não é obrigatório uma macro ter parâmetros, pode ser uma macro que simplesmente printa algum valor de alguma tabela, por exemplo. É possível também definir valores default para os parâmetros na definição da macro.
```html
<#macro macroName param1 param2="default value"></#macro>
```
Toda macro cria um novo escopo. Logo, definir uma variável dentro de uma macro com ``<#local>`` deixa a variável visível apenas dentro dela. Isso pode ser muito útil em alguns casos de uso com ``<#list></#list>``, por exemplo, chamando a macro a cada iteração.

Existe uma diretiva chamada ``<#nested>`` para ser usada dentro de uma macro. Se você já trabalhou com React, é similar à ``props.children``, renderizando o que vier dentro da utilização da macro.
```html
<#macro macroName param1>
	${param1} <#nested>
</#macro>
```
```html
<@macroName param1="Hello">Gabriel</@macroName>
<!-- Hello Gabriel -->
```
**Functions**

Existe também a diretiva ``<#function>``, que é quase similar à macro, com algumas diferenças. Enquanto a macro printa tudo o que tiver dentro dela (que for printável), a function não printa nada. Ao invés disso, ela exige um valor de retorno, usando a diretiva ``<#return>``. Essa diretiva pode ser usada quantas vezes for necessária dentro de uma function. Um caso de uso, em um dos componentes, foi o seguinte, aonde *localBrand* é uma variável que é recebida pelo componente (mais disso a frente), e retorna a cor de acordo com essa variável.

```html
<#function borderColor>
	<#if localBrand == "investnews">
		<#return "#DC2784">
	<#elseif localBrand == "blog">
		<#return "#9C63C0">
	<#elseif localBrand == "live-room">
		<#return "#AFEAF0">
	<#elseif localBrand == "nmi">
		<#return "#FAE74D">
	</#if>
</#function>
```
A utilização foi da seguinte forma:
```html
<div style="border-left: 8px solid ${borderColor()};">
	...
</div>
```
Como podem ver, podemos utilizar funções e interpolação em vários locais para criar ainda mais dinamicidade.

## Componentes base

Vamos falar dos dois arquivos base resumidamente, que são o ``bundler.html`` e o ``styles.html`` (nota de rodapé, .htm e .html são a mesma coisa, porém quando subimos um .html no Rsys ele omite a letra l.

O ``styles`` é um arquivo que contém todos os estilos CSS de todos os componentes e os estilos base da styleguide. Todos os componentes foram feitos mobile-first, assim evitando erros de layout em clientes como o app do Yahoo, que não lê CSS no ``<head>`` ou até ``<body>`` muitas vezes. Assim, no mobile todos funcionam perfeitamente, e existem estilos específicos para desktop dentro de um media query no ``styles``.

O ``bundler`` é o arquivo em que toda (ou quase toda) a mágica acontece. Ele que é responsável por ler os dados que colamos do Alohomora no Responsys nas Dynamic Variables, interpretar qual componente é qual, fazer os includes corretamente e distribuir os dados de cada componente para tal. Para saber mais, abra o arquivo que está tudo comentado lá. Ele é o arquivo mais complexo de todos, se tratando de RPL.