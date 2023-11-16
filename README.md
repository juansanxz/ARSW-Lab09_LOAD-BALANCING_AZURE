### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans
## Santiago Arévalo Rojas
## Juan Felipe Sánchez Pérez  

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/es-es/free/students/). Al hacerlo usted contará con $100 USD para gastar durante 12 meses.

### Parte 0 - Entendiendo el escenario de calidad

Adjunto a este laboratorio usted podrá encontrar una aplicación totalmente desarrollada que tiene como objetivo calcular el enésimo valor de la secuencia de Fibonnaci.

**Escalabilidad**  
Cuando un conjunto de usuarios consulta un enésimo número (superior a 1000000) de la secuencia de Fibonacci de forma concurrente y el sistema se encuentra bajo condiciones normales de operación, todas las peticiones deben ser respondidas y el consumo de CPU del sistema no puede superar el 70%.

### Parte 1 - Escalabilidad vertical

1. __Diríjase a el [Portal de Azure](https://portal.azure.com/) y a continuación cree una maquina virtual con las características básicas descritas en la imágen 1 y que corresponden a las siguientes:__
    * __Resource Group = SCALABILITY_LAB__
    * __Virtual machine name = VERTICAL-SCALABILITY__
    * __Image = Ubuntu Server__
    * __Size = Standard B1ls__
    * __Username = scalability_lab__
    * __SSH publi key = Su llave ssh publica__
      

![Imágen 1](images/part1/part1-vm-basic-config.png)    

Se crea la máquina virtual con las características indicadas.  
![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/b2a7022c-81a7-419b-980c-e4c948569043)  



2. __Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM (Revise la sección "Connect" de la virtual machine creada para tener una guía más detallada).__

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

   Se realiza la conexión a la VM.  
   ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/1285eedb-6fc0-43a9-9144-88c695fd78b0)  



3. __Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).__    

   Realizamos la descarga con el comando indicado.
   ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/08b2a2f4-0f74-439e-b10f-79229085e3fa)

   Verificamos que se haya instalado, consultando la versión:  
   ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/1321d55c-d685-4d5e-b129-84caa0787663)  

   Finalmente, instalamos node:  
   ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/a86af239-7381-4e6e-a184-d6116917d75a)  

   Verficamos la versión de node instalada:  
   ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/c33e4a1b-9570-49b7-a3a1-721b8544cd31)

   Además. se instalan dos versiones de node:  
   ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/791efb61-ba03-43f4-ba69-cf8e52fde099)  
   ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/4b181169-1db2-470d-9d5c-31028c07fc90)  

   Listamos las versiones de Node.Js para ver cual versión está en uso:  
   ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/600131b2-37a4-446f-a8eb-c2ee43d81a1d)  

4. __Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:__

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`

   Clonamos el repositorio e instalamos lo indicado:
   ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/04c7f8cd-e213-4206-81e2-e4c73e37c712)  


5. __Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.__

    ` node FibonacciApp.js`

   Ejecutamos la aplicación:
   ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/cb865c58-15da-4d74-b2e1-4b592cf7ce3d)  


6. __Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.__

![](images/part1/part1-vm-3000InboudRule.png)  

Se hace la configuración de la regla:  
![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/762727ba-af71-4703-92b6-fe23d7f76ec5)  

Se prueba el funcinamiento usando el endpoint, hallando el fibonacci de 6:    
![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/c6b0b199-2612-4c7c-ae58-f9817846bdf8)  

7. __La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:__  
    
    A continuación, se muestran los tiempos de respuesta:  
    * 1000000  
      ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/3eae1c19-6dca-4b6d-bd37-f41a7fd3a348)  

    * 1010000  
      ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/81a0a126-8a02-488c-ae2f-80f926a9aa60)  

    * 1020000    
      ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/fe812c9f-b419-415e-aac1-523a51d37345)

    * 1030000  
      ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/e830fb5d-7ee3-41db-84cd-ea61bf5d1ac1)     

    * 1040000  
      ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/fcfdec29-8135-4b4d-92f5-1b8786fab671)  

    * 1050000  
      ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/52f5c88f-3dca-4acc-acfd-8cd63e50d6e1)  

    * 1060000  
      ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/68d44f0d-8bfe-4449-8c1d-256a51ec5047)
      
    * 1070000  
      ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/570b6fdf-7ff0-4ff0-a4d7-f2c7c7ecf732)  

    * 1080000  
      ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/6784c27e-7fc2-494a-bec8-145dca890131)  

    * 1090000  
      ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/a50ec213-1f23-4879-abab-a4af046693f6)  
   

8. __Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).__  

![Imágen 2](images/part1/part1-vm-cpu.png)  
Desde Azure, observamos el consumo de CPU de la máquina virtual en la última hora:  
![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/31729807-f705-45ee-9faa-842d0516acaa)

9. __Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.__
    * __Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).__
    * __Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.__
    * __Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.__
    * __Ejecute el siguiente comando.__  

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```
    Antes de instalar newman, instalamos y cambiamos la versión de node a 18.18.2:  
   <img width="570" alt="image" src="https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/b1a7056b-d000-4b5f-af68-0678f6453abf">

   Y la configuramos para que sea la que se use por defecto:
   <img width="569" alt="image" src="https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/545ac520-6530-4292-897e-156203231ec3">

    Instalamos newman en la VM:
   ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/7c62f51b-59e8-4942-8a6a-9825f1918223)

   Ejecutamos el comando dado, y se empieza a simular la carga concurrente al sistema, obteniendo los siguientes resultados:
   <img width="1440" alt="image" src="https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/d55e2b5f-ba6b-4819-bbb9-c514eda7e5ae">
   <img width="618" alt="image" src="https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/93c4bab4-d33e-4fd7-999b-2701c9382597">  

10. __La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.__  

![Imágen 3](images/part1/part1-vm-resize.png)  
Se realiza el escalamiento vertical de la VM:  
<img width="906" alt="image" src="https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/5e493469-cfa9-4bb1-9144-8303c2c7b77b">  


11. __Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.__    
   Se repite el paso 7:  
   A continuación, se muestran los tiempos de respuesta:  
    * 1000000  
      ![imagen](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/307cc1a0-ec57-4475-8a5a-2fecaeb603d8)  

    * 1010000  
      <img width="1196" alt="image" src="https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/8e5924b7-7f97-491f-b0ea-752387f3e2da">  

    * 1020000  
      <img width="1200" alt="image" src="https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/0b35e12a-da30-41df-a60e-08e79ad32f30">  

    * 1030000  
      <img width="1196" alt="image" src="https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/afd3cd9c-126c-4717-8907-4c8f9502aaf6">  

    * 1040000  
      <img width="1200" alt="image" src="https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/f3975be3-9475-440a-9a43-bf52645bff56">  

    * 1050000  
       <img width="1198" alt="image" src="https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/adca9803-a08e-42cb-999c-4cd02dfa0bd4">  

    * 1060000  
        <img width="1197" alt="image" src="https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/5ad248dd-a795-44b7-bb5a-2ffadbb203d8">  

    * 1070000  
        <img width="1196" alt="image" src="https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/979b6721-c063-4629-a2fd-fbc960ad7299">  

    * 1080000  
        <img width="1197" alt="image" src="https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/d07e8f12-2d2c-474d-acc8-18904c655850">  

    * 1090000  
      <img width="1198" alt="image" src="https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/a9835743-b947-4177-aa22-96d2ffc243cd">  

   Se repite el paso 8:  
   Desde Azure, observamos el consumo de CPU de la máquina virtual en la última hora:  
   <img width="1187" alt="image" src="https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/b232a797-2a7b-4325-81a6-a8c179f18c88">  
   Al hacerse varias peticiones al tiempo, el consumo de CPU llegó a ser del 99% aproximadamente:  
   <img width="1165" alt="image" src="https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/3b15655b-99bd-4fa4-8bb8-6e9edc7e0b02">  

   Se repite el paso 9:  
   Ejecutamos el comando para simular la carga concurrente al sistema, obteniendo los siguientes resultados:  
   ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/0bde9757-015c-4069-9e79-82c940e83229)  
   <img width="1440" alt="image" src="https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/73900d52-7b05-4551-8126-746bb1fcbde2">  
   <img width="1439" alt="image" src="https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/c053a9b6-7089-4a7a-be5e-4582be4db7ec">  
   <img width="559" alt="image" src="https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/bed083d2-5283-444c-a19f-319c4ca64635">  


12. __Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.__    
   Usando este modelo de escalabilidad vertical, no se logra cumplir satisfactoriamente el escenario de calidad del requerimiento no funcional de escalabilidad. Como se evidencia en las imagenes anteriores, especificamente al repetir el punto 7; se observa que el tiempo que toma la aplicación en obtener el enésimo valor de la secuencia de Fibonnaci es mayor al 70% indicado en el escenario de calidad (cerca del 99%), e incluso mayor a la prueba realizada antes de aumentar los recursos de la máquina virtual. 
   
13. __Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.__
    Se deja la VM con el tamaño inicial.  
    <img width="890" alt="image" src="https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/5f10628a-0f85-4ef9-b389-a486ed527bcc">  


**Preguntas**

1. __¿Cuántos y cuáles recursos crea Azure junto con la VM?__
   Cuando se crea una VM se crea 7 recursos en total, los cuales son:  
   ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/79ee52b2-8613-4072-ae2a-f1a10ba81e7b)  
   Agregando la red virtual.  

2. __¿Brevemente describa para qué sirve cada recurso?__
   * __Máquina virtual:__ En Azure, una máquina virtual (VM) es una instancia de un sistema operativo en la nube que se ejecuta en hardware físico de Azure.
   * __Dirección IP pública:__ Se pueden asignar direcciones IP públicas o privadas a la máquina virtual, dependiendo lo que se requiera. En este caso, se estableció una dirección pública que permite la comunicación con el exterior (desde y hacia Internet).
   * __Grupo de Seguridad de Red:__ Es un componente importante para gestionar el tráfico de red en una red virtual. Un NSG contiene reglas que permiten o deniegan el tráfico de red       hacia y desde los recursos dentro de una red virtual en Azure.
   * __Red Virtual:__ Brinda aislamiento de red para la máquina virtual y permite la comunicación con otros recursos dentro de la misma red en el ambiente de Azure, limitando el acceso no autorizado y la visibilidad de la máquina. Puede incluir elementos como direcciones IP, subredes, reglas de firewall, tablas de ruteo, políticas de direccionamiento ip, etc.
   * __Interfaz de Red:__ Es un recurso que representa una interfaz de red en una máquina virtual o en una red virtual. La cual permite, conectividad de máquina virtual, asignación de direcciones IP, conectividad a redes virtuales, balanceo de carga, configuración de reglas de seguridad, integración con servicios de red y redirección de tráfico.
   * __Disco:__ Se refiere a los discos de almacenamiento que se pueden asociar a máquinas virtuales para proporcionar almacenamiento persistente. Hay dos tipos principales de discos en Azure: discos administrados y discos no administrados.
   * __Clave SSH:__ Este recurso es utilizado para realizar una conexión segura a nuestras máquinas virtuales en el proceso de autenticación.

3. __¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?__  
   Al cerrar la conexión que se realizó con la máquina virtual se cae la aplicación debido a que dicha conexión estaba proporcionando el entorno para que la aplicación se ejecutara, de esta forma, al cerrar la conexión se acaba cualquier proceso relacionada a esta. Por otra parte, era necesario crear una Inbound port rule antes de poder acceder al servicio porque este recurso nos permite gestionar el tráfico por los puertos de nuestra máquina y así especificar que puertos desea abrir para la conexión, de lo contrario, mis peticiones no se podrán realizar de manera exitosa.  

4. __Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.__  
   ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/73b35a5f-1f99-4510-82c3-c6afc175c0e1)  
   La función tarda bastante tiempo porque las peticiones que se realizan son de un enésimo número de Fibonnaci muy grande, y sólo hay una máquina con unas capacidades limitadas       
   respondiendo tales solicitudes, por lo que el consumo de recursos en la misma es alto y retrasa la respuesta de cada una.  

6. __Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.__  
   ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/31729807-f705-45ee-9faa-842d0516acaa)  
   Consume bastante CPU, teniendo en cuenta los recursos que tiene la VM (1 CPU, 1 GiB de RAM).  
   
7. __Adjunte la imagen del resumen de la ejecución de Postman. Interprete:__  
    * __Tiempos de ejecución de cada petición.__  
    * __Si hubo fallos documentelos y explique.__    
   Los resultados de ejecución de las peticiones de Postman se encuentran previamente, en el desarrollo del laboratorio en el punto donde solicitan realizar las pruebas. Y se logra observar que luego de agregar la escalabilidad, los tiempos de ejecución no mejoran mucho, quedamos en un promedios de 20 segundo, para un total de  2 minutos y 43 mientras que sin la escalabilidad era de 3:11.  
8. __¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?__  
   Lo primero que debemos saber es que las máquinas de series B, son utilizadas para cargas de trabajo que tienen requisitos de procesamiento variables, ya que utilizan un modelo de créditos de rendimiento. Obtienen créditos cuando la CPU está inactiva y pueden consumir estos créditos durante períodos de actividad intensa.  
   * __B2ms:__ 2vCPU, 8 Gib de Memoria, 16 GiB de SSD, 60% de rendimiento de CPU base de la máquina virtual, 60 créditos iniciales, 36 créditos ingresados/hora, 864 créditos máximos   ingreados, 4 discos de datos máximos, 1920/22.5 rendimiento máximo del disco sin almacenamiento en la caché IOPS/Mbps, 4000/100 rendimiento máximo del disco sin almacenamiento en la caché expandido y 3 Nº máx. NIC.  
   * __B1ls:__ 1 vCPU, 0.5 Memoria: GiB, 4 GiB de almacenamiento temporal (SSD), 10 rendimiento de CPU base de la máquina virtual (%), 30 créditos iniciales, 3 créditos ingresados/hora, 72 créditos máximos ingresados, 2	discos de datos máx, 160/10 rendimiento máximo del disco sin almacenamiento en la caché, 4000/100 rendimiento máximo del disco sin almacenamiento en la caché expandido y 2 Nº máx. NIC.  
9. __¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?__  
   No es una buena solución en este caso aumentar el tamaño de la VM en este caso, pues cada petición consume muchos recursos de la VM, y el escenario de calidad indica que no debe 
 superarse el 70% del uso de la CPU cuando múltiples usuarios usen la aplicación, lo cual no se cumple al realizar escalabilidad vertical. 
 Cuando se cambia el tamaño de la VM, se demora más en dar un resultado la aplicación FibonacciApp, incluso hay solicitudes que fallan al realizarlas con el comando newman.    
10. __¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?__  
    Azure debe acomodar los recursos físicos de una infraestructura cuando se cambia el tamaño de una VM. Esto impacta de forma negativa algunas aspectos, ya que se puede afectar la disponibilidad del servicio por causa de un breve periodo de inactividad, pueden haber intervalos de tiempo con un rendimiento más bajo mientras se aplican los nuevos ajustes, o puede que deban cambiarse configuraciones específicas que requieran ser modificadas para que la máquina virtual funcione con su nuevo tamaño.  
11. __¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?__  
    No hubo mejora ni en el consumo de CPU (que al hacer múltiples solicitudes a la aplicación fue de 99% aproximadamente) y tampoco en los tiempos de respuesta. Esto se debe a que cada petición consume muchos recursos de la VM, y al hacerse varias al tiempo, la máquina no es capaz de satisfacer el escenario de calidad teniendo tal volumen de trabajo.    
12. __Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?__  
    Al hacerlo con 4 ejecuciones paralelas:  
    ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/ccbd68d0-a56d-4f12-910a-bc6393098bd7)  
    Al hacer 4 ejecuciones paralelas sin realizar la escalabilidad veritical se obtienen los siguiente resultados:  
    ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/68acbb21-6c4a-43cb-a6b8-90bd48dc7354)  
    ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/201f804d-7a93-4de0-98ce-567d610d5acd)  
    ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/a424aeef-be6b-4278-a539-cf6fcaf3eee3)  
    ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/a22efdf0-e216-4ca5-a73e-ea8146e085d8)  
    Y con la escalabilidad vertical:  
    ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/dfb87c2e-7ad8-4ffc-aef2-5be9dab4dff4)  
    ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/9c91fe18-5db8-40aa-8fca-7c02bdad81bc)  
    ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/6deab623-04d3-4fa6-a371-b07afad53cbb)  
    ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/19f7b48b-2e57-42e6-b135-c857f3548b58)  
    En donde se evidencia que las peticiones no fueron resueltas y no es una solución adecuada. Además al ver el consumo de CPU, encontramos que antes del escalamiento se tenía el siguinete consumo:  
    ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/accd6e3f-521b-4a93-869f-e2ecb045ff13)  
    Una vez aumentado el tamaño, se obtuvo:  
    ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/9a7ca3d9-2e58-48d1-a326-d094b9f2a331)  
    Lo cual, aunque parece una buena señal porque disminuyó el consumo, consideramos que se ve relacionado a la cantidad de errores que se profujeron, bajando la cantidad de uso.  


### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![](images/part2/part2-lb-create.png)  
Se crea el balanceador de cargas como se indica:  
![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/7103aecb-fd43-4ff5-a7e1-929a62f643b5)  
![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/084ffa66-715d-4f24-84ad-62a2f220ec57)  
![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/b389b5a9-ae79-48aa-9461-7a66dde8f01f)  
![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/6bdf2eca-ebc5-4f69-bb99-da4f798a7856)  
![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/86813e9d-329b-4ced-be10-34c4f762b262)  



2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![](images/part2/part2-lb-bp-create.png)
Crear el Backend Pool:  
![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/8c5d039d-4e68-4815-8840-5106ab8baedb)  


3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)
El Health Probe ahora se crea en el mismo instante de crear un Loas Balancing Rule:  
![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/b12421ea-23ee-45f0-ad67-0b716e681300)  


4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)  
Se crea el Load Balancing Rule como se especificó:  
![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/cf2d7b77-7129-41f3-8dbf-e2366e3d78c1)  


5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)
Se empieza con la creación de la Virtual Network:  
![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/2931c9a2-92ce-464c-bbb6-f978d8ca2d28)
![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/3cff2fa9-7d7d-4b5d-99e5-9a04252ff250)    
![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/5ad1d0b7-368d-4087-a91b-748b407fba2e)  




#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

![](images/part2/part2-vm-create1.png)

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

![](images/part2/part2-vm-create2.png)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

![](images/part2/part2-vm-create3.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

![](images/part2/part2-vm-create4.png)  
Ya se tienen las 2 máquinas virtuales que nos permite crear con la suscripción de Student:  
![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/dfe1b7be-5145-4291-a9e6-8e0cadb7a4cc)  
![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/13882c9b-c4de-4e75-b31b-7829e5c68ad3)  



5. __ Finalmente debemos instalar la aplicación de Fibonacci en la VM. para ello puede ejecutar el conjunto de los siguientes comandos, cambiando el nombre de la VM por el correcto__

```
git clone https://github.com/daprieto1/ARSW_LOAD-BALANCING_AZURE.git

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
source /home/vm1/.bashrc
nvm install node

cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp
npm install

npm install forever -g
forever start FibonacciApp.js
```

__Realice este proceso para las 3 VMs, por ahora lo haremos a mano una por una, sin embargo es importante que usted sepa que existen herramientas para aumatizar este proceso, entre ellas encontramos Azure Resource Manager, OsDisk Images, Terraform con Vagrant y Paker, Puppet, Ansible entre otras.__

Se realiza la instalación solicitada en las 2 máquinas virtuales.

#### Probar el resultado final de nuestra infraestructura

1. __Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:__

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
```
Aquí podemos ver la dirección IP del balanceador de cargas que es al cual nos vamos a conectar:  
![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/2d11b481-d8ef-4c11-aa4a-ba4f0f1dd77e)  
Al entrar al primer recurso encontramos un mensaje de Hello World:  
![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/886f35cb-cd7a-4288-a520-5d9f3860a61e)  
Y si accedemos al recurso de fibonacci:  
![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/1e679872-c054-4d88-9c1f-40a8b77187e4)  


2. __Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.__
Al realizar una escalabilidad horizontal nos dimos cuenta que si mejora el rendimiento de la aplicación debido a que de las peticiones solicitadas solo 3 no fueron exitosas:
![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/27555483-1242-477b-973f-142c3c11f762)
![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/0b636776-79a9-482b-ab2a-166614936834)  
A comparación de cuando se tenía una infraestructura esacalada verticalmente en la cual fallaron 9 peticiones.
Por otro lado, al ver los timepos de respuesta obtenidos en ambas infraestrcutruras, no se evidencia un cambio significativo.  
4. __Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.__

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```
No fue posible realizar debido a que la memebresía de estudiante no permite tener más de 3 recursos.

**Preguntas**

* __¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?__  
  En la siguiente imagen se muestran los servicios de balanceador de carga de azure y se especifican dos características fundamentales:  
  ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/debdd77e-3ea2-416e-90c7-0d8b1b1712ee)  
  * __Front Door (Puerta de Enlace Frontal):__ Es un servicio de red global  de entrega de aplicaciones que ofrece enrutamiento de tráfico, equilibrio de carga y protección contra ataques DDoS a escala global. Es ideal para aplicaciones web y API distribuidas a nivel mundial, proporcionando aspectos como una alta disponibilidad, un mejor rendimiento, baja latencia y distribución del tráfico.  
 
  * __Azure Traffic Manager (Administrador de Tráfico de Azure):__ Está basado en DNS, por lo que solo equllibra la carga a nivel de dominio, y se encarga de distribuir el tráfico entre diferentes ubicaciones geográficas o entre diferentes servicios de Azure basándose en reglas de enrutamiento configurables. Es útil para implementaciones globales que requieren 
    alta disponibilidad y redundancia. Por su naturaleza,  no puede redirigir el tráfico de una instancia a otra en caso de fallas, tan rápidamente como con Azure Front Door, debido a 
    los desafíos comunes relacionados con el almacenamiento en caché de DNS y a los sistemas que no respetan los TTL de DNS.  
 
 * __Application Gateway (Puerta de Enlace de Aplicaciones):__ Está a nivel de aplicación (capa 7 del modelo OSI) que se encarga de proporcionar enrutamiento basado en URL. Está diseñado para aplicaciones web y proporciona funcionalidades como enrutamiento basado en contenido, SSL offloading y terminación (para la gestión de conexiones seguras), así como reglas de enrutamiento complejas. Útil para optimizar la productividad de las granjas de servidores web al traspasar la carga de la terminación SSL con mayor actividad de la CPU a la puerta de enlace.  
 
 * __Azure Load Balancer (Equilibrador de Carga de Azure):__ Admite una topología regional o global, y distribuye el tráfico de red entrante entre las instancias de máquinas virtuales 
   dentro de una red virtual. Opera a nivel de red (capa 4 del modelo OSI), ideal para cargas de trabajo TCP y UDP. Ofrece una latencia muy baja, alta disponibilidad y alto rendimiento 
   (entrante y saliente) para los protocolos de la capa de transporte mencionados anteriormente.  

SKU (Stock Keeping Unit) hace referencia a un identificador único utilizado para distinguir entre las diversas ofertas de servicios y recursos disponibles en la plataforma de azure. Los SKU se utilizan para identificar diferentes versiones, tamaños, niveles de rendimiento o capacidades de un servicio o producto en Azure.
Los SKU se aplican a una amplia gama de servicios y recursos de Azure, como máquinas virtuales, bases de datos, servicios de almacenamiento o servicios de red. Cada tipo de recurso tiene sus propios conjuntos de SKU, y las opciones de SKU disponibles pueden variar según el servicio específico.
Por ejemplo, los SKU de almacenamiento son:  
![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/4bb3bbca-92be-4e5b-aec8-8a25808719e2)  
El balanceador de carga requiere el uso de una IP pública porque es el punto de entrada para el tráfico externo que llega desde internet para consumir algún tipo de servicio, o acceder a una aplicación, y éste se encarga de distribuirlo entre las instancias correspondientes que ofrecen tal servicio.  
* __¿Cuál es el propósito del *Backend Pool*?__  
  El propósito principal del Backend Pool es definir y agrupar las instancias de servicio o recursos (en este caso las VM) que recibirán el tráfico entrante de forma equitativa después de que haya sido recibido por el balanceador de carga, facilitando la escalabilidad y disponibilidad del servicio.  
* __¿Cuál es el propósito del *Health Probe*?__  
  La funcionalidad del Health Probe dentro de un balanceador de cargas es mantener monitoreados las instancias de servicios y así poder ver que estén funcionando correctamente y que se 
  encunetren dispoinibles. La Health Probe permite al balanceador de cargas tomar decisiones informadas sobre la distribución del tráfico entre las instancias de servicio, mejorando la 
  disponibilidad y la eficiencia del sistema.  
* __¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.__  
  La Load Balancing Rule (Regla de Balanceo de Carga) en un balanceador de cargas de Azure se utiliza para definir cómo se distribuirá el tráfico entre las instancias de servicio. 
  Estas reglas especifican cómo se asignan las solicitudes de entrada a las instancias de destino y cómo se realiza la administración del tráfico.
  Los tipos de sesiones persistentes son:  
   * __Sin Sesión Persistente:__ Cada solicitud se enruta de forma independiente, sin relación con las solicitudes anteriores. Puede haber una variabilidad en la instancia de destino para cada solicitud.  
   * __Sesión por IP:__ La regla de balanceo de carga asigna una IP cliente específica a una instancia de destino. Todas las solicitudes desde la misma IP se enrutan a la misma instancia.  
   * __Sesión por Cookie:__ Se asigna una cookie específica a una instancia de destino. Las solicitudes del mismo cliente con la misma cookie se dirigen a la misma instancia.
 Las sesiones persistentes son importantes en aplicaciones que requieren que las conexiones del usuario se mantengan con la misma instancia para garantizar la continuidad y el estado de la sesión.  
En aplicaciones web que almacenan información en el lado del servidor, las sesiones persistentes ayudan a evitar problemas cuando las solicitudes de un usuario se enrutan a diferentes instancias.  
  
* __¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?__  
  * Una __Virtual Network__ (Red Virtual) en Azure es un servicio que permite crear y administrar redes privadas en la nube de Azure. Es una representación virtual de una propia red en la nube, donde se pueden implementar recursos, como máquinas virtuales (VM) y servicios, de manera segura y aislada.  
  * Una __subred__ es una división de una red virtual en segmentos más pequeños. Permite organizar y segmentar recursos dentro de la red virtual. Se hace esto con el fin de aprovechar de mejor manera la red que se tiene.  
  * El __Address Space__, también conocido como CIDR (Classless Inter-Domain Routing), es el rango de direcciones IP privadas que se asigna a una red virtual. Define el conjunto de direcciones IP que se pueden utilizar para los recursos dentro de la red virtual.  
  * Un __Address Range__ es un subconjunto del Address Space y se asigna a una subred específica dentro de la red virtual. Define el rango de direcciones IP que se puede usar dentro de esa subred.  
* __¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?__  
  Las Availability Zones son ubicaciones físicas y separadas dentro de una región específica, las cuáles se encuentran conectadas a mediante una red que proporciona una baja latencia y 
  una alta velocidad. Cada zona de disponibilidad es un centro de datos independiente con sus propias instalaciones de alimentación, enfriamiento y redes. El propósito principal de 
  tener múltiples zonas de disponibilidad en una región es mejorar la resiliencia y la disponibilidad de los servicios que se están ejecutando en la nube de azure.  
 
  Se usan tres zonas de disponibilidad, precisamente implementando y mejorando la presencia de este atributo de calidad en el sistema, permitiendo que pueda seguir funcionando aún 
  cuando ocurran fallas. Además, es una forma de distribuir el trabajo para mejorar el rendimiento y los tiempos de respuesta de solicitudes.  
 
  La redundancia de zona en las direcciones IP permite que la dirección IP asignada  a una VM puede ser utilizada por otros recursos en cualquier zona de disponibilidad dentro de la 
  misma región, con el fin de asegurar la continuidad del servicio en caso de que alguna zona falle o presente alguna interrupción del servicio.  
* __¿Cuál es el propósito del *Network Security Group*?__  
  Es un recurso en Azure que actúa como un firewall virtual para controlar el tráfico de red hacia y desde los recursos dentro de una red virtual. Su propósito principal es proporcionar una capa de seguridad a nivel de red, permitiendo o denegando el tráfico según las reglas de seguridad que se definan.  
* __Informe de newman 1 (Punto 2)__  
   El informe se encuentra previamente en la sección donde se pedía hacer las pruebas.  
* __Presente el Diagrama de Despliegue de la solución.__  
  ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812766/3f36303c-db7e-41c9-b670-7c4d0f3856d8)   





