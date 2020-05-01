# DIFERENCIAS ENTRTE  `git merge` Y ` git merge --no-ff` #
El indicador --no-ff evita que git merge ejecute un "avance rápido" si detecta que su HEAD actual es un antecesor de la confirmación que está intentando fusionar. Un avance rápido es cuando, en lugar de construir una confirmación de fusión, git simplemente mueve el puntero de su rama para que apunte a la confirmación entrante. Esto ocurre comúnmente cuando se hace un git pull sin ningún cambio local.

Sin embargo, ocasionalmente desea evitar que ocurra este comportamiento, generalmente porque desea mantener una topología de rama específica (por ejemplo, se está fusionando en una rama de tema y quiere asegurarse de que se vea de esa manera cuando lea el historial). Para hacer eso, puede pasar el indicador --no-ff y git merge will siempre construirá una combinación en lugar de un envío rápido.

De manera similar, si desea ejecutar un git pull o usar git merge para avanzar explícitamente y desea rescatar si no puede avanzar, entonces puede usar el indicador --ff-only. De esta manera, puede hacer algo como git pull --ff-only sin pensar, y luego, si se produce un error, puede volver atrás y decidir si desea fusionar o reajustar.

Ejemplo de flujo de trabajo
Se actualiso un paquete en un sitio web y  se tuvo que volver a las notas para ver el flujo de trabajo. 
Mi flujo de trabajo de comandos git:

' git checkout -b contact-form
(do your work on "contact-form")
git status
git commit -am  "updated form in contact module"
git checkout master
git merge --no-ff contact-form
git branch -d contact-form
git Push Origin master
A continuación: uso real, incluidas las explicaciones. '
Nota: la salida a continuación se recorta; git es bastante verboso. 
~~~
$ git status
# On branch master
# Changed but not updated:
#   (use "git add/rm <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#       modified:   ecc/Desktop.php
#       modified:   ecc/Mobile.php
#       deleted:    ecc/ecc-config.php
#       modified:   ecc/readme.txt
#       modified:   ecc/test.php
#       deleted:    passthru-adapter.igs
#       deleted:    shop/mickey/index.php
#
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#       ecc/upgrade.php
#       ecc/webgility-config.php
#       ecc/webgility-config.php.bak
#       ecc/webgility-magento.php 
~~~

Note 3 cosas desde arriba:
1) En la salida puede ver los cambios de la actualización del paquete ECC, incluida la adición de nuevos archivos.
2) También note que hay dos archivos (no en la carpeta /ecc) que eliminé independientemente de este cambio. En lugar de confundir esas eliminaciones de archivos con ecc, haré una ramificación cleanup diferente más adelante para reflejar la eliminación de esos archivos.
3) ¡No seguí mi flujo de trabajo! Me olvidé de git mientras intentaba que ECC volviera a funcionar.

A continuación: en lugar de hacer el git commit -am "updated ecc package" completo, normalmente lo haría, solo quería agregar los archivos en la carpeta /ecc. Esos archivos eliminados no eran específicamente parte de mi git add, pero debido a que ya fueron rastreados en git, necesito eliminarlos de la confirmación de esta rama:

~~~
$ git checkout -b ecc
$ git add ecc/*
$ git reset HEAD passthru-adapter.igs
$ git reset HEAD shop/mickey/index.php
Unstaged changes after reset:
M       passthru-adapter.igs
M       shop/mickey/index.php

$ git commit -m "Webgility ecc desktop connector files; integrates with Quickbooks"

$ git checkout master
D       passthru-adapter.igs
D       shop/mickey/index.php
Switched to branch 'master'
$ git merge --no-ff ecc
$ git branch -d ecc
Deleted branch ecc (was 98269a2).
$ git Push Origin master
Counting objects: 22, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (14/14), done.
Writing objects: 100% (14/14), 59.00 KiB, done.
Total 14 (delta 10), reused 0 (delta 0)
To git@github.com:me/mywebsite.git
   8a0d9ec..333eff5  master -> master
~~~
 La opción `--no-ff`  asegura que no se producirá una combinación de avance rápido, y que siempre se creará un nuevo objeto de confirmación . Esto puede ser deseable si quieres que git mantenga un historial de ramas de características.

![ejemplo.img](https://i.stack.imgur.com/FMD5h.png)

En la imagen de arriba, el lado izquierdo es un ejemplo del historial de git después de usar git merge --no-ff y el lado derecho es un ejemplo de usar git merge donde fue posible una fusión ff.
