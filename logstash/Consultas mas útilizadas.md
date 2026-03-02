
Datos del cluster: 
```json
GET _cat/indices/mercadona*?v&s=docs.count:desc
```

Resultado:

```txt
health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   mercadona_final  9S9KbXhURtKWOzb1tb4K0Q   1   1       4720            0    809.9kb        809.9kb
yellow open   mercadona_master 6zgTwEZ6T4KP0WHh05OVZA   1   1       4720            0    289.8kb        289.8kb
yellow open   mercadona_pro    cUDJ9LBURjS7cuVIEEmrmg   1   1          0            0       226b           226b

```

Búsqueda tolerante a errores `fuzziness`: 
```json
GET mercadona_master/_search
{
  "_source": ["subtitle", "price"], 
  "query": {
    "match": {
      "subtitle": {
        "query": "xocolate",
        "fuzziness": "AUTO"
      }
    }
  }
}
```

Filtro combinado Texto libre + filtro exacto
 +filtro de rango 
```json
GET mercadona_master/_search
{
  "query": {
    "bool": {
      "must": [ { "match": { "subtitle": "queso" } } ],
      "filter": [
        { "term": { "unidad": "kg" } },
        { "range": { "price": { "lte": 10.0 } } }
      ]
    }
  }
}
```

Operación múltiple con limites de resultados, para ranking:

```json
GET mercadona_master/_search
{
  "size": 5,
  "_source": ["name", "subtitle", "price"],
  "query": { "match_all": {} },
  "sort": [
    { "price": { "order": "desc" } }
  ]
}
```

Un resumen rápido y matemático: 

```json
GET mercadona_master/_search
{
  "size": 0,
  "aggs": {
    "radiografia_precios": {
      "stats": { "field": "price" }
    }
  }
}
```
Resultado: 
```json
  "hits" : {
    "total" : {
      "value" : 4720,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "radiografia_precios" : {
      "count" : 4717,
      "min" : 0.0,
      "max" : 902.0,
      "avg" : 8.130926225052463,
      "sum" : 38353.579003572464
    }
  }
```

Un group by:
```json
GET mercadona_master/_search
{
  "size": 0,
  "aggs": {
    "top_categorias": {
      "terms": { 
        "field": "name.keyword", 
        "size": 10 
      }
    }
  }
}
```
Resultado:
```json
      "buckets" : [
        {
          "key" : "Verdura",
          "doc_count" : 142
        },
        {
          "key" : "Leche y bebidas vegetales",
          "doc_count" : 121
        },
        {
          "key" : "Coloración cabello",
          "doc_count" : 100
        },
        {
          "key" : "Perfume y colonia",
          "doc_count" : 95
        },
        {
          "key" : "Insecticida y ambientador",
          "doc_count" : 73
        },
        {
          "key" : "Cerveza",
          "doc_count" : 70
        },
        {
          "key" : "Helados",
          "doc_count" : 70
        },
```

Subagregaciones: Agrupamos por categoría y calculamos su precio medio: 

```json
GET mercadona_master/_search
{
  "size": 0,
  "aggs": {
    "categorias": {
      "terms": { "field": "name.keyword", "size": 5 },
      "aggs": {
        "precio_medio": { "avg": { "field": "price" } }
      }
    }
  }
}
```

Resultado:
```json
GET mercadona_master/_search
{
  "size": 0,
  "aggs": {
    "categorias": {
      "terms": { "field": "name.keyword", "size": 5 },
      "aggs": {
        "precio_medio": { "avg": { "field": "price" } }
      }
    }
  }
}
```

Ejemplo de runtiem_fields:
```json
GET mercadona_master/_search
{
  "size": 3,
  "runtime_mappings": {
    "precio_con_iva": {
      "type": "double",
      "script": "if (doc['price'].size() > 0) emit(doc['price'].value * 1.21);"
    }
  },
  "query": { "match": { "name": "carne" } },
  "fields": ["subtitle", "price", "precio_con_iva"],
  "_source": false
}
```