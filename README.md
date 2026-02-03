p02
Descargo galería, la descomprimo, me meto en la carpeta y abro vscode en ella

En el terminal VSCode:
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

Ejecutar la app de Flask:
python src/app.py
o
gunicorn --chdir src (carpeta contenedora) app(nombre del modulo/fichero):app(Nombre de la variable) --bind 0.0.0.0:5000

Crear la imagen:
docker build - t hectordl/test .

Ejecutar la imagen:
docker run -d -p 5000:5000 --name test test
firefox --private-window localhost:5000

Para el servicio:
docker rm -f $(docker ps -aq)
docker ps -a

Compose:
docker compose up -d
docker compose down --volumes

Docker Push:
docker push hectordl/test

Secretos:
En dockerhub en Ajustes de la cuenta -> Settings -> Personal access tokens -> Generate new token -> Le damos una descripción, no le ponemos fecha limite y le damos permiso para: Read, Write, Delete 


Creamos un repositorio en GitHub

En la carpeta de galeria:
git init
git status
git branch -M main
git status
git add .
git status
git commit -m "inicial"
git remote add origin https://github.com/hdomlop728/devops-p02.git (repositorio)
git push -u origin main

En el repositorio de GitHub-> Settings -> Secrets and variables -> Actions -> New repository secret:
Name: DOCKERHUB_USERNAME
Secret: hectordl

De nuevo le damos a New repository secret:
Name: DOCKERHUB_TOKEN
Secret: El token 

En el repositorio -> Actions -> Get started with GitHub Actions -> set up a workflow yourself -> le ponemos el nombre que queramos y ponemos esto:
name: Docker CI/CD

on:
  push:
    branches: [ main ]
    paths:
      - src/**
  pull_request:
    branches: [ main ]
    paths:
      - src/**

jobs:
  build:
    name: build and push Docker image
    runs-on: ubuntu-latest
    steps: 
      # 1 
      - uses: actions/checkout@v4
      # 2  
      - name: Login to DockerHub
        uses: docker/login-action@v3 
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      # 3   
      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true 
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/galeria:latest
Commit changes ... y lo ponemos para la rama main

En local:
git pull
y modifico algo en src de la carpeta galeria (ej: dogs -> cats en app.py) 
git status
git add .
git commit "update to actions"
git push
Y en GitHub actions se debería hacer automaticamente push a Docker Hub
(Hice: docker rmi hectordl/test)
docker run -d -p 5000:5000 --name test hectordl/galeria
firefox --private-window localhost:5000


Despliegue en AWS
Debo tener una instancia EC2 con docker instalado y activa (la haces con la plantilla)

En el repositorio de GitHub -> Secrets and variables -> Actions -> New repository secret
Name: AWS_USERNAME
Secret: admin (por que estamos en debian casi al 100% será admin)

De nuevo le damos a New repository secret:
Name: AWS_HOSTNAME
Secret: IP de la máquina (lo suyo es hacer la IP elastica)

Y de nuevo le damos a New repository secret:
Name: AWS_PRIVATEKEY
Secret: El contenido del vockey.pem (Incluido la primera y última línea)

En el repositorio de GitHub -> Actions -> Buscamos Docker CI/CD en la barra lateral izquierda y le damos -> le damos al archivo .yml (devops2.yml) -> se nos abrirá el .yml, le daremos a editar y añadiremos el siguiente job:
# ... Cortado
jobs:
  build:
    # ... Cortado

  # código añadido
  aws:
    name: Deploy image to aws
    needs: build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    # 1
    - name: copy docker compose via ssh key  
      uses: appleboy/scp-action@v0.1.7
      with:
        host: ${{ secrets.AWS_HOSTNAME }}
        username: ${{ secrets.AWS_USERNAME }}
        port: 22 # ${{ secrets.PORT }}
        key: ${{ secrets.AWS_PRIVATEKEY }}
        source: "docker-compose.yml"
        target: /home/admin
    # 2
    - name: script deploy docker services 
      uses: appleboy/ssh-action@v1.0.3
      with:
        host: ${{ secrets.AWS_HOSTNAME }}
        username: ${{ secrets.AWS_USERNAME }}
        key: ${{ secrets.AWS_PRIVATEKEY }}
        port: 22  # ${{ secrets.PORT }}
        script: |
            sleep 60
            docker compose down --rmi all
            docker compose up -d


Con lo anterior hecho modificaremos algo en la carpeta src de nuestro repositorio y la subiremos a este.
Si lo hemos hecho bien, cuando termine el job podremos acceder a app con la ip_de_la_instancia_EC2:80 
-----


-----
Direcciones IP elásticas
Tenemos que tener creada una instancia EC2
En AWS -> EC2 -> Panel -> Recursos -> Direcciones IP elásticas
Allí le damos a Asignar Dirección IP Elástica -> No tocamos nada y le damos a Asignar
Cuando le demos nos llevará a la pantalla anterior y le daremos a Asociar esta dirección IP elástica
Cuando le demos, en Instancia le daremos a -> Instancia y escribiremos la instancia que queremos que tenga la IP elástica. Después le daremos a Asociados

Desasociar IPs elásticas
En AWS -> EC2 -> Panel -> Recursos -> Direcciones IP elásticas
Allí le damos la IP -> Acciones -> Desasociar dirección IP elástica
Una vez le demos le daremos a Desasociar

Liberar IPs elásticas (Eliminarla)
En AWS -> EC2 -> Panel -> Recursos -> Direcciones IP elásticas
Allí le damos la IP -> Acciones -> Liberar direcciones IP elásticas
Le damos a Publicar
-----


3. Segunda parte: Actions
3.1. Introducción
Las Actions de GitHub son una herramienta de integración continua (CI) y despliegue continuo (CD) proporcionada por GitHub. Te permiten automatizar flujos de trabajo relacionados con tu código, como la compilación, pruebas, despliegues y otras tareas personalizadas.

Principales características:

Automatización de flujos de trabajo: Puedes crear archivos de configuración (generalmente en formato YAML) que definen una secuencia de pasos a ejecutar cada vez que ocurre un evento, como un push, pull request o publicación de una nueva versión (release).
Ejecución en entornos personalizados: Las acciones pueden ejecutarse en entornos como Linux, Windows o macOS, utilizando runners proporcionados por GitHub o personalizados.
Acceso a un ecosistema amplio: GitHub Actions tiene un mercado (GitHub Marketplace) con miles de acciones listas para usar, como herramientas de compilación, despliegues a la nube y gestión de dependencias.
Alta flexibilidad: Puedes configurar desde tareas simples (por ejemplo, ejecutar tests) hasta flujos complejos que integren múltiples servicios.
Ventajas:

Integrado en GitHub: No necesitas herramientas externas para automatizar.
Escalabilidad: Puedes usar tanto runners gratuitos como configuraciones avanzadas.
Colaborativo: Perfecto para equipos distribuidos que trabajan en proyectos abiertos o cerrados.
Como ejemplo se añadirá un Action. Su objetivo es que cuando se haga un PR a la rama principal se haga el proceso de linting de los ficheros nuevos o modificados en esa rama. El Action dará como resultado si el proceso de linting es correcto o no. En este ejemplo se usarán ficheros Python y como linter las herramientas pylint o flake8.
