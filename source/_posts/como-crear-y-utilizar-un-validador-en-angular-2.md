---
title: Cómo crear y utilizar un validador en Angular 2
tags:
  - Angular 2
  - Forms
description: >-
  Aprende, junto a Jon Snow,  como crear y utilizar tus propios validadores en
  Angular 2
date: 2016-11-09 10:30:51
---


> Este artículo está actualizado a la version 2.2.0 de Angular 2

{% img https://s-media-cache-ak0.pinimg.com/236x/15/1d/41/151d4189b0b48debd2cb32e0d9e2b0f7.jpg "Jon Snow, Software Developer" %}

Este es Jon. Jon ha decidido dejar la espada y las guerras por los siete reinos
para convertirse en el mejor developer de todo Westeros. Jon utiliza exclusivamente **jQuery**
para desarrollar aplicaciones web, así que vamos a mostrarle algunas de las caracteristicas
de Angular 2 para mejorar su productividad y ayudarle a crear software más robusto. No sabes nada, Jon Nieve.

{% img http://i.giphy.com/has1WKhoorwLS.gif "Sí, sí, cómo quieras Jon" %}

Al igual que en la primera versión del framework Angular 2 incorporá el módulo `forms`
para manejar y validar, entre otras cosas, formularios. Este módulo proporciona las herramientas necesarias
para crear y manejar la interacción del usuario con un formulario de una forma muy similar
a la que se hiciese en Angular 1.

> La explicación en detalle del funcionamiento del módulo `forms` no entra dentro del alcance del artículo

## Utilizando una directiva para añadir una validación

Cuando tenemos un formulario típico al que queremos añadir las validaciones que HTML5 trae incorporadas
no hay que realizar ningún desarrollo adicional, el framework nos proporciona unas directivas que aplican
las validaciones necesarias y se integran con el sistema de validación de Angular de forma transparente.

Un ejemplo puede ser el siguiente:

```
<form name="myForm">
  <label for="firstName">First name:</label>
  <input type="text" name="firstName" required>
  <button type="submit">Send</button>
</form>
```

El formulario anterior contiene un caja de texto para introducir un nombre. Este campo está marcado como obligatorio,
pues tiene el atributo `required` que obliga al usuario a rellenarlo si quiere enviar la información que el formulario contiene.
Dicha validación se realiza nativamente en el navegador siguiendo el estándar de HTML5. En el ejemplo Angular aún no ha entrado en juego.

Bien, ahora vamos a utilizar el framework para validar el formulario, lo primero es añadir el atributo `novalidate` a la 
etiqueta `form` para evitar que la validación nativa se active.

Después, añadiremos las directivas necesarias para crear los controles del formulario, para ello necesitamos
la directiva `ngModel` (`formControl` o `formControlName` en caso de formularios reactivos) junto con el atributo `name`
para identificar cada control:

```
<form name="myForm" novalidate>
  <label for="firstName">First name:</label>
  <input type="text" name="firstName" [(ngModel)]="model.firstName" required>
  <button type="submit">Send</button>
</form>
```

Ahora podemos utilizar las propiedades que nos proporciona Angular para saber si el formulario
se ha modificado, si es válido y en caso de que no lo sea que errores tiene cada campo ([forms](https://angular.io/docs/ts/latest/guide/forms.html)).

Por ejemplo, en el caso de la directiva `required` esta añade el error `required: true` cuando el campo no se ha rellenado.

## Creando una directiva que aplique una validación

Para entender mejor el concepto vamos a crear nuestra propia validación. Esta validación comprobará
que el nombre que recibe sea el de un miembro legítimo de la [casa Stark](http://gameofthrones.wikia.com/wiki/House_Stark).

> Para los que no conozcan la serie [Juego de Tronos](https://g.co/kgs/9XLkiy) o no hayan leído la saga
> de libros [Canción de Hielo y Fuego](https://es.wikipedia.org/wiki/Canci%C3%B3n_de_hielo_y_fuego) los miembros legítimos (los bastardos no cuentan) de la casa Stark son los siguientes:
Eddard, Benjen, Lyanna, Catelyn, Robb, Sansa, Arya, Brandon y Ryckon.

{% img http://i.giphy.com/xONhGezd3gdB6.gif "Esta vez no Jon" %}

Teniendo esto claro, vamos a crear la directiva:

```javascript
@Directive({
  selector: '[stark]'
})
export class HouseStarkValidator { }
```

Por ahora la directiva aplica a todo elemento que tenga el atributo `stark`. Pero, ¿cómo convertimos la directiva
en un validador?.

### Implementando la interfaz `Validator`

Angular incluye la interfaz [Validator](https://angular.io/docs/ts/latest/api/forms/index/Validator-interface.html), la cual debe ser implementada
por toda clase que quiera actuar como un validador. La interfaz nos obliga a implementar el siguiente método:

```javascript
validate(c: AbstractControl) : {[key: string]: any}
```

Dicho método recibirá el control a validar y debe devolver un objeto con los errores de validación o `null` en caso de que
la validación sea correcta. Vamos a implementar la interfaz en nuestra directiva:

```javascript
@Directive({
  selector: '[stark]'
})
export class HouseStarkValidator implements Validator {
  private LEGITIMATE_STARK_MEMBERS: Array<string> = [
    'EDDARD', 
    'BENJEN', 
    'LYANNA', 
    'CATELYN', 
    'ROBB', 
    'SANSA', 
    'ARYA', 
    'BRANDON', 
    'RYCKON'
  ];

  validate(control: AbstractControl): {[key: string]: any} {
    if (control.value != null || typeof control.value === 'string' && control.value.length !== 0) {
      return this.isLegitimateStark(control.value) ? null : { 'stark': true };
    } else {
      return null;
    }
  }

  private isLegitimateStark (name: string): boolean {
    return this.LEGITIMATE_STARK_MEMBERS.indexOf(name.toUpperCase().trim()) !== -1;
  }
}
```

### Añadiendo el validador a `NG_VALIDATORS`

Con implementar la interfaz `Validator` no es suficiente. 

Para indicarle al framework que utilice nuestra clase como un validador debemos registrarla en el provider
`NG_VALIDATORS`, una colección que incluye Angular y que es extensible con las directivas
que queramos.

Una vez registrada la directiva Angular se encargará de invocar el método `validate` de la misma y de añadir el error
al objeto de errores del `FormControl`.

```javascript
const HOUSE_STARK_VALIDATOR: any = {
  provide: NG_VALIDATORS,
  useExisting: forwardRef(() => HouseStarkValidator),
  multi: true
};

@Directive({
  selector: '[stark]',
  providers: [ HOUSE_STARK_VALIDATOR ],
})
export class HouseStarkValidator implements Validator {
  ...
}
```

Sin entrar mucho en detalle, lo que estamos haciendo es añadiendo al *provider* `NG_VALIDATORS` nuestra
directiva `HouseStarkValidator`. Para ello, es necesario utilizar la propiedad `multi: true`, indicando que queremos
utilizar varias dependencias para el token `NG_VALIDATORS` y la propiedad `useExisting` que hace referencia a nuestra directiva.

¿Y para qué es la función `forwardRef`? La función `forwardRef` nos permite hacer referencia a instancias de clases
que aún no se hayan creado en el sistema de inyección de dependencias de Angular. Es decir, en este caso, el decorador `Directive` está haciendo
referencia a una instancia de `HouseStarkValidator` que aún no se ha creado.

Para entender mejor esto, podéis leer los siguientes artículos acerca de [forwardRef](http://blog.thoughtram.io/angular/2015/09/03/forward-references-in-angular-2.html)
y [multi providers](http://blog.thoughtram.io/angular2/2015/11/23/multi-providers-in-angular-2.html).

### Afinando el selector

Hasta ahora nuestra directiva aplica a todo elemento que tenga el atributo `stark`, pero esto no es muy eficiente
dado que nuestra validación está muy ligada a un control de formulario que esté integrado dentro de un formulario de Angular 2,
es decir, un elemento que tenga la combinación de atributos `ngModel` y `name` en caso de formularios *template-drive* o los atributos
`formControl` o `formControlName` en caso de formularios reactivos.

Así que, ¿por qué no afinar el selector para que solo aplique a los elementos que realmente sean controles de formulario?.

Las directivas en Angular 2 pueden tener multiples selectores (separados por comas) además de selectores que combinen varios atributos.
Podemos, por ejemplo, aplicar la directiva solo a aquellos elementos que tengan el atributo `stark` y la directiva `ngModel` de esta manera:

```javascript
@Directive({
  selector: '[stark][ngModel]'
})
```

En nuestro caso, y para cumplir todos los requisitos que hemos listado anteriormente, el selector de nuestra directiva quedaría así:

```javascript
@Directive({
  selector: '[stark][ngModel],[stark][formControl],[stark][formControlName]'
})
```

### Resultado final

Después de todos los pasos nuestra directiva está lista para ser utilizada, el código final quedaría así:

```javascript
const HOUSE_STARK_VALIDATOR: any = {
  provide: NG_VALIDATORS,
  useExisting: forwardRef(() => HouseStarkValidator),
  multi: true
};

@Directive({
  selector: '[stark]',
  providers: [ HOUSE_STARK_VALIDATOR ]
})
export class HouseStarkValidator implements Validator {
  private LEGITIMATE_STARK_MEMBERS: Array<string> = [
    'EDDARD', 
    'BENJEN', 
    'LYANNA', 
    'CATELYN', 
    'ROBB', 
    'SANSA', 
    'ARYA', 
    'BRANDON', 
    'RYCKON'
  ];

  validate(control: AbstractControl) : {[key: string]: any} {
    if (control.value != null || typeof control.value === 'string' && control.value.length !== 0) {
      return this.isLegitimateStark(control.value) ? null : { 'stark': true };
    } else {
      return null;
    }
  }

  private isLegitimateStark (name: string): boolean {
    return this.LEGITIMATE_STARK_MEMBERS.indexOf(name.toUpperCase().trim()) !== -1;
  }
}
```

Puedes ver la directiva en acción en el siguiente Plunker:

<iframe style="width: 100%; height: 600px" src="https://embed.plnkr.co/fnKjzidT4LreVT8aZ76Z?show=preview&deferRun" frameborder="0" allowfullscren="allowfullscren"></iframe>

{% img http://i.giphy.com/JJ59Hg2DGQ2ys.gif "S...now" %}

## Utilizando el validador en un formulario reactivo

Hasta ahora hemos hablado de validaciones en formularios *template-driven*, pero ¿qué hay de los formularios reactivos?. 

> Si quieres saber más acerca de los formularios reactivos te recomiendo leer [este artículo](http://blog.thoughtram.io/angular/2016/06/22/model-driven-forms-in-angular-2.html)

Cuando queremos añadir una validación en un formulario reactivo debemos hacerlo a la hora de construir los controles de formulario. Si recuerdas bien,
un formulario es una instancia de `FormGroup` que a su vez está formada por multiples `FormControl`. La forma más sencilla de crear un formulario reactivo sería así:

```javascript
let form = new FormGroup({
  firstName: new FormControl('', Validators.required),
  lastName: new FormControl('', Validators.required)
});
```

Si echas un vistazo al constructor de la clase [`FormControl`](https://angular.io/docs/ts/latest/api/forms/index/FormControl-class.html) podrás ver que este recibe tres argumentos:

* Valor inicial o objeto de inicialización
* Listado de validadores síncronos
* Listado de validadores asíncronos

Angular 2 provee la clase [`Validators`](https://angular.io/docs/ts/latest/api/forms/index/Validators-class.html), la cual tiene una serie de métodos estáticos para aplicar las validaciones más comunes:
`required`, `minLength`, `pattern`, `nullValidator`, etc. A la hora de crear un `FormControl` podemos pasar cualquiera de esos métodos estáticos
como segundo argumento:

```javascript
new FormControl('', [Validators.required, Validators.minLength(5)]);
```

De este modo, estaríamos aplicando las validaciones indicadas sobre el control de formulario. Vamos a refactorizar el código de nuestra directiva
para poder utilizar el validador en un formulario reactivo.

### Refactorizando el validador

Lo primero es extraer la lógica de la validación. En este caso vamos a sacar la función a un fichero que exporte la misma,
para poder utilizarla desde varios sitios:

```javascript house-stark.validator.ts
const LEGITIMATE_STARK_MEMBERS: Array<string> = [
  'EDDARD', 
  'BENJEN', 
  'LYANNA', 
  'CATELYN', 
  'ROBB', 
  'SANSA', 
  'ARYA', 
  'BRANDON', 
  'RYCKON'
];

export function isLegitimateStark (control: AbstractControl): {[key: string]: any} {
  if (control.value != null || typeof control.value === 'string' && control.value.length !== 0) {
    return LEGITIMATE_STARK_MEMBERS.indexOf(name.toUpperCase().trim()) !== -1 ? 
      null : { 'stark': true };
  } else {
    return null;
  }
}
```

Ahora, importaremos la función en nuestra directiva y la utilizaremos en el método `validate`:

```javascript house-stark-validator.directive.ts
import { isLegitimateStark } from './house-stark.validator';
...
validate (control: AbstractControl): {[key: string]: any} {
  return isLegitimateStark(control);
}
...
```

En el caso de un formulario reactivo el mecanismo es el mismo. Primero importamos la función y después la pasamos
como argumento en la lista de validadores:

```javascript
new FormControl('', [Validators.required, isLegitimateStark]);
```

### Parametrizando el validador

Hay validaciones que no dependen unicamente del valor del `FormControl` si no también de algún tipo de parametrización, por ejemplo,
la validación `minLength` depende de un valor que sirve como referencia para calcular si la cadena cumple con la longitud mínima.

Vamos a modificar nuestro validador para permitir que reciba un parámetro que indicará si se deben contar como legítimos los personajes
que sean bastardos, para ello nos vamos a ayudar de la interfaz `ValidatorFn`:

{% img http://i.giphy.com/FqOSrvVJhkP6M.gif "¿Contento Jon?" %}

```javascript house-stark.validator.ts
const LEGITIMATE_STARK_MEMBERS: Array<string> = [
  'EDDARD', 
  'BENJEN', 
  'LYANNA', 
  'CATELYN', 
  'ROBB', 
  'SANSA', 
  'ARYA', 
  'BRANDON', 
  'RYCKON'
];

const JUST_BASTARDS = ['JON', 'JON SNOW'];

function isLegitimateStarkWithoutBastards (name: string) {
  return LEGITIMATE_STARK_MEMBERS.indexOf(name.toUpperCase().trim()) !== -1 ? null : { 'stark': true })
}

function isLegitimateStarkWithBastards (name: string) {
  return LEGITIMATE_STARK_MEMBERS.concat(JUST_BASTARDS).indexOf(name.toUpperCase().trim()) !== -1 ? null : { 'stark': true })
}

export function isLegitimateStark (acceptBastards: boolean): ValidatorFn {
  return function (control: AbstractControl): {[key: string]: any} {
    if (control.value != null || typeof control.value === 'string' && control.value.length !== 0) {
      return acceptBastards ?
        isLegitimateStarkWithBastards(control.value) :
        isLegitimateStarkWithoutBastards(control.value); 
    } else {
      return null;
    }
  }
}
```

Ahora lo que tenemos es una función que recibe un parámetro y, utilizando un *closure*, devuelve la función validadora ya parametrizada.
Su uso sería idéntico a cualquier otra validación con parámetro:

```javascript
new FormControl('', [Validators.required, isLegitimateStark(true)]);
```

¡Oops! ¿Y ahora cómo parámetrizamos la directiva validadora?. Si queremos que nuestra directiva sea configurable tenemos que modificarla como se muestra a continuación.

Lo primero es definir una propiedad de entrada utilizando el decorador `@Input`:

```javascript
@Input() acceptBastards: boolean = false;
```

Y utilizar la propiedad para parametrizar la validación:
```
validate (control: AbstractControl): {[key: string]: any} {
  return isLegitimateStark(this.acceptBastards)(control);
}
```

Ahora la directiva se puede utilizar así:
```
<input type="text" name="character" [(ngModel)]="model.character" stark acceptBastards="true">
```

¡Espera! ¡¿Y qué ocurre si la propiedad `acceptBastards` cambia?!.

{% img http://i.giphy.com/dSFotB5BZwWd2.gif "Jon ha olvidado controlar el cambio de la propiedad" %}

Tranquilo Jon, está todo controlado. ¿Recuerdas los [lifecycle hooks](https://angular.io/docs/ts/latest/guide/lifecycle-hooks.html)? Bien,
pues resulta que hay un hook llamado [`ngOnChanges`]((https://angular.io/docs/ts/latest/guide/lifecycle-hooks.html#!#onchanges)) que nos permite estar al tanto de cualquier cambio en las propiedades de la directiva.

Es recomendable, aunque no necesario, implementar la interfaz `onChanges` en nuestra clase ya que nos ayudará a implementar correctamente
el método `ngOnChanges` que se ejecutará cada vez que haya un cambio en las propiedades de entrada (las que han sido marcadas con el decorador `@Input`)
de la clase. 

Bien, lo que tenemos que hacer ahora es parametrizar el validador cada vez que la propiedad `acceptBastards` cambie y guardar la función validadora
resultante para invocarla en el método `validate`. Además debemos implementar el método opcional `registerOnValidatorChange` que nos proporciona la interfaz `Validator`,
este método recibe una función que deberá ser invocada cada vez que el validador cambie, para ello la guardaremos en una propiedad interna de la clase y la invocaremos
cada vez que el validador cambie.

> El proposito del método `registerOnValidatorChange` es registrar la función que Angular estará escuchando para validar de nuevo el control cuando el validador se modifique

```javascript
export class HouseStarkValidator implements Validator, OnInit, OnChanges {
  @Input() acceptBastards: boolean;
  private validatorFn: Function;
  private onChange: Function;
  
  validate (control: AbstractControl): {[key: string]: any} {
    return this.validatorFn ? this.validatorFn(control) : null;
  }
  
  ngOnInit () {
    // Por defecto si no hay valor inicializamos la propiedad a false
    this.acceptBastards = this.acceptBastards === null || this.acceptBastards === undefined ? 
      false : this.acceptBastards;
  }
  
  ngOnChanges (changes: SimpleChanges) {
    let acceptBastardsChange: SimpleChange = changes['acceptBastards'];
    this.createValidatorFunction(acceptBastardsChange.currentValue);
    if (this.onChange) this.onChange();
  }
  
  registerOnValidatorChange(fn: () => void) { this.onChange = fn; }

  private createValidatorFunction (acceptBastards: any) {
    this.validatorFn = isLegitimateStark(this.acceptBastards);
  }
}
```

¡Y esto es todo! Con esto queda cubierto el uso de las validaciones en formularios *template-driven* y formularios reactivos, y además,
ya podemos empezar a crear nuestros propios validadores. ¿Qué te ha parecido Jon?.

{% img http://i.giphy.com/c8i9FWxTLvkT6.gif "Alucinante, ¿verdad?" %}

¡Hasta la próxima!

Si quieres ver todo el código resultante y jugar con lo que hemos tratado en el artículo puedes hacerlo en el siguiente Plunker:

<iframe style="width: 100%; height: 600px" src="https://embed.plnkr.co/x7HgdzfZwLfAGAMKEBaA?show=preview&deferRun" frameborder="0" allowfullscren="allowfullscren"></iframe>
