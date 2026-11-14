<img src="./imagenes/avista2.png" width="250" height="100" />

# Prueba técnica: análisis y pronóstico de transacciones

Análisis de datos transaccionales de una entidad bancaria para entender el volumen por operación y terminal y pronosticar la demanda diaria de los siguientes 15 días. La solución usa el histórico disponible hasta el 31 de mayo de 2017 y genera el pronóstico del 1 al 15 de junio de 2017.

## Contenido del repositorio

- [`Notebooks/Prueba_Analisis_Datos.ipynb`](Notebooks/Prueba_Analisis_Datos.ipynb): análisis exploratorio, validación y modelo predictivo.
- [`data/data_01/data_01.txt`](data/data_01/data_01.txt): registros de transacciones originales.
- [`data/pronostico_1_15_jun_2017.csv`](data/pronostico_1_15_jun_2017.csv): pronóstico por fecha, terminal y operación, con límites inferior y superior del intervalo del 80 %.
- [`Notebooks/imagenes/`](Notebooks/imagenes/): gráficos y archivos auxiliares generados para el análisis.
- [`doc/`](doc/): enunciado, informe ejecutivo y otros documentos de soporte de la prueba.

## Cómo reproducir el análisis

Se requiere Python y Jupyter. Instala las bibliotecas utilizadas por el notebook:

```bash
python -m pip install numpy pandas matplotlib scipy statsmodels lightgbm jupyter
```

Desde la carpeta raíz del repositorio, abre el notebook con su carpeta como directorio de trabajo:

```bash
cd Notebooks
jupyter lab Prueba_Analisis_Datos.ipynb
```

Ejecuta las celdas en orden. El notebook busca los datos en `../data/data_01/data_01.txt` cuando se ejecuta desde `Notebooks` y guarda los gráficos en `Notebooks/imagenes/`.

## Desarrollo y metodología

1. **Calidad y exploración:** se revisan las fechas, el volumen diario, la distribución por día de semana y hora, el calendario de pagos y festivos, y la concentración por operación y terminal.
2. **Selección de series:** para el análisis detallado del pronóstico se eligen cinco operaciones y cinco terminales. Para entrenar el modelo agrupado de LightGBM se aprovechan las series completas de cinco operaciones en 44 terminales.
3. **Modelado:** se comparan un pronóstico ingenuo de la semana anterior, la media reciente por día de semana, un GLM Poisson con variables de calendario, LightGBM Poisson y combinaciones de estos modelos. El ensamble final promedia GLM, LightGBM y la media semanal reciente.
4. **Validación:** se realizan cuatro backtests con origen móvil; en cada uno se pronostican 15 días posteriores al corte sin usar datos futuros. Se comparan WAPE, MAE, RMSE y el error del total de la ventana.
5. **Pronóstico:** el ensamble se vuelve a ajustar con el histórico disponible y produce estimaciones para el 1–15 de junio de 2017.

## Principales insights

- **El calendario explica buena parte de la demanda.** Se observan patrones semanales y picos en días de pago y el día hábil siguiente. Los domingos registran alrededor del 28 % del volumen de un miércoles; los días de pago pueden aumentar aproximadamente entre 41 % y 49 % el volumen frente a días comparables. Los festivos presentan caídas marcadas.
- **El volumen está concentrado.** Las cinco operaciones principales reúnen cerca del 92 % de las transacciones y la operación 0, aproximadamente el 55 %. También hay diferencias importantes de utilización entre terminales (alrededor de 7 veces entre extremos).
- **El ensamble mejora el pronóstico diario.** En los backtests, su WAPE diario se sitúa aproximadamente entre 33 % y 37 %, frente a cerca de 55 % para el modelo ingenuo semanal y 42 % para la media por día de semana. El sesgo promedio es moderado, alrededor de +5 %.
- **La agregación mejora la precisión útil para planificar.** El WAPE del total de 15 días por serie es cercano al 13 %, prácticamente empatado con la media semanal reciente. Para el volumen diario agregado de las 220 series, el error es de aproximadamente 20 %.
- **El error por serie y día sigue siendo alto.** El WAPE diario por serie ronda el 35 %. El calendario ayuda a anticipar patrones, pero no permite prever fallas o indisponibilidad de terminales ni el efecto de festivos no observados suficientemente.

## Uso y limitaciones

El pronóstico puede orientar la planeación de capacidad y personal alrededor de los días de pago, apoyar la programación de mantenimiento en periodos de menor demanda y servir como referencia para detectar desviaciones inusuales comparando las transacciones reales con el intervalo pronosticado.

Los resultados deben interpretarse considerando que solo hay cerca de cuatro meses de historia: no se puede estimar una estacionalidad anual y hay pocos festivos para aprender su efecto. Además, las transacciones observadas pueden no representar toda la demanda, no se cuenta con variables de disponibilidad de terminal ni con un diccionario de operaciones, y la selección se concentra en series de mayor volumen. Para mejorar el modelo se recomienda ampliar el histórico, incorporar disponibilidad y campañas, reentrenar periódicamente y monitorear el error por terminal.
