1. Abrimos archivos de configuración con `git config --global --edit`
2. Agregamos los alias  y que la rama por defecto deba ser "main" y no "master".
```bash
[init]
  defaultBranch = main
[alias]
  st = status
  co = checkout
  br = branch
  ci = commit
  unstage = reset HEAD --
  pm = push origin main
  lg = log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)' --all
```
3. Configuración local si estamos en otro dispositivo
```bash
git config --local user.name "nombre_personal"
git config --local user.email "tu@email-personal.com"
```
FIN