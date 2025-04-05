### 1- ¿Qué hace el hook useLayuotEffect? :

>[!NOTE] 
>El hook `useLayoutEffect` es una versión de `useEffect` que se activa antes de que el navegador vuelva a pintar la pantalla.

```tsx
useLayoutEffect(setup, dependencies?)
```

>[!WARNING]
> El hook `useLayoutEffect` puede perjudicar el rendimiento. Prefiera `useEffect` cuando sea posible.


### 2- Implementación :

Llame `useLayoutEffect` para realizar las mediciones de diseño antes de que el navegador vuelva a pintar la pantalla.

```tsx
import { useState, useRef, useLayoutEffect } from 'react';

function Tooltip() {  
	const ref = useRef(null);  
	const [tooltipHeight, setTooltipHeight] = useState(0);  

	useLayoutEffect(() => {    
		const { height } = ref.current.getBoundingClientRect();
		setTooltipHeight(height);  
 	}, []);  
```

#### Parámetros :

>[!IMPORTANT] 
>`setup (función de configuración)`
> - La función con la lógica de configuración, esta también puede devolver opcionalmente una _función de limpieza_. 
> - Antes de que su componente se agregue al DOM, se ejecutará la función de configuración. 
> - Después de cada re-renderizado con dependencias cambiadas, primero ejecutará la función de limpieza (si la proporcionaste) con los valores antiguos, y luego ejecutará la función de configuración con los nuevos valores. 
> - Antes de que el componente sea eliminado del DOM, se ejecutará la función de limpieza.

>[!IMPORTANT] 
>`dependencies (Arreglo de dependencias)` 
> - La lista de todos los valores reactivos a los que se hace referencia dentro del código. 
> - La lista de dependencias debe tener un número constante de elementos y escribirse en línea. 
> - React comparará cada dependencia con su valor anterior utilizando la comparación `Object.is`. Si se omite, el efecto se volverá a ejecutar después de cada reprocesamiento del componente.

#### Devuelve :

>[!CAUTION] 
>El hook `useLayoutEffect` devuelve un valor undefined.

#### Advertencias :

>[!CAUTION]
> El hook `useLayoutEffect` es un Hook, por lo que no se puede llamar dentro de bucles o condiciones.

>[!WARNING] 
>Cuando el modo estricto está activado, React **ejecutará un ciclo adicional de configuración + limpieza** antes de la primera configuración real. Se trata de una prueba de esfuerzo que garantiza que la lógica de limpieza "refleje" la lógica de configuración y que detenga o deshaga lo que sea que esté haciendo la configuración. 

>[!CAUTION] 
>Si algunas de las dependencias son objetos o funciones definidas dentro del componente, existe el riesgo de que **hagan que el efecto se vuelva a ejecutar con más frecuencia de la necesaria.** Para solucionar este problema, elimine las dependencias innecesarias de objetos y funciones. También puede extraer actualizaciones de estado y lógica no reactiva fuera de su efecto.
 
>[!NOTE] 
>Los efectos **solo se ejecutan en el cliente.** No se ejecutan durante la representación del servidor.

>[!NOTE] 
>El código interno y todas las actualizaciones de estado programadas desde él **impiden que el navegador vuelva a pintar la pantalla.** Cuando se usa en exceso, esto hace que la aplicación sea lenta.

---

### 3- Usos :

#### Medición del diseño antes de que el navegador vuelva a pintar la pantalla :

>[!NOTE]
>La mayoría de los componentes no necesitan conocer su posición y tamaño en la pantalla para decidir qué renderizar. A veces, eso no es suficiente:

Ejemplo de Uso: Imagine una información sobre herramientas que aparece junto a algún elemento al pasar el puntero. Si hay suficiente espacio, la información sobre herramientas debería aparecer encima del elemento, pero si no encaja, debería aparecer debajo. Para representar la información sobre herramientas en la posición final correcta, debe conocer su altura.

Para hacer esto, debe renderizar en dos pasadas:
- 1- Renderice la información sobre herramientas en cualquier lugar.
- 2- Mide su altura y decide dónde colocar la información sobre herramientas.
- 3- Vuelva _a_ renderizar la información sobre herramientas en el lugar correcto.

>[!WARNING] 
>Tenga en cuenta que, aunque el componente tiene que renderizarse en dos pasadas (primero, con inicializado a y luego con la altura real medida), solo verá el resultado final. Esta es la razón por la que necesita en lugar de `useEffect` para este ejemplo. 

Diferencias useLayoutEffect frente a useEffect :
- `useLayoutEffect:`  Bloquea el navegador para que no vuelva a pintar.
- `useEffect:` No bloquea el navegador.

#### Ejemplo 1 de 2: 

<!-- >[!cite] Renderizacion. -->
React garantiza que el código dentro y cualquier actualización de estado programada dentro de él se procesarán **antes de que el navegador vuelva a pintar la pantalla.** Esto le permite representar la información sobre herramientas, medirla y volver a renderizar la información sobre herramientas sin que el usuario note el primer procesamiento adicional. 

```tsx
import { useRef, useLayoutEffect, useState } from 'react';
import { createPortal } from 'react-dom';
import TooltipContainer from './TooltipContainer.js';
  
export default function Tooltip({ children, targetRect }) {
	const ref = useRef(null);
	const [tooltipHeight, setTooltipHeight] = useState(0);

	useLayoutEffect(() => {
		const { height } = ref.current.getBoundingClientRect();
		setTooltipHeight(height);

		console.log('Measured tooltip height: ' + height);
	}, []);

	let tooltipX = 0;
	let tooltipY = 0;
	
	if (targetRect !== null) {
		tooltipX = targetRect.left;
		tooltipY = targetRect.top - tooltipHeight;

		if (tooltipY < 0) {
		// It doesn't fit above, so place below.
			tooltipY = targetRect.bottom;
		}
	}
	
	return createPortal(
		<TooltipContainer 
			x={tooltipX} 
			y={tooltipY} 
			contentRef={ref}>
		{children}
		</TooltipContainer>, 
		document.body
	);
}
```
  
>[!CAUTION] Nota : 
>Renderizar en dos pasadas y bloquear el navegador perjudica el rendimiento. Trata de evitar esto cuando puedas.

---

### 4- Solución de problemas :

>[!CAUTION] 
>Recibo un error: No hace nada en el servidor `useLayoutEffect`.

El propósito de `useLayoutEffect` es permitir que el componente utilice la información de diseño para la representación.

Cuando usted usa la representación del servidor, la aplicación React se representa en HTML en el servidor para la representación inicial. Esto le permite mostrar el HTML inicial antes de que se cargue el código JavaScript.

El problema es que en el servidor no hay información de diseño.

En el ejemplo anterior, la llamada en el componente le permite posicionarse correctamente en función de la altura del contenido. Si intentara renderizar como parte del HTML inicial del servidor, esto sería imposible de determinar. ¡En el servidor, aún no hay diseño! Por lo tanto, incluso si lo renderizó en el servidor, su posición "saltaría" en el cliente después de que se cargue y ejecute JavaScript.

Normalmente, los componentes que se basan en la información de diseño no necesitan representarse en el servidor de todos modos. Por ejemplo, probablemente no tenga sentido mostrar un `Tooltip` durante el renderizado inicial. Se desencadena por una interacción con el cliente.