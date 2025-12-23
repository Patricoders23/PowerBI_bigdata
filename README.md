🏏 Cricket Data Analytics: Web Scraping & Power BI ETL

Descripción del Proyecto
Este proyecto demuestra una solución técnica avanzada para la extracción, transformación y carga (ETL) de estadísticas deportivas desde la web oficial de ESPN Cricinfo. El objetivo principal fue automatizar la captura de datos de bateo mediante técnicas de scraping dinámico en Power Query.

Flujo de Trabajo (Workflow)
Scraping Basado en Patrones: Se utilizó la función "Extraer tabla mediante ejemplos" para identificar los selectores CSS necesarios en el HTML de origen.

Modularización con Lenguaje M: Se convirtió la consulta inicial en una Función Personalizada parametrizada (ps as text). Esto permite que la URL sea dinámica (&page=" & ps & ") para manejar la paginación del sitio.

Arquitectura de Datos Robusta: * Se creó una tabla de cabeceras estática comprimida en binario para asegurar la consistencia del esquema.

Se generó una lista iterativa (del 1 al 3) para invocar la función sobre múltiples páginas de forma automática.

Pipeline de Limpieza:

Sustitución de valores nulos o caracteres especiales ("-") por valores numéricos calculables.

Configuración de Cultura/Locale (en-US) para garantizar que promedios y porcentajes se carguen correctamente sin importar la región del sistema.

Resultado Final
Tras el proceso de limpieza, los datos quedaron estructurados de la siguiente manera:

Volumen: Tabla consolidada de 106 filas y 15 columnas técnicas.

Calidad: Tipos de datos estrictos (Enteros para partidos/carreras y Decimales para promedios).

Listos para el Análisis: Limpieza completa de las columnas críticas: Player, Runs, Ave (Average) y SR (Strike Rate).

------------------------------------------------

Project Overview
This project showcases an advanced technical solution for ETL (Extract, Transform, Load) operations, sourcing sports statistics from the official ESPN Cricinfo website. The primary goal was to automate the capture of batting data using dynamic web scraping techniques within Power Query.

Workflow
Pattern-Based Scraping: "Extract table using examples" was utilized to identify the required CSS selectors within the source HTML.

M Language Modularization: The initial query was converted into a parameterized Custom Function (ps as text). This enables dynamic URL handling (&page=" & ps & ") to manage website pagination.

Robust Data Architecture: * A binary-compressed static header table was created to ensure schema consistency.

An iterative list (1 to 3) was generated to automatically invoke the function across multiple web pages.

Cleaning Pipeline:

Replacement of null values or special characters ("-") with calculable numeric values (0).

Culture/Locale (en-US) configuration to ensure averages and percentages load correctly regardless of the host system's regional settings.

Final Result
After the cleaning process, the data was structured as follows:

Volume: A consolidated table consisting of 106 rows and 15 technical columns.

Quality: Strict data typing (Integers for matches/runs and Decimals for averages).

Analysis-Ready: Complete sanitization of critical columns: Player, Runs, Ave (Average), and SR (Strike Rate).
