Estructurar y gestionar los datos antes de lleguen las queries. 

```json
PUT /products
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 1,
    "refresh_interval": "30s",
    "analysis": {
      "analyzer": {
        "spanish_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "stop", "snowball"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "name":      { "type": "text", "analyzer": "spanish_analyzer" },
      "sku":       { "type": "keyword" },
      "price":     { "type": "double" },
      "stock":     { "type": "integer" },
      "active":    { "type": "boolean" },
      "created_at":{ "type": "date" }
    }
  }
}

```

- shards: particiones de un índice, es un "mini-índice" independiente que contiene una porción de los datos reales. 
	- shard primario: almacena los datos principales.
	- shard replica: copia del primario para alta disponibilidad y balanceo de carga. 
	
¿Por que existen ?
- Al trabajar con muchos nodos, necesitamos **escalabilidad**, **tolerancia a fallos** y **balanceo**. Para ello dividimos los datos para búsquedas paralelas replicamos los índices en los nodos 

```ruby
output {
	elasticsearch {
		hosts => ['localhost:9200']
		index => "logs-%{+YYY.MM.dd}"
	}
	#elasticsearch crea automáticamente;
	# 1 shard primario + 1 réplica por defecto 
}
```

Por defecto elasticsearch adivina los índices, porque es malo para la producción, para ello: 
1. verificamos el indice si el  (template) existe.
2. si no existe, definir un archivo JSON en el repo, y enviarlo a elasticsearch. 
3. En un caso de **migración de datos** que se ejecutan durante el despliegue. 

El pipeline de despliegue lanza el comando `PUT /mi_indice` con la configuración guardada en tu repositorio git. 

Definimos los templates como esquemas para que a la hora de ingestar datos, tengan una buena estructura. 

## Consultas 

- Mapping estricto, en español. 

```json
PUT /mercadona_pro
{
  "settings": {
    "analysis": {
      "analyzer": {
        "spanish_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "spanish_stop", "spanish_snowball"]
        }
      }
    }
  },
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "id":       { "type": "keyword" },
      "name":     { "type": "keyword" },
      "subtitle": { "type": "text", "analyzer": "spanish_analyzer" },
      "price":    { "type": "float" },
      "gramos":   { "type": "float" },
      "unidad":   { "type": "keyword" }
    }
  }
}
```
- Mapping mas 
```json
PUT /mercadona_pro
{
  "settings": {
    "analysis": {
      "filter": {
        "spanish_stop": {
          "type": "stop",
          "stopwords": "_spanish_"
        },
        "spanish_snowball": {
          "type": "snowball",
          "language": "Spanish"
        }
      },
      "analyzer": {
        "spanish_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "spanish_stop",
            "spanish_snowball"
          ]
        }
      }
    }
  },
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "id":       { "type": "keyword" },
      "name":     { "type": "keyword" },
      "subtitle": { "type": "text", "analyzer": "spanish_analyzer" },
      "price":    { "type": "float" },
      "gramos":   { "type": "float" },
      "unidad":   { "type": "keyword" }
    }
  }
}
```
- Ordenación y búsqueda: 
```json
GET mercadona_final/_search
{
  "_source": ["name", "subtitle", "price"], 
  "query": {
    "bool": {
      "must": [
        { "match": { "subtitle": "yogur" } }
      ],
      "filter": [
        { "range": { "price": { "lte": 2.0 } } }
      ]
    }
  },
  "sort": [
    { "price": { "order": "asc" } }
  ],
  "size": 10
}
```

- Runtime Precio por kilo, creamos un campo en tiempo de consulta usando price y gramos.
```JSON
GET mercadona_final/_search
{
  "runtime_mappings": {
    "precio_por_kilo": {
      "type": "double",
      "script": {
        "source": """
          if (doc['gramos'].size() > 0 && doc['gramos'].value > 0 && doc['price'].size() > 0) {
            // Si cuesta X y tiene Y gramos, el precio por 1000g (1kg) es:
            emit((doc['price'].value / doc['gramos'].value) * 1000);
          } else {
            emit(0.0);
          }
        """
      }
    }
  },
  "query": {
    "match": { "name": "Queso" }
  },
  "fields": ["name", "price", "gramos", "precio_por_kilo"],
  "_source": false,
  "size": 5
}
```
