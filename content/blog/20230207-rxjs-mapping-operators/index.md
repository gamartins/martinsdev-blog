---
title: RxJS mapping operators
date: "2023-02-07T10:34:21.696Z"
description: "Explaining Higher-Order RxJs mapping operators"
---

Nesse artigo vou explicar as diferenças entre os operadores de mapeamento do RxJS. Para garantir que o artigo seja útil tanto para desenvolvedores iniciantes quanto avançados vou dividir o artigo em três partes:

  * Conceitos básicos
  * Operadores de mapeamento do RxJS
  * TL;DR; para quem só quer uma referência rápida

# Conceitos Básicos

## O que é o RxJS?

O RxJS é uma biblioteca para tratamento de eventos assincronos e sequenciais. Ela possui um tipo de dados principal, que é o Observable e funções que funcionam de maneira parecida com as disponíveis no objeto Array do javascript.

## O que são Observables?

O Observable é um tipo de dados que representa uma coleção de valores proveniente de eventos futuros. É importantte entender que podemos ter um, vários ou nenhum valor sendo emitido no decorrer do tempo.

Um clique do usuário na tela por exemplo, pode ser que o usuário clique ou não na tela, e esse evento pode ocorrer a qualquer momento. No exemplo de código a seguir utilizamos o método `fromEvent` disponibilizado na biblioteca para criamos um observable a partir do evento `click`.

```javascript
import { fromEvent } from 'rxjs';

const clickObservable = fromEvent(document, 'click');
clickObservable.subscribe(() => console.log('Clicked!'));
```

Um evento pode ser do tipo next, error ou complete. Os nomes são autodescritivos, mas observe que o evento do tipo `complete` indica que nenhum outro evento será emitido.

## O que são higher order operators ?

De acordo com o livro 'Eloquent Javascript' as _higher order functions_ são funcões que podem receber ou retornar outras funções. Um exemplo bem simples são as várias funções do objeto Array disponível no javascript.

```javascript
const words = ['spray', 'limit', 'elite', 'exuberant', 'destruction', 'present'];

const result = words.filter(word => word.length > 6);

console.log(result);
// Expected output: Array ["exuberant", "destruction", "present"]
```

O método _filter_ recebe como parâmetro uma função que deve retornar verdadeiro ou falso. Esse retorno será utilizado para definir quais elementos serão mantidos na lista.

O conceito se repete na biblioteca RxJS com os High Order Operators, com uma pequena diferença, os High Order Operators esperam receber como retorno um Operador. Um operador, no conceito do RxJS, é uma funcão que recebe um Observable como entrada e retorna outro Observable como saída. No exemplo a seguir temos um observable que retorna uma lista de URLs originadas da leitura de um arquivo. Esses elementos são transformados em requisições http.

```javascript
const fileObservable = urlObservable.pipe(
   map(url => http.get(url)),
);
```

# Operadores de mapeamento do RxJS

## map

Vamos começar com um conceito bem simples que é o operador map presente no RxJS. Seu funcionamento é igual ao map presente no Array, com a diferença que a lista é obtida através de um Observable. Digamos que recebemos uma lista de carros a partir de um observable e queremos alterar esses objetos para uma string. Para isso deveriamos utilizar o operador map do rxjs.

```javascript
const data = of([
  {
    brand: 'porsche',
    model: '911'
  },
  {
    brand: 'porsche',
    model: 'macan'
  },
  {
    brand: 'ferarri',
    model: '458'
  },
]);

data.pipe(
  map(cars => cars.map(car => `${car.brand} ${car.model}`))
).subscribe(cars => console.log(cars))
// Expected output: Array [ 'porsche 911, 'porsche macan', 'ferrari 458' ]
```

É muito importante observar que o valor emitido pelo Observable é uma *lista de objetos*. Isso será diferente nos próximos operadores.

## mergeMap

O operador mergeMap executa a mesma funcão que o operador map convertendo os valores recebidos, mas esses valores vem de uma *lista de Observable*. Se estivessemos usando apenas o operador map seria necessário se inscrever nesses eventos para ter acesso aos seus dados, criando uma concatenação de inscrições.

```javascript
from([1,2,3,4]).pipe(
  map(param => getData(param))
).subscribe(val =>  // Primeira inscrição
  val.subscribe(    // Segunda inscrição
    data => console.log(data)
  )
);
```

É nesse ponto que o mergeMap atua, recebendo o primeiro Observable, criando um segundo Observable e se inscrevendo para os valores emitidos por ele. O primeiro Observable é chamado de Observable externo enquanto o segundo é chamdo Observable interno. Utilizaremos essa denoninação a partir de agora.

```javascript
from([1,2,3,4]).pipe(
  mergeMap(param => getData(param))
).subscribe(val => console.log(val));
```

Em resumo o mergeMap:
  * recebe uma lista de Observable;
  * quando o Observable externo emite um valor ele é convertido em outro Observable;
  * retorna os eventos desse Observable interno;

## switchMap

O operador switchMap executa as mesmas funções do mergeMap MAS cancela a inscrição anterior se um novo evento for emitido. Se o Observable externo emitir um novo evento, antes do Observable interno emitir um valor, o Observable interno será cancelado e re-criado com os dados desse novo evento.

Exemplo: realizar uma busca baseada nas últimas entradas de dados do usuário.

```javascript
// Estamos criando um observable a partir do elemento input
const someInput = document.querySelector('input');
const inputObservable = fromEvent(someInput, 'updateValue');

inputObservable.pipe(
  switchMap((url) => {
    return fromFetch(url);
  })
).subscribe((response) => console.log(response.status));
```

## concatMap

O operador concatMap executa as mesmas funções do mergemMap MAS só cria uma nova inscrição quando a anterior emite um evento. Se o Observable externo emitir um novo evento, antes do Observable interno ser emitido, esse evento ignorado e não teremos um novo Observable interno.

Exemplo: para cada evento de clique, exibir um contador com intervalo de 3 segundos sem concorrência de eventos.

```javascript
import { fromEvent, concatMap, interval, take } from 'rxjs';

const clicks = fromEvent(document, 'click');
const result = clicks.pipe(
  concatMap(ev => interval(1000).pipe(take(4)))
);

result.subscribe(x => console.log(x));
```

## exhaustMap

O operador exhaustMap executa as mesmas funções do mergemMap MAS só cria uma nova inscrição quando a anterior *finaliza* o evento. Se o Observable externo emitir um novo evento, antes do Observable interno ser finalizado, esse evento é ignorado e não teremos um novo Observable interno.

Exemplo: executar um timer finito para cada clique do usuário se não existir outro time ativo

```javascript
import { fromEvent, exhaustMap, interval, take } from 'rxjs';

const clicks = fromEvent(document, 'click');
const result = clicks.pipe(
  exhaustMap(() => interval(1000).pipe(take(5)))
);
result.subscribe(x => console.log(x));
```

# TL;DR;

* mergeMap: mapeia valores para Observables, emite os valores.
* concatMap: mapeia valores para Observables internos, se inscreve e emite em ordem.
* switchMap: mapeia valores para Observables internos, completa o Observables interno anterior, emita valores.
* exhaustMap: mapeie valores Observables internos, ignore outros valores até que o Observable seja concluído.

# Referências

[RxJS Overview](https://rxjs.dev/guide/overview)

[RxJs Operators](https://rxjs.dev/api/operators)

[Higher-order Observables](https://rxjs.dev/guide/higher-order-observables)

[Understanding RxJS map, mergeMap, switchMap and concatMap](https://luukgruijs.medium.com/understanding-rxjs-map-mergemap-switchmap-and-concatmap-833fc1fb09ff)

[Comprehensive Guide to Higher-Order RxJs Mapping Operators: switchMap, mergeMap, concatMap (and exhaustMap)](https://blog.angular-university.io/rxjs-higher-order-mapping/)


