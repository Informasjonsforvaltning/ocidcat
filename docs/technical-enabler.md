# OCI Datacatalog & DCAT Application Profile Technical Enabler

The purpose of this document is to propose how an organization can use the service OCI DataCatalog as the metadata store for an Application Profile of a Service delivered by an organization.

#### Glossary
- `Data Catalog Vocabulary (DCAT)`: RDF Vocabulary and W3C standard for describing datasets and data services in a catalog. Designed to facilitate interoperability between data catalogs on the Web.
-	`DCAT Application Profile (DCAT-AP)`: The DCAT Application Profile for data portals in Europe (DCAT-AP) is an application profile of DCAT, maintained by the SEMIC action, Interoperable Europe. DCAT-AP defines a minimal common basis for data catalogs across borders and domains in Europe, to ensure findability across and interoperability between the different national data catalogs in Europe.
- `DCAT-AP-NO`: Norway's national application profile, based on DCAT-AP. DCAT-AP-NO is compatible with DCAT-AP but contains additional requirements and extensions specific to a Norwegian context. DCAT-AP-NO is used as the basis for the National Data Catalog (data.norge.no), which contains not only open data, but also data with restricted access. The National Data Catalog also contains an overview of the base registries, although there is no formal list of them.
-	`OCI DataCatalog(OCI-DCAT)`: Oracle Cloud Infrastructure (OCI) Data Catalog is a metadata management service that helps data professionals discover data and support data governance. Designed specifically to work well with the Oracle ecosystem, it provides an inventory of assets, a business glossary, and a common metastore for data lakes.
-	`RDF`: The Resource Description Framework (RDF) defines a language for describing resources and the relationships among them in terms of named properties and values.

## Concept
OCI-DCAT and the DCAT operate on different levels. OCI-DCAT is a service used to extract metadata from storage solutions (databases, object-based storage, etc.) and make it available to internal stakeholders so they can discover what data is available in those sources. On the other hand, DCAT is a standard to document dataservices and datasets (API’s, public assets, etc.) that an organization offers to the public or external stakeholders.

Even though the intended public of each one of them is different, this document describes how an organization could
- use metadata stored in OCI-DCAT as the basis to publish a DCAT data catalog in a machine readable format
- publish the data catalog to a DCAT-AP compliant data portal like data.norge.no.

## Logical description
OCI-DCAT has two main components
-	A Metadata store
-	A Business Glossary (simplified to _Glossary_)

A glossary can be used to model any kind of hierarchically structured system (e.g. a library inventory system). Since a DCAT data catalog is a hierarchical system, it can be modeled using OCI-DCAT glossaries, using its relationship and custom properties capabilities.

A DCAT data catalog stored in an OCI-DCAT glossary can be programmatically manipulated using the OCI REST API. This allows for a software component to transform a description according to DCAT from REST objects to an RDF serialization format.

A collateral advantage of such an implementation, is the fact that an organization can use OCI-DCAT glossaries with the DCAT model, to connect them to the metadata harvested from it’s data sources; allowing to have a better observability on 
-	Which internal storage system owns data that is being used by a dataservice published as a DCAT resource
-	Which internal stakeholder owns data offered by a dataservice published as a DCAT resource

![Figure 1](img/tech-enabler-logical.png)
> Figure 1: Logical overview of library interaction with OCI DCAT and a DCAT-AP RDF description file.

## Technical implementation

In this first approach, the simplified overview of the DCAT-AP-NO mandatory entities.
![DCAT-AP-NO simple description](https://informasjonsforvaltning.github.io/dcat-ap-no/images/DCAT-AP-NO-forenklet-fremstilling.png)
> Figure 2: Simplified overview of DCAT-AP-NO requirements, only with the mandatory and recommended fields.

### 1. Create an OCI-DCAT glossary modeled by the DCAT-AP-NO specification
<span id="t1">

| OCI DCAT | DCAT-AP-NO |
| --- | --- |
| Glossary | Catalog |
| Category | Dataset |
| N/A | Distribution |
| N/A | Actor |
| N/A | Kind |
| N/A | Dataservice |
| N/A | LegalResource |

> Table 1: Table mapping between OCI DCAT Glossary entities and DCAT-AP-NO classes
</span>

In this proposal, only Catalog and Dataset objects in the DCAT-AP-NO can be modeled in OCI-DCAT. The other objects do not have a relevant mapping, because they are used in the context of how data is distributed, which is not in the scope of OCI-DCAT.

Custom properties in OCI-DCAT can be used to supplement the properties that are mandatory. In the tables below it is described how.
<span id="t2">

| OCI DCAT (Glossary) | DCAT-AP-NO (dcat:Catalog) | Value |
| --- | --- | --- |
| description | dct:description |
| displayName | dct:title |
| (Custom Property) contactPoint | dcat:contactPoint | Email address of internal owner | 
| (Custom Property) publisher | dct:publisher | URI referring to the organization publishing the catalog |
| (Custom Property) rdf:type | rdf:type | `dcat:Catalog` |
| ~(Custom Property) dct:identifier~ | ~dct:identifier~ | if required: same as URI|
| (Custom Property) URI | URI |

> Table 2: Table mapping between OCI DCAT Glossary and DCAT-AP-NO Catalog class. 
</span>
<span id="t3">

Note: the URI pointed to by dct:publisher should follow the pattern as described [in the example](https://informasjonsforvaltning.github.io/dcat-ap-no/#Katalog-utgiver) in DCAT-AP-NO v3

| OCI DCAT (Category) | DCAT-AP-NO (dcat:Dataset) | Value |
| --- | --- | --- |
| description | dct:description |
| displayName | dct:title |
| (Custom Property) theme | dcat:theme |
| (Custom Property) contactPoint | dcat:contactPoint | Email address of internal owner | 
| (Custom Property) publisher | dct:publisher | URL string to the publisher |
| (Custom Property) rdf:type | rdf:type | `dcat:Dataset` |
| ~(Custom Property) dct:identifier~ | ~dct:identifier~ | if required: same as URI |
| (Custom Property) URI | URI |

> Table 3: Table mapping between OCI DCAT Category and DCAT-AP-NO Dataset class. 
</span>

Note: the URI pointed to by dct:publisher should follow the pattern as described [in the example](https://informasjonsforvaltning.github.io/dcat-ap-no/#Datasett-utgiver) in DCAT-AP-NO v3

#### 1a. Another mandatory objects in DCAT-AP
In this proposal, the classes
•	Distribution
•	~Actor~
•	Kind
•	DataService

Do not have a relevant mapping in OCI-DCAT, it is suggested that such information is provided as a flattened property either in a Glossary or a Category
But most of them can be represented as a single `string` custom property in OCI DCAT, and the library would be responsible of _expading_ as an separate term in the RDF generated file and viceversa.

<span id="t4">

| DCAT-AP-NO Parent class | DCAT-AP-NO Property | DCAT-AP-NO type | DCAT-AP-NO Mapped property  | OCI DCAT Parent Object | Custom Property Example Value |
| --- | --- | --- | --- | --- | --- |
| dcat:Catalog | dct:contactPoint | vcard:Kind | vcard:fn | Glossary | `ACME A/S` |
| dcat:Catalog | dct:contactPoint | vcard:Kind | vcard:hasEmail | Glossary | `<mailto:contact@acme.com>` |
| ~dcat:Catalog~ | ~dct:publisher~ | ~foaf:Agent~ | ~foaf:name~ | ~Glossary~ | ~`ACME A/S`~ |
| dcat:Dataset | dct:contactPoint | vcard:Kind | vcard:fn | Glossary | `Team Doe` |
| dcat:Dataset | dct:contactPoint | vcard:Kind | vcard:hasEmail | Glossary | `<mailto:teamdoe@acme.com>` |
| ~dcat:Dataset~ | ~dct:publisher~ | ~foaf:Agent~ | ~foaf:name~ | ~Category~ | ~`ACME A/S`~ |

> Table 4: Flattened mandatory objects in DCAT-AP-NO mapped to custom properties in OCI DCAT objects.
</span>

### 2. Export OCI-DCAT Glossary using OCI REST API
OCI-DCAT API has the following relevant REST operations for retrieval of glossary related objects:
-	`GetGlossary` | [Oracle Cloud Infrastructure API Reference and Endpoints](https://docs.oracle.com/en-us/iaas/api/#/en/data-catalog/20190325/Glossary/GetGlossary)
-	`ListTerms` | [Oracle Cloud Infrastructure API Reference and Endpoints](https://docs.oracle.com/en-us/iaas/api/#/en/data-catalog/20190325/Term/ListTerms)
-	`GetTerm` | [Oracle Cloud Infrastructure API Reference and Endpoints](https://docs.oracle.com/en-us/iaas/api/#/en/data-catalog/20190325/Term/GetTerm)

These REST operations can be used to navigate in a hierarchical manner the Glossary structure. Given the OCID (Oracle Cloud Identificator) of an OCI DCAT Glossary, an external agent, can get the details of the glossary, and list all the child categories with it's details.

### 3. Identifiers generated by OCI
> NB! Work in progress

When creating a Glossary term, OCI generates a key. This could be used to create a random key used as an inmutable string to be used as part of the URI of the RDF objects. But special considerations has to be made for it's use.

The URIs assigned to the different RDF resources in the Data Catalog (i.e. instances of dcat:Catalog and dcat:Dataset) should be persistent and globally unqiue, and ideally follow the [best practices for URIs](https://www.digdir.no/standarder/peikarar-til-offentlege-ressursar-pa-nett/1492)
