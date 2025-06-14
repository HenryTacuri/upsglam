# **Proyecto intercilo - UPSGlam 2.0**

**Integrantes:**
* Henry Tacuri
* Franklin Guapisaca
* Juan Quizhpi

# **Arquitectura de la aplicación**

<p align="center">
  <img src="imagenes_uso_app/cp_arq.jpg" width="800"/>
</p>

La arquitectura de la aplicación UPSGlam 2.0, representada en el diagrama, se basa en una estructura de microservicios distribuidos que se comunican entre sí mediante el protocolo HTTP. Este enfoque favorece la escalabilidad, la modularidad y la facilidad de mantenimiento del sistema. La solución se organiza en cuatro componentes principales: frontend móvil, backend reactivo, procesamiento de imágenes con GPU y servicios en la nube.

El frontend, desarrollado con Flutter, permite a los usuarios registrarse, iniciar sesión, subir imágenes, aplicar filtros y visualizar publicaciones en tiempo real. Este cliente móvil se comunica con un backend reactivo implementado con Spring WebFlux, estructurado en servicios que gestionan la autenticación, los usuarios, las fotos y el procesamiento de imágenes. Todo el backend realizado con spring boot está dockerizado.

El procesamiento de imágenes, está desarrollado con PyCUDA, incluye filtros personalizados como: Filtro Log, Filtro Media, Filtro Gaussiano, Filtro Cartoon, Filtro Sketch, Filtro Térmico y FiltroAsciiUps. Estos filtros se ejecutan de forma paralela sobre la GPU, optimizando el rendimiento y mejorando la experiencia del usuario. Por otra parte, todo este procesamiento de imágenes está dockerizado.

Con los contenedores backend y pycuda, se realiza la orquestación de contenedores para que puedan comunicarse entre sí. Finalmente, todos estos contenedores son levantados para que la aplicación móvil pueda consumir los respectivos servicios. La aplicación utiliza servicios en la nube de Firebase como Authentication, Firestore y Storage para gestionar la autenticación de usuarios, el almacenamiento de datos y la gestión de publicaciones.



# **Despliegue de la aplicación**

**1.** Para levantar los contenedores (Backend Spring Boot y PyCuda) se deberá crear el archivo ```docker-compose.yml```, el cual contendra el siguiente contenido:

```docker
version: '3.8'  # Versión de Docker Compose compatible con Docker Engine 19.03+

services:
  python_app:
    image: henrytacuri/upsglam_pycuda:latest  # Imagen que incluye la app Python con soporte CUDA
    container_name: python_app               # Nombre personalizado para el contenedor
    ports:
      - "5000:5000"                           # Mapea el puerto 5000 del host al 5000 del contenedor
    restart: unless-stopped                   # Reinicia el contenedor a menos que se detenga manualmente
    runtime: nvidia                           # Ejecuta con runtime NVIDIA para acceso a GPUs
    environment:
      - NVIDIA_VISIBLE_DEVICES=all            # Expone todas las GPUs disponibles al contenedor

  backend:
    image: henrytacuri/upsglam_backend:latest # Imagen del servicio backend
    container_name: upsglam_backend           # Nombre personalizado para el contenedor del backend
    ports:
      - "8080:8080"                           # Mapea el puerto 8080 del host al 8080 del contenedor
    restart: unless-stopped                   # Reinicia automáticamente salvo que se detenga manualmente
    depends_on:
      - python_app                            # Se inicia después de que python_app haya arrancado

```

Todos los Dockerfiles con los cuales se construyeron las imagenes de docker (ahora subidas a Docker Hub) están en las carpetas de los respectivos proyectos de Spring Boot y PyCuda.

**2.** Luego abrimos una terminal dentro del mismo directorio donde se creo el archivo ```docker-compose.yml``` y ejecutamos el comando ```docker compose up```.

**3.** En la terminal se vizualizará que los contenedores están ejecutandose correctamente.

**4.** Luego en el dispositivo android instalamos la aplicación móvil por medio del apk el cual tiene el nombre ```app-release.apk```.


# **Uso de la aplicación**

**1.** Creación de cuenta

Si no tenemos una cuenta creada, hacemos clic en el botón 'Regístrate' y completamos los campos.

<p align="center">
  <img src="imagenes_uso_app/Inicio.jpg" width="300"/>
  <img src="imagenes_uso_app/creacionCuenta.jpg" width="300"/>
  <img src="imagenes_uso_app/usuarioCreado.jpg" width="300"/>
</p>

**2.** Login

Iniciamos sesión con nuestra cuenta creada.

<p align="center">
  <img src="imagenes_uso_app\inicirSesion.jpg" width="300"/>
</p>

**3.** Procesamiento y publicación de fotos 

Para publicar una foto, vamos al apartado 'Subir fotos', seleccionamos una imagen, elegimos un filtro y hacemos clic en 'Aplicar filtro'. Finalmente, damos clic en 'Publicar foto'.

<p align="center">
  <img src="imagenes_uso_app\publicarFoto.jpg" width="300"/>
</p>

**4.** Galería de fotos

Dentro del apartado de galería de fotos, podemos encontrar únicamente nuestras publicaciones, donde podemos dar 'me gusta' y dejar comentarios.

<p align="center">
  <img src="imagenes_uso_app\verP.jpg" width="300"/>
  <img src="imagenes_uso_app\darMeGustaP.jpg" width="300"/>
  <img src="imagenes_uso_app\comentarP.jpg" width="300"/>
</p>

**5.** Feed de publicaciones

En el feed de publicaciones se muestran tanto nuestras publicaciones como las de otros usuarios, y podemos interactuar con ellas dando 'me gusta' y dejando comentarios.

<p align="center">
  <img src="imagenes_uso_app\feedP.jpg" width="300"/>
  <img src="imagenes_uso_app\comentarFeed.jpg" width="300"/>
</p>

**7.** Datos del usuario

En la sección de perfil se muestra toda nuestra información personal de nuestra cuenta.

<p align="center">
  <img src="imagenes_uso_app\perfil.jpg" width="300"/>
</p>


# **Reflexión sobre el uso de tecnologías paralelas, reactivas y en la nube en UPSGlam 2.0**

**Computación Paralela con CUDA**


La aplicación de PyCUDA para el procesamiento de imágenes resultó fundamental para alcanzar un alto rendimiento en la aplicación de filtros personalizados. Diseñamos e implementamos filtros tanto clásicos como creativos, incluyendo uno con elementos representativos de la UPS. La optimización y procesamiento paralelo tiene grandes ventajas para trabajar con grandes volúmenes de datos, ya que permite dividir y distribuir tareas en un entorno de procesamiento masivo como lo es una GPU.



**Programación Reactiva con WebFlux**

En el backend, utilizamos Spring WebFlux para construir una arquitectura reactiva basada en microservicios. Este enfoque nos permitió manejar múltiples peticiones concurrentes de forma eficiente, manteniendo una comunicación fluida entre los distintos servicios y facilitando la integración con Firebase. La naturaleza no bloqueante de WebFlux fue clave para mantener un flujo continuo de datos y para soportar escenarios de alta demanda. Aunque las interacciones cómo likes y comentarios se gestionaron desde el frontend, el backend brindó soporte estable para la persistencia y consulta en tiempo real a través de Firebase.


**Servicios en la Nube con Firebase**


Firebase fue una herramienta central en la estructura de la aplicación. Nos permitió implementar autenticación segura, almacenamiento de publicaciones y carga eficiente de imágenes procesadas. Gracias a Firebase Hosting y Storage, expusimos contenidos a través de URLs públicas, lo cual facilitó su integración con la aplicación móvil. Además, el uso de Firestore como base de datos NoSQL nos proporcionó flexibilidad y escalabilidad.


----------

**Link del informe:** https://drive.google.com/file/d/1_FsCDYix6oPOUuTGqLER_BTCgvzAQk38/view?usp=sharing

**Link de las presentaciones:** https://drive.google.com/file/d/1zXSEoE7oGkxbMT_I0GoPCaqOKRXAFDAN/view?usp=sharing
