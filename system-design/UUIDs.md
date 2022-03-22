#### UUID v1

uses host computers mac address, curr date and time + random component for uniqueness - guaranteed uniqueness unless generated from same computer and same time - chance of collision is there

`939b2440-a62d-11ec-b08c-c15cd01e3aff` (`939b2440` changes, `a62d-11ec-b08c-c15cd01e3aff` stays the same)
        
#### UUID v4

random generation with no inherent logic, no way to identify info about the source by looking at UUID (except for the 4 version number included in the spec `xxxxxxxx-xxxx-4xxx-xxxx-xxxxxxxxxxxx` 124 bits of randomness, 4 bits for version
    
#### UUID v5

non-random UUIDs generated via two things → (input string, namespace) where namespace itself is a fixed UUID generated per app → these are put into a SHA1 hashing algo to generate another UUID
    
So if namespace is profiles: `b878cf78-a62e-11ec-b909-0242ac120002`
Or namespace is identity: `c7403532-a62e-11ec-b909-0242ac120002`

Then, with the input string `Robert`, we’d generate [using](https://www.uuidtools.com/v5):
- `6d657a4c-df04-5ec5-8b15-ab267a378744` (profiles)
- `a228b637-06ba-522d-a149-16e801be5439` (identity)