# SolarBI Pascual | Inteligencia de Negocios, Grupo 01

**Integrantes:** Mateo Molina Arias y Melanny Alzate  
**Semestre:** 2026-II  
**Caso:** monitoreo de un inversor solar simulado de 5 kWp.

El proyecto organiza telemetría en capas Bronze, Silver y Gold. El flujo ETL limpia lecturas de cinco minutos, calcula energía diaria y permite examinar los mismos datos desde una perspectiva operativa en Grafana y una analítica en Power BI.

## Resultados

| Indicador | Resultado |
|---|---:|
| Registros Bronze | 877 |
| Rechazos por faltantes | 21 |
| Rechazos por duplicidad | 13 |
| Rechazos por rango o formato | 17 |
| Lecturas Silver válidas | 826 |
| Días Gold | 3 |
| Energía total | 78,542833 kWh |
| Cobertura de intervalos | 95,60 % |

El ETL se ejecutó dos veces con el backend SQLite de validación. Ambos registros, en `docs/ejecucion_1.txt` y `docs/ejecucion_2.txt`, muestran los mismos conteos y totales. El CSV Bronze conserva su hash SHA-256. El informe central presenta la metodología, el análisis y las figuras basadas en los datos procesados.

## Contenido del proyecto

| Carpeta | Contenido |
|---|---|
| `docs/` | Informe PDF, registros de ejecución y síntesis gráfica del ETL. |
| `data/bronze/` | Telemetría original simulada. |
| `data/silver/` | Lecturas válidas y métricas de calidad. |
| `data/gold/` | Energía y cobertura por día. |
| `etl/` | Simulador y flujo ETL reproducible. |
| `sql/` | Esquema, carga autónoma y consultas PostgreSQL. |
| `powerbi/` | Consulta M, medidas DAX, guía y prototipo analítico. |
| `grafana/` | Dashboard JSON, consulta de alerta, guía y prototipo operativo. |

## Reproducir el ETL

Con Python 3.10 o superior, desde la raíz del proyecto:

```powershell
python etl/run_etl.py --backend sqlite
python etl/run_etl.py --backend sqlite
```

El simulador crea `data/bronze/telemetria.csv` únicamente si falta. Cada ejecución vuelve a calcular Silver y Gold y aplica UPSERT a las tablas de validación. La proporción de filas válidas frente a Bronze es 826/877 = 94,18 %; la cobertura temporal es 826/864 = 95,60 % porque hay 288 intervalos esperados por cada uno de los tres días.

Para cargar el caso en PostgreSQL hay dos rutas:

1. **Archivo SQL autónomo:** crear una base `solarbi` y ejecutar `sql/carga_demo_postgres.sql` en pgAdmin Query Tool o con `psql -d solarbi -f sql/carga_demo_postgres.sql`. Incluye esquema, 826 lecturas Silver, tres registros Gold y consultas de verificación. La cláusula `ON CONFLICT` evita duplicados al repetir la carga.
2. **Flujo Python:** instalar `requirements.txt`, definir la conexión en `SOLARBI_DB_URL` y ejecutar `python etl/run_etl.py --backend postgres`. La URL se guarda en una variable local, no en el repositorio. `sql/verificacion_postgres.sql` reúne las consultas de control.

Las marcas temporales del archivo SQL corresponden a `America/Bogota` (`-05:00`). Para automatizar el ETL se propone una ejecución diaria a las 00:00 en esa zona horaria, mediante cron (`0 0 * * *`) o el Programador de tareas de Windows.

## Visualizaciones

`powerbi/conexion_postgresql.m` y `powerbi/medidas.dax` definen la conexión y los indicadores analíticos. `grafana/dashboard.json` define tres paneles y `grafana/alerta_potencia_cero.sql` aporta la consulta para el aviso operativo. Las guías de cada carpeta explican cómo instalar estas definiciones. Las imágenes PNG son **prototipos construidos con los CSV del proyecto**; identifican el diseño y los valores para reproducirlos en cada plataforma.

## Repositorio

El proyecto contiene archivos de texto adecuados para control de versiones y un `.gitignore` que excluye credenciales, cachés y la base SQLite local. Al publicarlo en GitHub, cada integrante debe registrar sus aportes con commits propios y revisar la invitación de colaboración. El informe central es `docs/MateoMolinaArias_MelannyAlzate_Consulta_BI_G01.pdf`.
