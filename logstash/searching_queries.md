
1. Búsqueda en términos o más campos
```JSON
GET mercadona_pro/_search
{
  "query": {
    "multi_match": {
      "query": "aceite de oliva",
      "fields": ["name^2", "subtitle"], 
      "type": "best_fields"
    }
  }
}
```
2. Combinación Booleana (Queries + Filters)
```json
GET mercadona_pro/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "subtitle": "chocolate" } }
      ],
      "must_not": [
        { "match": { "subtitle": "blanco" } }
      ],
      "filter": [
        { "range": { "price": { "lte": 2.50 } } }
      ]
    }
  }
}
```
3. Búsqueda Asíncrona
```json
POST mercadona_pro/_async_search?wait_for_completion_timeout=1s
{
  "query": {
    "match": { "name": "carne" }
  },
  "aggs": {
    "precio_medio": {
      "avg": { "field": "price" }
    }
  }
}
```
4. Agregaciones métricas y buckets

El término bucket, se refiere a una **agrupación lógica de datos en memoria**, equivalente a un  **GROUP BY** de SQL. 

Estas agrupaciones de datos en memoria se realizán al momento. 
```json 
GET mercadona_pro/_search
{
  "size": 0,
  "aggs": {
    "tipos_de_unidad": {
      "terms": { "field": "unidad.keyword", "size": 10 }
    },
    "precio_mas_caro_del_super": {
      "max": { "field": "price" }
    }
  }
}
```
5. Sub-agreagaciones
```json
GET mercadona_pro/_search
{
  "size": 0,
  "aggs": {
    "categorias": {
      "terms": { "field": "name.keyword", "size": 5 },
      "aggs": {
        "precio_medio_de_esta_categoria": {
          "avg": { "field": "price" }
        }
      }
    }
  }
}
```
6. Búsqueda a través de múltiples clusters:
Si poseemos mas de un cluster, en otra ubicación y realizamos una consulta por cada ubicación del cluster
```json
GET mercadona_pro,portugal_cluster:mercadona_pt/_search
{
  "query": {
    "match": { "subtitle": "bacalao" }
  }
}
```
7. Búsqueda utilizando un runtime Field
```json
GET mercadona_pro/_search
{
  "runtime_mappings": {
    "precio_con_iva": {
      "type": "double",
      "script": {
        "source": "if (doc['price'].size() > 0) { emit(doc['price'].value * 1.10); } else { emit(0.0); }"
      }
    }
  },
  "query": {
    "range": {
      "precio_con_iva": { "gte": 5.0 }
    }
  },
  "fields": ["name", "price", "precio_con_iva"],
  "_source": false
}
```