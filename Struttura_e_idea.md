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

### Passo 1: separare i paesi dagli aggregati

**Cosa facciamo.** Teniamo le righe in cui `iso_code` è compilato ed escludiamo quelle in cui è vuoto (NA).

**Perché.** Il file mescola i singoli paesi con aggregati come Europa, Asia e Africa. Le emissioni dell'Italia, per esempio, sono già comprese in quelle dell'Europa e del mondo: sommando tutto le conteremmo più volte. Nel 2025 la somma di tutte le righe, escluso `World`, è circa **208.854 Mt**, cioè più di 5 volte il totale mondiale (**38.995 Mt**).

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

**Da decidere**

- **Kosovo:** il filtro lo esclude anche se è un paese singolo. Proposta: recuperarlo a mano.
- **Continenti:** non vanno sommati ai paesi, ma le loro righe possono servire a parte per confrontare le grandi aree.
- **Aviazione e navigazione:** restano fuori dai paesi; per il totale mondiale conviene usare la riga `World`.
