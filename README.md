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

1. Diríjase a el [Portal de Azure](https://portal.azure.com/) y a continuación cree una maquina virtual con las características básicas descritas en la imágen 1 y que corresponden a las siguientes:
    * Resource Group = SCALABILITY_LAB
    * Virtual machine name = VERTICAL-SCALABILITY
    * Image = Ubuntu Server 
    * Size = Standard B1ls
    * Username = scalability_lab
    * SSH publi key = Su llave ssh publica

![Imágen 1](images/part1/part1-vm-basic-config.png)

* Primero creamos la maquina virtual con las especificaciones dadas

![](images/part1/creacionMaquina1.png)

![](images/part1/creacionMaquina2.png)

![](images/part1/creacionMaquina3.png)

![](images/part1/creacionMaquina4.png)

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM (Revise la sección "Connect" de la virtual machine creada para tener una guía más detallada).

    `ssh scalability_lab@xxx.xxx.xxx.xxx`
	
	* Para poder conectarnos a la maquina, nos dirigimos a la pestaña de connect, dentro de la maquina virtual y seleccionamos SSH
	
	![](images/part1/ConexionMaquina.png)
	
	* Despues abrimos una terminal en la misma ruta en donde tengamos descargada la llave que se genero anteriormente para poder conectarse
	
	![](images/part1/ubicacionLlave.png)
	
	* En la terminal ponemos el comando que da azure para conectarse por SSH, accediendo a la maquina virtual
	
	![](images/part1/conexion.png)
	

3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).
4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`
	
	![](images/part1/clonarRepositorio.png)

    `cd <your_repo>/FibonacciApp`

    `npm install`
	
	![](images/part1/instalarNodeMaquina.png)

5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    ` npm install forever -g`
	
	` forever start FibinacciApp.js`

6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![](images/part1/part1-vm-3000InboudRule.png)
![](images/part1/ReglaPuerto.png)

 * Iniciamos la aplicacion en la maquina virtual
   ![](images/part1/Puerto3000.png)
  
 * Verificamos en el navegador, que este funcionando obteniendo como respuesta 8
   ![](images/part1/FuncionamientoCorrecto.png)
  
 * Revisamos la consola de la maquina virtual
   ![](images/part1/ConsolaF.png)


7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
    * 1000000
	Se demnoro 17,04 segundos
	
	![](images/part1/1.png)
	
    * 1010000
	Se demnoro 17,7 segundos
	![](images/part1/2.png)
	
    * 1020000
		Se demnoro 18,04 segundos
	![](images/part1/3.png)
	
    * 1030000
			Se demnoro 18,55 segundos
	![](images/part1/4.png)
    * 1040000
		Se demnoro 19,15 segundos
	![](images/part1/5.png)
    * 1050000
			Se demnoro 19,45 segundos
	![](images/part1/6.png)
    * 1060000
		Se demnoro 19,73 segundos
	![](images/part1/7.png)
    * 1070000
		Se demnoro 20,31 segundos
	![](images/part1/8.png)
    * 1080000
		Se demnoro 20,63 segundos
	![](images/part1/9.png)
    * 1090000    
		Se demnoro 21,08 segundos
	![](images/part1/10.png)

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

	

![Imágen 2](images/part1/part1-vm-cpu.png)

![](images/part1/11.png)

![](images/part1/12.png)

9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
		![](images/part1/13.png)
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
	![](images/part1/14.png)
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

![](images/part1/15.png)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.
    * 1000000
	Se demnoro 14,55 segundos
	
	![](images/part1/16.png)
	
    * 1010000
	Se demnoro 14,92 segundos
	![](images/part1/17.png)
	
    * 1020000
		Se demnoro 15,09 segundos
	![](images/part1/18.png)
	
    * 1030000
			Se demnoro 15,51 segundos
	![](images/part1/19.png)
    * 1040000
		Se demnoro 15,57 segundos
	![](images/part1/20.png)
    * 1050000
		Se demnoro 16,34 segundos
	![](images/part1/21.png)
    * 1060000
		Se demnoro 16,17 segundos
	![](images/part1/22.png)
    * 1070000
		Se demnoro 16,46 segundos
	![](images/part1/23.png)
    * 1080000
			Se demnoro 16,82 segundos
	![](images/part1/24.png)

    * 1090000    
			Se demnoro 16,93 segundos
	![](images/part1/25.png)

    * Verificamos el consumo en informacion general
	![](images/part1/26.png)
	![](images/part1/27.png)
		

12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.

13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?
   ![](images/part1/28.png)
2. ¿Brevemente describa para qué sirve cada recurso?
   * Disco virtual: Se ultiliza para el almacenamiento de datos 
   * Clave SSH: Es la clave de conexión para el acceso remoto al servicio 
   * Interfaz de red: Sirve para señalar la conexión que se da de manera física, entre los dispositivos y el sistema 
   * Grupo de seguridad de red: Se utiliza para filtrar el tráfico de la red 
   * Dirección IP pública: Permite acceder a la vm y a conexiones 
   * Red virtual: Es la red vlan que se crea para darle conexión a la maquina virtual 
   * Network watcher: Es un observador el cual administra el trafico externo de la red
3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?

   * Si se utiliza el comando npm FibonacciApp.js y la máquina se suspende por inactividad, la aplicación dejaría de correr, al igual que si existe un error en la  máquina virtual. Por eso se utiliza forever start FibonacciApp.js.

   * El Inbound port rule sirve para permitir la entrada al servicio que se está levantando. En este caso la aplicación corre por el puerto 3000, así que es este el que se debe abrir.

4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.

   * Tabla de los tiempos dados con el tamaño de la maquina en b1ls 
   
     Para este caso la funcion tarda mucho debido a la capacidad de procesamiento dada por el tamaño de la maquina que esta tiene.
  
     ![](images/part1/graficoB1ls.png)
  
   * Tabla de los tiempos dados con el tamaño de la maquina en b2ms
   
     En este caso se nota una mejora en cuanto al tiempo ya que al cambiar de tamaño de b1ls a b2ms, la capacidad de procesamiento mejoro en comparacion al anterior. Esto porque 
	 b2ms tiene mas RAM y mas almacenamiento que b1ls.
  
     ![](images/part1/graficoB2ms.png)

5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.
![](images/part1/26.png)
6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.
    * Si hubo fallos documentelos y explique.
	
	![](images/part1/tamañob1ls.png)
![](images/part1/tamañoB2ms.png)
7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?

   * Los de la serie B solo funcionan en linux 
   * B2ms: Tiene 2 vCPUs, 8 GB de RAM, 1 data disk y cuesta $ 60.74 dólares mensuales.
   * B1ls: Tiene 1 vCPUs, 0.5 GB de RAM, 1 data disk y cuesta $ 3.80 dólares mensuales.

8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?
   * Si bien aumentar la capacidad de procesamiento a la máquina ayuda a bajar el tiempo de ejecución de la aplicación, no fue un cambio tan significativo, y la relacion costo beneficio de la maquina B1LS no se compara a la B2ms 



9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?

   * Puede generar sobrecostos donde no se tenga claro la capacidad que se requiere 

10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?
    * Se nota una leve mejora en cuanto a los calculos y en cuanto al uso de la CPU, este se vio disminuido 
11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![](images/part2/part2-lb-create.png)

Primero vamos a crear el balanceador de carga, con la informacion necesaria

![](images/part2/balanceador1.png)

![](images/part2/balanceador2.png)

![](images/part2/creacionIP1.png)

![](images/part2/creacionIP2.png)

![](images/part2/balanceador3.png)

2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![](images/part2/part2-lb-bp-create.png)

Para poder crear un Backen Pool, toca primero crear una red virtual para eso nos dirigimos al paso numero 5, antes de continuar con este

Despues nos dirigimos al grupo de recursos y despues a grupos backend y le damos agregar

![](images/part2/backendPool.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)

Nos dirigimos al grupo de recursos y despues a sondeo de estados y le damos agregar

![](images/part2/sondeoEstado.png)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)

Nos dirigimos al grupo de recursos y despues a reglas de equilibrio de carga y le damos agregar

![](images/part2/reglaEquilibrio1.png)

![](images/part2/reglaEquilibrio1.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)

![](images/part2/redVirtual1.png)

![](images/part2/redVirtual2.png)

![](images/part2/redVirtual3.png)

![](images/part2/redVirtual4.png)

#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

![](images/part2/part2-vm-create1.png)

A continuacion se van a crear tres maquinas virtuales, con la unica deferencia en que la zona de disponibilidad no es la misma

![](images/part2/creacionMaquina1.png)

![](images/part2/creacionMaquina2.png)

![](images/part2/creacionMaquina3.png)

El proceso es lo mismo para las otras dos maquinas virtuales

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

![](images/part2/part2-vm-create2.png)

Verificamos que la red virtual y sub red seleccionada sea la correcta

![](images/part2/verificacionSubred.png)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

![](images/part2/part2-vm-create3.png)

Ahora pasamos a crear el grupo de seguridad, con la informacion necesaria

![](images/part2/grupoSeguridad.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

![](images/part2/part2-vm-create4.png)

   * Asignacion de la maquina virtual 1 (VM1) al balanceador de carga
   
   ![](images/part2/vmAlBalanceador.png)
   Repetimos el mismo procedimiento para las otras dos maquinas virtuales

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

Verificamos accediendo a nuestra ip publica del balanceador de carga la cual es http://20.223.2.24/fibonacci/1, con lo cual obtenemos el siguiente resultado

![](images/part2/verificacionBalanceador.png)


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




