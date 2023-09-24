# Practica 2 ASR

## Primera Solución: Creación de la máquina de salto
En esta solución crearemos una máquina de salto, a través de la cual podremos acceder al servidor. Además, limitaremos el acceso a ambas máquinas configurando una serie de reglas de Firewall de nivel 4.
### Pasos a seguir:
1. Creamos la VPC Network "vpcp2" y le asignamos el rango de IP internas 10.0.0.0/27.
   <img width="691" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/924c68a5-c575-4fe4-829f-28902c074ead">

2. Creamos las dos instancias de VM, por un lado, la que es el servidor y por otro la que es la máquina de salto.
Ambas máquinas tienen dirección IP privada dentro de la subnetwork y dirección IP pública.

A continuación se incluyen las imágenes que muestran la configuración de las dos Máquinas virtuales, http-server y jump-server.

----------------------------------------------------------------------------------------------------------------------
#### Máquina http-server:
<img width="641" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/e0c0442c-d921-4d7b-850c-c41d2a74d664">

----------------------------------------------------------------------------------------------------------------------
#### Máquina jump-server:
<img width="602" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/27365389-0a53-4b1e-a17f-6baaafe64d6e">

----------------------------------------------------------------------------------------------------------------------




4. Para esta solución, habilitamos las siguientes reglas de Firewall:
   
   - 1. Se habilita el tráfico ssh desde la dirección IP de nuestro pc hasta el jump-server a través del puerto 22.
   - 2. Se habilita el tráfico interno, únicamente el tráfico TCP desde el jump-server al http-server.


![Reglas de Firewall](https://github.com/202306360/PracticasASR/assets/145692381/abacc4f3-7d9b-47e6-88fc-75d18f3be8ae)


Para permitir el acceso al jump-server desde nuestro pc hemos añadido la clave SSH en el "metadata" de Google Cloud. Una vez hecho esto y creadas las máquinas, ya podemos acceder al jump server desde nuestra consola de comandos. 

<img width="857" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/1379a9bb-950d-4f70-874f-efb4e9fb2d3f">
<img width="857" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/6b30c5c8-1d24-445a-87d2-5f147cb05d4f">













## Segunda solución: Introducción a los WAF - Web Application Firewall (firewall capa 7) - 4 puntos

Para la parte 2, lo primero que hacemos es crear de nuevo las máquinas virtuales, configurando una nueva subnet con los rangos de IP: 10.0.1.0/27.
En este caso, la máquina http-server solo tendrá dirección IP interna, mientras que la jump-server tendrá interna y externa. Para el acceso al servidor, instalaremos un Load Balancer de nivel L7 implementándolo con el servicio WAF y HTTPS offloading.

### 1. CREACIÓN DE LA SUBNET Y MÁQUINAS VIRTUALES
#### 1.1- Configuración de la subnet:
![Configuración de subnet](https://github.com/202306360/PracticasASR/assets/145692381/d2a439b1-07ba-4627-ac3b-9d4ed15d2bb8)

#### 1.2- Configuración de la máquina http-server:
![Máquina server](https://github.com/202306360/PracticasASR/assets/145692381/490c5498-73cd-4d4e-b02b-034613fe89b9)

#### 1.3- Configuración de la máquina jump-server:

![Máquina jump-server](https://github.com/202306360/PracticasASR/assets/145692381/e486cfef-19b0-42b4-9af6-d543366be0dd)

### 2. CONFIGURACIÓN DE LAS REGLAS DE FIREWALL
Para este apartado, mantendremos la misma configuración que en el ejercicio anterior, por lo que al firewall de la nueva subnet creada le aplicamos las siguientes reglas:

#### 2.1-Permitimos únicamente el tráfico SSH desde el jump-server al servidor-http por el puerto 22:
![Reglas de Firewall](https://github.com/202306360/PracticasASR/assets/145692381/f8dd88d1-feea-4a9b-9704-fa49a406da73)

#### 2.2-Permitimos también el tráfico SSH desde mi máquina al jump-server:
![Reglas de Firewall](https://github.com/202306360/PracticasASR/assets/145692381/cd0855f6-3575-4eb3-9c53-8ec57f971471)

Con estas reglas habilitadas, podemos acceder al jump-server desde nuestra IP con el comando SSH para posteriormente acceder también al http-server desde el jump-server.

<img width="545" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/415a5f2a-4f8d-4a94-b36b-f935671b75b9">

Sin embargo, para poder descargar los archivos en el http-server, tenemos que configurar el NAT para tener acceso a internet, ya que no tenemos IP pública en la máquina http-server.


### 3. CONFIGURACIÓN DEL NAT

#### 3.1-Habilitamos el NAT.
![Habilitación del NAT](https://github.com/202306360/PracticasASR/assets/145692381/d41092e4-bbf5-4f2a-b083-f65df419791e)

Una vez tengamos el NAT ya accedemos al Http-server e instalamos el Nginx.
<img width="960" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/4cb5f12d-1f01-4dec-b46c-2681823969e1">



A continuación, procedemos a configurar el Load Balancer.

### 4. CONFIGURACIÓN DEL LOAD BALANCER

#### 4.1-Generación del certificado:
En primer lugar, generamos el certificado a través de los comandos de SSL y lo cargamos en el servidor. El servidor web necesita estos documentos para poder firmar peticiones HTTPS.
![Generación de certificado](https://github.com/202306360/PracticasASR/assets/145692381/7e02c92b-8715-4a15-bbc1-719783e7a46b)
![Generación de certificado](https://github.com/202306360/PracticasASR/assets/145692381/72f7ee7f-5690-4bab-8839-8c6c00c454e2)

#### 4.1-Configuración del balanceador L7:
Una vez tenemos el certificado cargado, procedemos a crear el balanceador de L7. Configuramos el Frontend y el Backend, que se configura con un grupo de instancias que debemos crear previamente.

##### 4.1.1- Creación grupo de instancias:
Nos creamos un grupo de instancias a las que se dirige el balanceador.
<img width="803" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/3bffc9d1-4fc0-4353-ab3d-1ee3dd025a76">

Además, en el backend, debemos crear un Health Check para asegurarnos de que el servidor está operativo. Este Health Check se realiza mediante una conexión TCP en el puerto 80.

![Configuración del balanceador](https://github.com/202306360/PracticasASR/assets/145692381/aa87aabf-b302-4286-994d-95d47708abe2)
![Configuración del balanceador](https://github.com/202306360/PracticasASR/assets/145692381/b2030dc3-9bd2-4d2b-9f3e-6fa0579e993d)

#### 4.2-Health Check del Googel Firewall:
En este punto, también configuramos el Health Check de Google en el Firewall para permitir las comprobaciones desde ciertos rangos de IP externas a través del puerto 80.

![Configuración del Firewall](https://github.com/202306360/PracticasASR/assets/145692381/d6849c95-1d9e-4590-8d54-dc2ea4a9c4e3)

De esta forma, una vez tenemos el balanceador creado, ya podemos acceder al servidor a través de Internet con la dirección IP del balanceador.

![Acceso al servidor](https://github.com/202306360/PracticasASR/assets/145692381/a3a650f3-02bf-4893-b4d1-2e9c078662be)

Si lo hacemos a través del buscador de Google, observamos que debido al tipo de CA que hemos utilizado, nos sale un aviso de seguridad.

![Aviso de seguridad](https://github.com/202306360/PracticasASR/assets/145692381/b90f8717-07bc-4a55-b950-ce3e8155c340)
![Aviso de seguridad](https://github.com/202306360/PracticasASR/assets/145692381/78aeb9cc-dfd4-4a25-92d8-45cb464cc607)

#### 4.3-Firewall rule para permitir los Health Checks de Google:
Habilitamos una regla en el Firewall que permita enviar los Health Checks.
<img width="434" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/e8be0278-98b9-40ec-ab2b-43038a12274f">




#### 4.4-Configuración del WAF:
El objetivo de este apartado es proteger nuestra máquina de ataques SQL Injection, Cross-Site Scripting y restringir el tráfico solo a países de confianza de la UE implantando un WAF a nuestro balanceador.
Ahora procedemos a configurar el WAF a través de las políticas de Cloud Armor:

<img width="818" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/9450f733-b8c6-496d-a2bd-f9ad1fe163e2">

Las reglas instaladas son las siguientes:
<img width="813" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/33a8e4e4-b8bb-43b9-bd0a-da001b457b03">
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
**Preguntas apartado 2:**

¿Qué ventajas e inconvenientes tiene hacer HTTPS offloading en el balanceador?

- Ventajas: Reduce la carga en el servidor al encargarse el balanceador de gestionar las conexiones HTTPS y no el servidor final.
- Inconvenientes: Al deshabilitar el cifrado SSL/TLS en el servidor final, la comunicación entre el balanceador y el servidor no está cifrada, lo que podría hacer que el tráfico interno sea manipulado o interceptado. Se deben tomar medidas adicionales para proteger esta red interna.

¿Qué pasos adicionales has tenido que hacer para que la máquina pueda salir a internet y poder instalar el servidor Nginx?
He tenido que habilitar el NAT como router dentro de la red. La ruta se configura automáticamente.


## Tercera mejora solución: Zero Trust

En esta parte, buscamos que el contenido web esté cifrado también dentro del cloud.
La última parte consiste en hacer trabajar al Nginx por el puerto 443 y que el balanceador vaya también por el HTTPS en lugar del HTTP. Los pasos a seguir son los siguientes:


### 1. Modificar la configuración del instance group.
![Configuración del instance group](https://github.com/202306360/PracticasASR/assets/145692381/d63abee2-7ce1-4ccd-bfe7-3c00452ac980)

### 2. Cambiar la configuración en el Backend del Load Balancer para permitir la comunicación por el P443.

![Configuración del balanceador](https://github.com/202306360/PracticasASR/assets/145692381/c0167e17-724e-4783-8826-ae99f9993726)

### 3. Modificar el Health Check del Load Balancer en el Backend para que se ejecute sobre el puerto 443.

![Health Check del Load Balancer](https://github.com/202306360/PracticasASR/assets/145692381/955ec9d2-1b89-4b5c-9e9f-f5d4ab341efb)

### 4. Modificar la política de Health Check del Firewall para que se ejecute sobre el puerto 443.

![Configuración del Firewall](https://github.com/202306360/PracticasASR/assets/145692381/f5127714-006e-4109-a0e1-4765f412e59f)

### 5. Modificar la configuración de Nginx desde la consola del server para que trabaje sobre el puerto 443.

Para poder cambiar la configuración del Nginx dentro del servidor, debemos mover los archivos .cert y .KEY al http-server. Una vez tengamos estos archivos, debemos modificar la configuración del Nginx habilitando el parámetro ssl e indicando las rutas a los archivos .KEY y .crt.

![Configuración de Nginx](https://github.com/202306360/PracticasASR/assets/145692381/4fbcf2c2-b419-488a-b6a6-90b488283b05)
![Configuración de Nginx](https://github.com/202306360/PracticasASR/assets/145692381/ea765fcb-8928-4603-b308-c23fbd6c4f5e)

Una vez modificada la configuración del Nginx, la actualizamos y la recargamos. Procedemos a conectarnos al localhost y nos sale un aviso de que no se reconoce el CA, por lo que no se puede abrir la conexión de forma segura. Ejecutamos el comando `curl -k https://localhost` para conectarnos de manera insegura, sin comprobar la autoridad del CA.

![Aviso de seguridad](https://github.com/202306360/PracticasASR/assets/145692381/fb2ecb7d-4527-4ba3-a3c7-3b3b4ca57882)

Finalmente, nos conectamos al `https://localhost`.

![Acceso al localhost](https://github.com/202306360/PracticasASR/assets/145692381/a2ca7555-2773-4aec-bcb4-af240b7ff773)

También podemos acceder desde el navegador.

![Acceso desde el navegador](https://github.com/202306360/PracticasASR/assets/145692381/1ddc7cbb-b7a1-4ac7-9e79-53ab34956931)


## Cuarto apartado: Posibles mejoras

