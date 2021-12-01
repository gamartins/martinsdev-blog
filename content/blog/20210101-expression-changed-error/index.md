---
title: The `ExpressionChanged...` error
date: "2021-12-01T22:12:03.284Z"
description: "A description of what causes and how to handle the `ExpressionChangedAfterItHasBeenCheckedError` error"
---

Esse artigo é uma explicação do que causa, como debuggar e corrigir o erro ExpressionChangedAfterItHasBeenCheckedError

## O que causa esse erro

De uma maneira bem direta, o Angular lança esse erro sempre que

> O valor de uma expressão no componente é alterado depois que a detecção de mudanças foi iniciada mas enquanto o seu ciclo de vida ainda não foi concluido

É importante observar que esse erro será emitido apenas em ambiente de desenvolvimento porque isso é uma verificação para garantir que o componente vai se comportar de maneira esperada.

## Exemplo com código

No exemplo a seguir temos o componente pai `ExampleComponentA` e o componente filho `ExampleComponentB` . Lembre-se que a detecção de mudanças é realizada no sentido top-to-bottom, ou seja, primeiro os componentes pais e depois os seus filhos, mas ela só é considerada finalizada no pai quando o ciclo dos seus filhos tiver sido concluída.

A detecção de mudanças é realizada no `ExampleComponentA` e segue para o `ExampleComponentB` que modifica os valores de seu pai. Com isso, o valor da variável `title` foi modificado antes do seu ciclo ter sido terminado e somos apresentados ao erro `ExpressionChangedAfterItHasBeenCheckedError`

<iframe
  width="640"
  height="640"
  src="https://stackblitz.com/edit/expression-changed-after-checked-01?ctl=1&devtoolsheight=33&embed=1&file=src/app/app.component.ts&hideExplorer=1&view=preview"
></iframe>


O exemplo apresenta o erro da maneira mais simples possível. Existem inúmeras formas de causar esse problema mas é importante entender o seu conceito básico. Novamente:

> O valor de uma expressão no componente é alterado depois que a detecção de mudanças foi iniciada mas enquanto o seu ciclo de vida ainda não foi concluido

## Outros exemplos

O exemplo apresentado anteriormente é uma versão simples e didática do que pode causar esse erro. Vamos verificar alguns exemplos mais próximos de ambientes reais.

### Synchronous event broadcasting

Nesse exemplo temos um evento @Output ao invés de um serviço para atualizar o componente pai. Isso é uma má prática justamente por causar problemas no ciclo de detecção de mudanças.

<iframe
  width="640"
  height="640"
  src="https://stackblitz.com/edit/expression-changed-after-checked-03?ctl=1&devtoolsheight=33&embed=1&file=src/app/app.component.ts&hideExplorer=1&view=preview"
></iframe>

### Shared service

Nesse exemplo nós temos um serviço compartilhado chamado `SharedService` que envia uma atualização sempre que o valor de text é atualizado. Esse valor é atualizado em `ExampleBComponent` e em seguida `ExampleAComponent` lançando o nosso erro.

<iframe
  width="640"
  height="640"
  src="https://stackblitz.com/edit/expression-changed-after-checked-02?ctl=1&devtoolsheight=33&embed=1&file=src/app/app.component.ts&hideExplorer=1&view=preview"
></iframe>

### Dynamic component instantiation

Esse é um exemplo mais complexo pois estamos utilizando o ``ComponentFactoryResolver`` para instanciar dinamicamente um componente. Nesse caso o erro é emitido porque utilizamos o @ViewChild para referenciar o elemento onde iremos criar o componente. Como esse elemento não existia antes do primeiro ciclo de detecção o erro é emitido.

<iframe
  width="640"
  height="640"
  src="https://stackblitz.com/edit/expression-changed-after-checked-04?ctl=1&devtoolsheight=33&embed=1&file=src/app/app.component.ts&hideExplorer=1&view=preview"
></iframe>

## Como corrigir o problema

Como outros problemas de arquitetura em programação não existe uma "bala de prata" que sempre possamos utilizar quando esse erro é emitido. Contudo, aqui estão algumas sugestões de como resolver o problema:

### Verifique sua arquitetura
A estrutura de elementos do angular foi pensanda no formato top-to-bottom. Se o erro está ocorrendo porque um componente filho está atualizando o pai talvez seja melhor você reestruturar a organização desses componentes. No exemplo com [Synchronous event broadcasting](https://stackblitz.com/edit/expression-changed-after-checked-03) não deveriamos estar utilizando um evento no filho para atualizar o pai.

### setTimeout
O javascript funciona de maneira sincrona, mas essa função permite que executemos código no próximo 'ciclo de execução do Javascript' assim evitamos que uma modificação seja realizada antes do final do ciclo de vida do componente. No exemplo com o [SharedService](https://stackblitz.com/edit/expression-changed-after-checked-02) poderiamos escrever o seguinte código.

```jsx
export class ExampleAComponent {
  name = 'I am A component';
  text = 'A message for the child component';

  constructor(sharedService: SharedService) {
    sharedService.onNameChange((value) => {
      setTimeout(() => this.text = value)
    });
  }
}
```

Podemos utilizar o mesmo conceito para resolver o problema quando instaciamos o componente dinamicamente utilizando o [ComponentFactoryResolver](https://stackblitz.com/edit/expression-changed-after-checked-04)

```jsx
export class ExampleAComponent {
  @ViewChild('childElement', { read: ViewContainerRef }) el;

  name = 'I am A component';

  constructor(private compFactory: ComponentFactoryResolver) {}

  ngAfterViewInit() {
    setTimeout(() => {
      const factory = this.compFactory.resolveComponentFactory(ExampleBComponent);
      this.el.createComponent(factory);
    })
  }
}
```

### Forçar a detecção de mudanças
Podemos utilizar o serviço ``ChangeDetectorRef`` para forçar um novo ciclo de detecção de mudanças no componente. O melhor local para fazer isso é no hook ``ngAfterViewInit`` porque a detecção de mudança em todos os filhos que podem atualizar os pais já foi realizada ou no momento em que realizamos a alteração que gera o erro.

```javascript
export class ExampleAComponent {
  name = 'I am A component';
  text = 'A message for the child component';

  constructor(
    private sharedService: SharedService,
    private cd: ChangeDetectorRef) {
      sharedService.onNameChange((value) => {
        this.text = value;
        this.cd.detectChanges();
      });
  }
}
```

## Conclusão

O famigerado erro `ExpressionChangedAfterItHasBeenCheckedError` pode parecer que foi criado apenas para não permitir que você faça a entrega da sua tarefa e possa ir para casa. Por que criar um erro que só é lançado em ambiente de desenvolvimento? Só que na verdade, ele é uma forma da equipe de desenvolvedores do angular dizer: homi, cuidado, isso vai dar ruim.

Problemas relacionados a detecção de mudanças podem não causar nada ou podem fazer com que componente não exiba seus dados corretamente (depende muito do dia da semana e do prazo disponível para entrega). E com certeza vai criar um novo ticket muito mais díficil de debuggar no futuro, então refatore seu código e vá para casa mais tranquilo.

## Referências

[Everything you need to know about the ExpressionChangedAfterItHasBeenCheckedError error](https://indepth.dev/posts/1001/everything-you-need-to-know-about-the-expressionchangedafterithasbeencheckederror-error)

[Angular Debugging "Expression has changed after it was checked": Simple Explanation](https://blog.angular-university.io/angular-debugging/)

[NG0100: Expression has changed after it was checked](https://angular.io/errors/NG0100)