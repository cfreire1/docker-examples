# Sitio Oficial:
* Link: https://registry.hub.docker.com/r/gitlab/gitlab-ce/

# Descripcion:
* Git lab runner 
* (Material para estudio)

# Instrucciones 
# 1. Iniciar compose (GIT-LAB-RUNNER)
1. Asegurarse de estar corriendo el contenedor que pertenece a gitlab sitio web
2. Iniciar contenedor runner 
```
    docker-compose up -d 
```
3. Verificar contenedores corriendo
```
    docker ps

    CONTAINER ID   IMAGE                         COMMAND                  CREATED         STATUS                            PORTS   NAMES
    abbcb0ad083e   gitlab/gitlab-runner:latest   "/usr/bin/dumb-init …"   4 seconds ago   Up 3 seconds                              gitlab-runner
    58781119d54d   gitlab/gitlab-ce:latest       "/assets/wrapper"        5 seconds ago   Up 3 seconds (health: starting)           gitlab

```
# 2. Network
* Crear network
```
    docker network create --attachable network-gitlab
```

* Agregar contenedor a network
```
    docker network connect network-gitlab gitlab
    docker network connect network-gitlab gitlab-runner
```

# 3. Registrar (GIT-RUNNER)
1. Una vez que se encuentren corriendo los 2 contenedores, el gitlab-runner y el gitlab
    * Tendremos el siguiente error en el log del contenedor del gitlab-runner, lo cual es lo esperable
        * Y el error se debe a que aún no hemos configurado ni registrado nuestro corredor en ningún servidor de Gitlab.
```
    // error esperable
    gitlab-runner  | ERROR: Failed to load config stat /etc/gitlab-runner/config.toml: no such file or directory  builds=0
```
2. Ingresamos al runner por consola interactiva al contenedor
```
    docker exec -it gitlab-runner bash
```
3. Dentro del contenedor gitlab-runner registramos nuestro corredor 
    * Utilizaremos la URL: `http://gitlab:3201/`
    * Token: (Se obtiene desde gitlab-web)
        * Ir a sitio web `Menu/Admin/Overview/Runners`
        * Hacer click en boton azul 'Register an instance runner' y copiar token
```
    //registrar elegir solo una
        [op1] gitlab-runner register
        [op2] gitlab-runner register --docker-network-mode 'host' --docker-volumes /var/run/docker.sock:/var/run/docker.sock

    //Ejemplo de registro de runner de opciones (op1 o op2)
        Enter the GitLab instance URL (for example, https://gitlab.com/):
            http://gitlab-web/  
        Enter the registration token:
            8JLD4WogjkDoonPshxDC
        Enter a description for the runner:
            [60bf953184e1]: runnershared
        Enter tags for the runner (comma-separated):
            runner-shared
        Enter optional maintenance note for the runner:

        Registering runner... succeeded                     runner=8JLD4Wog
        Enter an executor: shell, ssh, docker+machine, docker-ssh+machine, parallels, docker, docker-ssh, virtualbox, kubernetes, custom:
            docker
        Enter the default Docker image (for example, ruby:2.7):
            docker:19.03.12
        Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 

```
* Tras ejecutar [runner register:op1]
    * configurar archivo `config.toml` y agregar o modificar la siguiente configuracion:
    ...
    ```
        [runners.docker]
        ...
        volumes = ["/cache","/var/run/docker.sock:/var/run/docker.sock"]
        ...
        network_mode = "host"

    ```
* Tras ejecutar [runner register:op2] 
    * Seguir a la siguiente seccion
# 4. OP1 - Configurar runner shared para repositorio
1. En `Menu/Admin/Overview/Runners` seleccionar a editar runner creado 
    * Marcamos o activamos la opcion `Run untagged jobs` y guardamos cambios

2. Ir a repositorio y revisamos
    * `<repo_selecionado>/setting/CICD/RUNNERS`
        * Activar `Enable shared runners for this project`

3. Creacion de `.gitlab-ci.yml` y ejecucion de Pipelines
    * Ir `<repo_selecionado>/CICD/Editor`
        * Seleccionamos `Configure Pipelines`
        * Agregamos nuestro Pipelines y hacemos commit al repositorio.
        * (Comenzara automaticamente la ejecucion de nuestro primer Pipelines)

#  4. OP2 - Configurar runner para repositorio
1. Ir a repositorio y revisamos
    * `<repo_selecionado>/setting/CICD/RUNNERS`
        * Desactivamos `Enable shared runners for this project`
2. Revisar seccion `Specific runners`
    * Registraremos otro runner con el token que nos genera para ese repositorio
    * Tras agregarlo actualizamos la pagina (F5) y nos aparecera con un icono de 'alerta'
    * Tras voler a actualizar la pagina (F5) y nos aparecera en color verda
3. Habilitar que el runner pueda ejecutar Pipelines
    * Editamos el runner creado 
        * En `Menu/Admin/Overview/Runners` seleccionar a editar runner creado 
            * Marcamos o activamos la opcion `Run untagged jobs` y guardamos cambios
4. Creacion de `.gitlab-ci.yml` y ejecucion de Pipelines
    * Ir `<repo_selecionado>/CICD/Editor`
        * Seleccionamos `Configure Pipelines`
        * Agregamos nuestro Pipelines y hacemos commit al repositorio.
        * (Comenzara automaticamente la ejecucion de nuestro primer Pipelines)

# 5. Revisar ejecucion Pipelines
4. Revisar ejecucion Pipelines
    * Ir `<repo_selecionado>/Pipelines`
        * Pipelines 

            ![exec-pipelines.png](images/exec-pipelines.png)

        * Ejecucion

            ![exec-pipelines-ok.png](images/exec-pipelines-ok.png)

# Notas

* https://bwgjoseph.com/how-to-setup-and-configure-your-own-gitlab-runner
* https://stackoverflow.com/questions/64675170/gitlab-ci-how-to-create-the-shared-runner-in-gitlab-which-does-not-depend-on
* https://stackoverflow.com/questions/33430487/how-to-use-gitlab-ci-to-build-a-java-maven-project

* Problem runner: Couldn't resolve host
    * https://gitlab.com/gitlab-org/gitlab-runner/-/issues/305
    * https://stackoverflow.com/questions/50325932/gitlab-runner-docker-could-not-resolve-host

* Problem runner Operation timed out
    * https://askto.pro/question/how-to-fix-failed-to-connect-to-gitlab-web-port-80-in-gitlab

* custom-network
    * https://twig2let.github.io/docker/docker_networkBetweenMultipleDockerComposeServices.html
    * https://stackoverflow.com/questions/52757357/create-custom-network-for-docker-compose-via-command-line
    * config.toml: https://stackoverflow.com/questions/38737862/gitlab-ci-add-net-host-option-to-docker
    * https://medium.com/devops-with-valentine/how-to-build-a-docker-image-and-push-it-to-the-gitlab-container-registry-from-a-gitlab-ci-pipeline-acac0d1f26df

* yml:
    * https://forge.etsi.org/rep/help/ci/docker/using_docker_build.md
	* https://gitlab.com/gitlab-org/gitlab-foss/-/issues/17769
	* https://forum.gitlab.com/t/example-gitlab-runner-docker-compose-configuration/67344/1
    * https://www.adictosaltrabajo.com/2018/04/10/primeros-pasos-con-gitlab-ci/

    * VARIABLES:
	    * https://docs.gitlab.com/ee/ci/variables/predefined_variables.html
        * http://mpegx.int-evry.fr/software/help/ci/environments/index.md
        * https://stackoverflow.com/questions/66673631/how-to-trigger-a-gitlab-pipeline-for-a-tagged-commit-on-branch-master
        * https://docs.gitlab.com/ee/ci/yaml/#tags
        * https://docs.gitlab.com/ee/ci/pipelines/

    * grep
	    * https://ubunlog.com/comando-grep/?msclkid=6ee599ecc80711ec922549d3ff968769
	    * https://yaroslavgrebnov.com/blog/bash-docker-check-container-existence-and-status/
