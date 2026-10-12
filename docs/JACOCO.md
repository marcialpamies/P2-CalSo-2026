# Introducción a JaCoCo

[Volver al enunciado](../README_P2.md)

## Para qué sirve

JaCoCo mide qué partes del código Java han sido **ejecutadas** al correr las pruebas. No comprueba por sí mismo si las aserciones son correctas. Una línea cubierta puede seguir conteniendo un defecto que nadie ha detectado.

En esta práctica interesa especialmente distinguir **cobertura de instrucciones** y **cobertura de ramas**. La segunda refleja las alternativas recorridas en las decisiones del código; no debe confundirse con el número de casos de prueba.

## Obtener el informe desde Eclipse

1. Abrir `robot-cleaning/` como **Existing Maven Projects**.
2. Hacer clic derecho sobre el proyecto: **Run As → Maven build…**.
3. Indicar el objetivo `clean verify` y ejecutar.
4. Abrir `robot-cleaning/target/site/jacoco/index.html` en un navegador.
5. Navegar desde el resumen hasta el paquete, la clase y la línea de interés.

También se puede ejecutar `mvn clean verify` en una terminal situada junto al `pom.xml`. Los informes se regeneran en `target/`; esta carpeta no se entrega.

## Cómo leer los resultados

- **Instructions**: instrucciones de bytecode ejecutadas y no ejecutadas.
- **Branches**: alternativas ejecutadas y no ejecutadas en decisiones.
- **Lines**: líneas ejecutadas o parcialmente ejecutadas.
- **Methods/Classes**: métodos o clases alcanzados.

Si un `if` contiene dos caminos, ejecutar solo uno de ellos puede dejar una rama sin cubrir aunque la línea esté ejecutada. Los porcentajes globales pueden ocultar que una regla importante no se ha comprobado.

## Preparación del vídeo

Seleccionad una clase concreta del proyecto. Mostrad el informe, identificad una condición o tramo sin cubrir, localizadlo en el código y explicad **qué prueba añadiríais y con qué datos**. No hace falta implementarla. Si la clase está totalmente cubierta, examinad un camino ya ejecutado y justificad qué test lo recorre.

No os limitéis a leer el porcentaje: explicad qué permite y qué no permite concluir.

## Problemas habituales

- **No aparece `index.html`**: comprobad el objetivo Maven, la configuración JaCoCo del `pom.xml` y que las pruebas se hayan ejecutado.
- **Cobertura 0 %**: verificad que las pruebas se ejecutaron en ese mismo proyecto y que JaCoCo está asociado a la fase adecuada.
- **Resultados antiguos**: regenerad con `clean verify` y comprobad que el análisis corresponde al commit final.

Documentación oficial: https://www.jacoco.org/jacoco/trunk/doc/maven.html
