## Core Entities

`identityBusiness`: a Wave Business (think business switcher)
`identityUser`: a Wave Account owned by a Person
`LegalEntity`: a Person or Legal Business in the physical world (does this extend `Business` or `Person`)
`Business`: a Legal Business (may be modelled by multiple `identity` `Businesses`), a `LegalEntity`
`Person`: a Person associated with a Business (may represent an `identityUser`), a `LegalEntity`

