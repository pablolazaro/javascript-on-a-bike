---
title: Angular 2 - Controlando componentes hijos con ViewChild y ViewChildren
tags:
  - Angular 2
  - Recetas
date: 2016-10-13 21:18:52
description: Aprende a controlar directivas y elementos que estén en el template de tus componentes usando @ViewChild y @ViewChildren.
---


> Este artículo está actualizado a la version ^4.0.0 de Angular 2

Una de las cosas que más me ha sorprendido de Angular 2 es la cantidad de 
posibilidades que nos ofrece el framework para que los componentes
[se comuniquen entre si](https://angular.io/docs/ts/latest/cookbook/component-communication.html).

Hoy voy a hablar de cómo controlar los componentes o directivas que tengamos en el *template* de nuestro componente 
utilizando los [decoradores](https://www.typescriptlang.org/docs/handbook/decorators.html)
 `ViewChild` y `ViewChildren`.

Durante el artículo vamos a utilizar como ejemplo un componente al que llamaremos `SystemMessage`
que mostrará un mensaje al usuario cuando la aplicación lo requiera.

```javascript
@Component({
  selector: 'system-message',
  template: `
    <div *ngIf="active" class="system-message">
      <p>{{message}}</p>
    </div>
  `
})
export class SystemMessageComponent {
  @Input() message: string;
  @Input() active: boolean;

  show () {
    this.active = true;
  }

  hide () {
    this.active = false;
  }    
}
```

# `ViewChild`

Tanto `ViewChild` como `ViewChildren` son dos decoradores que, utilizados sobre una propiedad
de la clase que representa a un componente, permiten obtener las instancias de elementos nativos, 
directivas y componentes que estén en el *template* del mismo.

{% blockquote Angular Docs https://angular.io/docs/ts/latest/api/core/index/ViewChild-decorator.html ViewChild API Reference %}
Configures a view query.

You can use ViewChild to get the first element or the directive matching the selector from the view DOM. If the view DOM changes, and a new child matches the selector, the property will be updated.

View queries are set before the ngAfterViewInit callback is called.

{% endblockquote %}

Es importante remarcar que las propiedades decoradas con `ViewChild` se resuelven justo antes de ejecutarse el
*hook* `ngAfterViewInit`. Esto quiere decir que si intentamos acceder a la propiedad en un *hook* previo, como por ejemplo en el *hook*
`ngOnInit`, esta tendrá como valor `undefined`.

Para realizar una *query* utilizando `ViewChild` podemos utilizar principalmente dos tipos de selectores:

* El tipo del componente. Se obtendrá el primer componente que se encuentre en la vista con el tipo informado
* El nombre de una *template variable*

## Usando un tipo

Usar `ViewChild` utilizando como selector el tipo del componente o directiva que queremos obtener
nos permite realizar búsquedas en la vista sin necesidad de crear una *template variable*.

Hay que tener en cuenta que en caso de haber más de un elemento con el tipo que se haya usado
como selector el framework nos devolverá la primera ocurrencia de la búsqueda.

```javascript app.component.ts
@Component({
  selector: 'my-app',
  template: `
    <h1>My App</h1>
    <system-message></system-message>
  `
})
export class AppComponent implements AfterViewInit {
  @ViewChild(SystemMessageComponent) message: SystemMessageComponent;

  ngAfterViewInit () {
    // Ahora puedes utilizar el componente hijo
  }
}
```

## Usando una *template variable*

Si queremos obtener una instancia en concreto entre varias posibilidades podemos usar una *template variable*
para utilizar su nombre como selector.


```javascript app.component.ts
@Component({
  selector: 'my-app',
  template: `
    <h1>My ViewChild Demo</h1>
    <system-message #firstMessage></system-message>
    <system-message #secondMessage></system-message>
  `
})
export class AppComponent implements AfterViewInit {
  @ViewChild('firstMessage') message: SystemMessageComponent;

  ngAfterViewInit () {
    // Ahora puedes utilizar el componente hijo
  }
}
```

La ventaja de este método es que podemos obtener también instancias de elementos nativos:


```javascript app.component.ts
@Component({
  selector: 'my-app',
  template: `
    <h1 #header>My ViewChild Demo</h1>
    <system-message></system-message> 
  `
})
export class AppComponent implements AfterViewInit {
  @ViewChild('header') header: ElementRef;

  ngAfterViewInit () {
    // Ahora puedes utilizar el componente hijo
  }
}
```

## Utilizando el API con ViewChild

Hasta ahora hemos aprendido a obtener las instancias de componentes, directivas y elementos nativos
que existan en el *template* de nuestro componente pero, ¿y cómo hacemos para comunicarnos con ellas?.

¡Muy sencillo! El objeto recuperado es una instancia de la clase del elemento
que hemos obtenido y por lo tanto todos sus métodos y propiedades están accesibles desde el mismo.

Lo único que hay que tener en cuenta es que solo se pueden utilizar después de la ejecución del *hook* `ngAfterViewInit`.

```javascript app.component.ts
@Component({
  selector: 'my-app',
  template: `
    <h1>My ViewChild Demo</h1>
    <system-message></system-message>
    <button (click)="toggleMessage()">Toggle message</button>
  `
})
export class AppComponent implements AfterViewInit {
  @ViewChild(SystemMessage) systemMessage: SystemMessage;

  ngAfterViewInit () {
    this.systemMessage.message = 'Hello World!';
  }

  toggleMessage () {
    this.systemMessage.active ? this.systemMessage.hide() : this.systemMessage.show();
  }
}
```

Puedes verlo en vivo y manipular el código en el siguiente **Plunkr**:

<iframe style="width: 100%; height: 600px" src="https://embed.plnkr.co/YmLiQgIbCMAUJq9LqS2s/" frameborder="0" allowfullscren="allowfullscren"></iframe>

# ViewChildren

El decorador `ViewChildren` nos permite obtener todos los elementos que existan en nuestro
*template* que sean del **tipo** que usamos como selector. El objeto devuelto es del tipo `QueryList`, 
el cual es resuelto antes del *hook* `ngAfterViewInit`. Podemos subscribirnos al `Observable` que nos proporciona
la propiedad `changes` para obtener los resultados de la búsqueda.

Puedes aprender más sobre la clase `QueryList` en la [documentación de la misma](https://angular.io/docs/ts/latest/api/core/index/QueryList-class.html).

```javascript app.component.ts
@Component({
  selector: 'my-app',
  template: `
    <h1 #header>My App</h1>
    <system-message message="First message"></system-message>
    <system-message message="Second message"></system-message> 
    <system-message message="Third message"></system-message> 
  `
})
export class AppComponent implements AfterViewInit {
  @ViewChildren(SystemMessageComponent) query: QueryList<SystemMessageComponent>;

  ngAfterViewInit () {
    this.query.changes.subscribe((items: Array<SystemMessageComponent>) => {
      messages.forEach((item: SystemMessageComponent) => console.log(item.message));
    });
  }
}
```

Y con esto ya tienes la información necesaria para empezar a utilizar tus componentes hijos
usando los decoradores `ViewChild` y `ViewChildren`. Es una técnica muy utilizada cuando se aplica
el patrón de [componentes de presentación y componentes contenedores](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.g879nu5gq).

¡Hasta la próxima!