# Diagramas del proyecto AndroidModApp

Carpeta: `docs/diagrams`

Contiene varios archivos PlantUML (.puml) que describen la arquitectura multimódulo y ejemplos de diagramas:

- `00-modulos-dependencias.puml` - Mapa de módulos (core, data, domain, feature, app).
- `01-capas-flujo.puml` - Diagrama de capas y flujo de dependencias (clean architecture).
- `02-secuencia-flujo-uso.puml` - Secuencia típica: UI → UseCase → Repo → DataSource.
- `03-core-paquetes.puml` - Paquetes y componentes del módulo `core`.
- `04-domain-paquetes.puml` - Paquetes y clases del módulo `domain`.
- `05-data-paquetes.puml` - Paquetes y estructuras del módulo `data`.
- `06-feature-paquetes.puml` - Ejemplo de paquetes dentro de un `feature`.
- `07-modelos-clases.puml` - Ejemplo de diagrama de clases para modelos y mappers.

Renderizado:

1. Instala PlantUML y Graphviz localmente, o usa un plugin de VS Code (PlantUML). 
2. Para generar PNG desde la línea de comandos (si tienes `plantuml`):

```bash
cd docs/diagrams
plantuml *.puml
```

3. Usualmente VS Code + PlantUML te permite previsualizar cada `.puml` y exportar PNG/SVG.

Notas:
- Los diagramas son plantillas basadas en la estructura multimódulo que hay en el repo. Si quieres diagramas exactos extraídos del código (clases reales, relaciones reales), puedo:
  - analizar los paquetes Kotlin y generar diagramas de clases concretos, o
  - extraer dependencias reales desde los `build.gradle.kts` y ajustar el diagrama de módulos.
