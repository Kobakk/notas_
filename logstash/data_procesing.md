Depende de la consulta: 
Nombre del producto debe servir tanto para buscar texto libre, como para agrupar en gráficas exactas. 

Ejemplo busqueda de producto "yogur"

```json
PUT /mercadona_master
{
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "id": { "type": "keyword" },
      "name": { 
        "type": "text", //name principal 
        "fields": {
          "keyword": { "type": "keyword" } //keyword subcampo 
        }
      },
      "price": { "type": "float" },
      "unidad": { "type": "keyword" }
    }
  }
}
```

El problema: 
text -> para busquedas amplias por ejemplo, buscar  "yogur natural " encentra " yogur", no funciona para agrupar, si tokeniza. 
keyword-> coincidencia exacta, perfecto para agrupar 

1. Cuidado, esto creara un index nuevo pero con datos vacios para moverlos necesitamos rellenarlos con los datos de la otra tabla, para ello ejecutaremos la consulta de reindex.
```json 
POST _reindex
{
  "source": {
    "index": "mercadona_final",
    "_source": ["id", "name", "price", "unidad"] 
  },
  "dest": {
    "index": "mercadona_master"
  }
}
```

2. Cuidado el **problema de spanish_analyzer** al no especificar el idioma, por ejemplo buscamos "yogur" y tenemos un fila "Yogures Naturales" -> "yogures,naturales" la normalización y tokenización se realiza pero la búsqueda por defecto de elk es muy rígida. 
Resultado:
Ningún producto encontrado 
```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  }
```

Eliminas la anterior molde: 
```json
DELETE /mercadona_master
```
Creamos el nuevo molde: 
```json
PUT /mercadona_master
{
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "id": { "type": "keyword" },
      "name": { 
        "type": "text",
        "analyzer": "spanish", 
        "fields": {
          "keyword": { "type": "keyword" } 
        }
      },
      "price": { "type": "float" },
      "unidad": { "type": "keyword" }
    }
  }
}
```
Realizamos el re-index:
```json
POST _reindex
{
  "source": {
    "index": "mercadona_final",
    "_source": ["id", "name", "price", "unidad"] 
  },
  "dest": {
    "index": "mercadona_master"
  }
}
```

Y ya deberia de dar el reslultado: 
```json 
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 81,
      "relation" : "eq"
    },
    "max_score" : 4.0682473,
    "hits" : [
      {
        "_index" : "mercadona_master",
        "_type" : "_doc",
        "_id" : "gsEnn5wBs0UJwQcTZ-pL",
        "_score" : 4.0682473,
        "_source" : {
          "price" : 6.0,
          "name" : "Yogures griegos",
          "id" : 1575
        }
      },
```

### reindexar y actualziar al vuelo 

Contexto: Tenemos un índice viejo y queremos pasarlo al nuevo y de paso añadirle un 10% de inflación. 
```json
POST _reindex
{
  "source": {
    "index": "mercadona_final",
    "_source": ["id", "name", "price", "unidad"] 
  },
  "dest": {
    "index": "mercadona_master"
  },
  "script": {
    "source": "ctx._source.price = ctx._source.price * 1.10;"
  }
}
```
Otro tip, si vemos que tenemos datos que poseen un error le añadimos una etiqueta:
```json
POST mercadona_final/_update_by_query
{
  "query": {
    "term": { "price": 0.0 }
  },
  "script": {
    "source": "ctx._source.revision_necesaria = true;"
  }
}
```

Si tenemos el anterior index rellenado de valores lo vacimos y lo volvemos a crear: 

```json
DELETE /mercadona_master

PUT /mercadona_master
{
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "id": { "type": "keyword" },
      "name": { 
        "type": "text",
        "analyzer": "spanish", 
        "fields": {
          "keyword": { "type": "keyword" } 
        }
      },
      "price": { "type": "float" },
      "unidad": { "type": "keyword" }
    }
  }
}
```

Ahora realizamos ese index con update: 

```json
POST _reindex
{
  "source": {
    "index": "mercadona_final",
    "_source": ["id", "name", "price", "unidad"] 
  },
  "dest": {
    "index": "mercadona_master"
  },
  "script": {
    "source": """
      if (ctx._source.price != null) {
        ctx._source.price = ctx._source.price * 1.10;
      }
    """
  }
}
```

Realizamos una consulta de comprobación:

```json
GET /mercadona_master/_search
{
  "size": 5,
  "_source": ["id", "name", "price", "unidad"],
  "query": {
    "match_all": {} 
  }
}
```

Resultado: 
```json
{
  "took" : 1022,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4720,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "mercadona_master",
        "_type" : "_doc",
        "_id" : "acEnn5wBs0UJwQcTZ-pL",
        "_score" : 1.0,
        "_source" : {
          "price" : 8.8,
          "name" : "Yogures naturales y sabores",
          "id" : 1550
        }
      },
```

Si queremos añadir un nuevo campo para etiquetar los datos con 0, no nos dejara porque el mapeo tiene el param `dynamic:strict` en el indice `mercadona_master` 

Añadimos la siguiente regla: 
```json
PUT /mercadona_master/_mapping
{
  "properties": {
    "revision_necesaria": { "type": "boolean" }
  }
}
```

Para que podamos realizar la siguiente consulta de etiquetado de valores nulos: 
```json
POST mercadona_master/_update_by_query
{
  "query": {
    "term": { "price": 0.0 }
  },
  "script": {
    "source": "ctx._source.revision_necesaria = true;"
  }
}
```

### tuberías de limpieza 

Definimos el pipeline, con este ejemplo de consulta: "si el producto no tiene unidad definida, ponle 'desconocida' por defecto y pasa el nombre a mayúsculas"

```json
PUT _ingest/pipeline/limpieza_mercadona
{
  "description": "Limpia unidades nulas y pone mayúsculas",
  "processors": [
    {
      "set": {
        "field": "unidad",
        "value": "desconocida",
        "override": false 
      }
    },
    {
      "uppercase": {
        "field": "name",
        "ignore_missing": true
      }
    }
  ]
}
```

Lo comprobamos con la siguiente consulta: 

```json 
POST _ingest/pipeline/limpieza_mercadona/_simulate
{
  "docs": [
    {
      "_source": {
        "id": "9999",
        "name": "galletas de chocolate",
        "price": 1.50
      }
    }
  ]
}
```

Y nos dara como resultado:

```json
{
  "docs" : [
    {
      "doc" : {
        "_index" : "_index",
        "_type" : "_doc",
        "_id" : "_id",
        "_source" : {
          "unidad" : "desconocida",
          "price" : 1.5,
          "name" : "GALLETAS DE CHOCOLATE",
          "id" : "9999"
        },
        "_ingest" : {
          "timestamp" : "2026-03-02T12:12:04.671348436Z"
        }
      }
    }
  ]
}
```
### runtime fields con painless scripting 

Creamos un campo que calsifique los productos al momento por ejemplo Barato (< 2€) Medio entre (2€ y 5€) o caro mas de 5€
```json 
GET mercadona_final/_search
{
  "size": 5,
  "_source": ["name", "price"],
  "runtime_mappings": {
    "categoria_precio": {
      "type": "keyword",
      "script": {
        "source": """
          if (doc['price'].size() == 0) { 
            emit('Sin Precio'); 
            return; 
          }
          double p = doc['price'].value;
          if (p < 2.0) { 
            emit('Barato'); 
          } else if (p <= 5.0) { 
            emit('Medio'); 
          } else { 
            emit('Caro'); 
          }
        """
      }
    }
  },
  "fields": ["name", "price", "categoria_precio"]
}
```

Y nos daría el siguiente resultado:

```json
{
  "took" : 53,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4720,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "mercadona_final",
        "_type" : "_doc",
        "_id" : "acEnn5wBs0UJwQcTZ-pL",
        "_score" : 1.0,
        "_source" : {
          "price" : 8.0,
          "name" : "Yogures naturales y sabores"
        },
        "fields" : {
          "price" : [
            8.0
          ],
          "name" : [
            "Yogures naturales y sabores"
          ],
          "categoria_precio" : [
            "Caro"
          ]
        }
      },
```
