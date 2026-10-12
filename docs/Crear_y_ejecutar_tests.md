# Guía para crear y ejecutar pruebas en Eclipse con JUnit 5

Esta guía explica cómo crear una nueva clase de prueba, añadir casos de prueba y ejecutar los tests con y sin análisis de cobertura de código.

---

## 1. Crear una nueva clase de prueba

1. En el **Explorador de paquetes (Package Explorer)**, sitúate en la carpeta `src/test/java`.
2. Busca el pauqete en el que quieras incorporar la clase de prueba (el mismo paquete que contiene la clase a probar en la carpeta `src/main/java` . Si el paquete no existe, crealo.  
3. Haz clic derecho sobre el paquete donde vas a añadir la prueba.  
4. Selecciona **New > JUnit Test Case**.  
5. En la ventana que aparece:  
   - **Name**: introduce el nombre de la clase de prueba, por ejemplo `RatingTest`.  
   - **JUnit version**: selecciona **JUnit 5**.  
   - Marca la opción **setUp/@BeforeEach** si quieres que se genere un método de inicialización (opcional).  
6. Pulsa **Finish**.  
→ Eclipse creará una clase de prueba vacía lista para usarse.

---

## 2. Crear un caso de prueba en una clase existente

Dentro de la clase de prueba, puedes añadir un método con la anotación `@Test` o `@ParameterizedTest`(en este uñtimo caso tendrás que introducir la fuente datos de las pruebas).

Ejemplo:
```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class RatingTest {

    @Test
    void test_given_valid_value_when_createRating_then_returnNormalized() {
        Rating r = Rating.of(4.75);
        assertEquals(4.8, r.asDouble());
    }
}
```

### Buenas prácticas:
- El nombre del método debe comenzar con la palabra test y describir claramente el caso (patrón *test_given–when–then*).  
- Cada método de prueba debe comprobar **una sola cosa**.  
- Si aún no está implementado, se puede usar:  
  ```java
  fail("Sin implementar todavía");
  ```

---

## 3. Ejecutar pruebas en Eclipse

### 3.1 Ejecutar una clase de prueba
1. Haz clic derecho sobre la clase de prueba (ej. `RatingTest.java`).  
2. Selecciona **Run As > JUnit Test**.  
3. Eclipse abrirá la vista de JUnit con el resultado:  
   - **Verde** = todos los tests pasaron.  
   - **Rojo** = al menos un test falló.

### 3.2 Ejecutar todas las pruebas de un paquete o proyecto
1. Haz clic derecho sobre el paquete o el proyecto.  
2. Selecciona **Run As > JUnit Test**.  
→ Eclipse ejecutará todos los tests de ese ámbito.

---

## 4. Ejecutar pruebas con cobertura de código (EclEmma)

Eclipse 2025-06 incluye por defecto el plugin **EclEmma Java Code Coverage 3.1.10**.

### 4.1 Ejecutar una clase con cobertura
1. Haz clic derecho sobre la clase de prueba.  
2. Selecciona **Coverage As > JUnit Test**.  
3. Eclipse ejecutará los tests y mostrará un informe de cobertura:  
   - **Verde** = líneas ejecutadas por los tests.  
   - **Rojo** = líneas no ejecutadas.  
   - **Amarillo** = líneas parcialmente cubiertas.

### 4.2 Ejecutar todo el proyecto con cobertura
1. Haz clic derecho sobre el proyecto.  
2. Selecciona **Coverage As > JUnit Test**.  
3. Revisa el informe en la vista **Coverage**. 
- Añadir casos: métodos con `@Test` y asertos de JUnit/Hamcrest.  
- Ejecutar: **Run As > JUnit Test**.  
- Cobertura: **Coverage As > JUnit Test**
