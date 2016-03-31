GraphAware Neo4j Elasticsearch Integration (ES Module)
======================================================

[![Build Status](https://magnum.travis-ci.com/graphaware/elasticsearch-to-neo4j.svg?token=tFjWxABA1S1VaGsxdhvX)](https://magnum.travis-ci.org/graphaware/elasticsearch-to-neo4j) | Latest Release: none yet!

GraphAware Elasticsearch Integration is an enterprise-grade bi-directional integration between Neo4j and Elasticsearch.
It consists of two independent modules plus a test suite. Both modules can be used independently or together to achieve
full integration.

The [first module](https://github.com/graphaware/neo4j-to-elasticsearch) is a plugin to Neo4j (more precisely, a [GraphAware Transaction-Driven Runtime Module](https://github.com/graphaware/neo4j-framework/tree/master/runtime#graphaware-runtime)),
which can be configured to transparently and asynchronously replicate data from Neo4j to Elasticsearch. This module is now
production-ready and officially supported by GraphAware for  <a href="http://graphaware.com/enterprise/" target="_blank">GraphAware Enterprise</a> subscribers.

The second module(this module) is a plugin to Elasticsearch that can consult the Neo4j database during an Elasticsearch query to enrich
the result (boost the score) by results that are more efficiently calculated in a graph database, e.g. recommendations.
This module is in active alpha development and isn't yet officially supported. We expect it to be production-ready by
the end of 2015.

# Elasticsearch -> Neo4j

## Feature Overview: Graph Aided Search

This module is a plugin for Elasticsearch that allow to improve the search result boosting or filtering them using data stored in the neo4j graph database. 
After performing the search on Elasticsearch, and just before returing results to the user, this plugin is able to submit some requests 
to the graph database through the REST api to get information needed to boost or filter the results and then get back the results to the user.

Two main features are exposed by the plugin: 

* **_Result Boosting_**: This feature allow to change the score value of the results. The score can be changed in different ways, 
mixing graph score with elasticsearch score or replacing it entirely are just two examples. 
It is possible to customize this behaviour with different formulas, rewriting some methods of the Graph Aided Search Booster. 
Usage examples include boosting (i) based on interest prediction (recommendations), (ii) based on friends interests/likes, (iii) whichever queries on neo4j
 
* **_Result Filtering_**: This feature allow to filter results removing documents from the results list. In this case, providing a cypher query, it is possible to return to the user only the document which id match results from cypher query.

More in details it operates in the following way:
1. Intercepts any “search” to the elasticsearch to find query extension;
2. Processes query extension identifying the type of the extension, if an boosting or a filter, and instantiates the related class;
3. Performs the operation required to boost or filter connecting to the neo4j rest API (or some extension to neo4j like Graphaware Recommendation Engine)
passing information needed, like cypher query, targe user, and so on;
4. Replies back to the user that submit the query.

## Usage: Installation

### Install Graph Aided Search Binary

    $ $ES_HOME/bin/plugin install com.graphaware/graph-aided-search/2.2.1

### Build from source

    $ git clone git@github.com:graphaware/elasticsearch-to-neo4j.git
    $ mvn clean deploy
    $ $ES_HOME/bin/plugin install file:///path/to/project/elasticsearch-to-neo4j/target/releases/elasticsearch-to-neo4j-2.2.1.zip
    
Start elasticsearch

### Configuration

Then configure indexes with the url of the neo4j. This can be done in two way:

    $ curl -XPUT http://localhost:9200/indexName/_settings?index.gas.neo4j.hostname=http://localhost:7474
    $ curl -XPUT http://localhost:9200/indexName/_settings?index.gas.enable=true

Or add it to the settings in the index template:

```
    GET  _template/template_gas
    {
      "template": "*",
      "settings": {
        "index.gas.neo4j.hostname": "http://localhost:7474",
        "index.gas.enable": true
      }
    }
```

### Disable Plugin

The query will continue to work with no issue, even with the "gas-boost" and "gas-filter" piece in the query. They will be removed automatically.

    $ curl -XPUT http://localhost:9200/indexName/_settings?index.gas.enable=false

## Usage: Search Phase

The integration with already existing query is seamless, since the plugin requires to add only some new pieces into the query. 
### Booster example

If you would like to search for all the Movie in the es dataset you should run a query like this:

```
  curl -X POST http://localhost:9200/neo4j-index/Movie/_search -d '{
    "query" : {
        "match_all" : {}
    }';
```
In this case you'll get as score value 1 for all the results. If you would like to boost results accordingly to user interest computed by Graphaware 
Recommendation Plugin on top of Neo4j you should change the query in the following way.


```
  curl -X POST http://localhost:9200/neo4j-index/Movie/_search -d '{
    "query" : {
        "match_all" : {}
    },
    "gas-booster" :{
          "name": "GraphAidedSearchNeo4jBooster",
          "recoTarget": "2",
          "maxResultSize": 10,
          "keyProperty": "objectId",
          "neo4j.endpoint": "/graphaware/recommendation/movie/filter/"
       }
  }';
```
The _gas-booster_ clause identify the type of operation, in this case it is required a boost operation. 
The _name_ parameter is mandatory and allows to specify the Booster class. The remaining parameters depends on the type of booster.
In the following paragraph the available boosters are described.

#### GraphAidedSearchNeo4jBooster


dsada

#### GraphAidedSearchCypherBooster

dsada



Filter Example...


The _gas-filter_ clause identify the type of operation, in this case it is required a filter operation.

The following Filter classes are alrea




## Customize the plugin

The plugin allows to implement custom booster and custom filter. In order to implements 

## Version Matrix

The following version are currently supported

| Version   | Elasticsearch |
|:---------:|:-------------:|
| master    | 2.3.x         |
| 2.2.1.x   | 2.2.1         |
| 2.2.0.x   | 2.2.0         |
| 2.1.1.x   | 2.1.1         |

### Issues/Questions

Please file an [issue](https://github.com/graphaware/elasticsearch-to-neo4j/issues "issue").

License
-------

Copyright (c) 2016 GraphAware

GraphAware is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License
as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.
This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied
warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.
You should have received a copy of the GNU General Public License along with this program.
If not, see <http://www.gnu.org/licenses/>.
