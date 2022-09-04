# Faceted filtering of the Knowledge Graph Visual Browser
## Overall architecture
The following picture depicts parts of the Knowledge Graph Browser that allow faceted filtering:  <br><br>
![Faceted filtering architecture](/faceted_filtering_architecture.png)

- Knowledge graph visual browser
  - KGVB frontend
    - [Faceted filtering component](#faceted-filtering-component)
      - [Facets from configuration](#facets-from-configuration)
      - [Dynamically generated facets](#dynamically-generated-facets)
  - KGVB backend
    - [GET facets items](get-facets-items)
- Triple store
  - [Configuration definitions](#configuration-definitions)
- RDF datasets
  - [SPARQL endpoints](#sparql-endpoints)

The next picture shows how the components interact during runtime:  <br><br>
![Sequence diagram](/configuration_facets_sequence_diagram.png)

<a id="faceted-filtering-component"></a>
## Frontend - Faceted filtering component


<a id="facets-from-configuration"></a>
### Facets from configuration


<a id="dynamically-generated-facets"></a>
### Dynamically generated facets


<a id="get-facets-items"></a>
## Backend - GET facets items


<a id="configuration-definitions"></a>
## Triple store - Configuration definitions


<a id="sparql-endpoints"></a>
## RDF datasets - SPARQL endpoints
