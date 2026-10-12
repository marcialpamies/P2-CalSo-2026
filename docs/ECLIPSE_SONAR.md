# Eclipse y SonarQube: herramientas mínimas y conexión opcional

[Volver al enunciado](../README_P2.md) · [Introducción a SonarQube](SONARQUBE.md)

## Qué necesitáis instalar

Para desarrollar la práctica es suficiente con **Eclipse IDE con herramientas Java y Maven (m2e)**, un JDK compatible y el proyecto Maven importado. **JUnit 5, JaCoCo y PIT se obtienen mediante las dependencias y plugins configurados en el `pom.xml`**: no necesitáis buscar un plugin de Eclipse para cada uno.

Es posible instalar **SonarQube for Eclipse** (antes SonarLint) para visualizar incidencias localmente y, si se autoriza, usar *Connected Mode*. Este complemento es útil, pero **no sustituye el análisis publicado desde GitHub Actions** y no es imprescindible para entregar el vídeo.

## Instalar SonarQube for Eclipse (opcional)

1. En Eclipse, abrid **Help → Eclipse Marketplace**.
2. Buscad **SonarQube for Eclipse** y comprobad que es el complemento de SonarSource.
3. Instaladlo y reiniciad Eclipse cuando se solicite.
4. Abrid el proyecto Java. Las incidencias del análisis local aparecerán en las vistas que proporcione el complemento.

Los nombres exactos de menús pueden variar con la versión de Eclipse y del complemento.

## Conectar con el servidor de la UMU (opcional)

1. Abrid **File → New → Other… → SonarQube → New SonarQube (Server, Cloud) Connection**, o la opción equivalente desde la vista de conexiones.
2. Elegid **SonarQube Server**, e indicad `https://sast.inf.um.es/`.
3. Si se solicita autenticación, usad un **token de usuario** personal obtenido en la página de seguridad de vuestra cuenta en SonarQube. No lo copiéis a archivos ni a GitHub.
4. Asignad un nombre a la conexión y finalizad el asistente.
5. Vinculad (*bind*) el proyecto local de Eclipse con el proyecto SonarQube de vuestro grupo y comprobad que se sincronizan las reglas.

**Importante:** el token para el modo conectado del IDE es de usuario; el token de análisis almacenado como secreto de GitHub Actions tiene otro uso. No compartáis ninguno de los dos. Connected Mode ajusta reglas y sincroniza información, **pero no envía el análisis local al servidor**.

Si el servidor no permite generar un token de usuario o la conexión está restringida, trabajad sin Connected Mode y consultad SonarQube desde el navegador. La entrega no depende de esta conexión.

## Problemas habituales

- **No encuentro el complemento**: verificad la compatibilidad de vuestra versión de Eclipse y utilizad el Marketplace oficial.
- **Conexión rechazada**: comprobad URL, credenciales, acceso a la red de la universidad y permisos del proyecto.
- **Distintas incidencias entre Eclipse y la web**: verificad el proyecto vinculado, las reglas y el commit analizado; además, los resultados locales y remotos no son idénticos por definición.

Documentación oficial: https://docs.sonarsource.com/sonarqube-for-eclipse/connect-your-ide/setup
