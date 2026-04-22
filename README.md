## Terms & Conditions

_config.yml          ← configura el sitio (pon tu username y repo)                                                                                                                          
Gemfile              ← para desarrollo local                                                                                                                                                
_layouts/default.html ← layout con selector EN | ES | PT                                                                                                                                    
assets/css/style.css  ← diseño limpio tipo legal

index.md                          ← home con lista de apps                                                                                                                                  
apps/myapp/index.md               ← tabla de links por idioma                                                                                                                               
apps/myapp/terms/{en,es,pt}.md    ← T&C en 3 idiomas                                                                                                                                        
apps/myapp/privacy/{en,es,pt}.md  ← Privacy Policy en 3 idiomas
apps/myapp/{terms,privacy}/index.md ← redirects automáticos

Para subir a GitHub Pages:
1. Crear repo en GitHub, hacer push de esta carpeta a main
2. Settings → Pages → Source: main / / (root)
3. Editar _config.yml: poner tu [YOUR-USERNAME] y si es proyecto (no usuario) añadir baseurl: "/nombre-del-repo"

Antes de publicar en la App Store, busca y reemplaza todos los [PLACEHOLDER]:
- [APP NAME], [DEVELOPER 1], [DEVELOPER 2], [CONTACT EMAIL], [DATE], [CITY]

Para añadir otra app: copia apps/myapp/ → apps/nueva-app/, cambia el front matter y añade la entrada en index.md.

La limitación de responsabilidad está fijada al precio de compra de la app — es el estándar para apps de pago indie y supera el review de App Store.   