### 1- ¿Qué hace el hook useRef? :

>[!NOTE] 
El hook `useRef` es un React Hook que te permite hacer referencia a un valor que no es necesario para el renderizado.

```tsx
const ref = useRef( initialValue );
```

----

### 2- Implementación :

Llame al nivel superior del componente para declarar una referencia con `useRef`.

```tsx
import { useRef } from 'react';

function MyComponent() {  
	const intervalRef = useRef(0);  
	const inputRef = useRef(null);
}
```

#### Parámetros :

>[!NOTE] initialValue
>- El valor que desea que sea la propiedad del objeto ref inicialmente. 
>- Puede ser un valor de cualquier tipo. 
>- Este argumento se ignora después de la representación inicial. `current`

#### Devuelve :

>[!NOTE] Devolución del hook
>El hook useRef devuelve un objeto con una sola propiedad [`current`]:

>[!IMPORTANT] current 
>- Inicialmente, se establece en el que ha pasado. 
>- Si pasas el objeto ref a React como un atributo a un nodo JSX, React establecerá su propiedad.`initialValue` `ref`  como `current`
>- En los siguientes renders, devolverá el mismo objeto.`useRef`

#### Advertencias :

>[!IMPORTANT] Diferencia de useState 
>A diferencia del estado, es mutable. Sin embargo, si contiene un objeto que se usa para la representación (por ejemplo, una parte de su estado), no debe mutar ese objeto.`ref.current`

>[!IMPORTANT] Renderizado condicional 
>Cuando cambias la propiedad, React no vuelve a renderizar tu componente. Porque no es consciente de cuándo lo cambias debido a que una referencia es un objeto JavaScript simple.`ref.current`

>[!WARNING] 
>No escriba _ni lea_ durante la representación, excepto para la inicialización. Esto hace que el comportamiento del componente sea impredecible.`ref.current`
 
>[!WARNING] Trabajo en Modo estricto 
>En el modo estricto, React **llamará a la función de su componente dos veces** para ayudarlo a encontrar impurezas accidentales. Este es un comportamiento exclusivo del desarrollo y no afecta a la producción. Cada objeto ref se creará dos veces, pero se descartará una de las versiones. Si la función del componente es pura (como debería ser), esto no debería afectar al comportamiento.

---

### 3- Ventajas de usarlo :

Mediante el uso de una referencia, se asegura de que:
- Puede **almacenar información** entre re-renders (a diferencia de las variables regulares, que se reinician en cada renderizado).
- Cambiarlo **no desencadena un nuevo procesamiento** (a diferencia de las variables de estado, que desencadenan un nuevo procesamiento).
- La **información es local** para cada copia de su componente (a diferencia de las variables externas, que se comparten).

>[!WARNING] 
>Cambiar una referencia no desencadena una nueva representación, por lo que las referencias no son adecuadas para almacenar la información que desea mostrar en la pantalla. En su lugar, use state para eso.


----

### 4 - Usos :

##### Ejemplo 1 de 2: 

>[!WARNING] Contador de clics :
Este componente utiliza una referencia para realizar un seguimiento de cuántas veces se hizo clic en el botón. Tenga en cuenta que está bien usar una referencia en lugar de un estado aquí porque el recuento de clics solo se lee y escribe en un controlador de eventos.

```tsx
import { useRef } from 'react';  

export default function Counter() {
	let ref = useRef(0);

	function handleClick() {
		ref.current = ref.current + 1;
		alert('You clicked ' + ref.current + ' times!');
	}

	return (
		<button 
			onClick={handleClick}>
			Click me!
		</button>
	);
}
```

##### Ejemplo 2 : 

>[!WARNING] Trampa :
**No escriba _ni lea_ `ref.current` durante el renderizado.** Leer o escribir una referencia durante el **renderizado** rompe estas expectativas de que el componente se una funcion pura.
 
```tsx
function MyComponent() {  // ...  // 🚩 Don't write a ref during rendering  
	myRef.current = 123;  // ...  // 🚩 Don't read a ref during rendering  
	
	return ( 
		<h1>{myOtherRef.current}</h1>;
	)
}
```

En su lugar, puede leer o escribir referencias **de controladores de eventos o efectos**.

```tsx
function MyComponent() {  // ...  

// ✅ You can read or write refs in effects
	useEffect(() => {        
		myRef.current = 123;  
	});  // ...  

//✅ You can read or write refs in event handlers
	function handleClick(){  
		doSomething(myOtherRef.current);  
	}  
}
```

Si _tiene_ que leer o escribir algo durante la representación, use state en su lugar.

>[!CAUTION] 
>Cuando rompes estas reglas, es posible que tu componente siga funcionando, pero la mayoría de las características más nuevas que estamos agregando a React dependerán de estas expectativas. 


#### 2- Manipulación del DOM con una referencia :

Es particularmente común usar una referencia para manipular el DOM. React tiene soporte incorporado para esto. En primer lugar, declare un Objeto ref Con un Valor inicial de `null`

```TSX
import { useRef } from 'react';

function MyComponent() {  
	const inputRef = useRef(null);  
// ...
```

A continuación, pase el objeto ref como atributo al JSX del nodo DOM que desea manipular:`ref`

```tsx
return <input ref={inputRef} />;
```

Después de que React cree el nodo DOM y lo coloque en la pantalla, React establecerá el `current` propiedad de su objeto `ref` a ese nodo DOM. Ahora puedes acceder al nodo DOM y llamar a métodos como `focus()`.

```tsx
function handleClick() {    
  inputRef.current.focus();  
}
```

React volverá a establecer la propiedad cuando el nodo se elimine de la pantalla.`current=null`


#### 2.1- Ejemplos de manipulación del DOM con useRef :

##### Ejemplo 1 : 

Enfocar una entrada de texto. En este ejemplo, al hacer clic en el botón, se enfocará la entrada:

```tsx
import { useRef } from 'react';
  

export default function Form() {
	const inputRef = useRef(null);

	function handleClick() {
		inputRef.current.focus();
	}  

	return (
		<>
			<input 
				ref={inputRef} 
			/>
			
			<button 
				onClick={handleClick}>
				Focus the input
			</button>
		</>
	);
}
```

---

##### Ejemplo 2 :

>[!WARNING] Evitar volver a crear el contenido de la referencia :
React guarda el valor de referencia inicial una vez y lo ignora en los siguientes renderizados.

```tsx
function Video() {  
	const playerRef = useRef( new VideoPlayer() );  
// ...
```

>[!WARNING] Problemas de rendimiento 
>Aunque el resultado de `new VideoPlayer()` solo se usa para el renderizado inicial, sigues llamando a esta función en cada renderizado. Esto puede ser un desperdicio si se trata de crear objetos caros.

Para resolverlo, puede inicializar la referencia de esta manera:

```tsx
function Video() {  
	const playerRef = useRef(null);  
	
	if (playerRef.current === null) {    
		playerRef.current = new VideoPlayer();  
	}  
}
```

>[!NOTE] 
>Normalmente, no se permite escribir o leer `ref.current` durante el renderizado. Sin embargo, está bien en este caso porque el resultado es siempre el mismo y la condición solo se ejecuta durante la inicialización, por lo que es totalmente predecible.


#### Ejemplo 4:

Solución de problemas No puedo obtener una referencia a un componente personalizado.
Es posible que recibas un error en la consola, si intentas pasar uno a tu propio componente de esta manera :

```tsx
const inputRef = useRef(null);

return <MyInput ref={inputRef} />;
```

>[!IMPORTANT] Ref de CustomComponents
>De forma predeterminada, sus propios componentes no exponen referencias a los nodos DOM dentro de ellos. Para solucionar esto, busque el componente para el que desea obtener una referencia y implemente el ` forwardRef ` .

>[!NOTE] forwardRef 
>Permite al componente principal obtener una referencia de un componente personalizado.

```tsx
import { forwardRef } from 'react';

const MyInput = forwardRef(({ value, onChange }, ref) => {  
	return (    
		<input      
			value={value}      
			onChange={onChange}      
			ref={ref}    
		/>  
	);
});

export default MyInput;
```
