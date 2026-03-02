1.  Ordenación de resultados:
```json 
GET mercadona_pro/_search
{
  "_source": ["subtitle", "price", "name"],
  "query": {
    "match": { "subtitle": "yogur" }
  },
  "sort": [
    { "price": { "order": "asc" } },
    { "name": { "order": "asc" } },
    { "_score": { "order": "desc" } } 
  ]
}
```
2.  Implementar Paginación 

- Paginación simple ( puede dar un error de memoría )
```json 
GET mercadona_final/_search
{
  "from": 20,
  "size": 10,
  "query": { "match_all": {} },
  "sort": [
    { "price": { "order": "asc" } }
  ]
}
```
- Paginación compleja 
```json 
GET mercadona_final/_search
{
  "size": 10,
  "query": { "match_all": {} },
  "sort": [
    { "price": { "order": "asc" } },
    { "id": { "order": "asc" } }
  ],
  "search_after": [1.5, "ID_DEL_ULTIMO_PRODUCTO_DE_LA_PAGINA_ANTERIOR"]
}
```

3. Definir un alias e índice 

Un alias para tener un acceso directo a un índice en especifico. 
```json 
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "mercadona_pro",
        "alias": "catalogo_actual"
      }
    }
  ]
}
```
Un alias que posea un filtro dentro de el como "ofertas", productos que lleven menos de 1 euro. 

```json 
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "mercadona_pro",
        "alias": "catalogo_ofertas_1euro",
        "filter": {
          "range": { "price": { "lt": 1.0 } }
        }
      }
    }
  ]
}
```

Y ahora realizamos una llamada sobre ese index como: 

```json
GET catalogo_ofertas_1euro/_search 
```