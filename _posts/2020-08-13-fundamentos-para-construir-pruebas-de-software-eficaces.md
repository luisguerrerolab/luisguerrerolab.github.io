---
layout     : post
title      : "Fundamentos Para Construir Pruebas de Software Eficaces"
date       : 2020-08-13 21:00:00 -0500
tags:
- testing
---

En este artículo quiero compartir las lecciones que aprendí construyendo el set de pruebas para el proyecto más reciente en el que trabajé.

## Escribir pruebas es una habilidad

A diferencia del código que forma parte de una aplicación dirigida a un usuario final, las pruebas que validan su comportamiento sólo son conocidas por el equipo de ingenieros que las crearon. **Es código que siempre vive detrás del escenario**. Su trabajo es asegurarse de que los actores principales (los módulos) den un espectáculo de primera en cada entrega.

**Ese aparente papel secundario de las pruebas puede provocar que relajemos los criterios de calidad en su construcción**, en contraste con la fuerte devoción con que seguimos las buenas prácticas de programación para construir la propia aplicación.

Además, durante situaciones de estrés, como bomberazos o cuando descubrimos un error crítico, ajustar las pruebas existentes o crear nuevas pueden parecer tareas fastidiosas que se interponen entre el desarrollador y el código liberado en producción.

Pero así como aprendimos a programar, **escribir pruebas también es una habilidad que podemos cultivar y mejorar cuanto más la practiquemos**. 

## Las pruebas son un lienzo de práctica

Las primeras pruebas para una aplicación suelen construirse usando el *patrón Copy/Paste*. Y eso está bien. La primera misión de una prueba es verificar que el sistema funciona. El refinamiento viene después, cuando el volumen de pruebas comienza a ser considerable. Pero según avanza el desarrollo, evitar código duplicado a veces no es tan simple como centralizar en una función todos los bloques de código repetidos por toda la aplicación.

Cuando la arquitectura del proyecto alcanza un cierto nivel de complejidad, debemos considerar estrategias de diseño software más sofisticadas. Por ejemplo: el Principio de Inversión de Dependencias es súper útil en las pruebas, para ayudarnos a acceder a versiones *ligeras* de una implementación compleja que vive en producción.

En ese sentido, **cada prueba es un lienzo donde podemos practicar nuestro oficio**.

Cada prueba es una oportunidad para simplificar la lógica de nuestra aplicación y reducir el código. Menos líneas, menos problemas para resolver.

Desde mi punto de vista, los dos principios más importantes que pueden servir como guía para cualquier proyecto de pruebas son DRY y SRP.

## Principio DRY

Don't Repeat Yourself. No te repitas.

### Pasar valores como variables

Pasar valores como cadenas directamente a funciones puede causar confusión al lector de las pruebas. Si una cadena parece fuera de lo ordinario, el lector se cuestionará por qué se eligió cierto valor para un parámetro o valor de retorno. Un mejor acercamiento es asignar esos valores a una variable que haga explícito su significado.

Evita valores mágicos.

Podría mejorar:

```csharp
var request = new ItemRequest
{
    Id = Guid.NewGuid(),
    Name = new string('a', 251), // ¡Ah, caray¡ ¿Por qué, 251? ¿Por qué? ¡Me esstoy perdiendo algo!
    Description = new string('b', 251),
};
```

Mejor:

```csharp
var lengthGreaterThanAllowed = 251; // Ahora queda claro a simple vista qué representa ese 251.
var request new ItemRequest
{
    Id = Guid.NewGuid(),
    Name = new string('a', lengthGreaterThanAllowed)
    Description = new string('b', lengthGreaterThanAllowed),
};
```

### Crear clases de ayuda para objetos reutilizables

El primer set de pruebas de una aplicación suele contener mucho código repetido. Yo considero que es normal y está bien. Cuando empezamos un proyecto completamente desde cero, nuestro objetivo debe ser asegurar que la funcionalidad principal esté respaldada por un set mínimo de pruebas. En esta etapa es muy probable que la lógica cambie con mucha frecuencia. **Para que el set de pruebas pueda mantener el ritmo de cambio de la aplicación tenemos que ceder un poco en la claridad de su código para ganar más rapidez en su desarrollo.**

Sin embargo, conforme la aplicación se acerca a una versión estable, **la primera refactorización que podemos aplicar al set de pruebas es centralizar todos los objetos y propiedades reutilizables en una clase de ayuda**, dándoles un nombre suficientemente explícito para comunicar su función.

Podría mejorar:

```csharp
public async Task AddItem_Success_ReturnsAddedItemId()
{
    // arrange
    // La instancia de un objeto es creada dentro de la función de prueba
        var newItem = new Item
        {
            Id = Guid.NewGuid(),
            Name = GetUniqueValue("Prefix"),
            Description = "New item"
        };
    
    // act
    var result = await _systemUnderTest.AddItem(newItem);

    // assert
    Assert.IsType<Guid>(result);
}
```

Mejor:

```csharp
public async Task AddItem_Success_ReturnsAddedItemId()
{
    // arrange
    // La instancia de Item es creada en algún otro lugar. Aquí sólo mandamos llamar a la clase de ayuda que genera esa instancia por nosotros.
    var newItem = TestUtils.GetValidItemInstance();
    
    // act
    var result = await _systemUnderTest.AddItem(newItem);

    // assert
    Assert.IsType<Guid>(result);
}
```

## Principio SRP

Single Responsibility Principle. Principio de Única Responsabilidad.

### Fábricas de dependencias

Las fábricas de dependencias son un gran ejemplo de la aplicación de SRP. Representan bloques de código que pueden acoplarse a múltiples sets de pruebas. Realizan una función específica y son mantenidos de forma independiente. Se vuelven necesarias cuando la aplicación está compuesta de múltiples capas de código.

Una API REST es un tipo de aplicación que puede estar formada por diferentes capas de código. Para construir un proyecto robusto como éste, eventualmente tendremos que escribir **pruebas de integración**. Ellas verifican que la interacción entre esas múltiples capas devuelva un resultado esperado.

Para crear las pruebas de integración de una API REST, nuestro set debe tener dos habilidades básicas:

- Realizar peticiones a través del protocolo HTTP.
- Obtener la respuesta de cada petición, para poder manipular su contenido dentro de la prueba.

La siguiente prueba aplica ambas habilidades en una forma simple:

```csharp
[Fact]
public void Add_Success_ReturnsCreatedCode()
{
    // arrange
    var itemRequest = TestUtils.GetValidItemRequest();

    // act
    var response = await _client.PostAsync("api/items", itemRequest);
    var stringContent = await response.Content.ReadAsStringAsync();
    var data = JsonConvert.DeserializeObject<<ItemResponse>(stringContent);

    // assert
    Assert.Equal((int)HttpStatusCode.Created, response.StatusCode);
    Assert.Equal(itemRequest.Name, data.Name);
}
```

En el bloque anterior:

- El objeto `itemRequest` fue creado usando la clase de ayuda `TestUtils`.
- Usa la función `PostAsync` para hacer la petición HTTP.
- Convierte la respuesta obtenida en un objeto de tipo `string` para después hacer otra conversión al tipo concreto `ItemResponse`.

Cuando el set de pruebas de integración para toda la aplicación es reducido (10 pruebas o menos), la implementación anterior puede ser suficientemente buena. Pero si esperamos que el volumen de pruebas crezca, podemos aplicar SRP desde las etapas tempranas para facilitarnos el desarrollo y mantenimiento en el futuro cercano.

En realidad, realizar peticiones por HTTP y obtener las respuestas no debe ser el fin, sino el medio de una prueba para obtener la información necesaria que nos ayude a comprobar si el resultado de una petición fue el esperado. Así que podemos pensar en delegar esas responsabilidades a alguien más. A una fábrica de dependencias, por ejemplo.

Creamos una fábrica que realiza peticiones HTTP y la llamamos `ApiClient`:

```csharp
[Fact]
public void Add_Success_ReturnsCreatedCode()
{
    // arrange
    var itemRequest = TestUtils.GetValidItemRequest();

    // act
    var created = await ApiClient.MakeRequestAsync<ItemResponse>(HttpMethod.Post, "api/items", itemRequest);

    // assert
	Assert.Equal(HttpStatusCode.Created, created.StatusCode);
    Assert.Equal(itemRequest.Name, created.Data.Name);
}
```

En el bloque anterior podemos notar varias mejoras:

- La petición HTTP se realiza a través de la función `MakeRequestAsync`.
- En la misma llamada se especifica el valor de retorno esperado, un objeto de tipo `ItemResponse`.
- La variable `created` contiene tanto el código de status HTTP como la información obtenida de la respuesta.

La fábrica `ApliClient` le dio a esta prueba elegancia y simplicidad. Ahora el código es mucho más fácil de leer por nuestros colegas. Además, separar en una clase las responsabilidades para consumir nuestra API convierte a este set de pruebas en un proyecto fácilmente adaptable. Si en un futuro decidimos cambiar la implementación para realizar peticiones HTTP, haríamos el trabajo una sola vez, dentro de la función `MakeRequestAsync`, en lugar de cambiar cada una de las llamadas a `PostAsync` que estarían dispersas por todo el set de pruebas.

## Las pruebas también son protagonistas

**El primer cliente en la historia de nuestra aplicación es el primer set de pruebas que construimos para verificar su funcionamiento.** Si mantenemos la cadencia de crear una prueba para cada nueva función, el set toma un rol principal: es el forjador que ayuda a determinar el diseño y a moldear la interface con la que interactúan entre sí múltiples partes de un sistema.

## Simplificación progresiva

Al momento de leer esta guía, es probable que nuestro proyecto ya tenga un volumen considerable de pruebas. Algunas verifican el comportamiento de cada módulo aislando sus dependencias (unitarias), mientras otras verifican el comportamiento de toda la aplicación mientras completa una tarea (de integración). Considerando este escenario, puede sentirse abrumador siquiera considerar la idea de aplicar todas las guías mencionadas en este artículo.

**Cambia la perspectiva.** En lugar de reestructurar el set de pruebas actual, piensa en que la próxima prueba que escribas puede ser el punto de partida para:

- Diseñar la arquitectura para almacenar clases de ayuda en un folder o proyecto dedicado.
- Declarar en clases separadas las fábricas de dependencias que utilizará la nueva prueba.
- Refactorizar las áreas de la aplicación que son verificadas por la prueba.


Escribe código de pruebas tan legible como sea posible, pensando en nuestros futuros colegas o lectores del código.

## Para continuar hablando de pruebas

- Qué áreas/capas/lógica probar y cuándo no escribir pruebas.
- Lenguaje apropiado en el mundo de las pruebas.
- Cómo simular las dependencias de una implementación compleja.

## Referencias

- [Unit testing best practices with .NET Core and .NET Standard](https://docs.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices)
- [Unit test controller logic in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/testing?view=aspnetcore-3.1)
