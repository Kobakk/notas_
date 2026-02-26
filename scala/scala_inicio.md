
Actividad Pokemon, partimos de leer limpiar, procesar varios archivos de datos. 
##### Lectura 
A la lectura cual es la diferencia entre una lectura normal, y una lectura con esquema explícitos:.

###### Ejemplos: 

Lectura normal: 
```scala
val df = spark.read
	.option(“header”, “true”)
	.option(“inferSchema”, “true”)
	.option(“datos.csv”)
```
Esquemas explícitos:
```scala
import org.apache.spark.sql.types._

val esquema = StructType(Array(
	StructField("id", IntegerType, nullable = false),
	StructField("nombre", StringType, nullable = true),
	StructField("fecha", DateType, nullable = false)
))

val df = spark.read
	.schema(esquema)
	.option("header", "true")
	.csv("datos.csv")
```

Pseudocódigo

```
funcion(param_especificado(
	campo_clase("param_entrada", tipoClase, booleano),
	campo_clase("param_entrada", tipoClase, booleano)
))
```

```scala
StructType(Seq(
	StructField("#", IntegerType, True)
))
```
##### Planteamiento

Una lectura estructurada frente a una lectura simple. 
###### Conclusión:

La lectura normal puede resultar **fácil**  y **rápida** pero es mas lenta y tiende a mas errores. 

La lectura con esquemas esta **mejor estructurada** y **tiende menos a errores**, es **una buena practica**.

El casteo de la clase. 
##### Consultas