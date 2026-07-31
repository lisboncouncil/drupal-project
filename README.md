# Lisbon Council Drupal project template

Composer template for new Lisbon Council Drupal 11 websites (Horizon projects).
It ships the contrib modules we use on every site plus the
[drupal-horizon-project](https://github.com/lisboncouncil/drupal-horizon-project)
boilerplate modules (`lc_*`), installed in `web/modules/custom/lc`.

## Create a new site

```bash
composer create-project lisboncouncil/drupal-project mysite/drupal \
  --repository='{"type":"vcs","url":"https://github.com/lisboncouncil/drupal-project"}'
```

Then follow the post-install message: create the database, then run:

```bash
drush site:install standard
drush recipe web/core/recipes/article_content_type   # run from the project root with an absolute path if drush complains
drush recipe web/core/recipes/page_content_type
drush en lc_hcommon lc_pages
```

On Drupal ≥11.4 the non-interactive `standard` profile no longer creates the
`article` and `page` content types, which `lc_hcommon` requires; applying
both core recipes before enabling the Horizon base modules is required.
Then enable the `lc_section_*` modules the site needs.

## What's included

- Drupal core ^11.3 (recipes-ready: `core-recipe-unpack`, `recipes/` path)
- Editorial/SEO: pathauto, redirect, metatag, schema_metatag, xmlsitemap,
  easy_breadcrumb, token, scheduler, publication_date
- Content modelling: ds, field_group, field_formatter_class, empty_fields,
  advanced_text_formatter, smart_date, paragraphs, svg_image, editor_file,
  format_bytes, calendar_view, taxonomy_manager, address
- Geo/maps: geofield, geocoder, geolocation_google_maps, leaflet
- Migration: migrate_plus, migrate_tools, migrate_source_csv
- Infrastructure: redis, smtp, matomo, mailchimp, eu_cookie_compliance,
  security_review, masquerade
- Themes: bootstrap_barrio, bootstrap_sass, gin (admin)
- Views: views_bootstrap, views_data_export
- Dev only: devel, upgrade_status

Not included by default (add per site when needed): search_api /
search_api_solr, webform, facets, tfa, bibcite.

## Patches

`cweagans/composer-patches` is preconfigured: add entries under
`extra.patches` in `composer.json`.
