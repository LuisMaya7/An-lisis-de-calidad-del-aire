# Análisis de Calidad del Aire - Detalles de la Solución

## Paso 1: Cargar Datos Demográficos

- **Código:** La función `cargar_datos_demograficos` carga los datos demográficos desde la URL proporcionada.

    ```python
    import pandas as pd

    def cargar_datos_demograficos() -> pd.DataFrame:
        url_demographics = "https://public.opendatasoft.com/explore/dataset/us-cities-demographics/download/?format=csv&timezone=Europe/Berlin&lang=en&use_labels_for_header=true&csv_separator=%3B"
        demographics = pd.read_csv(url_demographics, sep=';')

        columns_to_drop = ['Race', 'Count', 'Number of Veterans']
        demographics_cleaned = demographics.drop(columns=columns_to_drop, errors='ignore')

        demographics_cleaned.drop_duplicates(inplace=True)

        return demographics_cleaned
    ```

- **Acciones Realizadas:**
    - Eliminación de columnas: Race, Count y Number of Veterans.
    - Eliminación de filas duplicadas.

- **Resultados Esperados:** 
    - Se espera obtener un DataFrame limpio con datos demográficos sin columnas no deseadas y sin filas duplicadas. Esto mejora la calidad de los datos y facilita el análisis.

## Paso 2: Cargar Calidad del Aire

- **Código:** La función `cargar_calidad_aire` utiliza la API para obtener datos de calidad del aire.

    ```python
    import requests
    import pandas as pd

    def cargar_calidad_aire(ciudades: set) -> None:
        api_url = 'https://api.api-ninjas.com/v1/airquality?city={}'
        api_key = 'd07UAK21jfRuUOWCe3dPKQ==MXUcMShYimS6w6K1'

        data = {
            'City': [],
            'Concentration': []
        }

        for ciudad in ciudades:
            response = requests.get(api_url.format(ciudad), headers={'X-Api-Key': api_key})

            if response.status_code == 200:
                air_quality_data = response.json().get("data", {})
                concentration = air_quality_data.get("concentration", None)
                data['City'].append(ciudad)
                data['Concentration'].append(concentration)
            else:
                print(f"Error para la ciudad {ciudad}: {response.status_code} - {response.text}")

        df = pd.DataFrame(data)
        df.to_csv("ciudades.csv", index=False)
    ```

- **Acciones Realizadas:**
    - Obtención de datos de calidad del aire para cada ciudad.
    - Almacenamiento en un archivo CSV.

- **Resultados Esperados:** 
    - Se espera obtener un archivo CSV con datos de calidad del aire para cada ciudad. La función maneja posibles errores en la obtención de datos.

## Paso 3: Crear Base de Datos SQLite

- **Código:** Describe cómo creas la base de datos SQLite y cargas las dos tablas procesadas.

    ```python
    import sqlite3

    demographics_df = cargar_datos_demograficos()
    ciudades_set = set(demographics_df["City"].tolist())

    conn = sqlite3.connect('demographics_air_quality.db')

    demographics_df.to_sql('demographics', conn, index=False, if_exists='replace')
    pd.read_csv('ciudades.csv').to_sql('air_quality', conn, index=False, if_exists='replace')
    ```

- **Acciones Realizadas:**
    - Creación de una conexión a SQLite.
    - Carga de tablas demográficas y de calidad del aire.

- **Resultados Esperados:** 
    - Asegurarse de que las tablas se hayan cargado correctamente y que la estructura de la base de datos sea la esperada.

## Paso 4: Joins y Agregaciones

- **Código:** Proporciona el código SQL utilizado para realizar joins y agregaciones.

    ```sql
    -- Tu código SQL aquí
    SELECT d.City, d.State, d."Total Population", a.Concentration
    FROM demographics d
    LEFT JOIN air_quality a ON d.City = a.City
    ORDER BY d."Total Population" DESC
    LIMIT 10;
    ```

- **Acciones Realizadas:**
    - Join entre tablas demográficas y de calidad del aire.
    - Agregaciones para verificar la relación entre población y calidad del aire.

- **Resultados Esperados:** 
    - Muestra las primeras 10 columnas del resultado y comenta sobre cualquier patrón o tendencia observable.
