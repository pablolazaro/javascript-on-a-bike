---
title: Transclusión de contenido en Angular
tags:
  - Angular
  - Recetas
description: >-
  Aprende como proyectar contenido en Angular, ¡si has utilizado la
  transclusión en la primera versión del framework te será muy sencillo!
date: 2016-10-23 12:57:20
---


> Este artículo está actualizado a la version ^4.0.0 de Angular

La transclusión o proyección de contenido es una técnica muy interesante que nos permite
flexibilizar nuestros componentes permitiendo al consumidor de los mismos añadir elementos 
en las zonas de nuestros *templates* reservadas para ello.

# Transclusión en Angular 1

Si has trabajado alguna vez con la primera versión del framework el termino transclusion te será familiar. En Angular 1 la transclusión de contenido se realizaba mediante la directiva `ngTransclude`, que marcaba el punto de transclusión 
del contenido, y el atributo `transclude` que indicaba si se debía realizar la transclusión.

Pongamos como ejemplo que queremos desarrollar un componente llamado `Card` que tendrá los estilos básicos de una [tarjeta de Material Design](https://material.google.com/components/cards.html)
y que permitirá incluir el contenido que queramos dentro de la misma.

> Para la demostración se está utilizando el proyecto [Material Design Lite](https://getmdl.io/index.html)

Lo primero es definir el componente sobre el que se hará la transclusión de contenido. Para ello, es importante utilizar la propiedad
`transclude` en la definición del mismo para avisar al framework de que queremos proyectar contenido en el *template*.
Después señalaremos el punto de transclusión con la directiva `ngTransclude`. En el ejemplo estamos usando la directiva como una etiqueta HTML,
pero también se podría utilizar como atributo de un elemento nativo.

```javascript
  angular
    .module('demo')
    .component('card', {
      template: `
        <div class="card mdl-card mdl-shadow--4dp">
          <ng-transclude></ng-transclude>
        </div>
      `,
      transclude: true
    });
```

Una vez definido el componente ya podemos utilizarlo e incrustar el contenido dentro de el. Por defecto, y al solo haber
un punto de transclusión definido, todo el contenido que se introduzca bajo la etiqueta del componente será proyectado.

```javascript
  angular
    .module('demo')
    .component('app', {
      template: `
        <card>
          <!-- Contenido transcluido -->
          <h2>My card</h2>
        </card>
      `
    });
```

Puedes ver un ejemplo en el siguiente **Plunker**:
<iframe style="width: 100%; height: 600px" src="https://embed.plnkr.co/oHhXBzx2nALnyXJcQmBi?show=preview&deferRun" frameborder="0" allowfullscren="allowfullscren"></iframe>

También es posible señalar varios puntos de transclusión, flexibilizando aún más los componentes que desarrollemos.
Si te interesa saber más sobre la directiva `ngTransclude` en Angular 1 te recomiendo echar un vistazo a la [documentación oficial](https://docs.angularjs.org/api/ng/directive/ngTransclude).

Y mientras tanto, ¿qué te parece si echamos un vistazo a la transclusión en Angular?.

# Proyección de contenido en Angular

Hasta ahora hemos hablado del termino **transclusión**, que era el utilizado en la primera versión del framework.
Con la nueva versión de Angular este termino ha cambiado para pasar a llamarse **proyección de contenido** (o *content projection*), aunque en esencia,
el concepto es el mismo.

> La proyección de contenido está recogida en el [estándar Shadow DOM](http://webcomponents.org/articles/introduction-to-shadow-dom/).

La proyección se realiza mediante un proceso muy similar al que utilizabamos en Angular 1 aunque en este caso solo necesitamos utilizar
la etiqueta `<ng-content>` para marcar el punto de proyección.

Reescribir el componente `Card` en Angular sería algo así: 

```javascript
@Component({
  selector: 'card',
  template: `
    <div class="card mdl-card mdl-shadow--4dp">
      <ng-content></ng-content>
    </div>
  `
})
export class CardComponent { }
```

Y utilizar dicho componente y proyectar contenido en él sería identico al caso anterior:

```javascript
@Component({
  selector: 'my-app',
  template: `
    <div>
      <card>
        <!-- Contenido proyectado -->
        <h2>My card</h2>
      </card>
    </div>
  `,
})
export class App { }
```

## Proyección multiple

En esta segunda versión de Angular también es posible realizar la proyección multiple de contenido, o lo que es lo mismo,
podemos identificar distintos puntos de proyección para que el consumidor coloque los elementos que considere necesarios
en cada uno de ellos.

Para ello el framework nos proporciona la propiedad `select`, que utilizada sobre la etiqueta `<ng-content>` nos permite
crear un selector para identificar el contenido que queremos proyectar. Como selector podemos utilizar **atributos** o **componentes**.

### Utilizando atributos como selectores

Este es el método más sencillo y probablemente el más utilizado. Supongamos que ahora nuestro componente `Card` va a tener dos puntos
de proyección: título y cuerpo de la tarjeta:

```javascript
@Component({
  selector: 'card',
  template: `
    <div class="md-card">
      <div class="mdl-card__title">
        <!-- Aquí va el título -->
      </div>
      <div class="mdl-card__supporting-text">
        <!-- Aquí va el cuerpo -->
      </div>
    </div>
  `
})
export class CardComponent { }
```

Para añadir los puntos de proyeccion debemos incluir dos etiquetas `<ng-content>` en la zona del template que queramos. Además,
debemos añadir el selector para cada uno de ellos de tal modo que el framework sepa que elementos tiene que proyectar.
Para el caso del título vamos a usar la propiedad `cardTitle` como selector, y para el cuerpo usaremos `cardBody`: 

```javascript
@Component({
  selector: 'card',
  template: `
    <div class="md-card">
      <div class="mdl-card__title">
        <ng-content select="[cardTitle]"></ng-content>
      </div>
      <div class="mdl-card__supporting-text">
        <ng-content select="[cardBody]"></ng-content>
      </div>
    </div>
  `
})
export class CardComponent { }
```

Vale, espera, ¿por qué los nombres de los atributos están dentro de unas llaves?.

Para indicar que queremos utilizar como selector un atributo, al igual que hacemos con las [directivas](https://angular.io/docs/ts/latest/guide/attribute-directives.html),
debemos meterlo dentro de unas llaves. Este es el modo de especificar a Angular que queremos utilizar el atributo como selector de proyección.

Para utilizar el componente, unicamente tenemos que añadir los atributos que hemos utilizado como selectores sobre los elementos que queremos proyectar en cada punto:

```javascript
@Component({
  selector: 'my-app',
  template: `
    <div>
      <card>
        <h2 cardTitle>My title</h2>
        <p cardBody>My body</p>
      </card>
    </div>
  `,
})
export class App { }
```

### Utilizando componentes como selectores

Utilizar componentes como selectores de proyección es una técnica interesante que nos permite
asegurar que el elemento que se proyecte sea del tipo que queramos.

Vamos a crear el componente `CardTitle` para el ejemplo:

```javascript
@Component({
  selector: 'card-title',
  template: `
    <div class="mdl-card__title">
      <h2>
        <ng-content></ng-content>
      </h2>
    </div>
  `,
})
export class CardTitleComponent { }
```

Ahora vamos a cambiar el selector del título de la tarjeta para utilizar el componente `card-title`:

```javascript
@Component({
  selector: 'card',
  template: `
    <div class="md-card">
      <div class="mdl-card__title">
        <ng-content select="card-title"></ng-content>
      </div>
      <div class="mdl-card__supporting-text">
        <ng-content select="[cardBody]"></ng-content>
      </div>
    </div>
  `
})
export class CardComponent { }
```

Por último, en la aplicación vamos a utilizar el nuevo componente `card-title` en nuestra tarjeta (me he tomado la licencia de usar también el componente `card-body`):

```javascript
@Component({
  selector: 'my-app',
  template: `
    <div>
      <card>
        <card-title>My title</card-title>
        <card-body>My body</card-body>
      </card>
    </div>
  `,
})
export class App { }
```

¡Y esto es todo! La proyección de contenido en Angular es tan sencilla como la transclusión
en la primera versión del framework. Para finalizar, en el siguiente **Plunker** puedes ver el
ejemplo completo.

¡Hasta la próxima!

<iframe style="width: 100%; height: 600px" src="https://embed.plnkr.co/E6suFuoN5nPFP0Ho6s6o?show=preview&deferRun" frameborder="0" allowfullscren="allowfullscren"></iframe>
