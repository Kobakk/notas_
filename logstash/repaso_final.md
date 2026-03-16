
1. Índice con requisitos específicos, con un tipado estricto para evitar que se indexen campos basura. 
```js
PUT /covid-cases-v1
{
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "@timestamp": { "type": "date" },
      "cases": { "type": "integer" },
      "deaths": { "type": "integer" },
      "countriesAndTerritories": {
        "type": "text",
        "fields": {
          "keyword": { "type": "keyword" }
        }
      },
      "geoId": { "type": "keyword" },
      "countryterritoryCode": { "type": "keyword" },
      "continentExp": { "type": "keyword" },
      "popData2020": { "type": "long" }
    }
  }
}
```
Resultado, mensaje de true o realizado:
```js
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "covid-cases-v1"
}
```
2. Plantilla Dinámica
```js
PUT /covid-dynamic-2
{
  "mappings": {
    "dynamic_templates": [
      {
        "id_fields": {
          "match_pattern": "regex",
          "match": ".*Id$",
          "mapping": { "type": "keyword" }
        }
      },
      {
        "pop_fields": {
          "match_pattern": "regex",
          "match": "^pop.*",
          "mapping": { "type": "long" }
        }
      },
      {
        "strings_as_text_keyword": {
          "match_mapping_type": "string",
          "mapping": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      }
    ]
  }
}
```
Resultado: 
```js
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "covid-cases-v1"
}
```
3. ILM, para series temporales 
```js
PUT _ilm/policy/covid-ilm-policy-2
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_age": "30d",
            "max_size": "50gb"
          }
        }
      },
      "warm": {
        "min_age": "60d",
        "actions": {
          "shrink": {
            "number_of_shards": 1
          }
        }
      },
      "delete": {
        "min_age": "180d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```
Resultado: 
```js
{
  "acknowledged" : true
}
```

Creamos el primer índice enlazado a la política y el alias: 

```json
PUT /covid-timeseries-000001
{
  "settings": {
    "index.lifecycle.name": "covid-ilm-policy",
    "index.lifecycle.rollover_alias": "covid-timeseries"
  },
  "aliases": {
    "covid-timeseries": {
      "is_write_index": true
    }
  }
}
```

4. Creamos un datastream:
```json
PUT _index_template/covid-ds-template
{
  "index_patterns": ["covid-ds-*"],
  "data_stream": { },
  "template": {
    "settings": {
      "index.lifecycle.name": "covid-ilm-policy"
    },
    "mappings": {
      "properties": {
        "@timestamp": { "type": "date" },
        "cases": { "type": "integer" },
        "deaths": { "type": "integer" }
      }
    }
  }
}
```
## Busqueda de datos 

5. Buscamos el continente que sea "Europe" y el país que contenga la frase exacta "Austria": 

```json
GET /covid-cases-v1/_search
{
  "query": {
    "bool": {
      "must": [
        { "match_phrase": { "countriesAndTerritories": "Austria" } },
        { "term": { "continentExp": "Europe" } }
      ]
    }
  }
}
```
6. Casos mayores de 10.000 entre 01-07-22 al 01-10-22, cuando las muertes sean 0 y ordenando los casos de forma decente. 
```json 
GET /covid-cases-v1/_search
{
  "query": {
    "bool": {
      "must": [
        { "range": { "cases": { "gt": 10000 } } },
        { "range": { "@timestamp": { "gte": "2022-07-01", "lte": "2022-10-01" } } }
      ],
      "must_not": [
        { "term": { "deaths": 0 } }
      ]
    }
  },
  "sort": [
    { "cases": { "order": "desc" } }
  ]
}
```
7.  Busqueda asíncrona para calcular el total de casos del año 2022:
```json
POST /covid-cases-v1/_async_search
{
  "size": 0,
  "query": {
    "range": {
      "@timestamp": { "gte": "2022-01-01", "lte": "2022-12-31" }
    }
  },
  "aggs": {
    "total_casos_2022": {
      "sum": { "field": "cases" }
    }
  }
}
```
8. Agregaciones, calcular la suma total de casos, la media de muertes por continente y ordenando por el total de casos descendente. 
```json 
GET /covid-cases-v1/_search
{
  "size": 0,
  "aggs": {
    "por_continente": {
      "terms": {
        "field": "continentExp",
        "order": { "total_casos": "desc" }
      },
      "aggs": {
        "total_casos": { "sum": { "field": "cases" } },
        "media_muertes": { "avg": { "field": "deaths" } }
      }
    }
  }
}
```

10. Busqueda global de casos combinados
```json 
GET /covid-cases-*,europe_cluster:covid-cases-*/_search
{
  "size": 0,
  "aggs": {
    "total_global_casos": {
      "sum": { "field": "cases" }
    }
  }
}
```
11. Crear un campo evaluado en tiempo de ejecución: 
```json 
GET /covid-cases-v1/_search
{
  "_source": ["countriesAndTerritories", "@timestamp"],
  "fields": ["mortality_rate"],
  "runtime_mappings": {
    "mortality_rate": {
      "type": "double",
      "script": {
        "source": "if (doc['cases'].value > 0) { emit((double) doc['deaths'].value / doc['cases'].value); } else { emit(0.0); }"
      }
    }
  },
  "query": {
    "range": {
      "mortality_rate": { "gt": 0.01 }
    }
  }
}
```
13. Paginación 
```json 
GET /covid-cases-v1/_search
{
  "from": 20,
  "size": 10,
  "query": {
    "match_all": {}
  }
}
```
14. Index Alias 
```json
POST /_aliases
{
  "actions": [
    { "add": { "index": "covid-cases-v1", "alias": "covid-active" } }
  ]
}
```
Realizamos el cambio:
```json 
POST /_aliases
{
  "actions": [
    { "remove": { "index": "covid-cases-v1", "alias": "covid-active" } },
    { "add": { "index": "covid-timeseries-*", "alias": "covid-active" } }
  ]
}
```

## Procesamiento de datos 
15. Mapeo avanzado 
```json 
PUT /covid-adv-mapping
{
  "settings": {
    "number_of_shards": 2,
    "analysis": {
      "normalizer": {
        "lowercase_normalizer": {
          "type": "custom",
          "filter": ["lowercase"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "countriesAndTerritories": {
        "type": "text",
        "analyzer": "standard",
        "fields": {
          "english": {
            "type": "text",
            "analyzer": "english"
          }
        }
      },
      "continentExp": {
        "type": "keyword",
        "normalizer": "lowercase_normalizer"
      },
      "cases": {
        "type": "integer",
        "doc_values": true
      },
      "deaths": {
        "type": "integer",
        "index": false,
        "store": true
      }
    }
  }
}
```
16. Multi campos : Actualizamos el mapeo del índice anterior para añadir los multi-fields al continente: 
```json 
PUT /covid-adv-mapping/_mapping
{
  "properties": {
    "continentExp": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword",
          "ignore_above": 256
        },
        "wildcard": {
          "type": "wildcard"
        }
      }
    }
  }
}
```
17. Reindexar y actualizar por consulta: 
```json 
POST _reindex
{
  "source": { "index": "covid-cases-v1" },
  "dest": { "index": "covid-cases-v2" },
  "script": {
    "source": "if (ctx._source.popData2020 instanceof String) { ctx._source.popData2020 = Long.parseLong(ctx._source.popData2020) }",
    "lang": "painless"
  }
}
``` 
añadimos  el campo de high_incidence donde cases > 15000

```json
POST covid-cases-v2/_update_by_query
{
  "query": {
    "range": {
      "cases": { "gt": 15000 }
    }
  },
  "script": {
    "source": "ctx._source.high_incidence = true",
    "lang": "painless"
  }
}
```
18. Pipeline de ingesta: creamos el pipeline que transforma los datos justo anter de que se guarden en el indice: 
```json 
PUT _ingest/pipeline/covid-pipeline
{
  "processors": [
    {
      "convert": {
        "field": "popData2020",
        "type": "long",
        "ignore_missing": true
      }
    },
    {
      "script": {
        "lang": "painless",
        "source": """
          if (ctx.cases != null && ctx.deaths != null) {
            if (ctx.cases == 0) {
              ctx.case_fatality_ratio = 0.0;
            } else {
              ctx.case_fatality_ratio = (double) ctx.deaths / ctx.cases;
            }
          }
        """
      }
    },
    {
      "set": {
        "field": "ingestion_date",
        "value": "{{_ingest.timestamp}}"
      }
    }
  ]
}
```
19. Primero definimos el runtime field persistente en el mapeo de v2 con la fórmula: 
```json 
PUT /covid-cases-v2/_mapping
{
  "runtime_mappings": {
    "cases_per_100k": {
      "type": "double",
      "script": {
        "source": "if (doc['popData2020'].size() != 0 && doc['popData2020'].value > 0) { emit((doc['cases'].value / (double)doc['popData2020'].value) * 100000) } else { emit(0.0) }"
      }
    }
  }
}
```
Y por último lanzamos la búsqueda del top 10 de los días ordenados de mayor a menor por ese nuevo campo evaluado al vuelo: 
```json
GET /covid-cases-v2/_search
{
  "size": 10,
  "query": {
    "match_all": {}
  },
  "fields": [
    "@timestamp",
    "countriesAndTerritories",
    "cases_per_100k"
  ],
  "_source": false,
  "sort": [
    {
      "cases_per_100k": { "order": "desc" }
    }
  ]
}
```