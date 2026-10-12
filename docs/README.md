# Robot de limpieza — proyecto inicial para Eclipse

Proyecto Java 17 con Maven y JUnit 5. Se entrega deliberadamente con defectos de comportamiento y sin los tests de referencia.

La descripción funcional de los contratos está en [specification.md](specification.md). El alumnado debe implementar las pruebas y completar `StubCleaningInventory` en `src/test/java`.

Importación: **Eclipse → File → Import → Maven → Existing Maven Projects**.

Comandos desde este directorio:
- `mvn test`
- `mvn clean verify`
- `mvn org.pitest:pitest-maven:mutationCoverage`

El enunciado y las guías están en [../README.md](../README.md).
