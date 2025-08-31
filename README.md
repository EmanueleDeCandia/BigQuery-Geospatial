# BigQuery-Geospatial

# Visualizzazione Geospatial di BigQuery in Colab

[Google Colaboratory logo Apri in Colab](https://colab.research.google.com/github/GoogleCloudPlatform/bigquery-utils/blob/master/notebooks/bigquery_geospatial_visualization.ipynb) | [Google Cloud Colab Enterprise logo Apri in Colab Enterprise](https://console.cloud.google.com/vertex-ai/colab/import/https:%2F%2Fraw.githubusercontent.com%2FGoogleCloudPlatform%2Fbigquery-utils%2Fmaster%2Fnotebooks%2Fbigquery_geospatial_visualization.ipynb) | [BigQuery Studio logo Apri in BigQuery Studio](https://console.cloud.google.com/bigquery/import?url=https://github.com/GoogleCloudPlatform/bigquery-utils/blob/master/notebooks/bigquery_geospatial_visualization.ipynb) | [GitHub logo Visualizza su GitHub](https://github.com/GoogleCloudPlatform/bigquery-utils/blob/master/notebooks/bigquery_geospatial_visualization.ipynb) |

## Introduzione

L'obiettivo di questo notebook è illustrare i passaggi per visualizzare dati geospatiali in un notebook [Colab](https://colab.research.google.com/). I dati geospatiali verranno ottenuti da [Google BigQuery](https://cloud.google.com/bigquery).

Esamineremo i seguenti argomenti:
* Autenticazione su Google Cloud
* Lettura dei dati da BigQuery in Colab
* Utilizzo di strumenti Python per la data science per eseguire trasformazioni e analisi
* Utilizzo di `geemap` e `pydeck` per renderizzare scatter plot, poligoni, coropleth e heatmap

## Dataset e Obiettivi di Analisi

I dataset con cui lavoreremo includono:

* [San Francisco Ford GoBike Share](https://console.cloud.google.com/bigquery(cameo:product/san-francisco-public-data/sf-bike-share)
* [San Francisco Neighborhoods](https://console.cloud.google.com/bigquery?ws=!1m4!1m3!3m2!1sbigquery-public-data!2ssan_francisco_neighborhoods)
* [San Francisco Police Department (SFPD) Reports](https://console.cloud.google.com/bigquery(cameo:product/san-francisco-public-data/sfpd-reports))

Questi [dataset pubblici](https://cloud.google.com/bigquery/public-data) sono ospitati in Google BigQuery e sono inclusi nel livello gratuito di BigQuery di 1TB/mese di elaborazione. Ciò significa che ogni utente riceve 1TB di elaborazione BigQuery gratuita ogni mese, che può essere utilizzata per eseguire query su questo dataset pubblico. Leggi questa [pagina di documentazione](https://cloud.google.com/bigquery/docs/sandbox) per scoprire come iniziare rapidamente a utilizzare BigQuery per accedere ai dataset pubblici.

I nostri obiettivi sono interrogare e tracciare le seguenti informazioni a San Francisco:
* Tutte le stazioni di bike sharing utilizzando `ScatterplotLayer`
* Tutti i poligoni dei quartieri utilizzando un `GeoJSONLayer`
* Coropleth della densità delle stazioni di bike sharing (stazioni per chilometro quadrato) per quartiere utilizzando un `PolygonLayer`
* Densità continua degli incidenti SFPD utilizzando un `HeatmapLayer`

## Strumenti/Tecniche di Data Science Correlati

In ogni passaggio del percorso sopra descritto, mostriamo come BigQuery e le librerie Python per la data science e la creazione di grafici (tramite Colab) possano lavorare insieme per consentire questo tipo di analisi su larga scala. Alcuni strumenti e tecniche chiave che impiegheremo:
* [colabtools](https://github.com/googlecolab/colabtools) (moduli python `google.colab`)
* [Google BigQuery](https://cloud.google.com/bigquery/what-is-bigquery)
* Librerie Python per la Data Science: [pandas](https://pandas.pydata.org/)
* Librerie Python Geospatiali:
  * [geopandas](https://geopandas.org/en/stable/index.html) per estendere i tipi di dati utilizzati da `pandas` per consentire operazioni spaziali su tipi geometrici
  * [shapely](https://shapely.readthedocs.io/en/stable/index.html) per la manipolazione e l'analisi di singoli oggetti geometrici planari
* [branca](https://python-visualization.github.io/branca/) per generare colormap HTML + JS
* [pydeck](https://deckgl.readthedocs.io/en/latest/) per la visualizzazione di grafici basata su [deck.gl](https://deck.gl/#/)
* [geemap](https://geemap.org/) per la visualizzazione con `pydeck` e `earthengine-api`
  * modulo [geemap.deck](https://geemap.org/deck/)
  * [codice sorgente github](https://github.com/gee-community/geemap)
* [h3](https://uber.github.io/h3-py/intro.html) per il sistema di indicizzazione geospatiale gerarchica

## Prezzi di BigQuery e Costo di Esecuzione di Questo Notebook

Se non hai già un progetto Google Cloud, sono disponibili 2 opzioni gratuite per accedere a questi dati:

1. Iscriviti a [BigQuery sandbox](https://cloud.google.com/bigquery/docs/sandbox) per provarlo senza abilitare la fatturazione.
2. Se vuoi sperimentare più prodotti Google Cloud, attiva la [prova gratuita](https://cloud.google.com/free/) ($300 di credito per un massimo di 90 giorni).

Puoi utilizzare il [livello gratuito di BigQuery](https://cloud.google.com/bigquery/pricing#free-tier) per archiviare e analizzare una certa quantità di dati senza costi (1 TB di query, 10 GB di capacità di archiviazione al mese), anche dopo un periodo di prova gratuita.

L'esecuzione completa di questo notebook, con i valori predefiniti, dovrebbe rientrare nel livello di utilizzo gratuito e non dovrebbe costarti nulla. Tuttavia, la modifica delle impostazioni predefinite e/o l'esecuzione multipla potrebbero comportare costi aggiuntivi. Consulta la [guida ai prezzi di BigQuery](https://cloud.google.com/bigquery/pricing) per maggiori informazioni.

Questo non è un prodotto ufficiale di Google, ma un codice di esempio fornito a scopo didattico.

## Aiuta Google a migliorare i Tutorial

* Segnala problemi [qui](https://github.com/GoogleCloudPlatform/bigquery-utils/issues) su Github.
* Invia feedback [qui](https://docs.google.com/forms/d/e/1FAIpQLSdNSDbBM2rohqtqWtIPgKD_14oLc8TPqqdtKD11oqsrtA8_NQ/viewform) tramite un modulo Google.

# Configurazione

## Autenticazione GCP

Configuriamo il nostro Colab installando, importando e abilitando l'utilizzo di alcune librerie Python all'interno di Colab, oltre ad autenticare questo runtime di Colab con l'appropriato `GCP_PROJECT_ID` di Google Cloud. Questo segue da vicino le istruzioni nell'esempio Colab ["Getting started with BigQuery"](https://colab.sandbox.google.com/notebooks/bigquery.ipynb#scrollTo=SeTJb51SKs_W).

Il passaggio di autenticazione nella cella successiva richiederà di passare manualmente attraverso alcune finestre pop-up e copiare/incollare un codice di autenticazione da un'altra finestra nella cella per completare (alla prima esecuzione; potrebbe essere eseguito automaticamente in seguito).

## (Opzionale) Autenticazione GMP

Se desideri utilizzare Google Maps Platform (GMP) come provider di mappe per le mappe di base, devi fornire una [chiave API di Google Maps Platform](https://developers.google.com/maps/documentation/javascript/get-api-key). Questo notebook può recuperarla dai tuoi [Colab Secrets](https://colab.sandbox.google.com/github/google-gemini/cookbook/blob/main/quickstarts/Authentication.ipynb#scrollTo=dEoigYI9Jw_K) se fornisci un nome chiave.

Se non viene fornito alcun nome chiave, `pydeck` utilizzerà per impostazione predefinita la mappa `carto` fornita.

## Abilita BigQuery API

## (Opzionale) Abilita Google Maps Javascript API

# Installa Pacchetti

# Importa librerie

Abilita le tabelle interattive per i DataFrame `pandas` in Colab.

> Per maggiori dettagli, consulta [questo articolo](https://colab.google/articles/alive).

# Routine Condivise

Creiamo qui `display_pydeck_map` per renderizzare i grafici usando `pydeck`. Ciò comporta la creazione di un'istanza `pydeck.Map` e quindi l'aggiunta di layer (`pydeck.Layer`) all'istanza della mappa.

# Crea uno scatter plot con `ScatterplotLayer`

Gli scatter plot sono più utili quando è necessario rivedere un sottocampione di punti individuali. Questo viene talvolta definito "spot checking".

In questa sezione, creeremo uno scatter plot di tutte le stazioni di bike sharing elencate nel dataset pubblico [San Francisco Ford GoBike Share](https://console.cloud.google.com/bigquery(cameo:product/san-francisco-public-data/sf-bike-share).

## Importa dataset da BigQuery: Stazioni Ford GoBike a San Francisco

Usiamo la magic `%%bigquery` per eseguire una query e restituire i risultati in un `geopandas.GeoDataFrame`.

## Rendering di uno Scatter plot

Questo esempio dimostra come utilizzare [pydeck.Layer](https://deckgl.readthedocs.io/en/latest/layer.html#pydeck.bindings.layer.Layer) e lo [ScatterPlotLayer](https://deck.gl/docs/api-reference/layers/scatterplot-layer) per renderizzare i punti individuali come cerchi. Ciò richiede l'estrazione della longitudine e della latitudine come coordinate x e y rispettivamente dalla colonna `station_geom`.

Dal momento che `gdf_sf_bikestations` è un `geopandas.GeoDataFrame`, le coordinate possono essere accedute direttamente dalla sua colonna geometria `station_geom`. Il codice recupera la longitudine usando l'attributo `.x` della colonna e la latitudine usando il suo attributo `.y`, memorizzandole in nuove colonne `longitude` e `latitude`.

La cella sottostante renderizza uno scatter plot.

Se desideri personalizzarlo, consulta la documentazione API di [pydeck.Layer](https://deckgl.readthedocs.io/en/latest/layer.html#pydeck.bindings.layer.Layer) e la documentazione di [ScatterPlotLayer](https://deck.gl/docs/api-reference/layers/scatterplot-layer) per scoprire altri parametri.

# Visualizza poligoni con un `GeoJSONLayer`

Il tipo di dati [GEOGRAPHY](https://cloud.google.com/bigquery/docs/reference/standard-sql/data-types#geography_type) di BigQuery può rappresentare un valore geometrico o una collezione di geometrie. Esempi di forme includono:
* punti
* linee
* poligoni
* multi-poligoni

A volte ti vengono forniti dati geospatiali senza sapere in anticipo le loro forme previste.

In questo caso, la visualizzazione dei dati può aiutarti a scoprire le forme consentendo ulteriori analisi. Questa visualizzazione può essere fatta convertendo i dati geografici in GeoJSON e quindi visualizzandoli con un `GeoJSONLayer`.

Come esempio, importeremo dati geografici dal dataset [San Francisco Neighborhoods](https://console.cloud.google.com/bigquery?ws=!1m4!1m3!3m2!1sbigquery-public-data!2ssan_francisco_neighborhoods) e quindi li visualizzeremo.

## Importa dataset da BigQuery: Quartieri di San Francisco

Vediamo sopra che i dati sono un `POLYGON`.

La cella sottostante renderizza i dati affidandosi a `pydeck` per convertire ogni istanza di oggetto `shapely` nella colonna `geometry` nel [formato GeoJSON](https://datatracker.ietf.org/doc/html/rfc7946).

Se desideri personalizzare la visualizzazione, consulta la documentazione API di [pydeck.Layer](https://deckgl.readthedocs.io/en/latest/layer.html#pydeck.bindings.layer.Layer) e la documentazione di [GeoJsonLayer](https://deck.gl/docs/api-reference/layers/geojson-layer) per scoprire altri parametri.

# Renderizza un coropleth usando `PolygonLayer`

Se stai esplorando dati che sono poligoni non facilmente convertibili in GeoJSON, puoi considerare l'utilizzo di un [PolygonLayer](https://deck.gl/docs/api-reference/layers/polygon-layer) invece. Il `PolygonLayer` può elaborare dati di input di [tipi specifici](https://deck.gl/docs/api-reference/layers/polygon-layer#getpolygon) come un array di punti.

Mostreremo come utilizzare un `PolygonLayer` per renderizzare un array di punti e faremo un passo avanti per produrre un coropleth della densità delle stazioni di bike sharing per quartiere.

# Crea una heatmap con `HeatmapLayer`

I coropleth sono utili quando hai confini significativi noti in anticipo. Tuttavia, quando hai molti dati senza confini significativi noti, considera l'utilizzo di un `HeatmapLayer` per renderizzare la sua densità continua.

Per questo esempio, useremo il dataset [San Francisco Police Department (SFPD) Reports](https://console.cloud.google.com/bigquery(cameo:product/san-francisco-public-data/sfpd-reports)) per interrogare e visualizzare la distribuzione degli incidenti nel 2015.

Per le heatmap, si consiglia di quantizzare e aggregare i dati prima del rendering. Questo esempio lo farà utilizzando l'indicizzazione spaziale [H3](https://docs.carto.com/data-and-analysis/analytics-toolbox-for-bigquery/sql-reference/h3) di Carto.

## Importa dataset da BigQuery: Incidenti del Dipartimento di Polizia di San Francisco

La quantizzazione può essere fatta usando la libreria python [h3](https://github.com/uber/h3-py). Questa aggregherà i punti incidenti in esagoni.

Nello specifico, usiamo:
* [h3.latlng_to_cell](https://uber.github.io/h3-py/api_verbose.html#h3.latlng_to_cell) per mappare la posizione dell'incidente (latitudine e longitudine) a un indice di cella H3. Una [risoluzione](https://h3geo.org/docs/core-library/restable/) H3 di `9` fornisce esagoni aggregati sufficienti per la heatmap.
* [h3.cell_to_latlng](https://uber.github.io/h3-py/api_verbose.html#h3.cell_to_latlng) per determinare il centro di ogni esagono.

> Si prega di notare che è possibile utilizzare anche le [funzioni H3](https://docs.carto.com/data-and-analysis/analytics-toolbox-for-bigquery/sql-reference/h3) di Carto in BigQuery SQL per eseguire una conversione simile.

La cella sottostante renderizza un layer heatmap.

Se desideri personalizzarlo, consulta la documentazione API di [pydeck.Layer](https://deckgl.readthedocs.io/en/latest/layer.html#pydeck.bindings.layer.Layer) e la documentazione di [HeatmapLayer](https://deck.gl/docs/api-reference/aggregation-layers/heatmap-layer) per scoprire altri parametri.

# Ulteriori letture

Hai imparato come visualizzare dati geospatiali archiviati in BigQuery usando `pydeck`.

Se vuoi saperne di più su `pydeck` e altri tipi di grafici `deck.gl`, consulta la [Galleria](https://deckgl.readthedocs.io/en/latest/) di `pydeck` per esempi. Puoi anche consultare il [Catalogo Layer](https://deck.gl/docs/api-reference/layers) di `deck.gl` e il [codice sorgente github](https://github.com/visgl/deck.gl) per altri tipi di grafici.

Se vuoi leggere di più sui dati geospatiali, consulta la pagina [Getting Started](https://geopandas.org/en/stable/getting_started.html) di GeoPandas e la [User guide](https://geopandas.org/en/stable/docs/user_guide.html). Per la manipolazione di oggetti geometrici, consulta il [manuale utente](https://shapely.readthedocs.io/en/stable/manual.html) di Shapely.

Puoi leggere di più nella documentazione di BigQuery sull'[analisi geospatiale](https://cloud.google.com/bigquery/docs/geospatial-intro) e sulla [visualizzazione di dati geospatiali con BigQuery](https://cloud.google.com/bigquery/docs/geospatial-visualize).

Per esplorare l'utilizzo dei dati di Earth Engine con BigQuery per la visualizzazione, consulta Google Earth Engine [Exporting to BigQuery](https://developers.google.com/earth-engine/guides/exporting_to_bigquery). Per maggiori dettagli sull'analisi e visualizzazione geospatiale interattiva con Google Earth Engine, consulta [geemap.org](https://geemap.org/).

## Pulizia

Se hai creato un progetto Google Cloud specificamente per questo tutorial, puoi [eliminare il progetto](https://cloud.google.com/resource-manager/docs/creating-managing-projects#shutting_down_projects).

Se hai creato una chiave API di Google Maps Platform per questo tutorial, si consiglia di [eliminare la chiave](https://developers.google.com/maps/api-security-best-practices#deleting-unused-apikeys).

## Aiuta Google a migliorare i Tutorial

Segnala problemi [qui](https://github.com/GoogleCloudPlatform/bigquery-utils/issues) su Github.

Invia feedback [qui](https://docs.google.com/forms/d/e/1FAIpQLSdNSDbBM2rohqtqWtIPgKD_14oLc8TPqqdtKD11oqsrtA8_NQ/viewform) tramite un modulo Google.
