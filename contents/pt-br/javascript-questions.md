---
title: Perguntas sobre JavaScript
---

Respostas para [perguntas de entrevistas Front-end - Perguntas sobre JS](https://github.com/h5bp/Front-end-Developer-Interview-Questions/blob/master/src/questions/javascript-questions.md). _Pull requests_ para sugestões e correções são bem-vindos!

## Tabela de conteúdos

- [Explique delegação de evento](#explain-event-delegation)

### Explique delegação de evento

Delegação de evento é uma técnica que envolve adicionar _listeners_ de eventos a um elemento pai ao invés de adicioná-los a elementos filhos. O _listener_ irá "escutar" sempre que o evento é disparado nos elementos filhos devido ao _bubbling_ de eventos do DOM. Os benefícios desta técnica são:

- A _Memory Footprint_ diminui porque somente um manipulador (_handler_) é necessário no elemento pai, ao invés de ter que anexar um manipulador em cada filho.
- Não há necessidade de fazer o _unbind_ do manipulador de elementos que são removidos e o _bind_ de novos elementos.

###### Referências (Inglês)

- https://davidwalsh.name/event-delegate
- https://stackoverflow.com/questions/1687296/what-is-dom-event-delegatio

###### Referências (pt-BR)

https://developer.mozilla.org/pt-BR/docs/Aprender/JavaScript/Elementos_construtivos/Events#Delega%C3%A7%C3%A3o_de_eventos

[[↑] Voltar ao topo](#table-of-contents)

### Explique como `this` funciona no JavaScript

Não há uma simples explicação para o `this`; Esse é um dos conceitos mais confusos do JavaScript. Uma explicação simplificada é que o valor do `this` depende de como a função é invocada. Li diversas explicações sobre `this` online, e achei a explicação do [Arnav Aggrawal](https://medium.com/@arnav_aggarwal) a mais clara. As seguintes regras se aplicam:

1. Se a palavra-chave `new` é usada ao invocar a função, o `this` dentro da função será um novo objeto.
2. Se `apply`, `call` ou `bind` são usados para invocar/criar uma função, o `this` dentro da função é um objeto passado como argumento.
3. Se uma função é invocada como um método, como em `obj.method()`–O `this` é o objeto ao qual a função pertence.
4. Se uma função é invocada de maneira livre, ou seja, ela foi invocada sem nenhuma das condições apresentadas anteriormente, o `this` será um objeto global. Em um navegador, seria o objeto `window`. Se usado o _strict mode_ (`'use strict'`), O `this` seria `undefined` ao invés do objeto global.
5. Se múltiplas regras anteriores se aplicam, a regra mais alta ganha e atribuí o valor do `this`.
6. Se a função é uma `arrow function` _(ES2015)_, todas as regras anteriores são ignoradas e o valor do `this` é referente ao escopo ao seu redor no momento que foi criado.

Para uma explicação mais profunda, leia seu [artigo (inglês) no Medium](https://codeburst.io/the-simple-rules-to-this-in-javascript-35d97f31bde3)

#### Você poderia dar um exemplo de uma das maneiras de como trabalhar com this mudou no ES6?

ES6 te permite usar [arrow functions](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Functions/Arrow_functions) que usam [escopo léxico](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Functions/Arrow_functions#Sem_this_separado). Isto é muitas vezes conveniente, mas não previne o invocador de controlar o contexto via `call` ou `.apply`–as consequências sendo que uma biblioteca como o `jQuery` não irá fazer o _bind_ ao `this` de maneira apropriada nas funções do seu manipulador de evento. Além do mais, é importante ter isso em mente ao refatorar grandes aplicações legadas.

[[↑] Voltar ao topo](#table-of-contents)

### Explique como funciona a herança por protótipos

Esta é uma pergunta de JavaScript extremamente comum em entrevistas. Todos os objetos em JavaScript possuem uma propriedade `__proto__`, que é uma referência a outro objeto, que é chamado de protótipo do objeto. Quando uma propriedade é acessada em um objeto e essa propriedade não é encontrada, a _engine_ do JavaScript procura pela propriedade no `__proto__` do objeto, depois no `__proto__` do `__proto__` e assim por diante, até que a propriedade seja encontrada em um dos `__proto__`s ou até que chegue ao final da cadeia de protótipos. Este comportamento simula herança clássica, mas na verdade é muito mais delegação por herança.

#### Exemplo de herança prototipada

Nós já temos um método `Object.create` nativo, mas se quiséssemos fornecer um _polyfill_ para ele, seria da seguinte maneira:

```js
if (typeof Object.create !== 'function') {
  Object.create = function (pai) {
    function Tmp() {}
    Tmp.prototype = pai;
    return new Tmp();
  };
}

const Pai = function () {
  this.name = 'Pai';
};

Pai.prototype.cumprimentar = function () {
  console.log('"Olá" do pai');
};

const filho = Object.create(Pai.prototype);

filho.chorar = function () {
  console.log('waaaaaahhhh!');
};

filho.chorar();
// Imprime: waaaaaahhhh!

filho.cumprimentar();
// Imprime: "Olá" do pai
```

Coisas para se ter em mente aqui:

- `.cumprimentar` não está definido no _filho_, portanto a _engine_ navega na cadeia de protótipos e encontra `.cumprimentar` herdado do _Pai_.
- Precisamos invocar `Object.create` em uma das seguintes maneiras para que os métodos do protótipo sejam herdados:
  - Object.create(Pai.prototype);
  - Object.create(new Pai(null));
  - Object.create(objLiteral);
  - Atualmente, `filho.constructor` aponta para o `Pai`:

```js
filho.constructor

f() {
  this.name = 'Pai';
}

filho.constructor.name; // Pai
```

- Se quisermos corrigir isso, uma opção seria fazer:

```js
function Filho() {
  Pai.call(this);
  this.name = 'filho';
}

Filho.prototype = Pai.prototype;
Filho.prototype.constructor = Filho;

const c = new Filho();

c.chorar();
// Imprime: waaaaahhhh!

c.cumprimentar();
// Imprime: "Olá" do Pai

c.constructor.name;
// Imprime: "Filho"
```

###### Referências (Inglês)

- http://dmitrysoshnikov.com/ecmascript/javascript-the-core/
- https://www.quora.com/What-is-prototypal-inheritance/answer/Kyle-Simpson
- https://davidwalsh.name/javascript-objects
- https://crockford.com/javascript/prototypal.html
- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain

[[↑] Voltar ao topo](#table-of-contents)

### O que você acha de _AMD_ vs _CommonJS_?

Ambos são um modo de implementar um sistema modular, algo que não estava presente nativamente no JavaScript até que o ES2015 surgisse. _CommonJS_ é síncrono enquanto AMD (Definição de Módulo Assíncrona) é, obviamente, assíncrono. O design do CommonJS foi feito com desenvolvimento _server-side_ em mente, enquanto AMD, com seu suporte para carregamento de módulos assíncrono, é mais voltado para navegadores.

Eu acho a síntaxe do AMD muito verbosa e o CommonJS está mais próximo ao estilo que você escreveria declarações de _import_ em outras linguagens. Na maior parte do tempo, acho AMD desnecessário, porque se você servisse todo seu JavaScript empacotado em um único arquivo, você não se beneficiaria das propriedades de carregamento assíncrono. Além disso, a síntaxe do CommonJS é mais próxima do estilo Node de escrever módulos e há menos sobrecarga de mudança de contexto ao alternar entre client side e server side durante o desenvolvimento.

Estou feliz que com os módulos no ES2015, há suporte para carregamentos tanto síncrono quanto assíncrono, podemos finalmente ficar com uma única abordagem. Embora não tenha sido lançado totalmente em navegadores e no Node, sempre podemos usar transpiladores para converter nosso código.

###### Referências (Inglês)

- https://auth0.com/blog/javascript-module-systems-showdown/
- https://stackoverflow.com/questions/16521471/relation-between-commonjs-amd-and-requirejs

[[↑] Voltar ao topo](#table-of-contents)

### Explique por que o seguinte código não funciona como IIFE: `function foo() {}();`. O que precisa mudar para funcionar corretamente como uma IIFE?

IIFE significa Expressão de Função chamada imediatamente (Immediately Invoked Function Expression). O _parser_ do lê `function foo() {}();` como `function foo() { }` e `();`, onde a primeira parte é uma _declaração de função_ e a segunda (um par de parênteses) é uma tentativa de invocar uma função mas há nome especificado, consequentemente um erro é lançado: `Uncaught SyntaxError: Unexpected token )`.

Há duas maneiras de consertar que envolvem adicionar mais parênteses: `(function foo() {})()` e `(function foo() {}())`. Declarações que começam com `function` são consideradas _declarações de função_; Ao envolver essa função dentro de `()`, ela se torna uma _expressão de função_ que pode então ser executada com o `()` subsequente. Estas função não são expostas ao escopo global e você pode omitir seu nome se você não necessita fazer referência a ela dentro do corpo da função.

Você também pode utilizar o operador `void`: `void function foo(){ }();`. Infelizmente, há um problema com tal abordagem. A validação da expressão dada é sempre `undefined`, então se sua função IIFE retorna algo, você não poderá usar esse algo. Um exemplo:

```js
const foo = void (function bar() {
  return 'foo';
})();

console.log(foo);
```

###### Referências (Inglês)

- http://lucybain.com/blog/2014/immediately-invoked-function-expression/
- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/voi

[[↑] Voltar ao topo](#table-of-contents)

### Qual é a diferença entre uma variable que é: `null`, `undefined` ou não declarada? Como você faria para conferir cada um desses estados?

Variáveis **não declaradas** são criadas quando você atribui um valor a um identificador que não foi previamente criado usando `var`, `let` ou `const` Essas variáveis serão definidas globalmente, fora do escopo atual. Em _strict mode_, um _`ReferenceError`_ será lançado ao tentar atribuir um valor a uma variável não declarada. Variáveis não declaradas podem ser nocivas assim como variáveis globais. Evite-as te todas as maneiras! Para verificá-las, envolva seu uso com `try`/`catch`.

```js
function foo() {
  x = 1; // Lança um ReferenceError em Strict Mode
}

foo();
console.log(x); // 1
```

Uma variável que é `undefined` é uma variável que foi declarada, mas não teve um valor atribuído. Ela é do tipo `undefined`. Se uma função não retorna nenhum valor como resultado, e a chamada dessa função é atribuída a uma variável, a variável também será `undefined`. Para verificá-la, compare usando o operador de igualdade estrita (`===`) ou `typeof` que retornará a string `'undefined'`. Tenha em mente que você não deve usar o operador de igualdade abstrata para verificação, já que ele também retornará `true` se o valor for `null`.

```js
var foo;
console.log(foo); // undefined
console.log(foo === undefined); // true
console.log(typeof foo === 'undefined'); // true

console.log(foo == null); // true. Errado, não use esta verificação!

function bar() {}
var bar = bar();
console.log(baz); // undefined
```

Uma variável que é `null` será explicitamente atribuída com o valor `null`. Ela não representa valor algum e é diferente de `undefined` no sentido de ter recebido o valor `null` explicitamente. Para verificar um valor `null`, simplesmente compare usando o operador de igualdade estrito. Tenha em mente que, assim como o exemplo anterior, você não deve utilizar o operador de igualdade abstrata (`==`) para verificar, já que ele também retornará `true` se o valor for `undefined`.

```js
var foo = null;
console.log(foo === null); // true
console.log(typeof foo === 'object');

console.log(foo == undefined); // true. Errado, não use esta verificação!
```

Como um hábito pessoal eu nunca deixo minhas variáveis não declaradas ou destribuídas. Eu também explicitamente atribuo `null` logo após declará-las se eu não pretendo utilizá-las de início. Se você utiliza um _linter_ no seu workflow, ele usualmente verificará se você não está fazendo referências a variáveis não declaradas.

###### Referências (Inglês)

- https://stackoverflow.com/questions/15985875/effect-of-declared-and-undeclared-variables
- https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/undefined

[[↑] Voltar ao topo](#table-of-contents)

### O quê é uma closure, e como/por que você usaria uma?

Uma _closure_ é a combinação de uma função e um ambiente léxico dentro do qual aquela função foi declarada. A palavra "léxico" faz referência ao fato do escopo léxico usar a localização onde a variável é declarada dentro do código-fonte para determinar até onde aquela variável está disponível. Closures são funções que tem acesso ao escopo de variáveis de sua função externa mesmo depois da função externa ter retornado.

**Por que você usaria?**

- Privacidade de dados / Simular métodos privados com closures. Comumente usado no [pattern module (referência em inglês)](https://addyosmani.com/resources/essentialjsdesignpatterns/book/#modulepatternjavascript)
- [Aplicação parcial ou currying (referência em inglês)](https://medium.com/javascript-scene/curry-or-partial-application-8150044c78b8#.l4b6l1i3x)

###### Referências (Inglês)

- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures
- https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-closure-b2f0d2152b36

[[↑] Voltar ao topo](#table-of-contents)

### Você pode descrever a principal diferença entre um loop `.forEach` e um loop `.map()` e por que você escolheria um versus o outro?

Para entender a diferença entre os dois, vamos ver o que cada função faz.

**`forEach`**

- Itera entre os elementos do array.
- Executa um _callback_ para cada elemento.
- Não retorna um valor.

```js
const a = [1, 2, 3];
const doubled = a.forEach(num, index) => {
  // Faz algo com num e/ou index.
};

// doubled = undefined;
```

**`map`**

- Itera entre os elementos do array.
- "Mapeia" cada elemento a um novo elemento através da invocação da função para cada elemento, criando um novo array como resultado.

```js
const a = [1, 2, 3];
const doubled = a.map((num) => {
  return num * 2;
});

// doubled = [2, 4, 6]
```

A principal diferença entre `.forEach` e `.map()` é que `.map()` retorna um novo array. Se você precisa do resultado, mas não deseja transformar o array original, utilize `.map`. Se você deseja simplesmente iterar o array, `forEach` é a escolha certa.

###### Referências (Inglês)

- https://codeburst.io/javascript-map-vs-foreach-f38111822c0f

[[↑] Voltar ao topo](#table-of-contents)

### Qual é um uso de caso típico para funções anônimas?

Elas podem ser usadas em IIFEs para encapsular código dentro de um escopo local, para que então variáveis declaradas não "vazem" para o escopo global

```js
(function () {
  // Algum código aqui.
})();
```

Como um _callback_ que é usado uma vez e não precisa ser usado em mais nenhum lugar. O código parecerá mais independente e legível quando manipuladores são definidos dentro do código que os está invocando, ao invés de ter que procurar em outro lugar para encontrar o corpo da função.

```js
setTimeout(function () {
  console.log('Hello world!');
}, 1000);
```

Argumentos para _constructs_ de programação funcional ou Lodash (similar a _callbacks_).

```js
const arr = [1, 2, 3];
const double = arr.map(function (el) {
  return el * 2;
});

console.log(double); // [2, 4, 6]
```

###### Referências (Inglês)

- https://www.quora.com/What-is-a-typical-usecase-for-anonymous-functions
- https://stackoverflow.com/questions/10273185/what-are-the-benefits-to-using-anonymous-functions-instead-of-named-functions-fo

[[↑] Voltar ao topo](#table-of-contents)

### Como você organiza seu código? (module pattern, herança clássica?)

No passado, usei Backbone para meus _models_ o que me encorajava seguir uma abordagem mais Orientada a Objeto, criando _models_ em Backbone e anexando métodos a eles.

O pattern _module_ ainda é ótimo, mas nos dias de hoje, eu uso React/Redux que utilizam um fluxo de dados unidirecional baseado na arquitetura Flux. Eu representaria os modelos do app usando objetos simples e escreveria funções puras para manipular estes objetos. O _State_ é manipulado usando ações e _reducers_ como em qualquer outra aplicação Redux.

Eu evito usar herança clássica onde possível. Quando e se eu preciso, eu sigo [essas regras](https://medium.com/@dan_abramov/how-to-use-classes-and-sleep-at-night-9af8de78ccb4)

[[↑] Voltar ao topo](#table-of-contents)

### Qual a diferença entre objetos hospedeiros e objetos nativos?

Objetos nativos são objetos que são parte do JavaScript definidos pela especificação ECMAScript, tais como `String`, `Math`, `RegExp`, `Object`, `Function`, etc.

Objetos hospedeiros são fornecidos pelo ambiente de execução (navegador ou Node), tais como `window`, `XMLHTTPRequest`, etc.

###### Referências (Inglês)

- https://stackoverflow.com/questions/7614317/what-is-the-difference-between-native-objects-and-host-objects

[[↑] Voltar ao topo](#table-of-contents)

### Diferença entre: `function Pessoa(){}`, `var pessoa = Pessoa()`, e `var pessoa = new Pessoa()`?

Esta pergunta é bem vaga. Meu melhor palpite é que sua intenção é saber a respeito de construtores em JavaScript. Falando tecnicamente, `function Pessoa(){}` é apenas uma declaração de função normal. A convenção é usar PascalCase para funções que são destinadas a serem usadas com construtores.

`var person = Pessoa()` invoca `Pessoa` como uma função, e não como um construtor. Invocar dessa maneira é um erro comum se a pretensão for usar a função como construtor. Tipicamente, o construtor não retorna nada, consequentemente invocar o construtor como uma função normal irá retornar `undefined`, não a instância, que será atribuído à variável.

`var person = new Pessoa()` cria uma instância do objeto `Pessoa` usando o operador `new`, o qual herda de `Pessoa.prototype`. Uma alternativa seria usar `Object.create`, such as: `Object.create (Person.prototype)`.

```js
function Pessoa(nome) {
  this.nome = nome;
}

var pessoa = Pessoa('John');
console.log(pessoa); // undefined
console.log(pessoa.nome); // Uncaught TypeError: Cannot read property 'nome' of undefined

var pessoa = new Pessoa('John');
console.log(pessoa); // Pessoa { nome: "John" }
console.log(pessoa.nome); // "John"
```

###### Referências (Inglês)

- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new

[[↑] Voltar ao topo](#table-of-contents)

### Qual a diferência entre `.call` e `.apply`?

Ambos `.call` e `.apply` são usados para invocar funções cujo primeiro parâmetro irá ser usado como valor do `this` dentro da função. Entretanto, `.call` aceita argumentos separados por vírgula a partir do segundo argumento enquanto `.apply` aceita um array de argumentos como segundo parâmetro. Um modo fácil de lembrar é: C de `call` e comma-separated (separados por vírgula) e A de `apply` e array de argumentos.

```js
function add(a, b) {
  return a + b;
}

console.log(add.call(null, 1, 2));
console.log(add.apply(null, [1, 2]));
```

[[↑] Voltar ao topo](#table-of-contents)

### Explique `Function.prototype.bind`

> O método bind() cria uma nova função que, quando chamada, tem sua palavra-chave this definida com o valor fornecido, com uma sequência determinada de argumentos precedendo quaisquer outros que sejam fornecidos quando a nova função é chamada.

Em minha experiência, é o mais útil para _bindar_ o valor do `this` em métodos de classes aos quais você quer passar para outras funções. Isso é frequentemente feito em componentes React.

###### Referências (Inglês)

- https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_objects/Function/bin

[[↑] Voltar ao topo](#table-of-contents)

### Quando você usaria `document.write()`?

O `document.write()` escreve uma string à uma stream de documento aberta pelo `document.open()`. Quando `document.write()` é executado depois da página ter carregado, ele irá chamar o `document.open` que limpará todo o documento (`<head>` e `<body>` são removidos!) e substitui o conteúdo com o valor do parâmetro dado. Portanto, ele é geralmente considerado perigoso e propenso a mau uso.

Há algumas respostas online que explicam que o `document.write()` está sendo usado em códigos de analytics ou [quando você quer incluir estilos que somente devem funcionar se o JavaScript estiver habilitado](https://www.quirksmode.org/blog/archives/2005/06/three_javascrip_1.html). Ele até é usado em boilerplates de HTML5 para [carregar scripts em paralelo e preservar a ordem de execução](https://github.com/paulirish/html5-boilerplate/wiki/Script-Loading-Techniques#documentwrite-script-tag)! Entretanto, eu suspeito que esses motivos podem estar desatualizados e hoje em dia, podem ser alcançadas sem usar `document.write()`. Por favor me corrija se eu estiver errado a respeito disso.

###### Referências (Inglês)

- https://www.quirksmode.org/blog/archives/2005/06/three_javascrip_1.html
- https://github.com/h5bp/html5-boilerplate/wiki/Script-Loading-Techniques#documentwrite-script-tag

[[↑] Voltar ao topo](#table-of-contents)

### Qual é a diferença entre _feature detection_, _feature inference_, e _UA string_?

**Feature Detection**

_Feature detection_ envolve avaliar se um navegador suporta um determinado trecho de código, e executar um código diferente em caso dele não ser suportado, com isso o navegador poderia sempre oferecer uma experiência funcional ao invés de erros. Por exemplo:

```js
if ('geolocation' in navigator) {
  // Pode usar navigator.geolocation
} else {
  // Lidar com a falta da feature
}
```

[Modernizr](https://modernizr.com/) é uma ótima biblioteca para lidar com detecção de feature.

**Feature Inference**

_Feature inference_ avalia uma _feature_ assim como a _feature detection_, mas usa outra função porque é assumido que ela também existirá, por exemplo:

```js
if (document.getElementsByTagName) {
  element = document.getElementById(id);
}
```

Isso não é recomendado. _Feature detection_ é mais a prova de falhas.

**UA String**

É uma string de relatório do navegador que permite peers de protocolo de rede identificar o tipo da aplicação, sistema operacional, vendor de software ou versão de software do user agent fazendo a requisição. Pode ser acessado via `navigator.userAgent`. Entretanto, a string é complicada de _parsear_ e pode ser burlada. Por exemplo, o Chrome relata ambos Chrome e Safari. Então, para detectar o Safari você tem que procurar pela string do Safari e a ausência da string Chorme. Evite este método.

###### Referências (Inglês)

- https://developer.mozilla.org/en-US/docs/Learn/Tools_and_testing/Cross_browser_testing/Feature_detection
- https://stackoverflow.com/questions/20104930/whats-the-difference-between-feature-detection-feature-inference-and-using-th
- https://developer.mozilla.org/en-US/docs/Web/HTTP/Browser_detection_using_the_user_agent

[[↑] Voltar ao topo](#table-of-contents)

### Explique Ajax com o máximo de detalhes possível

Ajax (JavaScript e XML assíncrono) é um conjunto de técnicas de desenvolvimento web que usam diversas tecnologias web do lado do cliente para criar aplicações web assíncronas. Com Ajax, aplicações web podem enviar e receber dados de um servidor de maneira assíncrona sem interferir o layout ou comportamento da página existente. Ao desacoplar a camada de troca de dados da camada de apresentação, o Ajax permite que páginas web, e por extensão aplicações web, troquem conteúdo dinamicamente sem a necessidade de recarregar a página inteira. Na prática, implementações modernas comumente usam JSON ao invés de XML, devido às vantagens do JSON ser nativo do JavaScript.

A API `XMLHttpRequest` é frequentemente usada para comunicação assíncrona, assim como a API `fetch`.

###### Referências (Inglês)

- https://en.wikipedia.org/wiki/Ajax_(programming)
- https://developer.mozilla.org/en-US/docs/AJAX

[[↑] Voltar ao topo](#table-of-contents)

### Quais são as vantagens e desvantagens de usar o Ajax?

**Vantagens**

- Melhor interatividade. Novos conteúdos do servidor podem ser alterados dinamicamente sem a necessidade de recarregar a página inteira.
- Reduz conexões ao servidor já que scripts e folhas de estilo só necessitam ser solicitadas uma vez.
- O _State_ pode ser mantido em uma página. As variáveis em JavaScript e o _state_ do DOM são persistidos porque o container principal não é recarregado.
- Basicamente a maioria das vantagens de uma SPA.

**Desvantagens**

- Webpages dinâmicas são mais difíceis de favoritar.
- Não funciona se o JavaScript estiver desabilitado no navegador.
- Alguns _webcrawlers_ não executam JavaScript e não "veem" conteúdo que é carregador por JavaScript.
- Páginas web que usam Ajax para buscar dados provavelmente terão que combinar os dados remotos com templates do lado do cliente para atualizar o DOM. Para isso acontecer, o JavaScript terá que ser parseado e executado no navegador, e aparelhos mais antigos podem ter dificuldade com essas tarefas.
- Basicamente a maioria das desvantagens de SPAs.

[[↑] Voltar ao topo](#table-of-contents)
