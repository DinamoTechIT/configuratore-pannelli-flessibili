# configuratore-pannelli-flessibili

Tool online per determinare velocemente **quanti** pannelli fotovoltaici flessibili
da 220 W si possono collegare ai principali sistemi di accumulo / microinverter,
**come cablarli** (serie/parallelo per ogni ingresso MPPT) e **che cavo** serve in
base alla distanza.

Mini-app **statica e autonoma**: nessun backend, nessuna build, nessun framework.

## File

| File | Ruolo |
|---|---|
| `index.html` | Presentazione + **logica elettrica** (`arrange`, `perMPPT`, `cableSection`). |
| `data.json` | **Dati**: pannello di riferimento + elenco sistemi. Per aggiungere/modificare un accumulo si edita **solo** questo file. |

> ⚠️ I due file devono stare **nella stessa cartella** e il sito va servito via
> **http(s)**: aperto da `file://` (doppio click) il `fetch` di `data.json` non
> funziona. In locale: `python3 -m http.server` nella cartella, poi apri
> `http://localhost:8000/`.

## Deploy

1. Pubblica `index.html` + `data.json` su un host statico (Netlify / Vercel /
   Cloudflare Pages, anche drag&drop) o su una sottocartella/sottodominio di
   `app.dinamotech.it` (es. `configuratore.dinamotech.it` con CNAME verso l'host).
2. Se metti un CDN davanti, imposta un TTL basso su `data.json` (il `fetch` usa
   già `cache:"no-cache"`).

## Embed

L'iframe **comunica la propria altezza** al parent via `postMessage`
(`{ type:"dt-configuratore-height", height:<px> }`), così non serve un'altezza
fissa. Snippet completo da incollare nella pagina che ospita l'iframe (Shopify
Custom Liquid/HTML, Notion embed, ecc.):

```html
<iframe id="dt-config"
        src="https://configuratore.dinamotech.it/"
        style="width:100%; border:0" loading="lazy"
        title="Configuratore pannelli flessibili"></iframe>
<script>
  addEventListener("message", function (e) {
    if (e.data && e.data.type === "dt-configuratore-height") {
      document.getElementById("dt-config").style.height = e.data.height + "px";
    }
  });
</script>
```

Se la piattaforma non consente di aggiungere lo `<script>` (alcuni embed di
Notion), usa un'altezza fissa di fallback: `style="width:100%;height:1400px;border:0"`.

## Aggiornare i dati

- **Aggiungere/modificare un sistema:** edita `data.json` (campo `systems`). Campi:
  `inputs` (n. ingressi MPPT), `maxSeries`, `maxParallel`, `vmax`, opz. `startup`
  e `minSeries`, `note` (chiave della nota testuale).
- **Brand nuovo:** aggiungi il colore del tag alla mappa `BRAND_COLOR` in `index.html`.
- **Cambiare pannello:** edita `data.json` → `panel` e **ricontrolla** `maxSeries`
  = `floor(vmax / Voc)` (con Voc 19,26 V e 60 V resta 3) e i limiti di parallelo (Isc).
- I **calcoli** restano nel JavaScript: Airtable/Make portano solo i *dati*,
  rigenerando `data.json` senza toccare l'HTML.
