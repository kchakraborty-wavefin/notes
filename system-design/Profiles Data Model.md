## Core Entities

`identityBusiness`: a Wave Business (think business switcher)
`identityUser`: a Wave Account owned by a Person
`LegalEntity`: a Person or Legal Business in the physical world (does this extend `Business` or `Person`)
`Business`: a Legal Business (may be modelled by multiple `identity` `Businesses`), a `LegalEntity`
`Person`: a Person associated with a Business (may represent an `identityUser`), a `LegalEntity`

##### Sole Proprietors
> Not technically Legal Businesses? But still treated as a Business

##### Known Relationships
`Businesses` and `LegalEntities` can have two types of relationships:
- `Roles`
- `Ownership`

We don't need classes to represent kinds of ownership since it's pretty straightforward
> where does the concept of shares come from? Is this multiple owners for a business or some other definition?

Example of `roles` - `director`, `contractor`, `employee`, `payor` (contractor or payor can be another `Business`)
> Type checks for contractor and payor

```python
import datetime
import decimal
import typing
import uuid

class DomainObject:
    def __init__(self, **kwargs):
        for key in kwargs:
            setattr(self, key, kwargs[key])

class Address(DomainObject):
    # ...
    pass

class LegalEntity(DomainObject):
    # may want to create types (classes) for these aggregates of Roles/SharedRoles
    roles_by_business: dict[str, list[str]]  # { business_id: [role1_label, role2_label] }
    # This could be 'ownership_by_business' if we use percentages
    shares_by_business: dict[str, int]  # { business_id: shares }
    addresses: dict[str, Address]  # keyed by label
    phone_numbers: dict[str, str]  # keyed by label
    email_addresses: dict[str, str]  # keyed by label
    websites: dict[str, str]  # keyed by label
    taxpayer_identification_numbers: dict[str, str]  # keyed by label

class Person(LegalEntity):
    first_name: str
    last_name: str
    middle_names: str
    birth_date: datetime.date
    full_name: str
    first_and_last_name: str

class Business(LegalEntity):
    business_name: str
    established_on: datetime.date  # When the business was established in the physical world
    dissolved_on: datetime.date
    # may want to create a type (class) for this aggregate of Persons
    # {
    #     legal_entities: {
    #         'type': 'Person' or 'Business'
    #         'roles': [role1_label, role2_label],
    #         'ownership': ownership
    #     }, ...
    # }
    legal_entities: dict[str, dict[str, typing.Union[str, list[str], decimal.Decimal]]]
```

Example Instantiation:
```python
identity_namespace = 'f5f6e98d-2a00-4e42-9dc9-10c3f178ee87'  # uuid4
profiles_namespace = 'c521b0f0-6d27-4ffc-a462-93589cdae572'  # uuid4

business = Business(
    id = 'b6e7187e-5590-5fce-924c-69ca0c2a6578',  # 'random' uuid5 based on identity_namespace
    business_name = 'In Search of Entropy',
    established_on = datetime.date(2022, 2, 8),
    dissolved_on = None,
    associated_legal_entities = {
        '73ba9e5e-5502-522a-89e4-e5ecd32a934a': {
            'type': 'Person',
            'roles': ['director'],
            'ownership': decimal.Decimal(1),
        }
    },
    roles_by_business = {},
    shares_by_business = {},
    addresses = {
        'office': Address(),
        'mailing': Address(),
    },
    phone_numbers = {
        'office': PhoneNumber(value = '555-555-5555'),
    },
    email_addresses = {
        'primary': EmailAddress(value = 'contact@insearchofentropy.com'),
    },
    websites = {
        'primary': Website(value = 'https://insearchofentropy.com'),
    },
    taxpayer_identification_numbers = {
        'cra-payroll': TaxpayerIdentificationNumber(value = '123456789RP5720'),
        'cra-harmonized_sales_tax': TaxpayerIdentificationNumber(value = '123456789RT8294'),
    },
)

person = Person(
    id = '73ba9e5e-5502-522a-89e4-e5ecd32a934a',  # 'random' uuid5 based on identity_namespace
    first_name = 'Gunther',
    last_name = 'Penrose',
    middle_names = 'Herman Leonard',
    birth_date = datetime.date(1973, 2, 8),
    full_name = 'Gunther Herman Leonard Penrose',
    first_and_last_name = 'Gunther Penrose',
    roles_by_business = {
        'b6e7187e-5590-5fce-924c-69ca0c2a6578': ['director'],
    },
    shares_by_business = {
        'b6e7187e-5590-5fce-924c-69ca0c2a6578': 1000,
    },
    addresses = {
        'home': Address(),
        'work': Address(),
    },
    phone_numbers = {
        'work': PhoneNumber(value = '555-555-5555'),
    },
    email_addresses = {
        'work': EmailAddress(value = 'gunther@insearchofentropy.com'),
    },
    websites = {},
    taxpayer_identification_numbers = {
        'cra-social_insurance_number': TaxpayerIdentificationNumber(value = '123456789'),
    },
)
```

#### Problem: Data Changes Over Time
- find way to represent current and historical data associated with `LegalEntities`
- this data may be used to generate forms retroactively, its interface will need to be able to reconstitute this data by effective date
- 
```python
associated_legal_entities = {
        '73ba9e5e-5502-522a-89e4-e5ecd32a934a': {
            'type': 'Person',
            'roles': ['director'],
            'ownership': decimal.Decimal(1),
        }
    },
```

> dict of [key: str, val: Person | Business] ?

