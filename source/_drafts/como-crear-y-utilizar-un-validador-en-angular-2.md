---
title: como crear y utilizar un validador en angular 2
tags:
---

Al igual que en la primera versión del framework, Angular 2 incorporá un módulo (`@angular/forms`)
para manejar y validar formularios. Este módulo proporciona las herramientas necesarias
para crear y manejar la interacción del usuario con un formulario de datos de una forma muy similar
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
<form name="myForm" novalidate">
  <label for="firstName">First name:</label>
  <input type="text" name="firstName" [(ngModel)]="model.firstName" required>
  <button type="submit">Send</button>
</form>
```

## Creando una directiva que aplique una validación

### Implementando la interfaz `Validator`

### Refactorizando el validador

## Utilizando el validador en un formulario reactivo

