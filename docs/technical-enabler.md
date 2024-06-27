# OCI Datacatalog & DCAT Application Profile Technical Enabler

The purpose of this document is to propose how an organization can use the service OCI DataCatalog as the metadata store for an Application Profile of a Service delivered by an organization.

#### Glossary
-	`DCAT Application Profile (DCAT-AP)`: The DCAT Application profile for data portals in Europe (DCAT-AP) is a specification based on the Data Catalogue vocabulary (DCAT) for describing public sector datasets in Europe. Its basic use case is to enable cross-data portal search for data sets and make public sector data better searchable across borders and sectors. This can be achieved by the exchange of descriptions of datasets among data portals.
- `DCAT-AP-NO`: Norway's national application profile, namely DCAT-AP-NO. DCAT-AP-NO is used as the basis for their national data catalogue, which contains not only open data, but also data with restricted access. The national data catalogue also contains an overview of the base registries, although there is no formal list of them.
-	`OCI DataCatalog(OCI-DCAT)`: Oracle Cloud Infrastructure (OCI) Data Catalog is a metadata management service that helps data professionals discover data and support data governance. Designed specifically to work well with the Oracle ecosystem, it provides an inventory of assets, a business glossary, and a common metastore for data lakes.
-	`RDF`: The Resource Description Framework (RDF) defines a language for describing relationships among resources in terms of named properties and values.

## Concept
OCI-DCAT and the DCAT-AP operate on different levels. OCI-DCAT is a service used to extract metadata from storage solutions (databases, object-based storage, etc.) and make it available to internal stakeholders so they can discover what data is available in those sources. On the other hand, DCAT-AP is a standard to document dataservices and datasets (API’s, public assets, etc.) that an organization offers to the public or external stakeholders.

Even though the intended public of each one of them is different, this document describes how an organization could use metadata stored in OCI-DCAT as the basis to publish an DCAT-AP document in machine readable format so it can be published in an AP database like data.norge.no.

## Logical description
OCI-DCAT has two main components
-	A Metadata store
-	A Business Glossary (simplified to _Glossary_)

A glossary can be used to model any kind of hierarchically structured system (e.g. a library inventory system). Since DCAT-AP is a hierarchical system, it can be modeled using OCI-DCAT glossaries, using its relationship and custom properties capabilities.

A DCAT-AP stored in an OCI-DCAT glossary can be programmatically manipulated using the OCI REST API. This allows for a software component to transform a description according to DCAT-AP from REST objects to RDF format.

A collateral advantage of such implementation, is the fact that an organization can use OCI-DCAT glossaries with the DCAT-AP model, to connect them to the metadata harvested from it’s data sources; allowing to have a better observability on 
-	Which internal storage system owns data that is being used by a dataservice published as an DCAT-AP
-	Which internal stakeholder owns data offered by a dataservice published as an DCAT-AP

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

In this proposal, only Catalog and Dataset objects in the DCAT-A-NO can be modeled in OCI-DCAT. The other objects do not have a relevant mapping, because they are used in the context of how data is distributed, which is not in the scope of OCI-DCAT.

Custom properties in OCI-DCAT can be used to supplement the properties that are mandatory. In the tables below it is described how.
<span id="t2">

| OCI DCAT (Glossary) | DCAT-AP-NO (dcat:Catalog) | Value |
| --- | --- | --- |
| description | dct:description |
| displayName | dct:title |
| (Custom Property) contactPoint | dcat:ContactPoint | Email address of internal owner | 
| (Custom Property) publisher | dct:Publisher | URL string to the publisher |
| (Custom Property) rdf:type | rdf:type | `dcat:Catalog` |
| (Custom Property) dct:identifier | dct:identifier | 
| (Custom Property) URI | URI |

> Table 2: Table mapping between OCI DCAT Glossary and DCAT-AP-NO Catalog class. 
</span>
<span id="t3">

| OCI DCAT (Category) | DCAT-AP-NO (dcat:Dataset) | Value |
| --- | --- | --- |
| description | dct:description |
| displayName | dct:title |
| (Custom Property) theme | dcat:theme |
| (Custom Property) contactPoint | dcat:ContactPoint | Email address of internal owner | 
| (Custom Property) publisher | dct:Publisher | URL string to the publisher |
| (Custom Property) rdf:type | rdf:type | `dcat:Dataset` |
| (Custom Property) dct:identifier | dct:identifier |
| (Custom Property) URI | URI |

> Table 3: Table mapping between OCI DCAT Category and DCAT-AP-NO Dataset class. 
</span>

#### 1a. Another mandatory objects in DCAT-AP
In this proposal, the classes
•	Distribution
•	Actor
•	Kind
•	DataService
•	LegalResource
Do not have a relevant mapping in OCI-DCAT, it is suggested that such information is provided as a flattened property either in a Glossary or a Category
But most of them can be represented as a single `string` custom property in OCI DCAT, and the library would be responsible of _expading_ as an separate term in the RDF generated file and viceversa.

<span id="t4">

| DCAT-AP-NO Parent class | DCAT-AP-NO Property | DCAT-AP-NO type | DCAT-AP-NO Mapped property  | OCI DCAT Parent Object | Custom Property Example Value |
| --- | --- | --- | --- | --- | --- |
| dcat:Catalog | dct:contactPoint | vcard:Kind | vcard:fn | Glossary | `John Doe` |
| dcat:Catalog | dct:publisher | foaf:Agent | foaf:name | Glossary | `ACME A/S` |
| dcat:Dataset | dcat:contactPoint | vcard:Kind | vcard:fn | Category | `John Doe` |
| dcat:Dataset | dct:publisher | foaf:Agent | foaf:name | Category | `ACME A/S` |

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