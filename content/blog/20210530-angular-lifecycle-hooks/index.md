---
title: Another lifecycle hooks article...
date: "2022-05-30T22:12:03.284Z"
description: "A TL;DR; article about lifecycle hooks"
---

Olá desenvolvedor, provavelmente você, ou meu eu do futuro está recorrendo a esse artigo para entender qual hook utilizar para uma funcionalidade específica. O conceito de lifecycle hook não é complexo, mas escolher o melhor hook, que vai garantir o funcionamento do sistema e ao mesmo tempo não sobrecarrega-lo com chamadas excessivas nem sempre é uma escolha simples.

Para facilitar a nossa vida eu vou colocar aqui um resumo de cada hook e quando eles devem ser utilizados. Nos próximos paragrafos vou ser mais detalhista e colocar alguns exemplos de código.
  
  * ngOnChanges: chamado uma vez na criação do componente e todas as vezes que uma mudança nos @Inputs são detectados. É muito útil se você precisar lidar com qualquer lógica específica no componente com base na propriedade de entrada recebida.

  * ngOnInit: chamado apenas uma vez durante o ciclo de vida do componente, após a primeira chamada ngOnChanges, mas é chamado mesmo quando o ngOnChanges não é. É onde você pode executar qualquer inicialização logo após a construção do componente ( tente não usar o construtor para isso ).

  * ngDoCheck: chamado em cada detecção de mudança, imediatamente após ngOnChanges e ngOnInit. Você pode usar esse método para detectar alterações que o Angular não pode ou não irá detectar. Este hook possui um custo de performance alto, uma vez que é chamado frequentemente.

  * ngAfterContentInit: chamado apenas uma vez durante o ciclo de vida do componente, após o primeiro ngDoCheck. Ele é chamado após o conteúdo projetado, através do ng-content, ser totalmente inicializado e inserido no componente. É nesse hook que são atualizados as propriedades ContentChild e ContentChildren.

  * ngAfterContentChecked: chamado após o ngAfterContentInit, a diferença entre ele e o ngAfterContentInit é que ele continua a ser chamado depois de cada ciclo de deteção de mudanças após o ngDoCheck.

  * ngAfterViewInit: chamado apenas uma vez durante o ciclo de vida do componente, após ngAfterContentChecked. Esse hooks é chamado depois que a detecção de mudanças completa a inicialização do template do componente e dos seus filhos. É nesse hook que são atualizados as propriedades ViewChild e ViewChildren.

  * ngAfterViewChecked: chamado uma vez após ngAfterViewInit e depois de cada ngAfterContentChecked subsequente. A diferença entre ele e o ngAfterViewInit é que ele continua a ser chamado depois de cada ciclo de deteção de mudanças após o ngDoCheck.

  * ngOnDestroy: por último, este método é chamado apenas uma vez durante o ciclo de vida do componente, logo antes do Angular destruí-lo. Aqui é onde você deve informar o resto de sua aplicação que o componente está sendo destruído e deve colocar toda a sua lógica de limpeza para esse componente.

  Perguntas:
  
    - Qual a diferença entre o ngAfterContentInit e o ngAfterContentChecked
    - Qual a diferença entre o ngAfterViewInit e o ngAfterViewChecked

### References

* [Complete Guide: Angular lifecycle hooks](https://indepth.dev/posts/1494/complete-guide-angular-lifecycle-hooks)
* [AfterViewInit, AfterViewChecked, AfterContentInit & AfterContentChecked in Angular](https://www.tektutorialshub.com/angular/afterviewinit-afterviewchecked-aftercontentinit-aftercontentchecked-in-angular/)
