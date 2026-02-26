## Planteamiento de grafo
- Cada `commi -m "Commit inicial: libreria texto"` -> `commit b` 
- El puntero inicial es `(main)`, que se encarga de moverse al estado actual del cambio.
- Crear una nueva rama es crear un nuevo puntero que seguirá un nueva seria de commits.
	- `(feature_libreria_capitulos)` que parte del commit donde lo creamos `b`
	- A partir de los commit seguiremos avanzando.
### ej1
1. Partimos de main, creamos una nuva rama para implementar feature:
```
(mi-nueva-feature)
     /
a -> b
     \
      (main)
```
2. Volvemos a main y creamos una nueva rama para otra feature: 
```
(mi-nueva-feature)
           /
      ...-> c -> d -> e -> f -> g
     /
a -> b -- ... -> d (arreglo-urgente)
     \
      (main)

```
3. Fusión de arreglo `git merge arreglo_urgente` : Ahora main avanzo hasta `d`
```
(mi-nueva-feature)
           /
      ...-> c
     /
a -> b -> d (main)
```
- `main` : apunta siempre al ultimo commit 

`main (b) -> feature-personalizada (c) -> feature-grito (d)`
%%301020258%%
Conclusión:
- Las ramas, tienen como objetivo avanzar en desde un punto del código y para luego ser `merged` al la rama principal.
- Como funciones o avances aislados, el objetivo es acabar en la rama `develop`.

4. Creación de un proyecto
5. Divergencia en ramas
6. Unificación de ramas

### [[Diferencia entre Rebase vs Merge]]

**Clave** : Tanto las Ramas como los rebase, merge todo esta hecho para trabajar en equipo y que nuestro cambios no afecten al equipo. 

**Rebase**: Une dos ramas, a partir del último commit de esta, con los commits de la rama b, es decir que `a->x->x->b->z` se une creando una sola línea. 

**Merge**: Une dos ramas, respetando el historial de commits. 

Ejemplo de uso: 
1. Estoy en rama `feature-token` me salen un bug "bucle infinito y mal filtrado", realizo la rama `fix: filter and bug` y para volverlo a añadir uso el rebase. (Es una rama en la que trabajas solitario que no **afecta a otras personas** además de que no te interesa conservar el historial del bug)
```bash
# Realizar rebase de a -> b
git rebase 
# Ejemplo: Dos commits iguales "commit: api key addd" y "commit: added api key"
# para solucionar esto, unimos los dos commits similares con:
git rebase -i 
```
2. El merge, se usa generalmente. 
### `Squash`
**Ejemplo** :  Es algo natural que a lo largo del desarrollo de una función hemos realizado muchos commits: (1 empecé función, 2 arregle parámetro, 3 ahora si funciona al 85%).
Cuando vayamos a realizar un `merge` no queremos que tenga commits **inútiles**, para eliminar esos 4 commits inútiles. 
```bash
git rebase -i HEAD~1
# Nos llevara a un archivo de configuración donde eligiremos cuales deberan de ser aplastados
pick 5455f35 #  add: misiones.md -> ',' al final de cada oracion
squash c5386eb # add: misiones.md -> '.' al final de cada oracion
```
### Stash

**Ejemplo** : Hemos trabajado en muchas ramas, y mas funcionalidades añadidas. "feature_tokenizer , fix: parametes" pero nos piden una entrega inmediata del código. Este comando aparta temporalmente los cambios realizados para mostrar la rama "funcional, lo que se puede enseñar y que funcione"
```bash
git stash 
git stash pop
```
#### Recuperar rama borrada
```bash
# Creamos la rama de nuevo 
git branch 
# Recuperamos los datos anteriores 
```