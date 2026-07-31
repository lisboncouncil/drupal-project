# Design: template Composer `lisboncouncil/drupal-project`

Data: 2026-07-31
Stato: approvato da Marcello

## Obiettivo

Un template Composer riutilizzabile per le nuove installazioni Drupal di Lisbon Council, che sostituisca il copia-incolla manuale del `composer.json` tra progetti. Un nuovo sito si crea con un solo comando `composer create-project` e arriva già con i moduli contrib standard e i moduli Horizon (`lc_*`) pronti da abilitare.

## Decisioni prese

| Decisione | Scelta |
|---|---|
| Strumento | Template Composer (no install profile, no recipes per ora; il template resta recipes-ready) |
| Set moduli | Nucleo comune (≥4 progetti su 6) + gruppo geo/mappe + migrate, modello chief |
| Integrazione Horizon | `composer.json` aggiunto al repo `drupal-horizon-project` + tag `v1.0.0`, require via repository VCS |
| Hosting | GitHub `lisboncouncil/drupal-project`, uso con flag `--repository` (niente Packagist per ora) |
| Automazione | Solo `create-project`; l'installazione sito (drush si, abilitazione moduli) resta manuale |

## 1. Repo template

- Nome pacchetto: `lisboncouncil/drupal-project`, `type: project`, derivato da `drupal/recommended-project`.
- Drupal core `^11.3`, docroot `web/`.
- Recipes-ready come chief: `drupal/core-recipe-unpack` + installer-path `recipes/{$name}`.
- `cweagans/composer-patches` incluso e configurato (sezione `extra.patches` vuota pronta all'uso).
- `drupal-core-project-message` personalizzato: passi post-creazione del workflow Lisbon Council (creazione DB, `drush site:install`, abilitazione moduli `lc_*`).
- `minimum-stability: stable`, `prefer-stable: true`; eccezioni per singolo modulo con flag `@RC` nel constraint (es. `advanced_text_formatter: ^3.0@RC`).

Creazione nuovo sito:

```bash
composer create-project lisboncouncil/drupal-project nuovosito/drupal \
  --repository='{"type":"vcs","url":"https://github.com/lisboncouncil/drupal-project"}'
```

## 2. Set moduli

### `require`

- **Editoriale/SEO**: pathauto, redirect, metatag, schema_metatag, xmlsitemap, easy_breadcrumb, token, scheduler, publication_date
- **Modellazione contenuti**: ds, field_group, field_formatter_class, empty_fields, advanced_text_formatter, smart_date, paragraphs, svg_image, editor_file, format_bytes, calendar_view, taxonomy_manager, address
- **Geo/mappe** (dipendenze di lc_events): geofield, geocoder_address, geocoder_field, geolocation_google_maps, leaflet, geocoder-php/google-maps-provider
- **Migrazione**: migrate_plus, migrate_tools, migrate_source_csv
- **Infrastruttura/servizi**: redis, smtp, matomo, mailchimp, eu_cookie_compliance, security_review, masquerade
- **Temi**: bootstrap_barrio, bootstrap_sass, gin
- **Viste**: views_bootstrap, views_data_export
- **Tooling**: drush (in require perché usato da cron/deploy in produzione)
- **Horizon**: lisboncouncil/drupal-horizon-project ^1.0

### `require-dev`

- devel, upgrade_status

### Esclusi di proposito (da aggiungere per sito quando servono)

search_api / search_api_solr (richiede Solr dedicato; solo chief e cocyber), webform, facets, tfa, bibcite, social_auth, password_policy — presenti in 1-2 progetti soltanto.

I constraint di versione partono da quelli di chief (il più aggiornato) e si allargano dove serve.

## 3. Integrazione drupal-horizon-project

Due interventi:

1. **Repo horizon** (`github.com/lisboncouncil/drupal-horizon-project`): aggiungere `composer.json` alla radice con `name: lisboncouncil/drupal-horizon-project`, `type: drupal-custom-module`, licenza e require minimi (`composer/installers`). Creare tag `v1.0.0`. Le modifiche future ai moduli `lc_*` si rilasciano con nuovi tag.
2. **Template**: repository `{"type": "vcs", "url": "https://github.com/lisboncouncil/drupal-horizon-project.git"}` e installer-path esplicito:

```json
"web/modules/custom/lc": ["lisboncouncil/drupal-horizon-project"]
```

L'intero pacchetto si monta in `web/modules/custom/lc` (stesso layout di chief: `lc/lc_events`, `lc/lc_pages`, …). Sparisce la definizione `package` inline con versione finta "1.0" e ref congelato su `main`.

## 4. Verifica

1. `composer validate --strict` sul `composer.json` del template.
2. `composer create-project` di prova in una directory scratch sul server: l'albero completo (core 11.3 + contrib + horizon in `modules/custom/lc`) deve risolversi e installarsi senza conflitti su PHP 8.3.
3. Smoke-test opzionale: `drush site:install` con DB MySQL temporaneo, abilitazione di `lc_hcommon` e `lc_pages`.

## Fuori scope

- Conversione dei moduli `lc_*` in recipes native (possibile evoluzione futura, il template è già predisposto).
- Registrazione su Packagist.
- Script di provisioning del server (vhost Apache, DB, ecc.).
- Migrazione dei 6 siti esistenti al template (il template serve per i siti nuovi).
