# 🎬 Rentabilidad y factores de éxito en la industria cinematográfica

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-1557b0?style=for-the-badge)
![Power Query](https://img.shields.io/badge/Power%20Query%20(M)-217346?style=for-the-badge)
![Formato](https://img.shields.io/badge/formato-.pbip-555?style=for-the-badge)

> **¿Qué hace rentable a una película?** Análisis de **8.035 películas** con presupuesto y taquilla confiables (después de excluir 104 casos atípicos), cruzando calificaciones, géneros, países y premios Óscar.

## 📌 Resumen

| | |
|---|---|
| Consultas Power Query | 31, en 6 grupos (parámetros → fuentes → preparación → integración → controles → modelo) |
| Modelo | Estrella con `Fact_Peliculas` al centro, 6 tablas |
| Medidas DAX | 37 |
| Informe | 10 páginas de análisis + páginas originales |

## 🔑 Hallazgos principales

1. **Rentable no es lo mismo que exitoso.** El 70 % recupera la inversión, pero solo el 42 % llega a 2,5x. El ROI mediano es 99 % y el conjunto recauda 2,9 veces lo invertido.
2. **La calidad es el factor más fuerte.** Calificación < 5: ROI mediano −24 %. Calificación ≥ 8: ROI mediano 483 % y 83 % de éxito.
3. **El Óscar acompaña al éxito.** Las 238 premiadas tienen ROI mediano de 625 % y 93 % de éxito.
4. **El presupuesto tiene forma de U.** De 15 a 40 M es la zona más riesgosa (ROI 55 %). Con 100 M o más, el 88 % es rentable.
5. **Terror es el género más eficiente:** 158 % de ROI mediano con un presupuesto mediano de 9 M.
6. **Cada década rinde menos:** de 219 % en los 80 a 25 % en los 2020.
7. **La mediana es el indicador correcto:** sin atípicos el ROI promedio es 372 %, la mediana 99 %.

**Recomendación:** apostar por la calidad y elegir un extremo del presupuesto (bajo costo en géneros con público fiel como terror, o grandes producciones de animación y aventura). Evitar el presupuesto medio en drama, suspenso e historia.

## 🗂️ Fuentes

Todas se leen desde la carpeta [`Carpeta_Datos`](Carpeta_Datos), definida en el parámetro `Carpeta_Datos` de Power Query.

| Archivo | Aporta |
|---|---|
| `The_movies.xlsx` | Título, año, género, subgéneros, presupuesto, taquilla, director, estudios |
| `Puntajes_con_Calificaciones_y_Criticas.xlsx` | Calificación TMDb, votos, popularidad |
| `Zona_Geografica_y_ROI.xlsx` | Países de producción |
| `oscars_1927_a_presente.csv` | Premios Óscar por año y categoría |

## 🧱 Modelo de datos

```
                    Puente_Pelicula_Subgenero
                               │ (N:1, ambas)
Puente_Pelicula_Pais ──► Fact_Peliculas ◄── Fact_Oscars
     (N:1, ambas)              ▲            (N:1, única)
                               │ (N:1, ambas)
                        Oscar_Ganadores (DAX)
```

- Se simplificó de 11 a 6 tablas eliminando dimensiones de una sola columna.
- Las tablas puente permiten que una película cuente en **todos** sus subgéneros y países (13.275 asignaciones película-subgénero).

## 🧪 Técnicas destacadas

- **Reparación de acentos** (`AcciÃ³n` → `Acción`): función M que re-decodifica Windows-1252 → UTF-8 por fila, con fallback a Latin-1.
- **Normalización de categorías del Óscar**: los nombres que la Academia cambió con los años se unifican en categorías y áreas.
- **Detección de atípicos con la regla de Tukey** sobre `log10(taquilla / presupuesto)` (límites Q1 − 3·IQR y Q3 + 3·IQR, redondeados a 0,5 % y 500x).
- **Ajuste por inflación** con IPC de EE.UU. a dólares de 2025.
- **Consultas de control** (conteos, duplicados, coincidencias) que no se cargan al modelo.

```dax
ROI Mediano =
MEDIANX (
    FILTER ( Fact_Peliculas, Fact_Peliculas[Es_Evaluable_Financiera] = TRUE () ),
    DIVIDE (
        Fact_Peliculas[Ingresos_usd] - Fact_Peliculas[Presupuesto_usd],
        Fact_Peliculas[Presupuesto_usd]
    )
)
```

## 📄 Páginas del informe

| # | Página | Pregunta |
|---|---|---|
| 1 | Resumen | ¿Qué hace rentable a una película? |
| 2 | Géneros | ¿Qué géneros rinden más y con qué riesgo? |
| 3 | Presupuesto | ¿Conviene gastar más? |
| 4 | Calidad y Óscar | ¿La calidad se traduce en taquilla? |
| 5 | Países, estudios y directores | ¿Quién produce los éxitos? |
| 6 | Subgéneros (control) | ¿Se cuentan todos los subgéneros? |
| 7 | Ganadores del Óscar | ¿Quién ganó y por qué película? |
| 8 | Casos atípicos | ¿Qué se excluyó y por qué? |
| 9 | Género × presupuesto y tiempo | ¿Dónde conviene invertir? |
| 10 | Conclusiones | ¿Qué aprendimos? |

## ▶️ Cómo abrirlo

1. Clonar el repo y abrir `Rentabilidad Cine.pbip` con Power BI Desktop (o directamente `Rentabilidad Cine.pbix`).
2. En **Transformar datos → Parámetros**, apuntar `Carpeta_Datos` a la carpeta `Carpeta_Datos` del repo.
3. Actualizar.

> El documento completo del proceso está en [`Proyecto Power BI Cine qué hicimos y cómo.docx`](Proyecto%20Power%20BI%20Cine%20qu%C3%A9%20hicimos%20y%20c%C3%B3mo.docx).
