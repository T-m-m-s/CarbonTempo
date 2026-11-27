# CarbonTempo — Il suono delle emissioni

Progetto di **Data Science & Data Art** basato sul dataset [Our World in Data – CO₂ and Greenhouse Gas Emissions](https://github.com/owid/co2-data): analisi statistica in `R` (`tidyverse`) combinata con un sampler interattivo a 16 pad e 16 step (`Web Audio API`) capace di incidere ogni loop musicale su un **vinile 12" data-driven**.

**Gruppo:** Samuele Antonello (162640), Tommaso Trentin (167340), ...

---

## Contenuto della repository

| File | Descrizione |
|---|---|
| [`progetto.Rmd`](progetto.Rmd) | Notebook R Markdown con importazione, pulizia, statistiche, 4 grafici `ggplot2`, mappa dei pad e riflessioni finali |
| [`progetto.html`](progetto.html) | Rendering HTML dell'analisi |
| [`sampler.html`](sampler.html) | Opera interattiva offline (16 pad, 4 strumenti selezionabili, sequencer 16 step, `PLAY STORIA 1900→2050` e `INCIDI VINILE` PNG) |
| [`sampler_data.json`](sampler_data.json) | Parametri sonori dei 16 pad esportati da R (`freq_hz`, `freq_hz_log`, `gain_norm`, `decay_sec`, quote dei combustibili) |
| [`owid_co2.csv`](owid_co2.csv) | Dataset completo *Our World in Data – CO₂ and Greenhouse Gas Emissions* (1750–2025) incluso per riproducibilità immediata |
| [`presentazione_data_art.Rmd`](presentazione_data_art.Rmd) | Slide `ioslides` della presentazione in aula (max 10 minuti) |
| [`presentazione_data_art.html`](presentazione_data_art.html) | Rendering HTML delle slide |
| [`Struttura_e_idea.md`](Struttura_e_idea.md) | Appunti di progettazione sul dataset e sulle scelte di mappatura |

---

## Come aprire e provare il progetto

### 1. Opera artistica (`sampler.html`)
Apri direttamente **`sampler.html`** in qualsiasi browser moderno (Chrome, Firefox, Edge). Funziona al **100% offline** senza dipendenze esterne:
- Premi **`Spazio`** o **`AVVIA LOOP`** per ascoltare il preset dell'epoca selezionata (`1900`, `1950`, `1990`, `2025`, `2050`).
- Usa **`PLAY STORIA (1900→2050)`** per l'esecuzione cronologica automatica.
- Suona dal vivo i 16 pad con mouse/touch o tastiera (`1–4`, `Q–R`, `A–F`, `Z–V`) e cambia strumento per ciascuna colonna.
- Clicca **`INCIDI VINILE`** e **`Salva PNG`** per esportare il disco inciso dal loop.

### 2. Analisi e slide (`progetto.html` e `presentazione_data_art.html`)
Apri direttamente i due file `.html` nel browser.

Per rieseguire il **Knit** dei file `.Rmd` in RStudio (il file [`owid_co2.csv`](owid_co2.csv) è già presente nella cartella):
1. Assicurati di avere installati i pacchetti `tidyverse`, `jsonlite`, `prettydoc` e `rmarkdown`.
2. Apri `progetto.Rmd` (o `presentazione_data_art.Rmd`) e clicca **Knit**.

---

## Mappatura dati → suono & vinile

| Variabile OWID | Parametro | Effetto percepito |
|---|---|---|
| `co2_per_capita` | Altezza (`freq_hz_log`) | Mappatura lineare in semitoni (~2,7 semitoni per `+1 t/ab`) tra 65 e 585 Hz |
| `co2` (annuale) | Volume (`gain_norm`) | Radice quadrata normalizzata (`0.2–1.0`) per mantenere udibili i valori bassi |
| `cumulative_co2` | Durata & riverbero (`decay_sec`) | Coda persistente (`0.25–2.45 s`) proporzionale all'accumulo storico |
| `coal_co2`, `oil_co2`, `gas_co2` | Carattere timbrico & etichetta | Modellano filtro/armoniche e l'anello colorato sul vinile inciso |

## Trasparenza e fonti

- Tutti i valori di CO₂ (totali mondiali, per continente, pro capite, cumulativi, curva 1900–2025 e quote dei combustibili) provengono dal dataset **Our World in Data (1750–2025)**.
- Le anomalie termiche mostrate nel sampler (`+0.05 °C` … `+1.20 °C`) sono riferimenti esterni indicativi (NASA GISTEMP / HadCRUT rispetto all'era pre-industriale); lo scenario **2050 (`+2.50 °C`)** è una proiezione narrativa non contenuta nel CSV.
