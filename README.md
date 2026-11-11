# 🎬 What Makes a Movie Profitable?

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-1557b0?style=for-the-badge)
![Power Query](https://img.shields.io/badge/Power%20Query%20(M)-217346?style=for-the-badge)
![Format](https://img.shields.io/badge/format-.pbip-555?style=for-the-badge)

> **What makes a movie profitable?** Analysis of **8,035 movies** with reliable budget and box-office figures (after excluding 104 outliers), combining ratings, genres, countries and Academy Awards.

*The report itself (page names, measures, labels) is in Spanish.*

## 📌 At a glance

| | |
|---|---|
| Power Query queries | 31, in 6 groups (parameters → sources → preparation → integration → checks → model) |
| Model | Star schema with `Fact_Peliculas` at the center, 6 tables |
| DAX measures | 37 |
| Report | 10 analysis pages + original pages |

## 🔑 Key findings

1. **Profitable is not the same as successful.** 70% recover their investment, but only 42% reach 2.5x. Median ROI is 99% and the whole set grosses 2.9 times what was invested.
2. **Quality is the strongest driver.** Rating < 5: median ROI −24%. Rating ≥ 8: median ROI 483% and an 83% success rate.
3. **Oscars go hand in hand with success.** The 238 winners have a median ROI of 625% and a 93% success rate.
4. **Budget follows a U-shape.** 15–40 M is the riskiest range (ROI 55%). At 100 M or more, 88% are profitable.
5. **Horror is the most efficient genre:** 158% median ROI on a median budget of 9 M.
6. **Each decade returns less:** from 219% in the 1980s to 25% in the 2020s.
7. **The median is the right metric:** without outliers the average ROI is 372%, the median 99%.

**Recommendation:** bet on quality and pick one end of the budget range (low cost in genres with a loyal audience such as horror, or big animation and adventure productions). Avoid mid-range budgets in drama, thriller and history.

## 🗂️ Sources

All files are read from the [`Carpeta_Datos`](Carpeta_Datos) folder, set in the `Carpeta_Datos` Power Query parameter.

| File | Provides |
|---|---|
| `The_movies.xlsx` | Title, year, genre, subgenres, budget, box office, director, studios |
| `Puntajes_con_Calificaciones_y_Criticas.xlsx` | TMDb rating, votes, popularity |
| `Zona_Geografica_y_ROI.xlsx` | Production countries |
| `oscars_1927_a_presente.csv` | Academy Awards by year and category |

## 🧱 Data model

```
                    Puente_Pelicula_Subgenero
                               │ (N:1, both)
Puente_Pelicula_Pais ──► Fact_Peliculas ◄── Fact_Oscars
     (N:1, both)               ▲            (N:1, single)
                               │ (N:1, both)
                        Oscar_Ganadores (DAX)
```

- Simplified from 11 to 6 tables by removing single-column dimensions.
- Bridge tables let a movie count in **all** of its subgenres and countries (13,275 movie–subgenre assignments).

## 🧪 Notable techniques

- **Accent repair** (`AcciÃ³n` → `Acción`): an M function that re-decodes Windows-1252 → UTF-8 row by row, with a Latin-1 fallback.
- **Oscar category normalization:** category names the Academy changed over the years are unified into categories and areas.
- **Outlier detection with Tukey's rule** on `log10(box office / budget)` (Q1 − 3·IQR and Q3 + 3·IQR limits, rounded to 0.5% and 500x).
- **Inflation adjustment** to 2025 dollars using the US CPI.
- **Check queries** (counts, duplicates, matches) that are not loaded into the model.

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

## 📄 Report pages

| # | Page | Question |
|---|---|---|
| 1 | Resumen | What makes a movie profitable? |
| 2 | Géneros | Which genres perform best, and with what risk? |
| 3 | Presupuesto | Does spending more pay off? |
| 4 | Calidad y Óscar | Does quality translate into box office? |
| 5 | Países, estudios y directores | Who produces the hits? |
| 6 | Subgéneros (control) | Are all subgenres counted? |
| 7 | Ganadores del Óscar | Who won, and for which movie? |
| 8 | Casos atípicos | What was excluded, and why? |
| 9 | Género × presupuesto y tiempo | Where is it worth investing? |
| 10 | Conclusiones | What did we learn? |

## ▶️ How to open it

1. Clone the repo and open `Rentabilidad Cine.pbip` in Power BI Desktop (or `Rentabilidad Cine.pbix` directly).
2. Under **Transform data → Parameters**, point `Carpeta_Datos` to the repo's `Carpeta_Datos` folder.
3. Refresh.

> The full write-up of the process (in Spanish) is in [`Proyecto Power BI Cine qué hicimos y cómo.docx`](Proyecto%20Power%20BI%20Cine%20qu%C3%A9%20hicimos%20y%20c%C3%B3mo.docx).
