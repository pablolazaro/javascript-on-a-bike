---
title: Angular 2 - Controlando componentes hijos con ViewChild y ViewChildren
tags:
  - Angular 2
  - Recetas
---

Una de las cosas que más me ha sorprendido de Angular 2 es la cantidad de posibilidades que nos 
ofrece el framework para que los componente [se comuniquen entre si](https://angular.io/docs/ts/latest/cookbook/component-communication.html).

Hoy voy a hablar de cómo hacer que se comunique un componente padre con sus 
componentes hijos utilizando los [decoradores](https://www.typescriptlang.org/docs/handbook/decorators.html) `ViewChild` y `ViewChildren`.

# ViewChild



{% jsfiddle 45Fqf %}

