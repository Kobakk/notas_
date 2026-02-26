1. Dime bajo tu criterio cual es la mejor interfaz para practicar SQL de manera gratuita. Puede ser online o en local.
Mejor interfaz para practicar SQL: 
- [Leetcode](https://leetcode.com/): Una web basada en realizar ejercicios basados en entrevistas que realizan empresas. Hay tanto para código como código como para python. 
- SQL SERVER: Adventure Works2008
1. Dime cuales son los elementos de la estructura de la sintaxis básica de una consulta o query en SQL. 
```sql
SELECT * LIMIT -> 'selección de columnas y filtrado'
FROM -> 'tabla'
JOIN -> 'union con x'
WHERE -> 'condicion'
GROUP BY  -> 'agrupación'
```

Estas sintaxis son mas usadas en motores SQL.
- Hive SQL
- Claudera Impala, Apache Impala
1. Haz una demo donde me enseñes la aplicación de un join tipo left-semi y explícame un caso de uso dónde tenga sentido aplicarlo.
Un `left-semi` suele usarse como alternativa eficiente a una subconsulta `SELECT WHERE EXITS`
```sql  
SELECT * 
FROM tb_izq
LEFT SEMI JOIN tb_d
ON tb_iqz.clave = tb_d.clave

SELECT 
FROM tb_izq
WHERE tb_izq.key EXITS IN (SELECT key FROM tb_d )

SELECT customers.customer_id, customers.name, orders.order_id
FROM customers
LEFT JOIN orders
ON customers.customer_id = orders.customer_id;

```
%%
solo queremos datos de la otra tabla, por tema de eficiencia 
-> mejor que subconsulta, poco precisa y consume muchos recursos
-> Diferencia con LEFT JOIN muestra "NULL" de aquellas coincidencias negativas
Como clientes que hayan realizado pedido de macbook:
- semi left (5 personas, y datos)
- left (10 personas, mas NULL metiando a las que no )

%%
1. Haz una demo donde me enseñes la aplicación de un join tipo left-anti y explícame un caso de uso dónde tenga sentido aplicarlo.
```sql
SELECT customers.customer_id, customers.name
FROM customers
LEFT JOIN orders ON customers.customer_id = orders.customer_id
WHERE orders.order_id IS NULL;

SELECT customers.customer_id, customers.name
FROM customers
LEFT ANTI JOIN orders
ON customers.customer_id = orders.customer_id;
```