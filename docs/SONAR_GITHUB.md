# Tutorial: conectar GitHub Actions con SonarQube de la UMU

[Volver al enunciado](../README_P2.md) · [Introducción a SonarQube](SONARQUBE.md) · [Conexión opcional desde Eclipse](ECLIPSE_SONAR.md)

Este tutorial explica cómo preparar **un proyecto de SonarQube por grupo**, configurar las credenciales de forma segura en GitHub y comprobar que los análisis se publican correctamente.

El resultado final será un análisis de la rama `main` del repositorio del grupo en https://sast.inf.um.es/. Se recomienda completar la configuración antes de finalizar la práctica, aunque los primeros análisis pueden fallar mientras se descubren y corrigen los defectos intencionados.

> **Importante:** los nombres `SONAR_TOKEN`, `SONAR_HOST_URL` y `SONAR_PROJECT_KEY` deben escribirse exactamente así. **Nunca** publiquéis el valor del token, ni lo incorporéis al código, al README, a un commit o a una captura de pantalla.

## 1. Comprobar el repositorio de GitHub

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
    └── robot-cleaning/
        ├── pom.xml
        └── src/
```

El workflow se guarda en **`.github/workflows/` de la raíz del repositorio**, no dentro de `P2/` ni del proyecto Java. La ruta exacta al proyecto Maven será `P2/robot-cleaning/`.

En grupos de dos, uno de los integrantes puede configurar los permisos y secretos, siempre que disponga de autorización en GitHub. No hace falta que ambos creen tokens ni duplicar la configuración.

## 2. Crear o localizar el proyecto SonarQube del grupo

1. Entrad en [SonarQube UMU](https://sast.inf.um.es/) con vuestra cuenta.
2. Si el profesorado **ya os ha asignado un proyecto**, abridlo y no creéis otro.
3. Si tenéis autorización para crearlo, utilizad la opción **Create Project** o **Create new project** y escoged la creación manual/local cuando aparezca.
4. Indicad un nombre que identifique al grupo y la práctica. Por ejemplo, `CalSo-P2-Grupo-07`.
5. Estableced una **Project Key** única, por ejemplo, `calso-p2-grupo-07`. Es el identificador técnico del proyecto: no es necesariamente idéntico al título visible.
6. Conservad la Project Key. Se utilizará en GitHub y en el enlace al análisis.

Si no aparece la opción de crear proyectos o el servidor rechaza la creación por permisos, **solicitad al profesorado que os asigne un proyecto**. No intentéis modificar los permisos generales del servidor ni reutilicéis un proyecto de otro grupo.

Algunas pantallas pueden ofrecer la configuración mediante GitHub Actions. Podéis consultar esas indicaciones, pero utilizad los nombres de variables y el workflow de esta práctica para mantener una configuración común.

## 3. Generar el token de análisis en SonarQube

El token permite publicar análisis sin introducir una contraseña en el workflow.

1. Pulsad vuestro avatar en SonarQube.
2. Entrad en **My Account → Security**.
3. Buscad **Generate Tokens** y escribid un nombre identificable, por ejemplo, `CalSo-P2-Grupo-07-Actions`.
4. Seleccionad el tipo **Project Analysis Token** cuando esté disponible.
5. Elegid **únicamente el proyecto de vuestro grupo** y una caducidad que cubra la duración de la práctica.
6. Pulsad **Generate** y copiad inmediatamente el token. Por seguridad, SonarQube no vuelve a mostrar su valor después de cerrar el aviso.

Para generar un token de análisis del proyecto se necesita permiso **Execute Analysis** sobre él (o el permiso global correspondiente). Si no podéis crearlo, pedid al profesorado que revise los permisos. Un token restringido a un proyecto es preferible a uno con acceso general.

**No confundáis este token con el token de usuario empleado para conectar Eclipse en Connected Mode.** Para el IDE se utiliza un token de usuario personal y la conexión es opcional. Para GitHub Actions basta con el token de análisis del proyecto.

## 4. Guardar el token como secreto de GitHub

1. Abrid el **repositorio del grupo** en GitHub.
2. Entrad en **Settings → Secrets and variables → Actions**.
3. Seleccionad la pestaña **Secrets**.
4. Pulsad **New repository secret**.
5. Introducid estos datos:

| Campo | Valor |
| --- | --- |
| Name | `SONAR_TOKEN` |
| Secret | El token generado en SonarQube |

6. Pulsad **Add secret**.

GitHub mostrará el **nombre** del secreto, pero no permitirá consultar posteriormente su contenido. Esto es normal.

Si no podéis acceder a `Settings` o crear secretos, consultad con el propietario del repositorio. Un compañero sin permisos de administración no debe recibir el token por correo o mediante un archivo para intentar resolverlo.

## 5. Configurar las variables del repositorio

En la misma pantalla **Settings → Secrets and variables → Actions**, seleccionad ahora **Variables** y utilizad **New repository variable** para crear estas dos entradas:

| Name | Value |
| --- | --- |
| `SONAR_HOST_URL` | `https://sast.inf.um.es` |
| `SONAR_PROJECT_KEY` | La Project Key exacta de vuestro proyecto, p. ej. `calso-p2-grupo-07` |

La dirección del servidor y el identificador del proyecto **no son contraseñas**. Por eso se guardan como variables y no como secretos.

Antes de continuar, comprobad que existen **un secreto y dos variables**, con los nombres indicados. No mostréis ni compartáis el valor de `SONAR_TOKEN`.

## 6. Comprobar el workflow de GitHub Actions

El repositorio inicial incorpora el archivo `.github/workflows/p2-quality.yml`. No es necesario redactar otro workflow desde cero.

El procedimiento realiza, en este orden:

1. Obtiene el contenido de `main`.
2. Configura Java y Maven en el runner de GitHub.
3. Ejecuta `mvn clean verify` para correr JUnit 5 y generar JaCoCo.
4. Comprueba la presencia de los valores de conexión.
5. Ejecuta SonarScanner for Maven y envía al servidor el análisis, incluyendo la ruta al XML de JaCoCo.

En el workflow encontraréis expresiones equivalentes a estas:

```yaml
env:
  SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
  SONAR_HOST_URL: ${{ vars.SONAR_HOST_URL }}
  SONAR_PROJECT_KEY: ${{ vars.SONAR_PROJECT_KEY }}
```

Y el análisis utilizará la clave del proyecto y la dirección del servidor mediante las propiedades de SonarScanner. El informe de JaCoCo se genera en `target/site/jacoco/jacoco.xml`.

**No sustituyáis `secrets.SONAR_TOKEN` por el token escrito directamente.** La Action tiene acceso al secreto cuando se ejecuta, sin que este aparezca en el archivo YAML del repositorio.

### ¿Cuándo se ejecuta?

El workflow definitivo está preparado para analizar **`main`**, porque los resultados de la entrega deben corresponder al trabajo integrado. Las ramas individuales de las parejas pueden contener pruebas que fallen mientras se desarrolla la práctica.

Cuando integréis las correcciones en `main`, la Action se ejecutará si los cambios cumplen las condiciones `push` y `paths` del workflow. También se incluye `workflow_dispatch` para ejecutarla manualmente desde la pestaña **Actions** cuando GitHub la tenga disponible en la rama predeterminada.

Si aún no podéis ejecutarla manualmente, realizad una modificación pertinente en el proyecto y enviadla a `main` cuando esté listo para el análisis final. **No modifiquéis archivos solo para provocar ejecuciones innecesarias.**

## 7. Verificar la primera ejecución

1. En GitHub, abrid la pestaña **Actions** del repositorio.
2. Seleccionad **P2 - Analisis SonarQube**.
3. Abrid la ejecución más reciente correspondiente a `main`.
4. Comprobad los pasos de pruebas, generación de JaCoCo y publicación en SonarQube.
5. Si el workflow termina en verde, abrid vuestro proyecto en https://sast.inf.um.es/.
6. Verificad que podéis consultar la rama analizada, la fecha del análisis, las incidencias y el **Quality Gate**.
7. Revisad que SonarQube muestre datos de cobertura. Si aparecen como 0 % o no están disponibles, comprobad la importación del XML de JaCoCo antes de sacar conclusiones.
8. Registrad en `P2/README_P2.md` el enlace al proyecto de SonarQube y el **SHA del commit de `main`** utilizado para el análisis.

Un workflow verde significa que sus pasos se han ejecutado correctamente; **no significa necesariamente que el Quality Gate esté aprobado**, ni que las pruebas hayan detectado todos los errores.

Tampoco asegura por sí solo que la cobertura se haya importado: hay que comprobar el resultado en SonarQube.

## 8. Problemas habituales y cómo actuar

| Situación | Qué comprobar |
| --- | --- |
| No aparece **Create Project** | Es posible que no tengáis permisos; solicitad al profesorado un proyecto asignado |
| No puedo generar un Project Analysis Token | Revisad el permiso **Execute Analysis** y el proyecto seleccionado |
| GitHub no muestra **Settings** | El usuario no tiene los permisos necesarios en ese repositorio |
| El workflow informa de que falta `SONAR_TOKEN` | Debe existir un secreto con ese nombre exacto en **Actions → Secrets** |
| Faltan `SONAR_HOST_URL` o `SONAR_PROJECT_KEY` | Comprobad **Actions → Variables**, no solo los secretos |
| El escáner devuelve **Not authorized** | Revisad que el token pertenezca al proyecto correcto, tenga permisos y no haya caducado |
| **Project not found** | Comprobad que la Project Key coincide exactamente con la del servidor |
| La compilación o JUnit fallan | Revisad los tests y el código; los defectos intencionados pueden hacer fallar las primeras ejecuciones |
| No aparece el informe de cobertura | Verificad `target/site/jacoco/jacoco.xml` y el paso de importación de JaCoCo |
| No aparece el workflow en Actions | Comprobad la ruta del YAML, las condiciones del disparador y la rama predeterminada |
| El servidor no responde | Comprobad su disponibilidad y consultad con el profesorado si existe una restricción de red |
| El token caduca | Generad uno nuevo, sustituid el secreto en GitHub y revocad el anterior cuando corresponda |

Si un token se comparte por error, **revocadlo en My Account → Security**, cread uno nuevo y actualizad el secreto `SONAR_TOKEN`.

## 9. Qué debe quedar como evidencia en la entrega

En el apartado de resultados de calidad de `P2/README_P2.md` facilitad:

- Enlace al proyecto de SonarQube del grupo.
- Estado del análisis final de la rama `main` y del Quality Gate.
- SHA del commit de `main` analizado.
- Captura del resultado de GitHub Actions, **sin mostrar secretos**.

El análisis y las posibles actuaciones se explicarán en el **vídeo técnico de máximo 7 minutos**, tal como establece el enunciado. No se exige reproducir por escrito todas las incidencias.

## 10. Documentación para resolver dudas

- [SonarQube: gestionar tokens](https://docs.sonarsource.com/sonarqube-community-build/user-guide/managing-tokens/)
- [SonarQube: permisos](https://docs.sonarsource.com/sonarqube-community-build/instance-administration/user-management/user-permissions/)
- [GitHub: secretos de Actions](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets)
- [GitHub: variables de Actions](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-variables)
- [SonarScanner para Maven](https://docs.sonarsource.com/sonarqube-community-build/analyzing-source-code/scanners/sonarscanner-for-maven)
