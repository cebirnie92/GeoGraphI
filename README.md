# GeoGraphI
An interactive graph database of openly available seismic datasets

Overview
--------
Over the years numerous geophysical datasets have been released for public usage. However, with no central storage location or consistent description strategy, finding suitable openly available datasets still poses a large challenge to the geophysics community. GeoGraphI aims to tackle this problem by providing a single access point with the necessary structured information to search for suitable datasets.

GeoGraphI is a graph database, originally built using the Neo4J graph database management system, with the provided example queries wrote in the Cypher query language. With data subsets ranging from passive seismic to migrated volumes, and from core images to interpreted horizons, GeoGraphI can be queried either by key information matching, such as *Return a field seismic dataset acquired over a salt body*, or by computing similarity scores, such as *Return similar field datasets to the SEAM dataset*.

The database schema is an extension to that developed by the Open Subsurface Data Universe with additional, less-technical dataset descriptors which highlight the interesting features related to a dataset. For example, is the survey acquired in an area of CO_2 injection, or in an area of turbidites, or perhaps the data is plagued by simultaneous shooting, or surface related multiples. 

Finally, being a graph database, GeoGraphI can naturally handle a large number of relationships between different data features as well as being able to easily adapt for future growth, either through further population of the database or by modification of the underlying schema.

Loading GeoGraphI
-----------------
A dump of the dataset is in the `*.dump` file and can be loaded into a local instance of neo4j by running the following command:

```neo4j-admin load --from=<archive-path> --database=<database> [--force]```

Or by using the neo4J browser - see https://tbgraph.wordpress.com/2020/11/11/dump-and-load-a-database-in-neo4j-desktop/ for guidance.

Schema
------
The current schema is as follows:

|          Node         |    Properties    |                      Description                     |              Example             |
|:---------------------:|:----------------:|:----------------------------------------------------:|:--------------------------------:|
| Dataset               | id               | Unique dataset identifier                            | 001                              |
|                       | name             | The common name given to the dataset                 | Northern Lights                  |
|                       | link             | web link to the general overview of the dataset      | https://data.equinor.com/data... |
| Operating Environment | type             | Marine, Land or Cross-environment                    | Marine                           |
| Generation Procedure  | type             | Synthetic, Lab or Field                              | Field                            |
| Geographic Region     | name             | Continent of survey                                  | Europe                           |
| Seismic Geometry      | type             | 2D, 3D, 4D, or Passive                               | 2D                               |
| Energy Source         | type             | Type of seismic source used                          | Airgun                           |
| Receiver Type         | type             | Recording Instrument                                 | Geophone                         |
|                       | location         | Surface or Borehole                                  | Borehole                         |
|                       | no_of_components | Number of components in each receiver                | 3C                               |
| Seismic Subset        | id               | Unique seismic subset identifier                     | 002                              |
|                       | processing_stage | The data processing stage                            | VSP                              |
|                       | data_format      | The file format in which the data is available       | segy                             |
|                       | link             | Web link for downloading the data                    | https://data.equinor.com/data... |
| Supplementary Subset  | id               | Unique supplementary subset identifier               | 003                              |
|                       | data_type        | The type of supplementary data available             | Well Logs                        |
|                       | link             | Web link for downloading the supplementary data      | https://data.equinor.com/data... |
| Interesting Features  | type             | Interesting features whether geological or artefacts | CO2 Storage                      |

With the relationships between the nodes as illustrated in the following plot:
![Alt text](Figures/schema.png?raw=true "GeoGraphI Schema")

Input data
----------
The input data is stored in an excel spreadsheet of which the sheets are saved seperately prior to being used to populate the graph. 

Example queries
---------------
A simple use case is identifying a dataset which matches a few criteria, for example it has a post-stack seismic dataset as well as a horizon set.

```
MATCH p=(s:SeismicSubset)-[]-(d:Dataset)-[]-(n:SupplementarySubset) 
WHERE n.data_type='Horizons' and s.processing_stage='Poststack Seismic'
RETURN p
```

A more advanced use case is to find a dataset that is most similar to another dataset. In this case we need to make a projection of the graph with only the relevant nodes for the similarity comparison (query 1) and then we can compute the Jaccard Similarity Score between the different datasets (query 2). 

Query 1:
```
CALL gds.graph.create("dataset-graph",
['Dataset', 'OperatingEnvironment', 'SeismicGeometry', 'Feature', 'ReceiverType', 'EnergySource'],
['from_operating_environment','has_survey_geometry', 'contains_feature', 'recorded_by', 'uses_seismic_source'])
YIELD nodeCount, relationshipCount
```

Query 2:
```
CALL gds.nodeSimilarity.stream('dataset-graph')
YIELD node1, node2, similarity
WHERE gds.util.asNode(node2).name='SEAM P1 Elastic'
RETURN gds.util.asNode(node1).name as Dataset1, gds.util.asNode(node2).name as Dataset2, similarity
ORDER BY similarity DESCENDING, Dataset1, Dataset2
```

Whilst originally developed for identifying a dataset to work with, GeoGraphI can also be queried to see all available information on a specific dataset. 

```
MATCH p=(d:Dataset)-[]-() 
WHERE d.name='Northern Lights' 
RETURN p
```

Future Plans
------------
GeoGraphI was built to be shared with the geoscience community. On its initial release (April 2021), GeoGraphI has a strong seismic focus and is in at an MVP-stage. The hope is that via crowd-sourcing GeoGraphI will grow to have a more general geoscience focus, for example by including substantially more well datasets. As it grows the schema will undergo many revisions to remain flexible and providing the best possible information relevant to the datasets that populate the graph.

Contributing
------------
If you would like to add data to the GeoGraphI, please add the necessary information into the excel input file. 

Additionally, if you have any feedback on the schema or notice any data inaccuracies then please open a GitHub Issue which we will respond to asap.

If GeoGraphI helps you find data then please let us know - we love to hear that it has been of use 😊
