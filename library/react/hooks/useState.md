## 1- ¿Qué es un componente de React?

>[!NOTE] 
>`Componente de React:` Es una pequeña pieza de código encapsulada re-utilizable que puede tener estado o no.

```ts
export function Component() {
    return (
        <div>
            <h1>Hello</h1>
        </div>
    )
}
```

----

## 2- ¿Qué es un estado en React?

>[!NOTE] 
>`Estados en React:` Es como se encuentra la información del componente en un punto determinado del tiempo.

----

## 3- ¿Qué hace el hook useState?

>[!NOTE] 
>`useState:` es un React Hook que le permite agregar una variable de estado a su componente.
```ts
const [state, setState] = useState(initialState)
```

----

### 4- Estado inicial :

>[!NOTE] 
>`initial State:` La convención es nombrar variables de estado `[something, setSomething]` como si se usara la desestructuración de matrices.

```ts
import { useState } from 'react';
   
function MyComponent() {  
	const [age, setAge] = useState(28);  
	const [name, setName] = useState('Taylor'); 
	const [todos, setTodos] = useState(() => createTodos());
}
```
##### Parámetros del estado inicial :

>[!NOTE] 
>El valor que desea que tenga el estado inicialmente. Puede ser un valor de cualquier tipo. Este argumento se ignora después del renderizado inicial. 


>[!NOTE]
>Si pasa una función como estado inicial, será tratada como una _función inicializadora_ . Esta debe ser pura, no debe aceptar argumentos y debe devolver un valor de cualquier tipo.

>[!NOTE]
>React llamará a su función inicializadora al inicializar el componente y almacenará su valor de retorno como estado inicial.

##### Devoluciones :

>[!IMPORTANT] 
>El hook useState devuelve una matriz con exactamente dos valores:
> - El estado actual. Durante el primer renderizado, coincidirá con el `initialState` que has pasado.
> - La [`setfunción`] que le permite actualizar el estado a un valor diferente y activar una nueva representación.


##### Advertencias :

>[!CAUTION] 
>`Usos del Hook`
>El useState es un Hook, por lo que solo puedes llamarlo **en el nivel superior de tu componente** o en tus propios Hooks. No puedes llamarlo en bucles internos o condiciones. Si lo necesita, extraiga un nuevo componente y mueva el estado a él.

>[!WARNING] 
>En modo estricto, React **llamará a su función de inicializador dos veces** para ayudarlo a encontrar impurezas accidentales. Este es un comportamiento exclusivo de desarrollo y no afecta la producción. Si su función inicializadora es pura (como debería ser), esto no debería afectar el comportamiento. Se ignorará el resultado de una de las llamadas.

----

### 5- Actualización del estado ( setFunction ) :

>[!NOTE]
>La `setfunción` devuelta por  `useState` le permite actualizar el estado a un valor diferente y activar una nueva representación. Puedes pasar el siguiente estado directamente, o una función que lo calcule a partir del estado anterior:

```ts 
const [age, setAge] = useState(21);
const [name, setName] = useState('Edward');
		
function handleClick() {  
	setName('Taylor');
	setAge((prevState) => prevState + 1); 
}
```

##### Parámetros de la ( setFunction ) :

>[!NOTE] 
>`nextState:` El valor que desea que tenga el estado. Puede ser un valor de cualquier tipo, pero existe un comportamiento especial para las funciones.

>[!IMPORTANT] 
>Si pasa una función como `nextState`, será tratada como una _**función de actualización**_ . Debe ser puro, debe tomar el estado pendiente como único argumento y debe devolver el siguiente estado.

>[!IMPORTANT] 
>React pondrá su función de actualización en una cola y volverá a renderizar su componente. Durante el siguiente renderizado, React calculará el siguiente estado aplicando todos los actualizadores en cola al estado anterior.

##### Devoluciones :

>[!NOTE] 
>Las funciones de actualizacion no tienen un valor de retorno.

##### Advertencias :

>[!CAUTION] 
>La `setfunción` **sólo actualiza la variable de estado para el _siguiente_ renderizado** . Si lee la variable de estado después de llamar a la `setfunción`, aún obtendrá el valor anterior que estaba en la pantalla antes de su llamada.

>[!IMPORTANT] 
>Si el nuevo valor que proporciona es idéntico al actual `state`, según lo determinado por una comparación, React omitirá **volver a renderizar el componente y sus hijos** . Aunque en algunos casos es posible que React aún necesite llamar a su componente antes de omitir los elementos secundarios.

>[!IMPORTANT] 
>Realiza las actualizaciones de estado por lotes. Actualiza la pantalla **después de que todos los controladores de eventos se hayan ejecutado** y hayan llamado a sus `setfunciones`. Esto evita múltiples renderizaciones durante un solo evento.

>[!NOTE] 
>En el raro caso de que necesites forzar a React a actualizar la pantalla antes, por ejemplo para acceder al DOM, puedes usar `flushSync`.

>[!NOTE] 
>Llamar a la `setfunción` _durante la renderización_ solo se permite desde el componente de renderización actual. React descartará su salida e inmediatamente intentará renderizarla nuevamente con el nuevo estado.