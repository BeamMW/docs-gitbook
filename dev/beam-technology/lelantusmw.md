# LelantusMW

Cryptographer [Aram Jivanyan](https://www.linkedin.com/in/aramjivanyan/) of Firo (formerly ZCoin) initially developed Lelantus as a new protocol for a confidential exchange of value.

The core principles of Lelantus describe the related proofs placing a UTXO set into a [shielded pool](https://beamx.gitbook.io/glossary/-Ma56WMqUB1AsnhoithB/) while the [anonymity set](https://www.google.com/search?q=anonymity+set+crypto\&sxsrf=ALeKk03stmb--lt3dzzlAfJEASZzekFidw%3A1625620539632\&ei=OwDlYKXhJfW3qtsP4f6o8Aw\&oq=anonymity+set+crypto\&gs\_lcp=Cgdnd3Mtd2l6EAMyBQghEKABMgUIIRCgAToHCCMQsAMQJzoHCAAQRxCwAzoCCAA6BggAEBYQHjoFCCEQqwJKBAhBGABQ2UdY1FdggVpoAXACeACAAcYBiAGFCZIBAzAuOJgBAKABAaoBB2d3cy13aXrIAQnAAQE\&sclient=gws-wiz\&ved=0ahUKEwjl6ra25M\_xAhX1m2oFHWE\_Cs4Q4dUDCA8\&uact=5) accumulates. The UTXO set leaves the shielded pool as a "new" UTXO set equally the exact total value as UTXO set initially submitted.

Beam combined Lelantus with Mimblewimble to create a hybrid[ LelantusMW protocol](https://docs.beam.mw/Lelantus-MW.pdf). This hybrid protocol resolves the issue of [linkability](https://ieeexplore.ieee.org/document/8806723), which in some instances could allow an active attack to establish a link between wallets (though not to determine identities or values of the transactions).

See [LelantusMW](https://github.com/BeamMW/beam/wiki/Lelantus-MW) and [MW Confidential Lelantus Assets](https://github.com/BeamMW/beam/wiki/MW-CLA) for a more in-depth description of our protocol.

{% embed url="https://www.youtube.com/watch?v=cQGQZ1OWPmc&ab_channel=ZKProofStandards" %}



