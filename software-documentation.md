# Faceted filtering of the Knowledge Graph Visual Browser
## Overall architecture
The following picture depicts parts of the Knowledge Graph Browser that allow faceted filtering:  <br><br>
![Faceted filtering architecture](/resources/faceted_filtering_architecture.png)

- Knowledge graph visual browser
  - KGVB frontend
    - [Faceted filtering component](#faceted-filtering-component)
      - [Facets from configuration](#facets-from-configuration)
      - [Dynamically generated facets](#dynamically-generated-facets)
      - [How indexing of nodes and filtering works](#indexing-and-filtering)
  - KGVB backend
    - [GET facets items](get-facets-items)
- Triple store
  - [Configuration definitions](#configuration-definitions)
- RDF datasets
  - [SPARQL endpoints](#sparql-endpoints)

The next picture shows how the components interact during runtime:  <br><br>
![Sequence diagram](/resources/configuration_facets_sequence_diagram.png)

<a id="faceted-filtering-component"></a>
## Frontend - Faceted filtering component
The [faceted filtering component](https://github.com/JiriResler/knowledge-graph-browser-frontend/tree/master/src/component/faceted-filtering) contains code for rendering facets and also a script for working with data that get sent by the [KGVB server](#get-facets-items). There are two major categories of facets: those that are defined in the graph's configuration and those which are found locally based on available information in the graph.

<a id="facets-from-configuration"></a>
### Facets from configuration
The creator of a configuration can specify a facet like this:  <br><br>
![Given_name_facet](/resources/given_name_facet.png)

- `dct:title` and `dct:description` are displayed to the user.

- The `browser:facetType` can have two values - either the facet is a "label" facet or a "numeric" facet. A label facet is rendered using checkboxes and a numeric facet is rendered using a range slider. More about their implementation differences is in the [indexing and filtering](#indexing-and-filtering) chapter.  

- `browser:hasDataset` specifies a dataset on which the `browser:facetQuery` will be executed. The dataset needs to have a [SPARQL endpoint](#sparql-endpoints) available. 

- `browser:facetQuery` is a SPARQL query which returns nodes' IRIs and values associated with them given that they meet the query's criteria. For example for the above facet's query for a node specified by http://www.wikidata.org/entity/Q937 the associated value would be Albert. More on these internals in the [GET facets items](#get-facets-items) chapter.

An example of a "numeric" facet:  <br><br>
![Born in country with certain population_facet](/resources/born_in_country_population_facet.png)

<a id="dynamically-generated-facets"></a>
### Dynamically (locally) generated facets

<a id="indexing-and-filtering"></a>
### How indexing of nodes and filtering works
#### Indexing
There is a difference between label and numeric facets in that how items (nodes' IRIs) are indexed for filtering.  

A label facet has a map as its index where keys are labels (for example given names of people) and values are IRIs of nodes (for example people) which have the given property.  

An example of a label facet's index:  

```typescript
{
   "Albert" => ["IRI_1"], 
   "Marie" => ["IRI_2", "IRI_3"]
 } 
```

A numeric facet has an array for its index and its elements are nodes with their numeric value associated to them.  

An example of a numeric facet's index:
```typescript
[
   {
      "nodeIRI":"IRI_1",
      "value":5
   },
   {
      "nodeIRI":"IRI_2",
      "value":7
   }
]
```

Indexes for facets are created when facets are created and updated as the user adds or removes nodes to the graph. When an index becomes empty, its facet is marked as undefined and won't be rendered.

#### Filtering
When a user clicks the filter button, a set of nodes that pass all selected criteria is gathered and all nodes of the graph are tested if they are present in this set. If a node doesn't pass the filter, it will become hidden.

<a id="get-facets-items"></a>
## Backend - GET facets items


<a id="configuration-definitions"></a>
## Triple store - Configuration definitions


<a id="sparql-endpoints"></a>
## RDF datasets - SPARQL endpoints
