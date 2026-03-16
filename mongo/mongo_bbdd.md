# base de datos
Creamos la base de datos

```js
use database;
```
Errores: 
- Si no creamos todas las colecciones en una base de datos, nos darán error las consultas de unión para ello especificamos que deberán ser en diferentes columnas. 

## consultas simples
- clientes: 
```js
db.clientes.insertMany([
  { nombre: "Juan Pérez", telefono: "600111222", ciudad: "Logroño", cliente_habitual: true },
  { nombre: "María Gómez", telefono: "600333444", ciudad: "Madrid", cliente_habitual: false },
  { nombre: "Carlos Ruiz", telefono: "600555666", ciudad: "Barcelona", cliente_habitual: true },
  { nombre: "Ana López", telefono: "600777888", ciudad: "Valencia", cliente_habitual: false },
  { nombre: "Luis García", telefono: "600999000", ciudad: "Sevilla", cliente_habitual: true }
])
```
- vehículos : 
```js
db.vehiculos.insertMany([
  { matricula: "1234ABC", marca: "Seat", modelo: "Ibiza", anio: 2018, combustible: "Gasolina" },
  { matricula: "5678DEF", marca: "Ford", modelo: "Focus", anio: 2020, combustible: "Diesel" },
  { matricula: "9012GHI", marca: "Toyota", modelo: "Corolla", anio: 2019, combustible: "Híbrido" },
  { matricula: "3456JKL", marca: "Volkswagen", modelo: "Golf", anio: 2021, combustible: "Gasolina" },
  { matricula: "7890MNO", marca: "Renault", modelo: "Clio", anio: 2017, combustible: "Diesel" }
])
```
- mecánicos : 

```js
db.mecanicos.insertMany([
  { nombre: "Pedro Martínez", especialidad: "Motor", experiencia_anios: 10, turno: "Mañana" },
  { nombre: "Laura Fernández", especialidad: "Electricidad", experiencia_anios: 5, turno: "Tarde" },
  { nombre: "Javier Sánchez", especialidad: "Chapa y Pintura", experiencia_anios: 8, turno: "Mañana" },
  { nombre: "Sofía Díaz", especialidad: "Mantenimiento General", experiencia_anios: 3, turno: "Tarde" }
])
```

- reparaciones

```js
db.reparaciones.insertMany([
  { matricula_vehiculo: "1234ABC", descripcion: "Cambio de aceite y filtros", coste: 85.50, estado: "Completado" },
  { matricula_vehiculo: "5678DEF", descripcion: "Revisión de frenos", coste: 120.00, estado: "Pendiente" },
  { matricula_vehiculo: "9012GHI", descripcion: "Sustitución de batería", coste: 95.00, estado: "Completado" },
  { matricula_vehiculo: "3456JKL", descripcion: "Reparación junta culata", coste: 450.00, estado: "En proceso" },
  { matricula_vehiculo: "7890MNO", descripcion: "Pintura puerta lateral", coste: 150.00, estado: "Completado" }
])
```

## consultas 

Consultas combinadas:
```js
db.reparaciones.aggregate([
	{
		$lookup: {
			from: "vehiculos",
			localField: "matricula_vehiculo",
			foreignField: "matricula",
			as: "datos_del_coche"
		}
	},
	{
		$limit: 3
	}
])
```

1. historial de cada coche 
```js
db.vehiculos.aggregate([
	{
		$lookup: {
			from: "reparaciones", // tabla a lo que queremos unir
			localField: "matricula", // clave local 
			foreignField: "matricula_vehiculo", // clave foranea
			as: "historiaL_reparaciones" // alias
		}
	},
	{
		$limit: 1
	}
])
```
2. Reparaciones pendientes y sus coches ( Filtro + Join )
```js
db.vehiculos.aggregate([
	{
		$match: { estado: "En proceso" }
	},
	{
		$lookup: {
			from: "reparaciones",
			localField: "matricula",
			foreingField: "matricula_vehiculo",
			as:"historial_reperaciones"
		}
	},
	{
		$limit: 1
	}
])
```
Error: 
El match busca en nuestra tabla local que seria vehículos, no tenemos el campo "estado" en nuestro vehículo, es de la otra tabla.

Corrección el orden: 

```js
db.vehiculos.aggregate([

	{
		$lookup: {
			from: "reparaciones",
			localField: "matricula",
			foreingField: "matricula_vehiculo",
			as:"historial_reperaciones"
		}
	},
	{
		$match: { estado: "En proceso" }
	},
	{
		$limit: 1
	}
```

Error: Tenemos que nombrar el alias con el campo 

```js
$match: { "historial_reparaciones.estado": "En proceso"}
```

además de escribir `foreign` en vez de `foreing`

Resultado: 
```js
{
  _id: ObjectId('69a6a7ff5838c3b6c621dfa6'),
  matricula: '3456JKL',
  marca: 'Volkswagen',
  modelo: 'Golf',
  anio: 2021,
  combustible: 'Gasolina',
  historial_reparaciones: [
    {
      _id: ObjectId('69a6a7855838c3b6c621df9d'),
      matricula_vehiculo: '3456JKL',
      descripcion: 'Reparación junta culata',
      coste: 450,
      estado: 'En proceso'
    }
  ]
}
```

3. El primero coche Diésel y sus reparaciones, 
```js
db.vehiculos.aggregate([
	{
		$match: { "vehiculos.combustible": 'Diesel'  }
	},
	{
		$lookup: {
			from: "reparaciones",
			localField: "matricula",
			foreignField: "matricula_vehiculo",
			as: "historial_reparaciones"
		}
	},
	{
		$limit: 1
	}
])
```
Error: el `vehiculos.combustible` el alias.campo solo se usa cuando vamos a buscar en un alias o campo creado. Solo pondríamos el campo `combustible`

Resultado: 
```js
{
  _id: ObjectId('69a6a7ff5838c3b6c621dfa4'),
  matricula: '5678DEF',
  marca: 'Ford',
  modelo: 'Focus',
  anio: 2020,
  combustible: 'Diesel',
  historial_reparaciones: [
    {
      _id: ObjectId('69a6a7855838c3b6c621df9b'),
      matricula_vehiculo: '5678DEF',
      descripcion: 'Revisión de frenos',
      coste: 120,
      estado: 'Pendiente'
    }
  ]
}
```

4. Ver reparaciones que cuesten mas de 90€ pero menos de 200€ y que el resultado salgo ordenado alfabéticamente: 
```js
db.reparaciones.aggregate([
	{
		$match: { coste: $gt: 90, $lt: 200 }
	},
	{
		$sort: {  description: 1 }
	}
])
```

Error:  Corchete cuando hay varios parámetros en el  `$match: { coste: $gt:90, $lt:200}` -> `$match: { coste: { $gt:90, $lt: 200}}`

5. Múltiples condiciones exactas: buscar vehículos que sean de gasolina que sean del año 2018 o más nuevos lo cruzaremos con reparaciones y lo ordenaremos alfabéticamente: 
```js
db.vehiculos.aggregate([
  {
    $match: { combustile: "Gasolina", anio: { $gte: 2018 } }
  },
  {
    $lookup: {
      from: "reparaciones",
      localField: "matricula",
      foreignField: "matricula_vehiculo",
      as: "historial_reparaciones"
    }
  },
  {
    $sort: { marca: 1 }
  }
])
```

Error: Los errores de sintaxis dan como resultado la propia consulta, es combustible no combustile

Resultado: 
```js
{
  _id: ObjectId('69a6a7ff5838c3b6c621dfa3'),
  matricula: '1234ABC',
  marca: 'Seat',
  modelo: 'Ibiza',
  anio: 2018,
  combustible: 'Gasolina',
  historial_reparaciones: [
    {
      _id: ObjectId('69a6a7855838c3b6c621df9a'),
      matricula_vehiculo: '1234ABC',
      descripcion: 'Cambio de aceite y filtros',
      coste: 85.5,
      estado: 'Completado'
    }
  ]
}
```

Otro tip: 
```js
db.vehiculos.find({
	combustible: { $nin: ["Gasolina", "Diesel"]}
})
//si solo no queremos diesel 
db.vehiculos.find({
	combustible: { $ne: "Diesel"}
})
```

6. Ultima consulta doble join: 
```js
db.citas.aggregate([
    {
      $lookup: {
        from: "vehiculos",
        localField: "matricula_vehiculo",
        foreignField: "matricula",
        as: "datos_coche"
    	}
    },
    {
      $lookup: {
        from: "clientes",
        localField: "telefono_cliente",
        foreignField: "telefono",
        as: "datos_dueño"
      }
    }
])
```
Resultado:
```js
{
  _id: ObjectId('69a6ce663e01c34bf54816e0'),
  id_cita: 'C-001',
  fecha: '2026-03-04',
  matricula_vehiculo: '1234ABC',
  telefono_cliente: '600111222',
  motivo: 'Ruido en el motor',
  datos_coche: [
    {
      _id: ObjectId('69a6a7ff5838c3b6c621dfa3'),
      matricula: '1234ABC',
      marca: 'Seat',
      modelo: 'Ibiza',
      anio: 2018,
      combustible: 'Gasolina'
    }
  ],
  'datos_dueño': [
    {
      _id: ObjectId('69a6a81b5838c3b6c621dfa8'),
      nombre: 'Juan Pérez',
      telefono: '600111222',
      ciudad: 'Logroño',
      cliente_habitual: true
    }
  ]
}
```

7. Contar la cantidad de reparaciones que posee un vehículo: 
```js
db.reparaciones.aggregate([
	{
		$group: {
			_id: "$matricula_vehiculo", //campo que queremos agrupar
			cantidad_reparaciones: { $sum: 1} //nombre del contenedor y sume 1 por cada registro
		}
	},
	{
		$lookup: {
			from: "vehiculos", // a coches
			localField: "_id", // y apuntamos a nuestro id
			foreignField: "matricula",
			as: "datos_del_coche"
		}
	}
])
```

Resultado:

```js
{
  _id: '3456JKL',
  cantidad_reparaciones: 1,
  datos_del_coche: [
    {
      _id: ObjectId('69a6a7ff5838c3b6c621dfa6'),
      matricula: '3456JKL',
      marca: 'Volkswagen',
      modelo: 'Golf',
      anio: 2021,
      combustible: 'Gasolina'
    }
  ]
}
```

En sql: 

```sql
SELECT
	v.matricula,
	v.marca,
	v.modelo,
	COUNT(r.matricula_vehiculo) AS cantidad_reparaciones
FROM vehiculos v
INNER JOIN reparaciones r
	ON v.matricula, v.marca, v.modelo 
```

## consultas complejas

1. citas del taller, cruzarlas con vehículos, queremos solo fecha, motivo de cita, marca y modelo de coche
```js
db.citas.aggregate([
  {
    $lookup: {
      from: "vehiculos",
      localField: "matricula_vehiculo",
      foreignField: "matricula",
      as: "datos_coche"
    }
  },
  {
    $unwind: "$datos_coche"
  },
  {
    $project: {
      _id: 0,
      fecha_cita: "$fecha",
      motivo: 1,
      marca_coche: "$datos_coche.marca",
      modelo_coche: "$datos_coche.modelo"
    }
  }
])
```
Errorr: Necesitamos utilizar el $unwind para especificar mejor los campos que vamos  a mostrar, 0 o 1 para mostrar o no y $campo o $colección.campo si es de la tabla que traemos. 

2. Dinero gastado por marca de coche
```js
db.reparaciones.aggregate([
	{
		$lookup: {
			from: "vehiculos",
			localField: "matricula_vehiculo",
			foreignField: "matricula",
			as: "vehiculo_info"
		}
	},
	{
		$unwind: "$vehiculo_info",
	},
	{
		$group: {
			_id: "$vehiculo_info.marca" ,
			facturacion_total: { $sum: "$coste" } // suma costes de la marca
		}
	},
	{
		$project: {
			_id: 0,
			Marca: "$_id",
			total_Facturado: "$facturación_total"
		}
	},
	{
		$sort: { Total_Facturado: -1 }
	}
])
```
- reparaciones :
id 'objeto', 
matricula_vehículo: '1234abc',
descripción: 'cambio de aceite',
coste: 80.0 ,
estado: 'Completo'
- vehículos: 
id: 'objeto',
matricula: '1234abcd',
marca: 'seat',
modelo: 'Ibiza',
anio:2018,
combustible: 'gasolina' 
3. Si queremos contar vehículo por marca:
```js
{
	$group: {
		_id: "$vehiculo_info.marca",
		vehiculos_marca: { $sum: 1}
	}
},
{
	$project: {
		_id: 0, 
		Marca: "$_id"
		cant_vehiculos: "$vehiculos_marca"
	}
}
```

Un resumen de los tipos de group_by: 

```js
db.reparaciones.aggregate([
	{
		$group: {
			_id: "$estado",
			cantidad_total: { $sum: 1},
			dinero_acumulado: { $sum: "$coste" },
			precio_mas_caro: { $max: "$coste"},
			lsita_de_arreglos: { $push: "$descripcion"}
		}
	}
])
```