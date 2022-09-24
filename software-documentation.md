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
    - [GET facets items](#get-facets-items)
- Triple store
  - [Configuration definitions](#configuration-definitions)
- RDF datasets
  - [SPARQL endpoints](#sparql-endpoints)

<a id="faceted-filtering-component"></a>
## Frontend - Faceted filtering component
The [faceted filtering component](https://github.com/JiriResler/knowledge-graph-browser-frontend/tree/master/src/component/faceted-filtering) contains code for rendering facets and also a script for working with data that get sent by the [KGVB server](#get-facets-items). There are two major categories of facets: those that are defined in the graph's configuration and those which are found locally based on available information in the graph. If no facets are defined in a configuration, then only [locally generated facets](#dynamically-generated-facets) will be found.

Buttons' functionality:
- Filter
  - triggers filtering of the graph with parameters chosen by the user
- Reset
  - shows all hidden nodes, unchecks all checkboxes and resets all sliders
- Reload facets
  - clears all facets and loads them again

<a id="facets-from-configuration"></a>
### Facets from configuration
The creator of a configuration can define facets which will then be shown to the user in the filtering tab. More about defining facets is in [configuration definitions](configuration-definitions).  

The following sequence diagram shows how the components interact during runtime when the tool is loading configuration facets:  <br><br>
![Sequence diagram](/resources/configuration_facets_sequence_diagram.png)

1) The user wants to load facets
    - both from configuration and dynamically generated
    - when they click the filtering tab for the first time (when the component is mounted)
    - when they click the "reload facets" button
    - after an expansion for newly added nodes
2) The KGVB frontend calls the [getFacetsItems](#get-facets-items) function on the KGVB server
    - its inputs are the graph's configuration's IRI and nodes' IRIs for which the facets are supposed to be loaded
3) The server loads the configuration as a set of triples
4) For each defined facet in the configuration the server loads information about the facet and sends its query to a SPARQL endpoint (which is specified in the configuration)
    - when the KGVB server receives a response from a SPARQL endpoint, it parses the response and adds it to a variable named `facetsItems`
6) The server sends the `facetsItems` to the frontend

<a id="dynamically-generated-facets"></a>
### Dynamically (locally) generated facets
[Dynamically generated facets](https://github.com/JiriResler/knowledge-graph-browser-frontend/blob/master/src/component/faceted-filtering/DynamicallyGeneratedFacets.ts) are facets which are found locally based on the current state of the graph. They are not specified in a configuration. The principles of these facets are the same for every graph no matter what configuration is chosen.  

These are the currently supported facets:

- Type of node
- Number of edges (total/incoming/outgoing)
- Number of edges of the same type
  - this facet filters nodes by their number of edges of a specific type - for example a node can have 3 edges of type "awarded by" 
- Path
  - this type of facet is specified by a path in the graph
  - for example a facet can be specified by "colleague (outgoing edge)" / "awarded by (outgoing edge)" which means that the user can filter nodes based on whether there exists a path like this which starts from the node. The user can also select what label should the node at the end of that path have. 

<a id="indexing-and-filtering"></a>
### How indexing of nodes and filtering works
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

When a user clicks the filter button, a set of nodes that pass all selected criteria is gathered and all nodes of the graph are tested if they are present in this set. If a node doesn't pass the filter, it will become hidden. See the [implementation](https://github.com/JiriResler/knowledge-graph-browser-frontend/blob/353bffa676763f133ca837ff8b7265932a1b3c7a/src/component/faceted-filtering/FacetedFiltering.vue#L477).

#### Example of indexing and filtering:  
Let's consider this graph for an example:

![Example of indexing and filtering](/resources/indexing_and_filtering_graph.png)

The facets we will examine will be:

- Born in country facet - every person has a country they were born in
- Number of outgoing edges
- Path facet defined by this path: colleague (outgoing) / awardedByInstitution (outgoing)
  - this facet means that there must lead a path like this from a node which ends in nodes with labels chosen by the user

Let the configuration have only one facet defined - the born in country facet which allows to filter by countries where people were born in.  

So, the client sends a request to the server which will return facet items. The client will create a facet (if it wasn't created before) and create (or update) its index with the following content (the application works with nodes' IRIs, here we use human readable names of nodes for clarity):  

```typescript
{
   "Austria" => ["Erwin Schrödinger", "Otto Robert Frisch", "Josef Breuer"], 

   "Germany" => ["Immanuel Estermann"],

   "Czech Republic" => ["David Ernst Oppenheim", "Sigmund Freud"]
 } 
```

We can calculate rest of the facets on the client from information in the graph. The number of outgoing edges facet's index will have this content:

```typescript
[
   {
      "nodeIRI":"Erwin Schrödinger",
      "value":4
   },
   {
      "nodeIRI":"Otto Robert Frisch",
      "value":3
   },
   {
      "nodeIRI":"Sigmund Freud",
      "value":2
   },
   {
      "nodeIRI":"David Ernst Oppenheim",
      "value":1
   },
   {
      "nodeIRI":"Josef Breuer",
      "value":1
   },
   {
      "nodeIRI":"Immanuel Estermann",
      "value":2
   },
   {
      "nodeIRI":"Vladimir V. Tchernavin",
      "value":1
   },
   {
      "nodeIRI":"Otto Stern",
      "value":2
   }
]
```

The path facet's index will contain these information:

```typescript
{
   "Deutsche Physikalische Gesellschaft" => ["Vladimir V. Tchernavin"], 

   "Royal Society" => ["Vladimir V. Tchernavin", "David Ernst Oppenheim", "Josef Breuer"],

   "Frankfurt am Main" => ["David Ernst Oppenheim", "Josef Breuer"],

   "French Academy of Sciences" => ["Vladimir V. Tchernavin"],

   "American Physical Society" => ["Vladimir V. Tchernavin"],

   "Royal Swedish Academy of Sciences" => ["Vladimir V. Tchernavin"],

   "ETH Zürich" => ["Otto Robert Frisch", "Immanuel Estermann"]
 }
```

Let's say that the user chooses these values for facets:  

- Born in country facet: labels `Austria`, `Germany`, `Czech Republic`
- Number of outgoing edges facet: range `<1, 2>`
- The path facet defined by this path: **colleague (outgoing) / awardedByInstitution (outgoing)**: labels `Royal society`, `Deutsche Physikalische Gesellschaft`, `Royal Swedish Academy of Sciences`

When the filter button is pressed, each facet returns a set of nodes which pass the selected criteria.  

The born in country facet will return this set:  

```typescript
{"Erwin Schrödinger", "Otto Robert Frisch", "Sigmund Freud", "David Ernst Oppenheim", "Josef Breuer", "Immanuel Estermann"}
```

The number of outgoing edges facet will return this set:  

```typescript
{"Vladimir V. Tchernavin", "Sigmund Freud", "David Ernst Oppenheim", "Josef Breuer", "Immanuel Estermann"}
```

The path facet will return this set:  

```typescript
{"Vladimir V. Tchernavin", "Otto Robert Frisch", "David Ernst Oppenheim", "Josef Breuer", "Immanuel Estermann"}
```

We need the nodes to pass all filters, so an intersection of all nodes is made with this result:

```typescript
{"Josef Breuer", "David Ernst Oppenheim", "Immanuel Estermann"}
```

This is a set of nodes which should remain visible, so if a node is not present in this set, it will be hidden.


<a id="get-facets-items"></a>
## Backend - GET facets items
There's one function ([this one](https://github.com/linkedpipes/knowledge-graph-browser-backend/blob/0f5dd1be2d6df550350f355a761361aeeaa1f6a1/kgserver.js#L26)) which works as an intermediate between the [KGVB frontend](#faceted-filtering-component) and [RDF datasets](#sparql-endpoints). Its inputs are the graph's configuration's IRI and nodes' IRIs for which the facets are supposed to be loaded. It reads a query from a [facet definition](#configuration-definitions) (if there is any facet defined in a configuration), inserts nodes' IRIs from frontend into that query and sends it to a SPARQL endpoint. Then it parses its response and sends the result to the frontend.

Here is an [example](https://query.wikidata.org/#PREFIX%20wdt%3A%20%3Chttp%3A%2F%2Fwww.wikidata.org%2Fprop%2Fdirect%2F%3E%0APREFIX%20browser%3A%20%3Chttps%3A%2F%2Flinked.opendata.cz%2Fontology%2Fknowledge-graph-browser%2F%3E%0A%0ACONSTRUCT%20%7B%0A%20%20%23%20The%20result%20has%20to%20be%20an%20RDF%20triple%0A%20%20%3Fnode%20browser%3AqueryPath%20%3FtargetNode.%0A%7D%20WHERE%20%7B%20%0A%20%20%23%20Nodes%27%20IRIs%20get%20inserted%20here%20%20%0A%20%20VALUES%20%3Fnode%20%7B%3Chttp%3A%2F%2Fwww.wikidata.org%2Fentity%2FQ1035%3E%20%3Chttp%3A%2F%2Fwww.wikidata.org%2Fentity%2FQ937%3E%7D%0A%20%20%23%20A%20path%20which%20is%20the%20heart%20of%20the%20query%0A%20%20%3Fnode%20wdt%3AP735%2Frdfs%3Alabel%20%3FtargetNode.%0A%20%20FILTER%20%28LANG%28%3FtargetNode%29%20%3D%20%22en%22%29%0A%7D) of what a facet's query can look like after the function inserts nodes' IRIs which it gets sent from the frontend. When it is run there is also an example of a SPARQL endpoint response which the backend function parses.

<a id="configuration-definitions"></a>
## Triple store - Configuration definitions
The creator of a configuration can specify a facet like this:  <br><br>
![Given_name_facet](/resources/given_name_facet.png)

- `dct:title` and `dct:description` are displayed to the user.

- The `browser:facetType` can have two values - either the facet is a "label" facet or a "numeric" facet. A label facet is rendered using checkboxes and a numeric facet is rendered using a range slider. More about their implementation differences is in the [indexing and filtering](#indexing-and-filtering) chapter.  

- `browser:hasDataset` specifies a dataset on which the `browser:facetQuery` will be executed. The dataset needs to have a [SPARQL endpoint](#sparql-endpoints) available. 

- `browser:facetQuery` is a SPARQL query which returns nodes' IRIs and values associated with them given that they meet the query's criteria. For example for the above facet's query for a node specified by http://www.wikidata.org/entity/Q937 the associated value would be Albert. More on these internals in the [GET facets items](#get-facets-items) chapter.
  - `#INSERTNODES` serves for the backend function so that it can inserts nodes' IRIs which it gets sent from the frontend.
  - Here is a runnable [example](https://query.wikidata.org/#PREFIX%20wdt%3A%20%3Chttp%3A%2F%2Fwww.wikidata.org%2Fprop%2Fdirect%2F%3E%0APREFIX%20browser%3A%20%3Chttps%3A%2F%2Flinked.opendata.cz%2Fontology%2Fknowledge-graph-browser%2F%3E%0A%0ACONSTRUCT%20%7B%0A%20%20%23%20The%20result%20has%20to%20be%20an%20RDF%20triple%0A%20%20%3Fnode%20browser%3AqueryPath%20%3FtargetNode.%0A%7D%20WHERE%20%7B%20%0A%20%20%23%20Nodes%27%20IRIs%20get%20inserted%20here%20%20%0A%20%20VALUES%20%3Fnode%20%7B%3Chttp%3A%2F%2Fwww.wikidata.org%2Fentity%2FQ1035%3E%20%3Chttp%3A%2F%2Fwww.wikidata.org%2Fentity%2FQ937%3E%7D%0A%20%20%23%20A%20path%20which%20is%20the%20heart%20of%20the%20query%0A%20%20%3Fnode%20wdt%3AP735%2Frdfs%3Alabel%20%3FtargetNode.%0A%20%20FILTER%20%28LANG%28%3FtargetNode%29%20%3D%20%22en%22%29%0A%7D) of a query.

An example of a "numeric" facet:  <br><br>
![Born in country with certain population_facet](/resources/born_in_country_population_facet.png)

<a id="sparql-endpoints"></a>
## RDF datasets - SPARQL endpoints
An RDF dataset needs to provide an endpoint for SPARQL queries so that the [KGVB backend](#get-facets-items) can access the dataset.
