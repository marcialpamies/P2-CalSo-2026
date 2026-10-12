# Introducción a SonarQube

[Volver al enunciado](../README_P2.md) · [Configuración de GitHub Actions](SONAR_GITHUB.md) · [Conectar Eclipse](ECLIPSE_SONAR.md)

## Qué analiza

SonarQube examina el código de manera estática según un **Quality Profile** (reglas activas). Los resultados se muestran como incidencias que pueden afectar a fiabilidad, seguridad o mantenibilidad, según la clasificación y versión del servidor. El **Quality Gate** aplica condiciones a métricas y determina si el análisis las supera.

Un informe de SonarQube no sustituye a JUnit: encontrar una incidencia estática y comprobar un comportamiento funcional son tareas distintas.

El servidor utilizado en la práctica es https://sast.inf.um.es/.

## Consultar un proyecto

1. Acceded al proyecto del grupo en el servidor de la asignatura.
2. Comprobad que el análisis corresponde a la versión integrada de `main`.
3. Revisad el estado de **Quality Gate** y las condiciones que muestra.
4. Entrad en **Issues** (incidencias); seleccionad una y examinad la regla y su ubicación en el código.
5. Consultad las métricas principales sin confundir el número de incidencias con los defectos funcionales que deben detectar los tests.

Las incidencias no son todas igual de importantes. Antes de proponer una modificación, comprobad qué regla se aplica y si afecta al contexto de vuestro proyecto.

## Preparación del vídeo

Mostrad el Quality Gate y escoged una incidencia. Abrid su detalle, identificad la regla y el código señalado, explicad **qué consecuencia podría tener** y qué modificación consideraríais adecuada. Si no hay incidencias, explicad con una condición concreta del Quality Gate qué se ha evaluado y cómo interpretar el resultado.

No hace falta resolver incidencias ni perseguir un Quality Gate satisfactorio. La práctica se centra en analizar y argumentar.

## Relación con las otras herramientas

- **JUnit 5**: verifica resultados definidos por la especificación.
- **JaCoCo**: señala qué código se ejecutó durante las pruebas.
- **PIT**: explora la capacidad de las pruebas para detectar cambios.
- **SonarQube**: informa de reglas estáticas y condiciones de calidad; puede mostrar cobertura importada de JaCoCo si la Action está configurada para ello.

Documentación oficial: https://docs.sonarsource.com/sonarqube-community-build/
