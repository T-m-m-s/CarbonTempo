## Dataset di partenza

Il progetto parte da `owid_co2.csv`, il dataset **CO₂ and Greenhouse Gas Emissions** di Our World in Data, qui descritto prima di qualsiasi modifica. Ha **56.269 righe** e **79 colonne** e va dal **1750** al **2025**. Ogni riga è un paese o un'area in un certo anno, senza coppie `country`–`year` duplicate.

Tre caratteristiche contano per l'analisi:

- **Paesi e aggregati sono mescolati.** La colonna `country` contiene anche mondo, continenti, gruppi di reddito e voci come i trasporti internazionali. Sommare tutte le righe conterebbe più volte le stesse emissioni.
- **`iso_code` non basta sempre a distinguerli.** 48.306 righe hanno un codice e 7.963 no. Tra quelle senza codice ci sono però anche territori come il Kosovo e le isole Ryukyu: filtrando solo su `iso_code` vengono esclusi.
- **La copertura cambia molto.** I dati antichi sono molto incompleti e non tutte le colonne arrivano al 2025. Per i paesi, `co2` è presente nel 93% delle righe dal 1950, contro il 54% delle righe dell'intero file.

### Quali colonne tenere

La scelta dipende dall'obiettivo: confrontare le emissioni tra aree ed epoche e trasformarle in suono. Le percentuali indicano quante righe dei paesi (con `iso_code`) hanno il dato.

| Colonna | Cosa contiene | Dal 1950 | Nel 2025 | Perché tenerla |
|---|---|---|---|---|
| `country` | Paese o area | 100% | 100% | Identifica la riga |
| `year` | Anno | 100% | 100% | Serve per il confronto nel tempo |
| `iso_code` | Codice del paese | 100% | 100% | Aiuta a separare paesi e aggregati |
| `population` | Persone | 99% | 99% | Dà la scala e spiega il pro capite |
| `co2` | Emissioni annue (milioni di t) | 93% | 99% | È la misura principale |
| `co2_per_capita` | Emissioni per persona (t) | 93% | 98% | Permette di confrontare paesi di dimensioni diverse |
| `cumulative_co2` | Emissioni accumulate dal 1750 (milioni di t) | 93% | 99% | Racconta la responsabilità storica |
| `coal_co2` | Da carbone (milioni di t) | 63% | 62% | Mostra il combustibile dominante |
| `oil_co2` | Da petrolio (milioni di t) | 93% | 98% | Come sopra |
| `gas_co2` | Da gas (milioni di t) | 48% | 58% | Come sopra |

Per carbone, petrolio e gas i vuoti vanno trattati con cautela: un dato mancante non equivale con certezza a zero emissioni.

**Utili solo se si allarga il progetto:**

- `temperature_change_from_co2`: collega le emissioni al riscaldamento, ma arriva al 2024.
- `co2_including_luc`: include l'uso del suolo (per esempio la deforestazione), che può cambiare il confronto tra paesi. Parte dal 1850.
- `share_global_co2`: la quota sul totale mondiale, comoda per le classifiche.

**Da lasciare fuori per ora:**

| Gruppo | Perché |
|---|---|
| `gdp` e colonne `*_per_gdp` | Arrivano al massimo al 2022 e non servono all'obiettivo |
| Energia (`energy_*`, `primary_energy_consumption`, `*_per_unit_energy`) | Partono dal 1965 e si fermano al 2024 |
| `consumption_co2*`, `trade_co2*` | Partono dal 1990 e hanno molti vuoti |
| Metano, protossido di azoto, gas serra totali | Si fermano al 2024 e spostano il tema oltre la CO₂ |
| Cemento, flaring, altre industrie | Fonti minori: nel 2025, a livello mondiale, circa il 6% di `co2` in totale |
| Altre colonne `*_growth_*`, `share_global_*`, `cumulative_*`, `*_per_capita` | Dettagli o valori derivati che non servono alle domande del progetto |

### Esempio: alcune righe dell'Italia

Sono mostrate solo le colonne da tenere.

| country | year | iso_code | population | co2 | co2_per_capita | cumulative_co2 | coal_co2 | oil_co2 | gas_co2 |
|---|---|---|---|---|---|---|---|---|---|
| Italy | 1900 | ITA | 33332371 | 13.909 | 0.417 | 258.495 | 13.637 | 0.271 | 0.000 |
| Italy | 1950 | ITA | 46391674 | 41.262 | 0.889 | 1697.562 | 24.574 | 13.256 | 0.956 |
| Italy | 1990 | ITA | 57031457 | 438.208 | 7.684 | 11891.367 | 69.580 | 254.823 | 88.992 |
| Italy | 2025 | ITA | 59146263 | 293.920 | 4.969 | 26291.516 | 9.894 | 145.964 | 126.969 |

## Pulizia dei dati

### Passo 1: selezionare le colonne

**Cosa facciamo.** Da 79 colonne passiamo alle 10 indicate sopra (`data_selected`: 56.269 righe).

**Perché.** Sono quelle che servono per dire chi emette, quando, quanto e con quale combustibile.

### Passo 2: separare i paesi dagli aggregati

**Cosa facciamo.** Teniamo le righe in cui `iso_code` è compilato ed escludiamo quelle in cui è vuoto (NA).

**Perché.** Il file mescola i singoli paesi con aggregati come Europa, Asia, Africa , ecc. Le emissioni dell'Italia, per esempio, sono già comprese in quelle dell'Europa e del mondo: sommando tutto le conteremmo più volte.

**Risultato del filtro**

| Gruppo | Righe | Nomi diversi | Cosa contiene |
|---|---|---|---|
| Con `iso_code` (teniamo) | 48.306 | 223 | Solo paesi e territori singoli, nessun aggregato |
| Senza `iso_code` (escludiamo) | 7.963 | 36 | Aggregati e voci speciali |

Dopo il filtro **non rimangono agglomerati**. I 223 nomi non sono però tutti Stati sovrani: ci sono anche territori (per esempio Groenlandia, Hong Kong, Guyana francese) e l'Antartide. Sono aree distinte, quindi non creano doppi conteggi.

Le 36 voci escluse sono:

| Tipo | Voci |
|---|---|
| Mondo e continenti | World, Africa, Asia, Europe, North America, South America, Oceania, più le versioni "(GCP)" dei sei continenti |
| Altri raggruppamenti geografici | Central America (GCP), Middle East (GCP), Asia (excl. China and India), Europe (excl. EU-27), Europe (excl. EU-28), North America (excl. USA) |
| Gruppi politici ed economici | European Union (27 e 28), OECD (due versioni), Non-OECD, i quattro gruppi di reddito, Least developed countries |
| Voci non attribuite a un paese | International aviation, International shipping, Kuwaiti Oil Fires (due versioni) |
| Territori singoli senza codice | **Kosovo**, **Ryukyu Islands** (due versioni) |

**Controllo sui totali (2025)**

| Voce | CO₂ (Mt) |
|---|---|
| Somma delle righe con `iso_code` | 37.748 |
| Aviazione internazionale | 581 |
| Navigazione internazionale | 659 |
| Kosovo | 8 |
| **Totale** (calcolato sui valori non arrotondati) | **38.995** |
| Riga `World` | 38.995 |

I conti tornano: nel 2025 non ci sono doppi conteggi e non manca altro.

Il risultato del filtro è `data_countries`: **48.306 righe** e 10 colonne.

### Passo 3: valori mancanti dei combustibili

**Cosa facciamo.** Se `co2` esiste, mettiamo 0 al posto dei valori mancanti di `coal_co2`, `oil_co2` e `gas_co2`. Se manca anche `co2`, lasciamo NA.

**Perché.** Assumiamo che il vuoto significhi "nessuna emissione registrata". È un'ipotesi, non una certezza. Dove non c'è nemmeno il totale, mettere 0 vorrebbe dire inventare un dato.

| Colonna | Valori sostituiti con 0 |
|---|---|
| `coal_co2` | 5.791 |
| `oil_co2` | 1.816 |
| `gas_co2` | 9.487 |

### Passo 4: dataset di mondo e continenti

**Cosa facciamo.** Creiamo `data_continents` con `World` e i sei continenti: **1.932 righe** (7 aree per 276 anni). Partiamo da `data_selected`, che contiene ancora gli aggregati.

**Perché.** Serve per confrontare le grandi aree senza sommare i paesi a mano. Questa tabella va tenuta separata da `data_countries`, per non contare due volte le stesse emissioni. In questa tabella i vuoti dei combustibili non vengono sostituiti.

### Limiti da tenere presenti

- **Kosovo:** non ha `iso_code`, quindi è escluso da `data_countries` anche se è un paese singolo.
- **Aviazione e navigazione:** non sono assegnate ai paesi; per il totale mondiale va usata la riga `World`.

## L'idea: dai dati al suono

Ogni coppia **area + periodo** diventa un suono, cioè un pad del sampler. Ascoltando i pad si confrontano le emissioni di continenti ed epoche diverse.

### Come si arriva alla tabella finale (`sampler_pads`)

1. Si parte da `data_continents`.
2. Si tengono 4 aree (Europe, North America, Asia, Africa) e 4 anni (1900, 1950, 1990, 2025): **16 righe**.
3. I valori mancanti dei combustibili diventano 0 e si individua il combustibile dominante.
4. Si calcolano i parametri del suono:

| Dato | Parametro | Calcolo |
|---|---|---|
| `co2_per_capita` | frequenza | da 65 a 585 Hz (vedi sotto) |
| `co2` | volume (`gain_norm`) | da 0,2 a 1, con la radice quadrata del valore diviso il massimo |
| `cumulative_co2` | durata (`decay_sec`) | da 0,25 a 2,45 s, in proporzione al massimo |
| combustibile dominante | timbro | carbone, petrolio o gas |

I massimi (e il minimo) sono calcolati sulle sole 16 righe.

### Come si calcola la frequenza

**Versione lineare (`freq_hz`)**

```r
freq_hz = round(65 + (co2_per_capita / max(co2_per_capita, na.rm = TRUE)) * 520, 1)
```

**Versione logaritmica attuale nell'Rmd (`freq_hz_log`): sbagliata**

```r
freq_hz_log = round(65 * (585 / 65)^(co2_per_capita / max(co2_per_capita, na.rm = TRUE)), 1)
```

L'esponente è proporzionale a `co2_per_capita`, non al suo logaritmo. La formula è quindi esponenziale rispetto al dato: a uguali *differenze* di emissioni corrispondono uguali intervalli musicali, mentre noi vogliamo che accada per uguali *rapporti*. La frase dell'Rmd sui rapporti non è vera con questa formula.

**Versione corretta (da inserire nell'Rmd)**

```r
log_norm = (log(co2_per_capita) - log(min(co2_per_capita, na.rm = TRUE))) /
           (log(max(co2_per_capita, na.rm = TRUE)) - log(min(co2_per_capita, na.rm = TRUE))),
freq_hz_log = round(65 * (585 / 65)^log_norm, 1)
```

`log_norm` porta il logaritmo del dato tra 0 (valore minimo, 65 Hz) e 1 (valore massimo, 585 Hz). Così lo stesso rapporto tra due emissioni dà sempre lo stesso intervallo. Richiede valori maggiori di zero.

**Confronto sui 16 pad** (min 0,017 e max 14,158 t per persona)

| Area | Anno | t per persona | Lineare (Hz) | Log attuale (Hz) | Log corretta (Hz) |
|---|---|---|---|---|---|
| Europe | 1900 | 3,008 | 175,5 | 103,7 | 352,7 |
| Europe | 1950 | 4,328 | 224,0 | 127,2 | 397,2 |
| Europe | 1990 | 11,112 | 473,1 | 364,6 | 540,5 |
| Europe | 2025 | 6,499 | 303,7 | 178,2 | 453,6 |
| North America | 1900 | 6,680 | 310,3 | 183,3 | 457,7 |
| North America | 1950 | 12,458 | 522,6 | 449,3 | 561,1 |
| North America | 1990 | 14,158 | 585,0 | 585,0 | 585,0 |
| North America | 2025 | 10,166 | 438,4 | 314,8 | 525,0 |
| Asia | 1900 | 0,047 | 66,7 | 65,5 | 90,6 |
| Asia | 1950 | 0,290 | 75,7 | 68,0 | 164,2 |
| Asia | 1990 | 2,056 | 140,5 | 89,4 | 311,4 |
| Asia | 2025 | 4,876 | 244,1 | 138,5 | 413,0 |
| Africa | 1900 | 0,017 | 65,6 | 65,2 | 65,0 |
| Africa | 1950 | 0,410 | 80,1 | 69,3 | 183,9 |
| Africa | 1990 | 1,027 | 102,7 | 76,2 | 248,2 |
| Africa | 2025 | 1,007 | 102,0 | 76,0 | 246,6 |

### Proposta: periodi al posto dei singoli anni (da discutere)

Invece di 4 anni singoli, usare 4 intervalli di anni. Da decidere insieme se abbiano senso intervalli di uguale durata, perché nei primi secoli molti dati mancano (a volte c'è solo la popolazione) e le frequenze potrebbero risultare falsate.

## Il sampler: com'è ora e dove vogliamo arrivare

### Come funziona adesso

L'artifact (`Climate Data Sampler - OWID CO₂ Data Art (1).html`) è una pagina web che imita un sampler da produzione musicale. I suoni sono generati dal browser, senza file audio.

**I pad.** Sono 16, in una griglia 4 × 4: le **righe** sono gli anni (1900, 1950, 1990, 2025), le **colonne** le aree (Europe, North America, Asia, Africa). Si suonano con il mouse, il tocco o la tastiera (da `1`–`4` fino a `Z`–`V`). Ogni pad è una nota sintetizzata a partire dai dati:

| Caratteristica del suono | Dato |
|---|---|
| Altezza | CO₂ pro capite (per ora con la formula lineare `freq_hz`) |
| Volume | CO₂ totale |
| Durata | CO₂ cumulativa |
| Timbro (forma d'onda) | Combustibile dominante: carbone a dente di sega, petrolio quadra, gas sinusoidale |

**Il loop.** Un sequencer a 8 passi permette di accendere i pad nei vari passi, scegliere i BPM e far ripetere il pattern.

**Le epoche.** Si può scegliere tra 1900s, 1950s, 1980s, 2020s e una proiezione al 2050. La scelta cambia gli effetti generali (filtro, distorsione, riduzione dei bit), il colore dello sfondo e il testo di contesto. Il pulsante del preset carica un pattern d'esempio, più fitto e veloce nelle epoche recenti. Gli effetti si regolano anche a mano.

**L'opera visiva.** "Create Artwork" registra un giro del loop e lo trasforma in un'immagine ad anelli, salvabile in PNG.

**Da sapere.** I dati dei pad sono copiati dentro l'HTML. I valori delle epoche (emissioni globali, anomalia di temperatura, quote dei combustibili) sono scritti a mano nella pagina e non calcolati dall'Rmd: vanno verificati prima di presentarli come dati.

### Cosa vogliamo fare

- **Colonne = continenti, ognuno con uno strumento.** Per esempio basso o chitarra, con la possibilità di scegliere quale strumento assegnare a ogni continente.
- **Righe = epoche.** Al posto dei singoli anni, i periodi di cui sopra.
- **Obiettivo finale: comporre canzoni diverse a seconda del periodo** e raccontare così, attraverso la data art, come è cambiato il consumo (le emissioni) di CO₂.

**Da definire**

- Quali periodi usare e come riassumere i dati di ciascuno.
- Quali strumenti offrire e come produrli (sintetizzati o con campioni audio).
- Cosa resta guidato dai dati con gli strumenti: per esempio altezza, volume e durata. Il timbro dipenderebbe dallo strumento, quindi serve un altro modo per far sentire il combustibile dominante, come un effetto.
