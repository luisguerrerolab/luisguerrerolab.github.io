---
layout:   post
title:    "Escribir código de pruebas más rápido usando snippets en Visual Studio Code"
date:    2020-08-16 15:00:00 -0500
tags:
- xunit
- vscode
---

Hace poco escribí el set de pruebas unitarias para la primera versión de un proyecto Web API, usando el framework xUnit. Un endpoint podría presentar diferentes escenarios y quería escribir una prueba para cada uno de ellos. Mi cabeza estaba muy enfocada, pero mis manos difícilmente le podían llevar el ritmo, porque tenía que recrear la estructura de un método de prueba una y otra vez.

En este artículo compartiré cómo configurar Visual Studio Code para ayudarte a ser más eficiente escribiendo código de pruebas con xUnit, usando un snippet que automáticamente imprime la plantilla de código para un bloque `[Fact]`, en lugar de escribirlo todo manualmente.

La plantilla insertada sigue dos convenciones para escribir pruebas:

1. El **nombre de una prueba** debe estar formado por tres partes:
- **Nombre** del método a probar.
- El **escenario** específico que será probado.
- El **resultado** esperado.

1. El **contenido de una prueba** debe ser organizado siguiendo el patrón *Preparar, Actuar, Verificar*:
- Primero, **prepara** los objetos que serán usados dentro de la prueba.
- Después, **actúa** sobre el sistema bajo pruebas (system under test).
- Finalmente, **verifica** que esas acciones entregaron el resultado esperado.

## Instrucciones

> ℹ️ Los siguientes pasos fueron probados en MacOS, con Visual Studio Code en inglés. Pero estoy seguro de que tu **sofisticación técnica** te llevará a averiguar cómo replicarlos en tu sistema operativo e idioma particulares.

1. Abre la **Command Palette** tecleando `⇧⌘P`.
2. En la barra de texto, escribe **Configure User Snippets**.
3. Elige el lenguaje de programación para el snippet. Dado que ésta es una plantilla para xUnit, la opción debe ser `csharp`.
4. Si no has creado otro snippet para este lenguaje, reemplaza el contenido del archivo `csharp.json` con lo siguiente:
```json
{
	// Place your snippets for csharp here. Each snippet is defined under a snippet name and has a prefix, body and 
	// description. The prefix is what is used to trigger the snippet and the body will be expanded and inserted. Possible variables are:
	// $1, $2 for tab stops, $0 for the final cursor position, and ${1:label}, ${2:another} for placeholders. Placeholders with the 
	// same ids are connected.
	// Example:
	// "Print to console": {
	// 	"prefix": "log",
	// 	"body": [
	// 		"console.log('$1');",
	// 		"$2"
	// 	],
	// 	"description": "Log output to console"
	// }
    "New Xunit fact": {
        "prefix": "fact",
        "body": [
            "[Fact] public void ${1:Method}_${2:Scenario}_${3:Result}()\n{\n\t// arrange\n\t$4\n\n\t// act\n\t$5\n\n\t// assert\n\t$6\n}"
        ],
        "description": "Add Xunit [Fact] method"
    }
}
```

Si tienes otros snippets para C#, entonces sólo agrega el bloque de código de las líneas 15 a 21. 

Eso es todo.

Para probarlo, abre una de tus clases de pruebas (en realidad, puede ser cualquier archivo con código en C#) y escribe la palabra `fact`. Verás que visual Studio Code mostrará un montón de sugerencias para autocompletar, con el snippet que recién agregamos como primera opción:


{% include video-player.html url="/assets/video/vscode-snippet-fact-template.mov" width=708 height=286 %}


Ahora empieza la diversión. Arriba mencioné que este snippet sigue una convención específica para nombrar una prueba, que consta de 3 partes. En cuanto el snippet se inserta en el editor, puedes cambiar los valores para `Method`, `Scenario` y `Result` posando el cursor sobre ellos sólo presionando la tecla `Tab`. Lo mismo aplica para las secciones `arrange`, `act` y `assert`.

Pruébalo y diviértete.

> 📄 Una de las extensiones que instalé en Visual Studio Code **para** C# ya contiene un snippet para un bloque `[Fact]` de xUnit. Yo no uso ese formato particular, por eso es que decidí crear un snippet que se adapte mejor a mis necesidades.

## Enlaces de Recursos
- [Snippets en Visual Studio Code](https://code.visualstudio.com/docs/editor/userdefinedsnippets)
- [Unit Testing Best Practices - Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices)