# SQL-SERVER-UNI-ADMISION-ANALYSIS
SQL-SERVER-UNI-ADMISION-ANALYSIS
## Contexto del Proyecto
El presente proyecto aplica consultas estructuradas en SQL (nivel intermedio-avanzado) para extraer, limpiar y analizar los factores criticos que impactan los resultados del proceso de admision a la Universidad Nacional de Ingenieria (UNI) durante los periodos 2025-II y 2026-I. Utilizando datos abiertos y oficiales del Estado Peruano [Link](https://www.datosabiertos.gob.pe/dataset/postulantes-al-concurso-de-admisi%C3%B3n-de-la-universidad-nacional-de-ingenier%C3%ADa-del-2025-2-al), este repositorio busca identificar brechas socio-demograficas, el rendimiento academico por regiones y el nivel de competitividad entre las diferentes especialidades ofrecidas.

## Diccionario de Columnas

Los datos originales provienen de la Plataforma Nacional de Datos Abiertos y se encuentran consolidados en la tabla principal Postulantes_UNI. A continuacion, la descripción de las columnas analizadas:
- IDHASH: Identificador único y anonimizado del postulante.
- COLEGIO: Nombre de la institución educativa de procedencia.
- COLEGIO_DEPA / PROV / DIST: Departamento, provincia y distrito de ubicación del colegio.
- COLEGIO_ANIO_EGRESO: Año en el que el postulante finalizó la educación secundaria.
ESPECIALIDAD: Carrera profesional a la que postula.
- ANIO_POSTULA / CICLO_POSTULA: Periodo del examen de admisión (ej. 2025 - 2).
- DOMICILIO_DEPA / PROV / DIST: Ubicación geográfica de residencia actual del postulante.
- ANIO_NACIMIENTO: Año de nacimiento del postulante.
- SEXO: Género del postulante.
- CALIF_FINAL: Calificación vigesimal u oficial obtenida en las pruebas de admisión.
- INGRESO: Estado final del postulante (INGRESÓ o NO INGRESÓ).
- MODALIDAD: Modalidad del concurso (Ordinario, Dos Primeros Alumnos, etc.).

## Limpieza de Datos

Antes de realizar el Análisis Exploratorio de Datos (EDA), se estandarizaron los datos nulos para preparar la información y procesarla correctamente.


``` SQL
UPDATE Postulantes_UNI
SET 
    COLEGIO = NULLIF(COLEGIO, 'SIN DATO'),
    COLEGIO_DEPA = NULLIF(COLEGIO_DEPA, 'SIN DATO'),
    COLEGIO_PROV = NULLIF(COLEGIO_PROV, 'SIN DATO'),
    COLEGIO_DIST = NULLIF(COLEGIO_DIST, 'SIN DATO'),
    DOMICILIO_DEPA = NULLIF(DOMICILIO_DEPA, 'SIN DATO'),
    DOMICILIO_PROV = NULLIF(DOMICILIO_PROV, 'SIN DATO'),
    DOMICILIO_DIST = NULLIF(DOMICILIO_DIST, 'SIN DATO')
WHERE 
    COLEGIO = 'SIN DATO' 
    OR COLEGIO_DEPA = 'SIN DATO' 
    OR COLEGIO_PROV = 'SIN DATO' 
    OR COLEGIO_DIST = 'SIN DATO'
    OR DOMICILIO_DEPA = 'SIN DATO' 
    OR DOMICILIO_PROV = 'SIN DATO' 
    OR DOMICILIO_DIST = 'SIN DATO';
```



## Exploratory Data Analysis: 10 Key Insights

### Pregunta 1: 
¿Cuál es el promedio general de calificación y la tasa de ingreso por proceso de admisión?
- Query:

``` SQL
SELECT 
    ANIO_POSTULA,
    CICLO_POSTULA,
    COUNT(*) AS Total_Postulantes,
    SUM(CASE WHEN INGRESO = 'SI' THEN 1 ELSE 0 END) AS Total_Ingresantes,
    ROUND(SUM(CASE WHEN INGRESO = 'SI' THEN 1.0 ELSE 0 END) * 100.0 / COUNT(*), 2) AS Tasa_Ingreso_Pct,
    ROUND(AVG(CALIF_FINAL), 2) AS Nota_Promedio_General
FROM Postulantes_UNI
GROUP BY ANIO_POSTULA, CICLO_POSTULA
ORDER BY ANIO_POSTULA, CICLO_POSTULA;
```

-  Resultado:
![imagene_p1](./picture/P1.png)
- Conclusión:  
El proceso 2026-1 registró un incremento de 686 postulantes respecto al ciclo anterior, lo que elevó la competencia y redujo la tasa de ingreso en cerca de 1.9 puntos porcentuales (de 23.49% a 21.57%). Asimismo, se observa una ligera disminución en el puntaje promedio general en el examen 2026-1.


### Pregunta 2: 
¿Cuáles son las 5 carreras con mayor demanda en el examen Ordinario y cuál es su respectiva nota de corte?

- Query:

``` SQL
SELECT TOP 5
    ESPECIALIDAD,
    MODALIDAD,
    COUNT(*) AS Total_Postulantes,
    -- Se evalúa la nota de corte en la modalidad ORDINARIO para reflejar el puntaje mínimo real del concurso regular
    ROUND(MIN(CASE WHEN INGRESO = 'SI' THEN CALIF_FINAL END), 2) AS Nota_Corte_Minima
FROM Postulantes_UNI
WHERE MODALIDAD = 'ORDINARIO'
GROUP BY ESPECIALIDAD, MODALIDAD
ORDER BY Total_Postulantes DESC;
```

- Resultado:
![imagene_p2](./picture/P2.png)
- Conclusión:  
En la modalidad Ordinario, Ingeniería Civil y Sistemas lideran la demanda de postulantes. Sin embargo, Ingeniería de Sistemas registra la nota de corte más exigente del Top 5 (1223.40 puntos), superando incluso a Civil (1190.40 puntos) a pesar de tener una cantidad ligeramente menor de candidatos.


### Pregunta 3: 
¿Cómo se distribuyen los postulantes, la tasa de ingreso y el rango de calificaciones (mínima y máxima) según la modalidad de admisión?
- Query:

``` SQL
SELECT 
    MODALIDAD,
    COUNT(*) AS Total_Postulantes,
    -- Conteo condicional para contabilizar únicamente a los postulantes que lograron ingresar
    SUM(CASE WHEN INGRESO = 'SI' THEN 1 ELSE 0 END) AS Total_Ingresantes,
    CAST(SUM(CASE WHEN INGRESO = 'SI' THEN 1.0 ELSE 0 END) * 100.0 / COUNT(*) AS DECIMAL(10,2)) AS Tasa_Ingreso_Pct,
    CAST(ISNULL(MIN(CASE WHEN INGRESO = 'SI' THEN CALIF_FINAL END), 0) AS DECIMAL(10,2)) AS Nota_Minima_Ingreso,
    CAST(ISNULL(MAX(CASE WHEN INGRESO = 'SI' THEN CALIF_FINAL END), 0) AS DECIMAL(10,2)) AS Nota_Maxima_Ingreso
FROM Postulantes_UNI
GROUP BY MODALIDAD
ORDER BY Total_Postulantes DESC;
```

- Resultado: 
![imagene_p3](./picture/P3.png)

- Conclusión:  
La modalidad Ordinario concentra la mayor cantidad de postulantes (9,304) pero registra la menor tasa de éxito (16.17%). En contraste, CEPREUNI es la vía más efectiva con un 54.73% de ingreso y el puntaje más alto del proceso (2910.27 puntos). Por último, las modalidades extraordinarias aplican criterios especiales, registrando un puntaje de 0.00 al no evaluarse mediante el examen tradicional.

### Pregunta 4: 
 ¿Existe una brecha de rendimiento en las calificaciones promedio y tasas de ingreso diferenciada por sexo?
- Query:

``` SQL
SELECT 
    SEXO,
    COUNT(*) AS Total_Postulantes,
    SUM(CASE WHEN INGRESO = 'SI' THEN 1 ELSE 0 END) AS Total_Ingresantes,
    -- Tasa de ingreso redondeada a 2 decimales
    CAST(SUM(CASE WHEN INGRESO = 'SI' THEN 1.0 ELSE 0 END) * 100.0 / COUNT(*) AS DECIMAL(10,2)) AS Tasa_Ingreso_Pct,
    -- Promedio de nota general
    CAST(AVG(CALIF_FINAL) AS DECIMAL(10,2)) AS Nota_Promedio,
    -- Promedio de nota considerando únicamente a quienes ingresaron
    CAST(AVG(CASE WHEN INGRESO = 'SI' THEN CALIF_FINAL END) AS DECIMAL(10,2)) AS Nota_Promedio_Ingresantes
FROM Postulantes_UNI
WHERE SEXO IS NOT NULL
GROUP BY SEXO
ORDER BY Total_Postulantes DESC;
```

- Resultado: 
![imagene_p4](./picture/P4.png)
- Conclusión:  
Existe una brecha de participación, donde el volumen de postulantes masculinos representa más del 77% del total. Si bien los hombres presentan una mayor tasa de ingreso (23.68% frente al 18.24%) y un promedio general ligeramente superior, las mujeres que logran ingresar obtienen un desempeño superior, alcanzando un promedio de 1307.33 puntos frente a los 1281.03 del grupo masculino.

### Pregunta 5: 
¿Existe una diferencia en el rendimiento y tasa de ingreso entre postulantes de Lima/Callao y los del resto del país?
- Query:

``` SQL
SELECT 
    CASE 
        WHEN DOMICILIO_DEPA IN ('LIMA', 'CALLAO') THEN 'LIMA Y CALLAO'
        ELSE 'PROVINCIAS'
    END AS Region_Procedencia,
    COUNT(*) AS Total_Postulantes,
    CAST(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER() AS DECIMAL(10,2)) AS Porcentaje_Participacion_Pct,
    SUM(CASE WHEN INGRESO = 'SI' THEN 1 ELSE 0 END) AS Total_Ingresantes,
    CAST(SUM(CASE WHEN INGRESO = 'SI' THEN 1.0 ELSE 0 END) * 100.0 / COUNT(*) AS DECIMAL(10,2)) AS Tasa_Ingreso_Pct,
    CAST(AVG(CALIF_FINAL) AS DECIMAL(10,2)) AS Nota_Promedio,
    CAST(AVG(CASE WHEN INGRESO = 'SI' THEN CALIF_FINAL END) AS DECIMAL(10,2)) AS Nota_Promedio_Ingresantes
FROM Postulantes_UNI
GROUP BY 
    CASE 
        WHEN DOMICILIO_DEPA IN ('LIMA', 'CALLAO') THEN 'LIMA Y CALLAO'
        ELSE 'PROVINCIAS'
    END
ORDER BY Total_Postulantes DESC;
```

- Resultado: 
![imagene_p5](./picture/P5.png)
- Conclusión:  
Existe una marcada centralización en la postulación, ya que Lima y Callao concentran el 91.1% de los candidatos. Aunque ambas regiones muestran una tasa de ingreso prácticamente idéntica (~22.3%), los ingresantes procedentes de la capital alcanzan un rendimiento académico superior en el examen, superando por más de 132 puntos en promedio a los ingresantes de provincias (1297.38 vs 1165.26).

