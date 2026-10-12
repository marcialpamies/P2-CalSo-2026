# Práctica 2. Pruebas unitarias y análisis de calidad

**Calidad del Software · Curso 2026-2027**

## Descripción del caso práctico

Una empresa está desarrollando el software de un robot que ejecuta distintos programas de limpieza. Cada programa requiere una cantidad fija de energía y puede necesitar agua, jabón, cera o limpiacristales. El robot consulta las existencias antes de empezar: si falta algún producto, informa de las unidades necesarias y no consume nada; si hay suficiente de todos, realiza los descuentos. En esta práctica la energía identifica el consumo del programa, pero no se gestiona una batería.

Recibiréis un proyecto Java que compila y contiene defectos de comportamiento. Vuestro trabajo consiste en **descubrirlos mediante pruebas JUnit 5 diseñadas a partir de la especificación**, corregirlos y analizar los resultados de calidad obtenidos al terminar. No se facilita la cantidad ni la ubicación de los defectos.

Trabajaréis principalmente en **Eclipse**. El proyecto utiliza Maven y una organización basada en dominio, aplicación y adaptadores. La especificación que determina el comportamiento correcto está en `docs/specification.md`. **El código entregado no es la especificación**: precisamente puede contener errores.

### Bounded context y alcance del slice

El siguiente diagrama presenta los conceptos principales de **Robot Cleaning**, sus relaciones y su distribución entre las capas **Adapters**, **Application** y **Domain**. Permite distinguir el código comprendido en el slice de esta práctica de la funcionalidad que queda fuera de su desarrollo.

```mermaid
flowchart TB
    subgraph AD["ADAPTERS"]
        direction LR
        SERVICE["CleaningRobotService<br/>Adaptador de entrada"]
        STUB["StubCleaningInventory<br/>Suplantador para pruebas"]
        REAL["Inventario real<br/>Infraestructura no implementada"]
    end

    subgraph AP["APPLICATION"]
        direction LR
        PORT["IExecuteCleaningProgram<br/>Puerto de entrada"]
        UC["ExecuteCleaningProgramUseCase<br/>Caso de uso"]
        COMP["CleaningProgramByEnergyComparator<br/>Comparador"]
    end

    subgraph DOM["DOMAIN"]
        direction TB
        subgraph CORE["Modelo de dominio"]
            direction LR
            PROGRAM["CleaningProgram<br/>Entidad: id, name, energy, requirements"]
            RESULT["ExecutionResult<br/>executed, missingProducts"]
            INVENTORY["ICleaningInventory<br/>Puerto de salida del dominio"]
        end
        subgraph VALUES["Conceptos de valor"]
            direction LR
            ENERGY["Energy<br/>Objeto de valor"]
            PRODUCT["CleaningProduct<br/>WATER · SOAP · WAX · GLASS_CLEANER"]
        end
    end

    SERVICE --> PORT
    PORT --> UC
    UC --> PROGRAM
    UC --> RESULT
    UC --> INVENTORY
    COMP --> PROGRAM
    PROGRAM --> ENERGY
    PROGRAM --> PRODUCT
    RESULT --> PRODUCT
    STUB -.-> INVENTORY
    REAL -.-> INVENTORY

    classDef included fill:#E8F5EC,stroke:#248447,stroke-width:2px,color:#183B29
    classDef excluded fill:#FDEBEC,stroke:#CE3548,stroke-width:2px,color:#7D1F2C
    class SERVICE,STUB,PORT,UC,COMP,PROGRAM,RESULT,INVENTORY,ENERGY,PRODUCT included
    class REAL excluded

    style AD fill:#F7FAFD,stroke:#B8CADB,stroke-width:1px
    style AP fill:#F7FAFD,stroke:#B8CADB,stroke-width:1px
    style DOM fill:#F7FAFD,stroke:#B8CADB,stroke-width:1px
    style CORE fill:#FFFFFF,stroke:#D9E3EB
    style VALUES fill:#FFFFFF,stroke:#D9E3EB
```

**Cómo interpretarlo:** los elementos **verdes** pertenecen al slice y los **rojos** están fuera de su alcance o no se implementan en esta práctica. La interfaz `ICleaningInventory` está situada en la **capa de dominio**, aunque sea utilizada por el caso de uso de aplicación. El inventario real no se implementa: en las pruebas del caso de uso se desarrolla `StubCleaningInventory` dentro de `src/test/java`, para controlar las existencias sin depender de infraestructura real.

El diagrama es una visión arquitectónica, no sustituye la [especificación de comportamiento](docs/specification.md), que es la referencia para diseñar los casos de prueba.

## Estructura del repositorio de GitHub

Cada grupo dispone de un único repositorio para todas las prácticas. Dentro de él se encuentra el proyecto Maven de la segunda práctica:

```text
Repositorio-del-grupo/
├── .github/
│   └── workflows/
│       └── p2-quality.yml
├── P1/
└── P2/
    ├── README_P2.md
    ├── docs/
    └── P2_INICIALES/ (Cambiar las iniciales por las que correspondan)
        ├── pom.xml
        └── src/
```

Para conectar las acciones de GitHub con la herramienta externa configuraremos un flujo de trabajo. El workflow se guardará en **`.github/workflows/` de la raíz del repositorio**, no dentro de la carpeta `P2/` ni del proyecto Java. La ruta exacta al proyecto Maven será `P2/P2_INICIALES/`.

LA estructura del archivo `.yml` será la siguiente, modificando las rutas pertinentes:

```yml
name: P2 - Analisis SonarQube
on:
  push:
    branches: [main]
    paths:
      - 'P2/P2_INICIALES/**'
      - '.github/workflows/p2-quality.yml'
  workflow_dispatch:

permissions:
  contents: read

jobs:
  quality:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: P2/P"_INICIALES
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Configurar Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '21'
          cache: maven
          cache-dependency-path: P2/P2_INICIALES/pom.xml
      - name: Pruebas JUnit e informe JaCoCo
        run: mvn -B clean verify
      - name: Comprobar configuracion
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ vars.SONAR_HOST_URL }}
          SONAR_PROJECT_KEY: ${{ vars.SONAR_PROJECT_KEY }}
        run: |
          test -n "$SONAR_TOKEN" || { echo 'Falta SONAR_TOKEN'; exit 1; }
          test -n "$SONAR_HOST_URL" || { echo 'Falta SONAR_HOST_URL'; exit 1; }
          test -n "$SONAR_PROJECT_KEY" || { echo 'Falta SONAR_PROJECT_KEY'; exit 1; }
      - name: Publicar analisis en SonarQube
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ vars.SONAR_HOST_URL }}
          SONAR_PROJECT_KEY: ${{ vars.SONAR_PROJECT_KEY }}
        run: >
          mvn -B org.sonarsource.scanner.maven:sonar-maven-plugin:5.1.0.4751:sonar
          -Dsonar.projectKey="$SONAR_PROJECT_KEY"
          -Dsonar.host.url="$SONAR_HOST_URL"
          -Dsonar.token="$SONAR_TOKEN"
          -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
```

En grupos de dos, uno de los integrantes puede configurar los permisos y secretos, siempre que disponga de autorización en GitHub. No hace falta que ambos creen tokens ni duplicar la configuración.

## 1. Preparación y pruebas en Eclipse

Importad `robot-cleaning/` mediante **File → Import → Maven → Existing Maven Projects**. Comprobad que Eclipse reconoce las fuentes de producción y las de pruebas. Leed primero la especificación y diseñad casos con datos normales, valores límite, situaciones inválidas y comportamientos de igualdad u ordenación, según corresponda.

Consultad también la documentación oficial de **JUnit 5**, especialmente Jupiter, las aserciones y la ejecución de pruebas: [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/). No es necesario reproducir el manual en el informe, pero sí aplicar correctamente las herramientas del framework.

Como guía rápida para la creación de pruebas con jUnit 5 puedes consultar el documento: [Crear y ejecutar test](docs/Crear_y_ejecutar_tests.md). Además, se puede consultar la guía rápida de jUnit del curso pasado: [Guía jUnit del curso 2026](https://github.com/marcialpamies/P2-CalSo/blob/main/docs/manual_JUnit5.md).

### Localización de las pruebas asociadas a una clase concreta

Las clases de jUnit 5 que contengan los casos de prueba de una clase concreta `ClaseDePrueba.java`, localizada en el paquete `paquete.de.clase` de la carpeta `src` deberán localizarse en el mismo paquete `paquete.de.clase` pero en la carpeta `test` y se nombrarán siguiendo el convenio `testClaseDePrueba.java`. 

Por ejemplo, las pruebas de la clase siguiente que se encuentran en la carpeta del proyecto `src/main/java` en el paquete `com.calso.robot.domain.valueobjects`

```java
package com.calso.robot.domain.valueobjects;

public final class Energy {
...
}
```
Deben localizarse en la clase jUnit 5 siguiente, que se localizará en la carpeta `src/test/java` y en el mismo paquete `com.calso.robot.domain.valueobjects`:

```java
package com.calso.robot.domain.valueobjects;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class EnergyTest {
...
}
```
De esta forma las pruebas quedan alineadas con la estructura de la arquitectura seleccionada.


### Consulta rápida de JUnit 5

Una prueba escrita con Jupiter suele utilizar `@Test`. Cada método prepara sus datos, ejecuta una operación y comprueba el resultado esperado. Por ejemplo:

```java
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;

import org.junit.jupiter.api.Test;

class EnergyTest {

    @Test
    void testGivenEnergy10WhenCreatingEnergyThenStoresValue10() {
        Energy energy = Energy.of(10);
        assertEquals(10, energy.value());
    }

    @Test
    void testGivenEnergyMinus1WhenCreatingEnergyThenThrowsIllegalArgumentException() {
        assertThrows(IllegalArgumentException.class,
                () -> Energy.of(-1));
    }

    @Test
    void testGivenTwoEnergyValues10WhenComparingThenTheyAreEqual() {
        Energy first = Energy.of(10);
        Energy second = Energy.of(10);
        assertEquals(first, second);
    }
}
```

### Convención obligatoria y precisión de los nombres de las pruebas

Todos los métodos JUnit 5, incluidos los parametrizados, tendrán nombres en **inglés**, en formato **camelCase**, con esta estructura:

```text
testGiven<Condición>When<Acción>Then<ResultadoEsperado>
```

Las partes `testGiven`, `When` y `Then` son obligatorias; cada una irá seguida de una descripción que comience por mayúscula. `And` es opcional y permite añadir condiciones o resultados relacionados.

**El nombre debe identificar el escenario concreto**, no solo la regla general que se está comprobando. Incluid los valores y condiciones que permiten distinguir el caso de otros similares, especialmente límites, existencias, cantidades requeridas y diferencias entre objetos. No es necesario reflejar todos los datos auxiliares. El nombre debe corresponderse con las entradas y las aserciones reales.

Por ejemplo, es demasiado genérico `testGivenInsufficientStockWhenExecutingProgramThenReportsMissingProducts`. Un nombre más preciso sería:

```java
@Test
void testGivenWater2AndSoap0WhenExecutingProgramRequiringWater2AndSoap1ThenReportsSoap1MissingAndPreservesStock() {
    // ...
}
```

Otros escenarios específicos:

```java
@Test
void testGivenEnergy0WhenCreatingEnergyThenThrowsIllegalArgumentException() {
    // ...
}

@Test
void testGivenWater2WhenExecutingProgramRequiringWater2ThenWaterStockBecomes0() {
    // ...
}
```

Los nombres `testExecuteProgram()`, `givenStockWhenExecutingThenSucceeds()` y `testGivenStockWhenExecuting()` incumplen la estructura. Un nombre que respeta esa estructura también puede ser insuficiente si no distingue el caso concreto.

**Control en SonarQube.** El Quality Profile Java **CalSo - Java Docencia** incorpora la regla `java:S3578`, que comprueba la estructura de nombres. Su incumplimiento genera incidencias y **hace fallar el Quality Gate configurado para las prácticas CalSo**. El profesorado comprobará además la correspondencia entre el nombre, los datos y las aserciones. Usad nombres precisos desde la primera prueba.

El fallo del Quality Gate no significa necesariamente que la Action de GitHub falle: son controles distintos.

Estos ejemplos son orientativos: debéis comprobar las firmas y los contratos reales del proyecto. En particular, los métodos y resultados esperados de cada prueba deben deducirse de la especificación.

| Aserción | Uso |
| --- | --- |
| `assertEquals(expected, actual)` | Comprueba igualdad |
| `assertNotEquals(unexpected, actual)` | Comprueba desigualdad |
| `assertTrue(condition)` / `assertFalse(condition)` | Comprueba una condición |
| `assertNull(value)` / `assertNotNull(value)` | Comprueba valores nulos |
| `assertThrows(Type.class, executable)` | Comprueba una excepción |
| `assertDoesNotThrow(executable)` | Comprueba que no se lanza una excepción |
| `assertAll(...)` | Agrupa comprobaciones relacionadas |

Si el contrato especifica un mensaje de excepción, comprobadlo también. Usad `@BeforeEach` únicamente para una preparación común, independiente y sencilla. No compartáis estado mutable entre pruebas.

Para comparar colecciones, elegid una aserción acorde con el contrato: `assertIterableEquals` cuando el orden sea relevante, `assertEquals` cuando la colección proporcione la igualdad necesaria o una comprobación específica cuando únicamente importe la pertenencia.

Escribid pruebas JUnit 5 para `Energy`, `CleaningProduct`, `CleaningProgram` y `CleaningProgramByEnergyComparator`. Sus nombres deben describir la condición y el resultado esperado, por ejemplo `testGivenDifferentEnergiesWhenComparingProgramsThenSortsAscending`. Las pruebas serán **deterministas, independientes y lo más atómicas posible**: no deberán depender del orden de ejecución ni de datos creados por otra prueba. Evitad reunir comprobaciones no relacionadas en un mismo método, pues esto dificulta localizar los fallos.

Ejecutad las pruebas con **Run As → JUnit Test** desde Eclipse o con `mvn test`. Para la verificación completa podéis utilizar `mvn clean verify`. Cuando una prueba falle, comprobad primero que refleja la especificación. Si es correcta, localizad el defecto, corregidlo y ejecutadla de nuevo. Conservad cada prueba útil en la batería final.

No se valora crear muchas pruebas parecidas, sino comprobar comportamientos diferentes y justificar las elecciones.

## 2. Probar la aplicación cuando falta una dependencia

El inventario real todavía no está desarrollado. El caso de uso `ExecuteCleaningProgramUseCase` conoce únicamente la interfaz `ICleaningInventory`, de modo que puede probarse sin esperar a que exista ese módulo.

Implementad **vosotros mismos** `StubCleaningInventory` a partir del esqueleto incluido en `src/test/java/com/calso/robot/application/usecases/`. Debe aceptar unas existencias iniciales, devolver cero para productos que no estén registrados y descontar cantidades válidas sin dejar valores negativos. Los intentos de descontar cantidades negativas o superiores a las disponibles deben producir la excepción indicada en el contrato sin alterar el inventario. Cada instancia debe conservar su propio estado.

Usad ese suplantador para diseñar pruebas del caso de uso: ejecución permitida, existencias exactas, falta de uno o varios productos, ausencia de cambios si la ejecución se rechaza y programas que no requieren productos. Verificad tanto el resultado devuelto como las existencias finales. El suplantador **forma parte del código de pruebas**, no de producción. No se piden contadores de llamadas ni comprobaciones de interacciones internas.

En el informe explicad brevemente qué dependencia estáis sustituyendo, por qué el uso de una interfaz lo permite y cómo ayuda a probar una parte del sistema antes de completar el resto.

## 3. Trabajo individual, integración y análisis

La asignatura utiliza **un repositorio por grupo**, con una carpeta para cada práctica. Incorporad esta actividad dentro de `P2/`, manteniendo `README_P2.md`, el proyecto Maven y la carpeta `evidencias/`.

**Parejas.** Cada integrante crea desde `main` una rama propia y se responsabiliza de una parte del proyecto. Se propone repartir *dominio y comparador* frente a *suplantador y caso de uso*; podéis acordar otro reparto de carga semejante y dejarlo escrito antes de empezar. Desarrollad y realizad commits en vuestra rama hasta completar el bloque. A continuación, abrid una **Pull Request** hacia `main`. El compañero revisará los cambios y, después, integraréis ambas ramas.

Conservad un historial que permita reconocer la autoría y evolución de cada aportación. Utilizad commits pequeños y coherentes, con mensajes descriptivos, realizados por quien implementa cada cambio. Por ejemplo, separad los commits de pruebas, correcciones y documentación. En la Pull Request, describid los cambios, enlazad las pruebas relevantes y revisad el código del compañero antes de aprobarla. Si hay conflictos, resolvedlos conjuntamente y comprobad después que la suite completa sigue pasando. No trabajéis los dos directamente sobre la misma rama ni compartáis credenciales. Los resultados finales deben corresponder a la versión integrada en `main`.

**Grupos individuales.** Realizad todos los cambios y commits directamente en `main`; no se exigen ramas ni Pull Requests. Mantened commits descriptivos que permitan relacionar las pruebas con los defectos detectados y sus correcciones.

Al finalizar, ejecutad la suite completa desde Eclipse y generad los análisis desde **Run As → Maven build…**:

| Análisis | Objetivo de Maven | Dónde consultar el resultado |
| --- | --- | --- |
| Pruebas y JaCoCo | `clean verify` | Vista JUnit de Eclipse y `target/site/jacoco/index.html` |
| PIT | `org.pitest:pitest-maven:mutationCoverage` | Informe HTML dentro de `target/pit-reports/` |
| SonarQube | Action preparada en GitHub, rama `main` | Proyecto del grupo en https://sast.inf.um.es/ |

### Guías de apoyo y configuración

Las explicaciones breves de este enunciado se complementan con los siguientes documentos. Consultadlos cuando necesitéis interpretar un informe o configurar las herramientas; no es necesario reproducir sus definiciones en el vídeo.

| Guía | Contenido |
| --- | --- |
| [Introducción a JaCoCo](docs/JACOCO.md) | Cobertura, informe HTML y selección de evidencias |
| [Introducción a PIT](docs/PIT.md) | Mutantes, resultados y posibles pruebas |
| [Introducción a SonarQube](docs/SONARQUBE.md) | Reglas, incidencias y Quality Gate |
| [Configuración de GitHub Actions](docs/SONAR_GITHUB.md) | Proyecto, token, variables y publicación del análisis |
| [Eclipse y SonarQube](docs/ECLIPSE_SONAR.md) | Instalación opcional del complemento y modo conectado |

JaCoCo y PIT funcionan mediante Maven con la configuración del proyecto. La conexión de SonarQube con Eclipse es **opcional**; el análisis que se evaluará es el publicado desde GitHub Actions sobre `main`.

### Consulta rápida de JaCoCo

JaCoCo es una herramienta de cobertura de código. Indica qué partes de la implementación han sido ejecutadas por las pruebas, pero no demuestra que las pruebas sean correctas ni que hayan detectado todos los defectos.

El informe HTML se genera en `target/site/jacoco/index.html`. Conviene distinguir:

- **Instruction coverage**: instrucciones de bytecode ejecutadas.
- **Branch coverage**: ramas de decisión ejecutadas.
- **Line coverage**: líneas ejecutadas.
- **Method coverage**: métodos invocados.
- **Class coverage**: clases utilizadas.

En esta práctica registraréis principalmente instrucciones y ramas. La cobertura de ramas aporta información sobre decisiones alternativas, pero no garantiza que se hayan comprobado los valores límite ni los resultados esperados. Desde el informe podéis navegar hasta cada clase y observar el código ejecutado total o parcialmente. No incorporéis `target/` al repositorio.

### Consulta rápida de PIT

PIT realiza análisis de mutación. Introduce pequeños cambios temporales en el código y ejecuta las pruebas para comprobar si los detectan.

- **Killed mutants**: mutantes detectados por las pruebas.
- **Survived mutants**: mutantes que no provocaron fallos.
- **No coverage**: mutantes en código que no han alcanzado las pruebas.
- **Mutation coverage**: proporción de mutantes eliminados respecto al total considerado.
- **Test strength**: proporción de mutantes eliminados entre los que tenían cobertura.

Un mutante superviviente no demuestra necesariamente que el programa original contenga un error: puede ser equivalente o señalar un comportamiento que las pruebas no comprueban. Revisad tanto el resumen como el detalle por clase y método. El informe HTML se encuentra dentro de `target/pit-reports/`. No alteréis los resultados para ocultar mutantes.

### SonarQube

Para configurar SonarQube, seguid `docs/SONAR_GITHUB.md`. El profesorado facilitará el identificador del proyecto y cómo registrar el token como secreto. **No incluyáis credenciales en el repositorio**. Consultad el análisis de `main`, el estado del Quality Gate y las incidencias encontradas.

### Vídeo de análisis de calidad del software

El grupo elaborará un vídeo de **7 minutos como máximo**, incluida la presentación inicial, en el que analizará resultados concretos obtenidos con JaCoCo, SonarQube y PIT sobre la versión final del proyecto.

El vídeo comenzará con una presentación de los participantes, mostrando su imagen en directo durante un máximo de 30 segundos. A continuación, se grabará la pantalla mientras se navega por los informes y el código.

El análisis deberá incluir:

- **JaCoCo:** seleccionar un resultado concreto de cobertura, identificar el código o las ramas afectadas, explicar su significado y proponer una prueba que permita mejorar la comprobación de ese comportamiento.
- **SonarQube:** seleccionar una incidencia concreta, identificar la regla aplicada y el código afectado, explicar el problema y plantear una posible solución. También se mostrará el estado general del Quality Gate.
- **PIT:** seleccionar un mutante superviviente, mostrar el cambio introducido, explicar por qué las pruebas no lo detectan y proponer una actuación para resolver esa situación.

Si no existen incidencias o mutantes supervivientes, podrá seleccionarse un resultado alternativo que permita realizar un análisis técnico equivalente.

No se trata de describir las herramientas ni de leer sus métricas. El objetivo es **interpretar evidencias del propio proyecto, relacionarlas con el código desarrollado y razonar posibles vías de actuación**. No será necesario implementar las mejoras propuestas.

Finalizad el vídeo indicando qué actuación consideraríais prioritaria y justificad vuestra elección. En grupos de dos, ambos estudiantes deberán intervenir de forma equilibrada y explicar resultados concretos.

#### Requisitos de grabación y calidad

El vídeo podrá realizarse en español o en inglés y deberá cumplir estas condiciones:

- La imagen debe permitir leer claramente el código y los informes.
- La voz debe escucharse con claridad, con volumen adecuado y sin ruidos que dificulten la comprensión.
- Deben mostrarse las acciones realizadas, el movimiento del ratón y la navegación real por las herramientas.
- La explicación debe realizarse al mismo tiempo que se muestran las acciones; no se sustituirá por una voz en off añadida posteriormente.
- Debe mantenerse la continuidad entre las acciones y la explicación.
- El vídeo debe reproducirse correctamente desde el enlace entregado.

No se evalúa la edición del vídeo, pero sí su legibilidad, el sonido, la duración y la posibilidad de comprobar las explicaciones técnicas.

#### Publicación y entrega del vídeo

Alojad el vídeo en una plataforma externa, como **YouTube, con visibilidad «Oculto/No listado»**, accesible mediante un enlace directo sin solicitar permisos adicionales. No es necesario hacerlo público.

Incluid el enlace de reproducción en este `README_P2.md` y comprobad que funciona desde otra sesión o navegador. El profesorado podrá realizar una entrevista individual breve si necesita verificar la comprensión de los resultados presentados.

## 4. Qué debéis entregar

La entrega es el contenido de `P2/` en el repositorio del grupo, incluido el proyecto Maven con sus tests, el suplantador implementado y las correcciones. Utilizad **`README_P2.md` como informe**: completad la sección final conservando estas instrucciones.

| Evidencia | Información que debe poder comprobarse |
| --- | --- |
| Integrantes y reparto | Nombre y apellidos, responsabilidades y, en parejas, enlaces a las ramas y Pull Requests |
| Casos de prueba | Identificador, clase probada, condición, resultado esperado y método JUnit |
| Defectos detectados | Test que falló, esperado frente a observado, causa, archivo o método corregido y enlace al commit |
| Resultado JUnit | Captura de Eclipse con el resultado final y total de pruebas |
| Suplantador | Archivo Java implementado y explicación breve de la dependencia sustituida |
| JaCoCo | Indicadores de cobertura de instrucciones y ramas |
| PIT | Mutation coverage y test strength |
| SonarQube | Enlace al proyecto, estado del Quality Gate e incidencias |
| Vídeo | Enlace directo al vídeo externo no listado de máximo 7 minutos |
| Versión analizada | SHA del commit de `main` al que corresponden los resultados |

**Rastro de los defectos.** Los commits y las pruebas son la evidencia principal; una descripción por sí sola no demuestra que el defecto se haya descubierto. Conservad el test que lo identifica y señalad el commit de la corrección. No hace falta una captura por cada error cuando el historial permite comprobarlo.

Guardad las capturas en `P2/evidencias/` y enlazadlas desde este documento con rutas relativas. No subáis `target/`, archivos `.class`, claves ni tokens. Los informes generados pueden permanecer en el equipo: la interpretación detallada se realizará en el vídeo, no por escrito.

## 5. Evaluación

La evaluación se basará en **evidencias verificables del repositorio y del vídeo**, no en el número de pruebas ni en alcanzar un porcentaje concreto de cobertura o mutación.

| Criterio | Puntos | Evidencia principal |
| --- | ---: | --- |
| Diseño y ejecución de pruebas JUnit 5 | 3,50 | Tests que verifican contratos y resultados |
| Identificación y corrección de defectos | 3,00 | Tests reveladores y commits de corrección |
| Construcción y uso del suplantador | 0,75 | Implementación en `src/test/java` y pruebas del caso de uso |
| Interpretación de JaCoCo | 0,40 | Análisis técnico en el vídeo |
| Interpretación de PIT | 0,40 | Análisis técnico en el vídeo |
| Interpretación de SonarQube | 0,40 | Análisis técnico en el vídeo |
| Propuestas de actuación justificadas | 0,35 | Viabilidad y relación con resultados reales en el vídeo |
| Relación entre resultados y prioridad final | 0,20 | Conclusión razonada del vídeo |
| Documentación y trabajo con Git | 1,00 | README, trazabilidad, ramas e integración, cuando proceda |
| **Total** | **10,00** | |

Una prueba que pasa sin comprobar correctamente el resultado esperado no es evidencia suficiente. El vídeo se valorará por la comprensión y el razonamiento técnico, no por sus efectos visuales.

---

## Resolución del grupo

### Integrantes, responsabilidades y enlaces de integración

*Completar.*

### Casos de prueba

| ID | Clase | Condición y resultado esperado | Método JUnit |
| --- | --- | --- | --- |
| | | | |

### Defectos detectados y corregidos

| ID | Prueba que revela el defecto | Esperado / observado | Causa y corrección | Commit |
| --- | --- | --- | --- | --- |
| | | | | |

### Suplantador y desarrollo incremental

*Explicar brevemente.*

### Indicadores de calidad

| Herramienta | Resultado obtenido | Evidencia |
| --- | --- | --- |
| JUnit 5 | | |
| JaCoCo (instrucciones y ramas) | | |
| PIT (mutation coverage y test strength) | | |
| SonarQube (Quality Gate e incidencias) | | |

**Commit de `main` analizado:** *completar*.

**Enlace al vídeo externo (oculto/no listado):** *completar*.
