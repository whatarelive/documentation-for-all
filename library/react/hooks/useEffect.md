## 1- ¿Qué hace el hook useEffect? :

>[!NOTE] useEffect: 
>Es un React Hook que te permite sincronizar un componente con un sistema externo.

```ts
useEffect(setup, dependencies?)
```

### 2- Implementación :
Llame al hook `useEffect` en el nivel superior de su componente para declarar un efecto:

```ts
import { useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {  
const [serverUrl, setServerUrl] = useState('https://localhost:1234');

useEffect(() => { 
	const connection = createConnection(serverUrl, roomId);
	connection.connect();    

	// Función de limpieza 	
	return () => connection.disconnect();  	
}, [serverUrl, roomId]); 
```

#### Parámetros :

>[!IMPORTANT] setup (función de configuración):
> - Es la función con la lógica de tu efecto, esta puede devolver opcionalmente una función _de limpieza_ . Cuando su componente se agregue al DOM, React ejecutará su función de configuración. 
> - Después de cada renderizado con dependencias modificadas, React primero ejecutará la función de limpieza (si la proporcionó) con los valores anteriores y luego ejecutará su función de configuración con los nuevos valores. 
> - Después de que su componente se elimine del DOM, React ejecutará su función de limpieza.

>[!IMPORTANT] dependencies (Arreglo de dependencias): 
>Es la lista opcional de todos los valores reactivos a los que se hace referencia dentro del código `setup`(función de configuración).
>- Los `valores reactivos` incluyen accesorios, estado y todas las variables y funciones declaradas directamente dentro del cuerpo del componente. 
>- La lista de dependencias debe tener un número constante de elementos y estar escrita en línea como `[dep1, dep2, dep3]`. 
>- React comparará cada dependencia con su valor anterior usando la comparación. 
>- Si omite este argumento, su efecto se volverá a ejecutar después de cada repetición del componente.

#### Devoluciones :

>[!CAUTION]
>El useEffect tiene como devoluciones un valor undefined.

#### Advertencias :

>[!CAUTION] Usos del hook 
>El `useEffect` es un Hook, por lo que solo puedes llamarlo **en el nivel superior de tu componente** o en tus propios Hooks. No puedes llamarlo bucles internos o condiciones. Si lo necesita, extraiga un nuevo componente y mueva el estado a él.

>[!IMPORTANT] Sistema externo es cualquier fragmento de código que no esté controlado por React :
>
>- Un temporizador gestionado con `setInterval()` y `clearInterval()` .
>- Una suscripción a un evento usando `window.addEventListener()` y `window.removeEventListener()`.
>- Una biblioteca de animación de terceros con una API como `animation.start()`y `animation.reset()`.

>[!WARNING] Trabajo en Modo estricto
>Cuando el modo estricto está activado, React ejecutará **un ciclo adicional de configuración y limpieza de solo desarrollo** antes de la primera configuración real. Esta es una prueba de esfuerzo que garantiza que su lógica de limpieza "refleje" su lógica de configuración y que detenga o deshaga cualquier cosa que esté haciendo la configuración. 

>[!CAUTION] 
>Si algunas de sus dependencias son objetos o funciones definidas dentro del componente, existe el riesgo de que hagan que **el efecto se vuelva a ejecutar con más frecuencia de la necesaria.** Para solucionar este problema, elimine las dependencias innecesarias de objetos y funciones. También puede extraer actualizaciones de estado y lógica no reactiva fuera de su Efecto.

>[!NOTE]  
>Si su efecto no fue causado por una interacción (como un clic), React generalmente permitirá que el navegador **pinte la pantalla actualizada primero antes de ejecutar su efecto.** 

>[!NOTE] 
>Si su efecto hace algo visual (por ejemplo, colocar una información sobre herramientas) y el retraso es notable (por ejemplo, parpadea), reempláce `useEffect` con `useLayoutEffect`.

>[!NOTE] 
>Los efectos **sólo se ejecutan en el cliente.** No se ejecutan durante el procesamiento del servidor.



### 3- Usos : 

#### > Conexión a un sistema externo :

`Conexion Sistema Externo :` Algunos componentes deben permanecer conectados a la red, a alguna API del navegador o a una biblioteca de terceros, mientras se muestran en la página. Estos sistemas no están controlados por React, por eso se denominan _externos._

Para conectar su componente a algún sistema externo, llame `useEffect`al nivel superior de su componente :

```ts
import { useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {  
	const [serverUrl, setServerUrl] = useState('https://localhost:1234');
	
	useEffect(() => {  	
		const connection = createConnection(serverUrl, roomId);
		connection.connect();  	
		
		return () => connection.disconnect();   
	}, [serverUrl, roomId]); 
}
```

**Debe pasar dos argumentos a `useEffect`:**
1. Una _función de configuración_ con código de configuración que se conecta a ese sistema.
    - Debería devolver una _función de limpieza_ con un código de limpieza que se desconecte de ese sistema.
2. Una lista de dependencias que incluye todos los valores de su componente utilizados dentro de esas funciones.

**React llama a tus funciones de configuración y limpieza cuando es necesario, lo que puede suceder varias veces:**
1. Su código de configuración se ejecuta cuando su componente se agrega a la página.
2. Después de cada repetición de su componente donde las dependencias han cambiado:
    - Primero, su código de limpieza se ejecuta con los accesorios y el estado antiguos.
    - Luego, su código de configuración se ejecuta con los nuevos accesorios y estado.
3. Su código de limpieza se ejecuta por última vez después de que su componente se elimina de la página.

#### > Peticiones Http con efectos : 

Puede utilizar un efecto para recuperar datos para su componente. Tenga en cuenta que si utiliza un marco, utilizar el mecanismo de obtención de datos de su marco será mucho más eficiente que escribir efectos manualmente.

Si desea recuperar datos de un efecto manualmente, su código podría verse así:

```ts
import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {    
    const [bio, setBio] = useState(null);  

    useEffect(() => {    
        let ignore = false;    
        setBio(null);    
        
        fetchBio(person).then(result => {      
            if (!ignore) setBio(result);      
        });    

        return () => ignore = true;      
    }, [person]);
```

>[!CAUTION] 
Tenga en cuenta la `ignore`variable que se inicializa `false`y se establece `true`durante la limpieza. Esto garantiza que su código no sufra "condiciones de carreras" las respuestas de la red pueden llegar en un orden diferente al que usted las envió.