# Abstract
The assets are described [here](https://github.com/BeamMW/beam/wiki/Confidential-assets)

This document's purpose is to specify the asset metadata, to enable 3rd party tools to consume and display the subset of asset-related information in a simple and well-standardized manner.

## General notes

Asset meta is a byte buffer associated with the asset and stored on chain. 

* Asset metadata should be an UTF-8 string
* Asset metadata should be serialized as a simple name=value set of pairs, delimited with `;`
* The `keyN=valueN;` sequence must be preceded with `STD:` prefix, there is no need for `;` at the sequence end
* The total size of the metadata cannot exceed 16 Kbytes (16,384 bytes)
* Every description should be in English

## Asset metadata has 2 groups of properties:

* Standard properties
* Optional properties

*WARNING:* Both types of properties are immutable and can't be changed after the asset is created. Therefore it is strongly recommended to double-check the values before starting the asset registration.

# Standard properties, mandatory
* `SCH_VER=` metadata schema version, currently must be `1`
* `N=` human-readable asset name, for example Beam Coin
* `SN=` human-readable short name/asset code, for example BEAM. Length should not exceed 6 characters.
* `UN=` human-readable asset unit name, for example Beam
* `NTHUN=` human-readable smallest asset unit name (nth-unit name), for example Groth

# Standard properties, non-mandatory
* `NTH_RATIO=` descriptive metadata for the number of smallest units per whole asset unit. Use `100000000` (eight decimal places) for new assets, or omit the field. It is not a consensus precision setting; see the precision note below.


## Asset precision and compatibility

Beam wallet amounts use eight decimal places for Confidential Assets: one whole
asset unit corresponds to `100000000` smallest units. The core treats
`NTH_RATIO` as opaque metadata, not as a parameter that changes transaction
arithmetic, issuance amounts or supply limits. A value of `8` is a ratio of eight,
not a request for eight decimal places.

For asset-creation interfaces, show eight decimal places as fixed. If emitting
`NTH_RATIO`, use `100000000`; do not offer a configurable precision selector.
Third-party frontends may interpret metadata differently. To avoid inconsistent
amounts, continue using eight-decimal amount formatting rather than rescaling
balances according to this field.

Existing assets may contain a different or invalid ratio. Preserve their stored
metadata and amounts; displaying that declared ratio must not imply that it is
the effective precision. This clarification does not migrate existing balances
or implement the precision-change proposal in
[Beam #2172](https://github.com/BeamMW/beam/issues/2172).

# Optional properties
* `OPT_SHORT_DESC=` an one-liner, should not exceed 128 characters
* `OPT_LONG_DESC=` a paragraph, should not exceed 1024 characters
* `OPT_SITE_URL=` URL of the asset website
* `OPT_PDF_URL=` URL for the description paper
* `OPT_FAVICON_URL=` URL of the favicon
* `OPT_LOGO_URL=` URL of the logo in SVG vector format
* `OPT_COLOR=` color in the hex format (#FF00FF for example) to be used by the UI to display this asset. If not specified default color would be generated. Currently default colors are based on the asset id

