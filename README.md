# PracticasASR
1a Solución: creación de máquina de salto - 4 puntos
Pasos a seguir:
Creamos la VPC Network "vpcp2" y le asignamos el rango de IP internas 10.0.0.0/27
Creamos las dos instancias de VM, por un lado la que es el servidor y por otro la que es la máquina de salto.

  Máquina http-server:
  
<img width="700" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/0d94df5a-6d55-44b9-9499-befe8dd5a9da">


  Máquina jump-server:
  
  <img width="728" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/e91a1769-abf5-48a4-a626-9dfa0effb863">


Ambas máquinas tienen dirección IP privada dentro de la subnetwork y la máquina de salto es la única que tiene dirección IP pública.
Para esta solución, habilitamos las siguientes reglas de Firewall:
1) Se habilita el tráfico ssh desde la dirección IP de nuestro pc hasta el jump-server a través del puerto 22.
2) Se habilita el tráfico interno entre las máquinas, esto para comprobar que existía conectividad entre ambas y poder hacer un ping.
3) Se deshabilita la regla anterior y se habilita únicamente el tráfico TCP desde el jump-server al http-server.
4) 
   <img width="790" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/abacc4f3-7d9b-47e6-88fc-75d18f3be8ae">

   
 <img width="757" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/e237223b-8d8c-4306-b14f-7cd310da1a62">

Para permitir el acceso al jump-server desde nuestro pc, añadimos la clave -ssh en el "metadata" de google cloud.
Una vez hecho esto y creadas las máquinas, ya podemos acceder al jump server desde nuestra consola de comandos.
Una vez estamos dentro del jump server, podemos hacer un -ping a la dirección IP interna de la máquina http-server y comprobar que efectivamente podemos llegar a ella a través del bastión.


![image](https://github.com/202306360/PracticasASR/assets/145692381/16cb4bea-d6a2-46d8-90f5-79089eb8d898)




-2da mejora solución: introducción a los WAF - Web Application Firewall (firewall capa 7) - 4 puntos

Para la parte 2, lo primero que hacemos es crear de nuevo las máquinas virtuales, para ello he configurado una nueva subnet con los rangos de IP: 10.0.1.0/27:

<img width="802" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/d2a439b1-07ba-4627-ac3b-9d4ed15d2bb8">

La máquina server: únicamente con dirección IP interna
<img width="782" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/490c5498-73cd-4d4e-b02b-034613fe89b9">


La máquina jump-server: con IP interna y externa
<img width="781" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/e486cfef-19b0-42b4-9af6-d543366be0dd">


El firewall: En principio con la misma configuración que en el ejercicio anterior pero esta vez aplicamos las siguientes reglas:

1)Permitimos únicamente el tráfico ssh desde el jump-server al servidor-http por el P22.
<img width="814" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/f8dd88d1-feea-4a9b-9704-fa49a406da73">


2)Permitimos también el tráfico ssh desde mi maquina al jump-server.
<img width="806" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/cd0855f6-3575-4eb3-9c53-8ec57f971471">


Con esto podemos acceder al jump-server desde nuestra Ip con el comando ssh IP -i y la clave y un avez dentro del jump-server, podemos acceder de la misma manera al hht-server con su IP interna.

![image](https://github.com/202306360/PracticasASR/assets/145692381/0f896475-3cd7-4200-a100-5fcdb8b93f2b)

Sin embargo, para poder descargar los archivos en el server, tenemos que crear el NAT, para poder tener acceso a internet (no tenemos IP pública en el server).
NAT
Habilitamos el hhtp bastion.

<img width="710" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/d41092e4-bbf5-4f2a-b083-f65df419791e">


Una vez tengamos el NAT, y habilitada la regla de acceso. Accedemos desde el jump-server al http-server.
Después de esto, ya accedo a el Http-server e instalo el nginx
![image](https://github.com/202306360/PracticasASR/assets/145692381/c59d8692-fd98-4bba-8363-159c80f16d3b)


A continuación, procedemos a instalar el Load Balancer
LOAD BALANCER

PREGUNTAS:
¿Qué ventajas e incovenientes tiene hacer https offloading en el balanceador?
El https offloading es una técnica que permite que el balanceador de carga sea el que se encarga de gestionar las conexiones HTTPS en lugar de que sea el server final el que lo haga.
Como ventaja: se reduce la carga en el servidor porque ya no tienen que hacer el cifrado SSL/TLS
Como desventaja: Al deshabilitar el cifrado SSL/TLS la comunicación entre balanceador y servidor no está cifrada por lo que ese tráfico interno podría ser manipulado o interceptado. Por esta razón se tendrían que tomar otras medidas para proteger esta red interna.
Además, se tiene que tener seguridad en el balanceador, pues este va a tener acceso a los datos sin cifrar.

¿Qué pasos adicionales has tenido que hacer para que la máquina pueda salir a internet para poder instalar el servidor nginx?
He tenido que habilitar el NAT como router dentro de la red. SE CONECTA AUTOMÁTICAMENTE EL NAT A EL HTTP SERVER? LA RUTA SE HACE AUTOMÁTICA??

Parte 2.2 Proteger nuestra máquina de ataques SQL Injection, Cross Syte Scripting y restringir el tráfico sólo a paises de confianza de la UE implantando un WAF a nuestro balanceador.

En primer lugar generamos el certificado a través de los comandos de SSL y lo cargamos en el servidor. EL servidor web necesita estos documentos para poder firmar peticiones HTTPS.
![image](https://github.com/202306360/PracticasASR/assets/145692381/7e02c92b-8715-4a15-bbc1-719783e7a46b)

![image](https://github.com/202306360/PracticasASR/assets/145692381/72f7ee7f-5690-4bab-8839-8c6c00c454e2)

Una vez tenemos el certificado cargado, procedemos a crear el balanceador de L7.
Configuramos en Frontend y el Backend que se configura con un grupo de instancias instance group que tenemos que crear previamente.
Además, en el backend debemos de crear un HealthCheck mediante el cual el balanceador se asegure que el servidor está operativo. Este healthCheck se hace con una conexión TCP en el puerto 80.

<img width="793" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/aa87aabf-b302-4286-994d-95d47708abe2">

<img width="946" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/b2030dc3-9bd2-4d2b-9f3e-6fa0579e993d">

En este punto, también configuramos el Health Check de google en el Firewall para permitir las comprobaciones desde ciertos rangos de IP externas y a través del P80:
<img width="956" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/d6849c95-1d9e-4590-8d54-dc2ea4a9c4e3">


De esta forma, una vez tenemos el balanceador creado, ya podemos acceder al servidor a través de internet con la dirección IP del balanceador.
![image](https://github.com/202306360/PracticasASR/assets/145692381/a3a650f3-02bf-4893-b4d1-2e9c078662be)

Si lo hacemos a través del buscador de google, observamos que por el tipo de CA que hemos utilizado, nos sale el siguiente aviso:
![image](https://github.com/202306360/PracticasASR/assets/145692381/b90f8717-07bc-4a55-b950-ce3e8155c340)
![image](https://github.com/202306360/PracticasASR/assets/145692381/78aeb9cc-dfd4-4a25-92d8-45cb464cc607)

Ahora procedemos a configurar el WAF, a través de las políticas de Cloud Armor:
En este caso hemos creado 3 políticas
1)Restringimos el acceso únicamente a IPs de España.
2)Protejemos el acceso contra SQL injection
3)Protejemos el acceso contra Cross-Scripting

<img width="817" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/23ab39dc-a294-4fa4-b6fd-af811cb32bf0">


###  3ra mejora solución: zero trust 
En esta parte, buscamos que el contenido web esté cifrado también dentro del cloud.
La ultima parte: hacemos trabajar al engineX por el 443 y que el balanceador vaya también por el HTTPS en lugar del HTTP. Pasos a seguir:

1) En primer lugar, tenemos que modificar la configuración del instance group.
 <img width="473" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/d63abee2-7ce1-4ccd-bfe7-3c00452ac980">

2) Cambiamos la configuración en el Backend del Load Balancer:
  <img width="841" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/c0167e17-724e-4783-8826-ae99f9993726">

3)  Modificamos el Health Check del Load Balancer en el Back end:
   <img width="848" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/955ec9d2-1b89-4b5c-9e9f-f5d4ab341efb">

4) Modificamos la política de Health Check del Firewall para que se ejecute sobre el P443:
   <img width="807" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/f5127714-006e-4109-a0e1-4765f412e59f">

5) Modificamos la configuración de NgineX desde la consola del server para que trabaje sobre el P443:
Para poder cambiar la configuración del NgineX dentro del servidor, tenemos que mover los archivos .cert y .KEY al http-server.
Una vez tengamos estos archivos, tenemos que modificar la configuracion del NgineX habilitando el parámetro ssl, e indicando las rutas a los archivos .KEY y .crt
<img width="602" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/4fbcf2c2-b419-488a-b6a6-90b488283b05">

<img width="796" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/ea765fcb-8928-4603-b308-c23fbd6c4f5e">



Una vez modificada la configuración del NgniX, la actualizamos y la hacemos reload.
Procedemos a conectarnos al localhost y nos sale un aviso de que no se reconoce el CA por lo que no se puede abrir la conexión de forma segura.
Ejecutamos para ello el comando: curl -k https://localhost que se conecta de manera insegura, sin comprobar la autoridad del CA.
<img width="608" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/fb2ecb7d-4527-4ba3-a3c7-3b3b4ca57882">

Finalmente nos conectamos al htttps://localhost

<img width="614" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/a2ca7555-2773-4aec-bcb4-af240b7ff773">


Podemos acceder también desde el navegador:
<img width="960" alt="image" src="https://github.com/202306360/PracticasASR/assets/145692381/1ddc7cbb-b7a1-4ac7-9e79-53ab34956931">
