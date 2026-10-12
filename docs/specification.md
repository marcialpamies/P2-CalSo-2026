# Especificación contractual — Práctica 2

Diseñad pruebas usando este documento antes de examinar las implementaciones. Los nombres, paquetes y firmas están definidos en el código entregado.

## `Energy` (objeto de valor)
Representa el valor de energía del robot.

- `Energy.of(int units)` acepta exclusivamente enteros **mayores que cero**; en otro caso lanza `IllegalArgumentException`.
- `getValue()` devuelve el entero que representa la energía actual del robót.
- El valor es inmutable; dos instancias con iguales unidades de energía deben ser iguales (`equals`) y devolver el mismo `hashCode`.

## `CleaningProduct` (enum)
Representa los diferentes productos que utiliza el robot.

- Valores: `WATER`, `SOAP`, `WAX`, `GLASS_CLEANER`.
- `toString()` devuelve respectivamente `agua`, `jabón`, `cera`, `limpiacristales` en español y minúsculas.

## `CleaningProgram` (entidad)
Representa la información de un programa de limpieza configurado en el robot.

- Constructor: `CleaningProgram(String id, String name, Energy energy, Map<CleaningProduct,Integer> requirements)`.
- `id`, `name`: no pueden ser nulos ni ser una cadena blanca. `energy` y `requirements`: no nulos. Entradas del mapa: claves y cantidades requeridas no nulos, cantidades tiene que ser enteras `>= 0`.
- El mapa puede estar vacío; no se limita la cantidad de productos.
- `getId()`, `getName()`, `getEnergy()`, `getRequirements()` devuelven las propiedades correspondientes.
- Copia defensiva al construir y mapa de lectura no modificable al consultar; modificaciones posteriores del mapa original nunca afectan al programa.
- Igualdad (`equals`/`hashCode`) basada **únicamente en `id`**, incluso cuando nombres o consumos difieran.

## `ExecutionResult` (resultado)
- `record ExecutionResult(boolean executed, Map<CleaningProduct,Integer> missingProducts)`.
- `executed == true` implica que `missingProducts` está vacío.
- Si `executed == false`, `missingProducts` contiene **todos** los productos insuficientes y **cuántas unidades adicionales** necesita cada uno.
- El mapa devuelto no se puede modificar externamente.

## `ICleaningInventory` (puerto de salida)
- `available(CleaningProduct product)`: devuelve unidades disponibles (`>= 0`).
- `consume(CleaningProduct product, int quantity)`: descuenta exactamente `quantity` unidades. `quantity` debe ser `>= 0`; si supera las existencias o es negativo, lanza `IllegalArgumentException` sin modificar inventario.
- Las operaciones se consideran síncronas; no existen escrituras concurrentes ni errores de infraestructura durante un descuento válido.

## `IExecuteCleaningProgram` / `ExecuteCleaningProgramUseCase`
- Constructor: `ExecuteCleaningProgramUseCase(ICleaningInventory inventory)` requiere inventario no nulo.
- `execute(CleaningProgram program)` requiere programa no nulo.
- Antes de descontar cualquier cantidad, comprueba que existen suficientes unidades **de todos los productos**.
- Si algún producto falta, retorna `ExecutionResult(false, faltantes)` y **no modifica ninguna existencia**.
- Si todos alcanzan o superan lo requerido (incluida disponibilidad **exacta**), descuenta las cantidades indicadas y retorna `ExecutionResult(true, mapaVacio)`.
- Programa sin productos: ejecución correcta sin descuentos.
- Requisitos de cantidad cero son válidos y no producen descuentos de existencias.
- Todas las necesidades se valoran en unidades enteras; la energía no se descuenta de una batería.

## `CleaningProgramByEnergyComparator`
- Orden ascendente por `energy().value()`; en empate, orden lexicográfico ascendente por `id()`.
- Cumple el contrato de `Comparator`, incluyendo retorno cero al comparar valores equivalentes según ambos criterios.

## `CleaningRobotService`
- `CleaningRobotService(ICleaningInventory)` crea el caso de uso usando el puerto.
- `executeProgram(CleaningProgram)` delega la ejecución y retorna su resultado.

## Escenario orientativo
- Inventario: agua=2; programa: agua=2 → ejecuta, inventario final agua=0.
- Inventario: agua=2, jabón=0; programa: agua=2, jabón=1 → no ejecuta, faltante jabón=1, inventario intacto.

Estos escenarios no sustituyen la obligación de identificar otros casos límite y comprobaciones de ausencia de efectos.
