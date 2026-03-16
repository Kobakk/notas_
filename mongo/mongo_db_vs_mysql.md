Depende de lo que necesitemos:
Mysql: 
- Estructura estricta y predecible: Elegir si los datos siempre van a tener la misma forma y encajar perfectamente en las tablas, filas, columnas. Un flujo de datos ordenado
- Buena integridad, donde las transacciones matemáticas deben ser infalibles y los datos huérfanos o mal puestos son inaceptables. 
- Relaciones complejas: Si la base de la aplicación consiste en cruzar muchas tablas constantemente, consultas con multiples joins (coches con reparaciones en x ciudad )

No relacional: 
- flexibilidad total: justificando si los datos cambian constantemente de forma o si cada registro es distinto ( coche, motos, scoters, )
- Si se prevee un crecimiento de datos bastante elevado, mongodb esta hecho para escalar verticalmente. 