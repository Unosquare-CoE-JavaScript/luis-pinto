## Sincrono vs Asincrono
### Codigo Sincrono
- VENTAJA: Facil de escribir y entender
- DESVENTAJA: Puede crear codigo bloqueante
- DESVENTAJA: Menos performance
### Codigo Asincrono
- VENTAJA: Bueno en performance
- DESVENTAJA: Elimina codigo bloqueante
- DESVENTAJA: Puede ser dificil de entener
- DESVENTAJA: Dificil de escribir

## Event Loop
El proposito de event loop es asegurarse que todo el codigo es manejado, Javascript es de un solo hilo y solo podemos ejecutar una pieza de codigo a al vez,pero el event loop hace posible la sincronizidad porq somos posible de dejar el codigo a un lado como el callback que fue creado con el *settimeout*
## Callbacks
Es un patron que nos permite pasar una funcion a otra funcion y ser ejecutada hasta que algo ocurra
## Problemcas con Callbacks
- Callback hell
- Dificultad to entender
- Inversion de control
## Ques una Promesa en Javascript
- Un objeto con Propiedades y Metodos
- Representa uuna finalizacion eventual( o falla) de una operacion asincrona
- Provee una valor resultante
## Async await
Asyn es usado como parte de una definicio de funcion y fuerza a la funcion a retornar una promesa y si la funcion retorna un valor, ese valor es envuelto en la promesa, si ningun valor es retornado aun la funcion retorna una promesa
## Generators
Basicamente un generador es una funcion que puede pausarse y continuar luego
Setear un generador envuelve 2 partes:
- Definir la funcion como generator con el uso del asterisco
- Requerir el uso de la palabra reservada yield
