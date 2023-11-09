### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

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

9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

    Instalamos newman en la VM:
   ![image](https://github.com/juansanxz/ARSW-Lab09_LOAD-BALANCING_AZURE/assets/123812331/7c62f51b-59e8-4942-8a6a-9825f1918223)
   


11. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)  



11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.
12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.
13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?
2. ¿Brevemente describa para qué sirve cada recurso?
3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?
4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.
5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.
6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.
    * Si hubo fallos documentelos y explique.
7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?
8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?
9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?
10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?
11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![](images/part2/part2-lb-create.png)

2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![](images/part2/part2-lb-bp-create.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)

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

5. Finalmente debemos instalar la aplicación de Fibonacci en la VM. para ello puede ejecutar el conjunto de los siguientes comandos, cambiando el nombre de la VM por el correcto

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

Realice este proceso para las 3 VMs, por ahora lo haremos a mano una por una, sin embargo es importante que usted sepa que existen herramientas para aumatizar este proceso, entre ellas encontramos Azure Resource Manager, OsDisk Images, Terraform con Vagrant y Paker, Puppet, Ansible entre otras.

#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
```

2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?
* ¿Cuál es el propósito del *Backend Pool*?
* ¿Cuál es el propósito del *Health Probe*?
* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.
* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?
* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?
* ¿Cuál es el propósito del *Network Security Group*?
* Informe de newman 1 (Punto 2)
* Presente el Diagrama de Despliegue de la solución.




