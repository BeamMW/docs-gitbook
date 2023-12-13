---
description: >-
  This document's purpose is to specify the asset metadata, to enable 3rd party
  tools to consume and display the subset of asset-related information in a
  simple and well-standardized manner.
---

# Asset Metadata Descriptor

## Requirements

Asset meta is a byte buffer associated with the asset and stored on chain.

* Asset metadata should be an UTF-8 string
* Asset metadata should be serialized as a simple `name=value` set of pairs, delimited with `;`
* The `keyN=valueN;` sequence must be preceded with `STD:` prefix, there is no need for `;` at the sequence end
* The total size of the metadata cannot exceed 16 Kbytes (16,384 bytes)
* Every description should be in English

## Properties

Asset metadata has 2 groups of properties:

* Standard properties
* Optional properties


Both types of properties are immutable and can't be changed after the asset is created. Therefore, double-checking the values before starting the asset registration is strongly recommended


### Required properties:

* `SCH_VER=` metadata schema version currently must be `1`
* `N=` human-readable asset name, for example `Beam Coin`
* `SN=` human-readable short name/asset code, for example `BEAM`. Length should not exceed 6 characters
* `UN=` human-readable asset unit name, for example `Beam`
* `NTHUN=` human-readable smallest asset unit name (nth-unit name), for example `Groth`

### Optional properties

* `NTH_RATIO=` integer that represents the ratio of the smallest asset unit to regular unit. If omitted default value of `100000000` is used
* `OPT_SHORT_DESC=` an one-liner, should not exceed 128 characters
* `OPT_LONG_DESC=` a paragraph, should not exceed 1024 characters
* `OPT_SITE_URL=` URL of the asset website
* `OPT_PDF_URL=` URL for the description paper
* `OPT_FAVICON_URL=` URL of the favicon
* `OPT_LOGO_URL=` URL of the logo in SVG vector format
* `OPT_COLOR=` color in the hex format (#FF00FF for example) to be used by the UI to display this asset. If not specified default color would be generated. Currently default colors are based on the asset id

## Examples

* with required properties only:

`--asset_meta "STD:SCH_VER=1;N=Beam Coin;SN=BEAM;UN=Beam;NTHUN=Groth"`

* with required and optional properties:

`--asset_meta "STD:SCH_VER=1;N=Beam Coin;SN=BEAM;UN=Beam;NTHUN=Groth;NTH_RATIO=1000;OPT_SHORT_DESC=Off with their heads!;OPT_LONG_DESC=The fact that this catch phrase ... this story remains a part of popular culture.;OPT_SITE_URL=https://***/*;OPT_PDF_URL=https://***;OPT_FAVICON_URL=https://***/;OPT_LOGO_URL=https://***;OPT_COLOR=#FF00FF" --fee 100000 --enable_assets`
