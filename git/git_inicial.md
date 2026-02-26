#29/10/25-8

###### Mostrar información:
```bash
#iniciar git
git init
# creara la carpeta oculta ".git" donde contiene el codigo local 
rm -rf .git # para eliminar git de nuestra carpeta 
#historial de commits
git log --oneline 
git log --pretty=format"s" 
#ver detalle commit
git show "id_commit" 
```

###### Configuración:
- `/.gitconfig` : archivo de configuración
	- windows ruta : `C:\Users\user\.gitconfig`
	- Orden de prioridad (Local > Global > Sistema)
		- Local: `.git/config` -> carpeta proyecto -> aplica a repo específico
			- `.gitignore`
		- Global: `/.gitconfig` -> carpeta usuario -> aplica a todos los **proyectos**
		- Sistema: `/etc/gitconfig` -> carpeta sistema -> aplica a todos los **usuarios**
	-  Para ver configuración:`git config --global --list`
	- Para editar: `git config --global --edit`
Usar dos cuentas: En el archivo de configuración local 
```bash
git config --local user.name "nombre_personal"
git config --local user.email "tu@email-personal.com"
```
Diferencia entre:
- Autenticación, de eso se encargara app de github. 
- Firma: De eso tendremos que configurar el usuario local con `config --local user.`
###### atajos
```bash
[alias]
    # Atajo para 'git status'
    st = status
    # Atajo para 'git checkout'
    co = checkout
    # Atajo para 'git branch'
    br = branch
    # Atajo para 'git commit'
    ci = commit
    # Un 'git log' mucho más bonito y legible
    lg = log --oneline --graph --decorate --all
```

###### [[Trabajo con ramas]]

- `git checkout nombre_rama` : moverse a rama
- `git branch -m main` : renombrar rama 
Cambios `staged` :  Con `git add` o sin.

###### Buenas prácticas:
	- Commit efectivos: `git commit -m "feat | fix |docs"`
	- Etiquetas para marcar hitos `git tag -d v1.0.0`
	- Commits átomicos
###### conexión con github
- Git -> Github: 
```bash
# 'origin' es el apodo estándar para tu repositorio remoto
git remote add origin https://github.com/TuUsuario/proyecto-extraccion-pdf.git
```
%%31102513%%
- Resultado: https://github.com/Jsbasv/proceso_etl
###### Trabajo con commits antiguos - merge
%%0111259%%

- Unir `commits`
La opción mas suave.
Actualizar commit, o corregir commit. 
```bash
git reset --soft HEAD^

# Opcion mas fuerte
git rebase -i
# Volver a la rama anterior 
git refllog # vemos todo el historial completo de commits
git reset --hard keyLog # example: 9e4c870
# renombrar archivo
git mv nombre nuevonombre
# Para eliminar archivo
git rm nombre
git commit -m "Eliminado: nombre"
```
Añadir - Eliminar Tags / Versiones:
```bash
git tag nombre

git tag

git tag -d nombre
# subir tag 
git push --tags 
```

Explicación de versiones 1.0.5
1 : Cambio mayor
0: Funcionalidad añadida
5: Cambio de Bug 