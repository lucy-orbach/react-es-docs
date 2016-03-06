---
id: component-specs
title: Especificación de componentes y ciclo de vida
permalink: component-specs-es-ES.html
prev: component-api-es-ES.html
next: tags-and-attributes-es-ES.html
---

## Especificaciones de Componentes

Cuando creas una clase componente al invocar `React.createClass()`, debes entregar un objeto de especificaion que tenga el método `render` y opcionalmente otros métodos del ciclo de vida descritos aquí.

> Nota:
>
> También es posible utilizar clases simples de JavaScript como clases de componentes. Estas clases pueden implementar la mayoría de los mismos métodos, pero con algunas diferencias. Para mas información acerca de estas diferencias, por favor lea nuestra documentación acerca de [clases en ES6](/react/docs/reusable-components.html#es6-classes).

### render

```javascript
ReactElement render()
```

El método `render()` es obligatorio.

Cuando es llamado, examina `this.props` junto a `this.state` y retorna un elemento hijo único . Este elemento puede ser tanto una representación visual de un componente DOM nativo (como un `<div />` o un `React.DOM.div()`) u otro componente definido por ti.

También puedes retornar `null` o `false` para indicar que no quieres generar un elemento. Detrás de escena, React genera un tag `<noscript>` para trabajar con nuestro actual algoritmo de diferenciación. Cuando retornas `null` o `false`, `ReactDOM.findDOMNode(this)` retornara `null`.

La función `render()` debe ser *pura*, es decir, que no modifica el estado del componente, debe retornar el mismo resultado cada vez que es invocada, no debe leer o escribir el DOM, ni interactuar de otra forma con el buscador (Ej., al utilizar `setTimeout`). Si necesitas interactuar con el buscador, realiza tu trabajo en `componentDidMount()` o en otro método del ciclo de vida. Mantener `render()` puro hace que el renderizado del servidor sea mas practico, y permite que los componentes sean mas simples de entender.

### getInitialState

```javascript
object getInitialState()
```

Es invocado por unica vez antes que el componente este montado. El valor de retorno sera utilizado como el valor inicial de `this.state`. 

### getDefaultProps

```javascript
object getDefaultProps()
```

Es invocado una sola vez, y guardado en cache cuando la clase es creada. Los valores mapeados son ubicados en `this.props` si la propiedad no es declarada por el componente padre (Ej., usando un chequeo `in`)   

Este método es invocado antes de que cualquier instancia del componente haya sido creada, y por lo tanto no puede confiar en que `this.props` este definida. Adicionalmente, este atento de que cualquier objeto complejo retornado por `getDefaultProps()` sera compartido a través de las instancias, no copiado.  


### propTypes

```javascript
object propTypes
```

El objeto `propTypes` permite validar las propiedades que son entregadas a tus componentes. Para mas información acerca de `propTypes`, mire [Componentes reutilizables](/react/docs/reusable-components.html).

### mixins

```javascript
array mixins
```

El arreglo `mixins` te permite utilizar mixins para compartir comportamientos a través de múltiples componentes. Para mas información acerca de los mixins, mire [Componentes reutilizables](/react/docs/reusable-components.html).

### statics

```javascript
object statics
```

El objeto `statics` te permite definir métodos estáticos que pueden ser llamados en la clase componente. Por ejemplo:

```javascript
var MyComponent = React.createClass({
  statics: {
    customMethod: function(foo) {
      return foo === 'bar';
    }
  },
  render: function() {
  }
});

MyComponent.customMethod('bar');  // true
```

Los métodos definidos dentro de este bloque son _estáticos_, esto quiere decir que los puedes llamar antes de que cualquiera instancia del componente sea creada, estos métodos no tienen acceso a los `props` o `state` de tus componentes. Si quieres probar el valor de `props` en un método estático, cuando lo llames debes entregar los valores de props como argumento a este método estático. 


### displayName

```javascript
string displayName
```

La cadena de texto `displayName` es utilizada en los mensajes de debugging. JSX fija este valor automáticamente; mire [JSX en profundidad](/react/docs/jsx-in-depth.html#the-transform). 


## Metodos del ciclo de vida

Varios métodos son ejecutas en puntos específicos del ciclo de vida de un componente.


### Montar: componentWillMount

```javascript
void componentWillMount()
```

Es invocado una única vez, tanto en el cliente como en el servidor, inmediatamente antes del que el rendering inicial ocurra. Si llamas a `setState` dentro de este método, `render()` vera el cambio de estado y se ejecutara solo una vez, a pesar del cambio de estacdo que ocurrio.  

### Montar: componentDidMount

```javascript
void componentDidMount()
```

Es invocado una unica vez, solo en el cliente (no en el servidor), inmediatamente despues de que el rendering inicial ocurra. En este punto del ciclo de vida, puedes acceder a cualquier `ref` de los componentes hijos (Ej., puedes acceder a la representación del DOM). El método `componentDidMount()` de los componentes hijos es invocado antes que el de su componente padre.

Si deseas hacer una integración con otros frameworks de JavaScript, fijar timers utilizando `setTimeout` o `setInterval`, o enviar peticiones AJAX, puedes realizar esas operaciones en este método.

### Actualizar: componentWillReceiveProps

```javascript
void componentWillReceiveProps(
  object nextProps
)
```

Es invocado cuando un componente recibe nuevas props. Este método no es llamado para el rendering inicial.

Se utiliza este método como una oportunidad para reaccionar a una actualización de las propiedades `props` antes que `render()` sea llamado al actualizar el estado utilizando `this.setState()`. Las propiedades antiguas pueden ser accedidas en `this.props`. Llamar `this.setState()` dentro de esta función no genera una llamada adicional a `render()`.

```javascript
componentWillReceiveProps: function(nextProps) {
  this.setState({
    likesIncreasing: nextProps.likeCount > this.props.likeCount
  });
}
```

> Nota:
>
> Un error común en la ejecución del código de este meotodo del ciclo de vida, es asumir que las propiedades `props` han cambiado. Para entender porque esto es invalido. lea [que A implique B no implica que B implique A](/react/blog/2016/01/08/A-implies-B-does-not-imply-B-implies-A.html)
>
> No existe un método análogo `componentWillReceiveState`. Una actualización de propiedades puede generar un cambio de estado, pero no lo contrario. Si necesitas realizar una operación en respuesta a un cambio de estado, utiliza `componentWillUpdate`.

### Actualizar: shouldComponentUpdate

```javascript
boolean shouldComponentUpdate(
  object nextProps, object nextState
)
```

Es invocado antes de renderizar, cuando nuevas propiedades y/o estado han sido recibidos. Este método no es llamado en el render inicial, o cuando `forceUpdate` es utilizado. 

Utiliza esto como una oportunidad para hacer `return false` cuando estas seguro que la transición de la nueva propiedad y/o estado no requiere que el componente sea actualizado.

```javascript
shouldComponentUpdate: function(nextProps, nextState) {
  return nextProps.id !== this.props.id;
}
```

Si `shouldComponentUpdate` retorna false, la llamada a `render()` no se realizara hasta el siguiente cambio de estado. Además, `componentWillUpdate` y `componentDidUpdate` no seran llamados.

Por defecto, `shouldComponentUpdate` siempre retorna `true` para prevenir sutiles errores cuando el `state` muta (cambia), pero si tienes el cuidado de siempre tratar el `state` como inmutable y solo leer desde `props` y `state` en `render()`, entonces puedes sobreescribir `shouldComponentUpdate` con una implementación que compare las antiguas propiedades y/o estado para su reemplazo.

Si el rendimiento es un cuello de botella, especialmente con docenas o cientos de componentes, usa `shouldComponentUpdate` para acelerar tu aplicación. 

### Actualizar: componentWillUpdate

```javascript
void componentWillUpdate(
  object nextProps, object nextState
)
```

Es invocado inmediatamente antes de renderizar, cuando nuevas propiedades y/o estado se reciben. Este método no es llamado en el render inicial.

Usa esto como una oportunidad para realizar preparar una acción antes de que una actualización ocurra.

> Nota:
>
> *No puedes* utilizar `this.setState()` en este método. Si necesitas actualizar el estado en respuesta al cambio de una propiedad, utiliza `componentWillReceiveProps`.


### Actualizar: componentDidUpdate

```javascript
void componentDidUpdate(
  object prevProps, object prevState
)
```

Es invocado inmediatamente despues de que las actualizaciones del componente son enviadas al DOM. Este método no es llamado en el render inicial.

Usa esto como una oportunidad para realizar operaciones en el DOM luego de que el componente ha sido actualizado.


### Demontar: componentWillUnmount

```javascript
void componentWillUnmount()
```

Es invocado inmediatamente antes de que el componente sea desmontado del DOM.

Puedes realizar cualquier limpieza necesaria en este método, como invalidar timers, o limpiar cualquier elemento del DOM que haya sido creado en `componentDidMount`. 
