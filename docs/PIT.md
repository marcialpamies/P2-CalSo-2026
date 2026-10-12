# Introducción a PIT (Mutation Testing)

[Volver al enunciado](../README_P2.md)

## Para qué sirve

PIT evalúa si las pruebas son capaces de **detectar pequeños cambios** introducidos temporalmente en el código. Esos cambios se llaman *mutantes*. Por ejemplo, puede sustituir una comparación `>` por `>=`, o alterar un valor devuelto.

Una prueba que ejecuta una línea no necesariamente detecta un error en ella. Por eso JaCoCo y PIT aportan información diferente.

## Ejecutar PIT

Desde Eclipse, **Run As → Maven build…**, con el objetivo:

```text
org.pitest:pitest-maven:mutationCoverage
```

La alternativa en terminal, dentro de `robot-cleaning/`, es:

```powershell
mvn org.pitest:pitest-maven:mutationCoverage
```

Abrid el informe HTML dentro de `target/pit-reports/`; según la configuración, puede haber una subcarpeta con identificador de ejecución.

## Interpretar el informe

- **Killed**: alguna prueba detectó el cambio, normalmente fallando.
- **Survived**: el cambio no fue detectado por las pruebas ejecutadas.
- **No coverage**: el código mutado no fue alcanzado por las pruebas.
- **Mutation coverage**: proporción de mutantes eliminados sobre el total considerado por el informe.
- **Test strength**: proporción de eliminados entre los mutantes cubiertos.

Un superviviente no implica automáticamente un defecto del programa original. Algunas transformaciones son equivalentes para el comportamiento observable o afectan a condiciones que el contrato no exige distinguir.

## Preparación del vídeo

Escoged un mutante superviviente. Mostrad la clase y la línea, explicad **qué ha cambiado**, qué comportamiento podría variar y por qué ninguna prueba actual lo distingue. Proponed una prueba concreta, con entradas y resultado esperado, o justificad por qué el cambio podría ser equivalente. Si no hay supervivientes, analizad uno eliminado y la prueba que lo detecta.

No tenéis que modificar el código ni mejorar la puntuación.

## Problemas habituales

- **PIT no encuentra pruebas**: comprobad JUnit 5, el plugin de soporte para JUnit 5 configurado en `pom.xml` y que `mvn test` funciona.
- **No hay informe**: buscad errores en la consola de Maven y revisad `target/pit-reports/`.
- **Resultados difíciles de interpretar**: comparad la línea mutada con la especificación funcional, no solo con la implementación original.

Documentación oficial: https://pitest.org/quickstart/maven/
