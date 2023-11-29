# Automatización
Coloco las notas que he tomado a lo largo de mi investigación de Ansible
Nodos de control | Para antes de usar Ansible

Ansible es fácil de instalar. El software Ansible solo debe instalarse en el nodo (o nodos) de control desde el que se ejecutará Ansible. Los hosts administrados por Ansible no necesitan tener Ansible instalado.

La instalación del conjunto de herramientas principal de Ansible implica relativamente pocos pasos y tiene requisitos mínimos. Por otro lado, la instalación de los componentes adicionales que proporciona Red Hat Ansible Automation Platform, como el controlador de automatización (anteriormente llamado Red Hat Ansible Tower), requiere un sistema Red Hat Enterprise Linux 8.2 o posterior, con un mínimo de dos CPU, 4 GiB de RAM y 20 GiB de espacio disponible en disco.

Python 3 (versión 3.5 o posterior) o Python 2 (versión 2.7 o posterior) debe instalarse en el nodo de control.

--------------------------------------------------------------------------
Para Versiones más actuales podemos usar el microservicio 
Primero instalamos ansible -navigator --version 
sudo dnf install ansible-navigator

y usamos su plataforma que tiene ambientes con todos los registros y palybook para ejecutar en ambientes virtuales 

tiene que ser la versión 2.1.0 y usamos el comando podman para logearnos sobre el ambiente $podman login utility.lab.example.com 

------------------------------------------Documentación Core--------------------------------

Documentación básica por parte de redhat

Instlación y configuración 
https://docs.ansible.com/ansible/latest/installation_guide/index.html

SSH troobleshoting
1: https://access.redhat.com/solutions/3356121
2: https://access.redhat.com/articles/3967261#troubleshooting-common-issues-19

Entendiendo la implementación modular
https://access.redhat.com/articles/3967261#modules-7

Playbooks de para troobleshoting
https://access.redhat.com/articles/3967261#troubleshooting-common-issues-19

Ansible en AWS y otras soluciónes core
https://access.redhat.com/articles/3967261#aws-14

Casos de uso 
https://content.redhat.com/content/rhcc/us/en/assets/display.html?id=fb2628d2-022b-4006-badb-f914662238ab

-----------------------------Terminos y condiciónes de Ansible platform----------------------------------




---------------------------------------------------------------------------------------------------------

Los host que tengan windows tendrian que tener cuando menos PowerShell 3.0 o posterios con una variable de entorno que tenga python y para hacer instalaciónes se necesitan tener NETFramework 4.0

Con ansible se recomienda tener agrupados los los hostname y dominios en base a un grupo fijo de invitados y ahcer grupos mas grades que encapsulen a tro grupos por ejemplo:

osts can be in multiple groups. In fact, recommended practice is to organize your hosts into multiple groups, possibly organized in different ways depending on the role of the host, its physical location, whether it is in production or not, and so on. This allows you to easily apply Ansible plays to specific hosts.

[webservers]
web1.example.com
web2.example.com
192.0.2.42

[db-servers]
db1.example.com
db2.example.com

[east-datacenter]
web1.example.com
db1.example.com

[west-datacenter]
web2.example.com
db2.example.com

[production]
web1.example.com
web2.example.com
db1.example.com
db2.example.com

[development]
192.0.2.42

Important

Two host groups always exist:

    The all host group contains every host explicitly listed in the inventory.

    The ungrouped host group contains every host explicitly listed in the inventory that is not a member of any other group.

Defining Nested Groups

Ansible host inventories can include groups of host groups. This is accomplished by creating a host group name with the :children suffix. The following example creates a new group called north-america, which includes all hosts from the usa and canada groups.

[usa]
washington1.example.com
washington2.example.com

[canada]
ontario01.example.com
ontario02.example.com

[north-america:children]
canada
usa

Para lanzar paybooks a un rango de nodos la sintaxis es:
Ranges match all values from START to END, inclusively. Consider the following examples:

    192.168.[4:7].[0:255] matches all IPv4 addresses in the 192.168.4.0/22 network (192.168.4.0 through 192.168.7.255).

    server[01:20].example.com matches all hosts named server01.example.com through server20.example.com.

    [a:c].dns.example.com matches hosts named a.dns.example.com, b.dns.example.com, and c.dns.example.com.

    2001:db8::[a:f] matches all IPv6 addresses from 2001:db8::a through 2001:db8::f.

Podemos ver toda la segmentación de grupos en el archivo /etc/ansible/hosts

-----------------RHEL 8 ---------------------
- Listar Hosts ligados y mapeados
  $  ansible ansible all --list-hosts
- Listar Hosts agrupados 
  $ ansible ungrouped --lists-hosts
- Grupos especificos
  $ ansible <namegroup> --lists-hosts

----------------RHEL 9 -----------------------
ansible-navigator <FILE> -i <FILE> -m stdout --list

Comando que se sugiere para checar los hosts que estan sobre un archivo plano 
-----------------RHEL 8 ---------------------
$ ansible <groupname> -i <file> --list-hosts
-----------------RHEL 9 ---------------------
$ ansible-navigator <FILE> -i <FILE> -m stdout --graph ungrouped

-----------------RHEL 8 ---------------------
Configuración de Ansible /etc/ansible/ansible.cfg
-----------------RHEL 9 ---------------------
$ ansible-navigator config




Algunas de las configuraciónes básicas son:

inventory  	    | 	Specifies the path to the inventory file
                | 
remote		    |	EL nombre de los usuarios logeados en los host 
                |   administrados, si este no esta especificado se usa 
                |   el ususraio normal
                | 
ask-pass	    | 	Si solicitar o no una contraseña SSH. 
                |   puede ser false si se utiliza la autenticación de 
                |   clave pública SSH.
                |
become          |       Si cambiar automáticamente de usuario en el host
                |       administrado (normalmente a raíz) después de
                |       conectarse. Esto también puede ser especificado por           
                |       una obra de teatro.
become_method   |       Cómo cambiar de usuario (normalmente sudo, que es el
                |       predeterminado, pero su es una opción).
                |
become_user     |       El usuario al que cambiar en el host gestionado 
                |       (normalmente root, que es el valor predeterminado).
                | 
become_ask_pass |       Ya sea para solicitar una contraseña para su método
                |       Become_method. El valor predeterminado es falso  

Al parecer es muy importante importar los archivos las direcciónes a los archivos de inventarios 

Ahora el archivo de mayor importancia a importar es [DEFAULTS]
para que apunte a directorios con el inventario de pod's
inventory = ./inventory

remote_user = root # logeo como root dentro de el arreglo de hosts
ask_pass = true    # yes fingerprint para entrar en la consola virtual que se encuentre mapeada 
para establecer una conección segura entre el host master  los nodos invitados es necesario cifrar la conexíon son llaves privadas en caso que soloquemos FALSE para importar la llave en caso de que sea necesario a todos los nodos usamos un playbook maestro tal como:
PALYBOOK SSH KEYGEN
- name: Public key is deployed to managed hosts for Ansible
  hosts: all

  tasks:
    - name: Ensure key is in root's ~/.ssh/authorized_hosts
      authorized_key:
        user: root
        state: present
        key: '{{ item }}'
      with_file:
        - ~/.ssh/id_rsa.pub

Aumento de privilegios

Por razones de seguridad y auditoría, es posible que Ansible deba conectarse a hosts remotos como un usuario sin privilegios antes de escalar los privilegios para obtener acceso administrativo como root. Esto se puede configurar en la sección [privilege_escalation] del archivo de configuración de Ansible.

Para habilitar la escalada de privilegios de forma predeterminada, establezca la directiva Become = True en el archivo de configuración. Incluso si está configurado de forma predeterminada, hay varias formas de anularlo cuando se ejecutan comandos ad hoc o Ansible Playbooks. (Por ejemplo, puede haber momentos en los que desee ejecutar una tarea o jugar que no aumente los privilegios).

La directiva Become_method especifica cómo escalar los privilegios. Hay varias opciones disponibles, pero la predeterminada es usar sudo. Del mismo modo, la directiva Become_user especifica a qué usuario escalar, pero el valor predeterminado es root.

Si el mecanismo de Become_method elegido requiere que el usuario ingrese una contraseña para escalar privilegios, puede configurar la directiva Become_ask_pass = true en el archivo de configuración.


Para algunas distribuciónes de redhat como rhel7 es necesario importar los permisos de en el archivos etc/passwd

cuidar mucho esté preconfigurado en ansible 

[defaults]
inventory = ./inventory
remote_user = someuser
ask_pass = false

[privilege_escalation]
become = true
become_method = sudo
become_user = root
become_ask_pass = false

Nota

También puede usar variables de grupo para cambiar el tipo de conexión para un grupo de host completo. Esto se puede hacer colocando archivos con el mismo nombre que el grupo en un directorio group_vars y asegurándose de que esos archivos contengan configuraciones para las variables de conexión.

Por ejemplo, es posible que desee que todos sus hosts administrados de Microsoft Windows utilicen el protocolo winrm y el puerto 5986 para las conexiones. Para configurar esto, puede colocar todos esos hosts administrados en ventanas de grupo y luego crear un archivo llamado group_vars/windows que contenga las siguientes líneas:

ansible_connection: winrm
ansible_port: 5986


---------------------------------APARTADO-------------------------

Para mandar ping o comandos para los distindo shost de nuestro inventario existen dos formas de logralo la primera consiste en hacer en el configurable que se apunte a na ruta con nuestro inentario de host y grupos en formato YAML
para que podamos hacer referencia a este archivo o a un grupo dado de anta en nuestro configurable /etc/ansible/ansible.cfg en la parte para defaults

la configuración  por defecto puede existir en un directorio local definido por nosotros mismos creando un ansible.conf para saber que segmentación de entradas se tienen para le momento de desplegar 

primero creamos nuestro configurable
luego cramos nuestro inventario con la segmentación de red y los host destinos para tenerlos agrupados para poder tirar comandos a hacia los destinos:
386  ansible all -m ping

  $  vi inventory
  $  ansible all -m ping
  $  ansible localhost --list-host
  $  ansible TSR -m ping
  $  ansible TSS -m ping


para hacer un playbook completo de esto ultimo tenemos que crear igualmente un archivo tipo inventory.ini cona el mismo contenido y un YAML con el siguiente contenido al igual que el directorio tal como:

[TSR]
192.168.1.[30:33] ansible_user=root ansible_password=Temporal123$

[TSS]
192.168.1.27 ansible_user=r ansible_password=T
192.168.1.17 ansible_user=r ansible_password=T

"notese que colocamos en este caso los usuarios y contraseñas para que podamos logearnos en los destinos pero despendiendo de la operación que se de esto puede ser u no necesario "

y para el YAML que va ejecutar un ping este seria el codigo:

---
- name: Enviar ping a los hosts
  hosts: TSR
  tasks:
    - name: Enviar ping
      ping:

$ansible TSR -i inventory.ini -u root -m command -a "ping -c 4 localhost"


y pero tienene en este caso que terner bien configurado el archivo raíz de configuración de ansible que se aloja en /etc/ansible/ansible.cfg

en general si se usamos el archivo configurable de accesos debe tener todo lo necesario para que el comando de ansible se logre se deben tener bien mapeados los host y el configurable debe aputan correctamente a los destinos para que pida correctamente las claves acialos destinos 

coamndo general:

ansible -m ping <Grupo, usuario ó Acumulados> -i <inventarioFILE>

------------------------- Comandos Ad Hoc ---------------------------------

Primero veamos que el manual sobre algun comando sobre ansible se abre con 
$ ansible-doc COMAND 

Documentación para uso de Ansible:
https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html

parametro spreconfigurables si no tenemos a la mano un preconfigurable bien implementado tenemos a la mano:

inventory 	         |        -i
remote_user 	       |        -u
become 	--become,    |        -b
become_method 	     |        --become-method
become_user          |	      --become-user
become_ask_pass      |        --ask-become-pass, -K

$ ansible --help

resources:
https://docs.ansible.com/ansible/2.9/user_guide/intro_patterns.html  ---> Trabajo con colaboradoes sobre los grupos 
https://docs.ansible.com/ansible/2.9/user_guide/intro_adhoc.html     --->
Guía de comandos Ad Hock

https://docs.ansible.com/ansible/2.9/modules/command_module.html     ---> 
Guía de comandos para ejecuaciónes remotas 

Comando Ad Hoc para copiar archivos con la siguiente sintaxis siempre reordemos que tenemos uqe estar sobre el mismo path donde se encuentra el confgurable 

PROMT
$ ansible localhost -m copy -a 'content="Managed by Ansible\n" dest=/etc/motd' -u devops --become

$ ansible all  -m  command  -a  'cat /etc/motd'  -u  devops
 ------------  --  -------  --  ---------------  --  ------
       1        2     3      4        5           6     7  

1.- indica que desplieges sobre multiples host 
2.- ejecuta como comando
3.- indica que tipo de ejecución tendra
4.- indica que va una cadena 
5.- indica que tipo usuario del sistema va a ejecutar este sistema 
-----------------------YAML--------------------------------------
  tasks:
    - name: newbie exists with UID 4000       | nombre para registro de ejecución
      user:
        name: newbie                          | lo que hace es lo mismo que el 
        uid: 4000                             | comando completo pero con mejor 
        state: present                        | sintaxis facil de entender

---------------------------------------------------------------------------------
  tasks:				| Habilitamos el servicio apache
    - name: web server is enabled
      service:
        name: httpd
        enabled: true

    - name: NTP server is enabled	| Habilitamos la sincronización horaria 
      service:
        name: chronyd
        enabled: true

    - name: Postfix is enabled		| Servicio de correo con playbook
      service:
        name: postfix
        enabled: true
-------------------------------------------------------------------------
Ejecución palybooks

sintaxis básica: $ ansible-playbook site.yml webserver.yml
---out-----
$ ansible-playbook webserver.yml

   PLAY [play to setup web server]
   ************************************************

    TASK [Gathering Facts]
    *********************************************************
    ok: [servera.lab.example.com]

    TASK [latest httpd version installed]
    ******************************************
    changed: [servera.lab.example.com]

    PLAY RECAP
    *********************************************************************
    servera.lab.example.com    : ok=2    changed=1    unreachable=0
    failed=0

---------------------------------------------------------------------
Tabla 2.5. Configuración de detalles de la salida de la ejecución de la guía

Opción	Descripción
-v	    | Se muestran los resultados de la tarea.
        |
-vv	    | Se muestran tanto los resultados de la tarea como la
        | configuración de la tarea.
        |
-vvv	  | Incluye información acerca de las conexiones a los hosts 
        | administrados.
        |
-vvvv	  | Agrega opciones de detalle adicionales a los complementos
        | (plug-ins) de conexión, incluidos los usuarios que se usan en
        | los hosts administrados para ejecutar scripts, y los scripts
        | que fueron ejecutados.

-------------------------------------------------------------------------


ahora que hemos visto como se crean los playbook vemos si su sintaxis es correcta con el siguiente comando:
ansible-playbook --syntax-check FILE.yml

algunos parametros que metemos con .yml y YAML permiten la elevación de permisos para con los usuarios como lo son:

remote_user: remoteuser
become: true
become_method: sudo
becomes_user: privileged_user

En general los playbooks es una  forma de tirar comandos sin saber nada de administración de linux la forma de consultar el manual al momento de ejecutar ansible-doc para los modulos

pero generalmente al presentarce errores con dependencias con o con las versiónes de los comandos dados de alta en el repositorio de RedHat 

Use el comando ansible-doc para buscar módulos y aprender a usarlos para sus tareas.

De ser posible, intente evitar los módulos command, shell y raw en las guías, por más simple que parezca su uso. Debido a que estos adoptan comandos arbitrarios, es muy fácil programar guías no idempotentes con estos módulos.

Por ejemplo, la siguiente tarea que usa el módulo shell no es idempotente. Cada vez que se ejecuta la reproducción, reescribe /etc/resolv.conf aunque ya conste de la línea nameserver 192.0.2.1.

- name: Non-idempotent approach with shell module
  shell: echo "nameserver 192.0.2.1" > /etc/resolv.conf
Hay varias maneras para escribir tareas que usen el módulo shell de manera idempotente, y algunas veces, realizar estas modificaciones y usar shell es el mejor enfoque. Una solución más rápida puede ser usar ansible-doc para descubrir el módulo copy y usarlo para obtener el efecto deseado.

El siguiente ejemplo no reescribe el archivo /etc/resolv.conf si ya consta del contenido correcto:

- name: Idempotent approach with copy module
  copy:
    dest: /etc/resolv.conf
    content: "nameserver 192.0.2.1\n"
El módulo copy comprueba si ya se alcanzó el estado y, de ser así, no realiza cambios. El módulo shell permite mucha flexibilidad, pero también requiere de más atención para garantizar que se ejecute de manera idempotente.

Las guías idempotentes se pueden ejecutar de manera repetida para garantizar que los sistemas se encuentren en un estado particular sin interrumpir esos sistemas si ya lo están.


comentarios en YAML formato de diccionarios:

{name: svcrole, svcservice: httpd, svcport: 80}

comentarios para texto en los archivos:
this is a string
'this is another string'
"this is yet another a string"

sangrado en los comandos:
  name: svcrole
  svcservice: httpd
  svcport: 80

listas:

hosts: [servera, serverb, serverc]

Servicios de host en particular:
  tasks:
    - name: normal form
      service:
        name: httpd
        enabled: true
        state: started
IdentacIón para que los playbooks sean leeidos 

Ejemplos de actualización, servicios y cortafuegos:
    - name: latest version of all required packages installed
        yum:
        name:
          - firewalld
          - httpd
          - mariadb-server
          - php
          - php-mysqlnd
        state: latest
    - name: firewalld enabled and running
      service:
        name: firewalld
        enabled: true
        state: started

    - name: firewalld permits http service
      firewalld:
        service: http
        permanent: true
        state: enabled
        immediate: yes

agregamos tareas necesarias para permitir el uso de instancias de httpd y mariadb

    - name: httpd enabled and running
      service:
        name: httpd
        enabled: true
        state: started

    - name: mariadb enabled and running
      service:
        name: mariadb
        enabled: true
        state: started


Instancia de pruebas en para sitio web Apache
Agregue las entradas necesarias para definir la tarea final a fin de generar contenido web para realizar pruebas. Use el módulo get_url para copiar http://materials.example.com/labs/playbook-review/index.php en /var/www/html/ en el host administrado.



- name: test php page is installed
      get_url:
        url: "http://materials.example.com/labs/playbook-review/index.php"
        dest: /var/www/html/index.php
        mode: 0644

---------------------------Gestión de Variables----------------------------

Los tipos de variables pueden ser archivos, directorios, sesiónes en sitios remoto y llaves incriptadas

Aquí hay una lista simplificada de formas de definir una variable, ordenada de menor a mayor precedencia:

    Variables de grupo definidas en el inventario.

    Agrupar variables definidas en archivos en un subdirectorio group_vars en el mismo directorio que el inventario o la guía de reproducción.

    Variables de host definidas en el inventario.

    Variables de host definidas en archivos en un subdirectorio host_vars en el mismo directorio que el inventario o la guía de reproducción.

    Datos del host, descubiertos en tiempo de ejecución.

    Reproducir variables en la guía de reproducción (vars y vars_files).

    Variables de tareas.

    Variables adicionales definidas en la línea de comandos.


Las variables como los suario pueden definirce en los mismos archivos de inventario para que al ser tipeados sobre los archivos .yml exitan y esten activas 

para definir varias variables como matrices de variables asignadas a las variables en el inventario 
Por ejemplo:

users:
  bjones:
    first_name: Bob
    last_name: Jones
    home_dir: /users/bjones
  acook:
    first_name: Anne
    last_name: Cook
    home_dir: /users/acook


Referenciado como objeto deacuerdo a su uso

# Returns 'Bob'
users.bjones.first_name

# Returns '/users/acook'
users.acook.home_dir

Debido a que la variable se define como un diccionario Python, hay disponible una sintaxis alternativa.

# Returns 'Bob'
users['bjones']['first_name']

# Returns '/users/acook'
users['acook']['home_dir']

Siempre tipear guion bajo antes de declarar las variables Ejemplo de variables declaradas sobre el miso playbook 
---------------------------------------------------------------------
---
- name: Deploy and start Apache HTTPD service
  hosts: webserver
  vars:
    web_pkg: httpd
    firewall_pkg: firewalld
    web_service: httpd
    firewall_service: firewalld
    python_pkg: python3-PyMySQL
    rule: http

  tasks:
    - name: Required packages are installed and up to date
      yum:
        name:
          - "{{ web_pkg  }}"
          - "{{ firewall_pkg }}"
          - "{{ python_pkg }}"
        state: latest

    - name: The {{ firewall_service }} service is started and enabled
      service:
        name: "{{ firewall_service }}"
        enabled: true
        state: started

    - name: The {{ web_service }} service is started and enabled
      service:
        name: "{{ web_service }}"
        enabled: true
        state: started

    - name: Web content is in place
      copy:
        content: "Example web content"
        dest: /var/www/html/index.html

    - name: The firewall port for {{ rule }} is open
      firewalld:
        service: "{{ rule }}"
        permanent: true
        immediate: true
        state: enabled

- name: Verify the Apache service
  hosts: localhost
  become: false
  tasks:
    - name: Ensure the webserver is reachable
      uri:
        url: http://servera.lab.example.com
        status_code: 200
----------------------------------------------------------------------------

gestión de contraseñas cifradas con RedHat Ansible Voult por ello primero se explican como usar los tipos de variable para los archivos tipo Vault que esta cifradas con una contraseña 

Creamos un secret.yml sobre el cual e encuentran nuestras credenciales tales como:
username: ansibleuser1
pwhash: password

para abrir en esdición este archivo usamos el comando:

$ ansible-voult edit secret.yml 

el cual nos pedira una contraseña cada que nosotros desemos usarlo 
importar contenido a variables que solo existen en el sistema:

 
---
- name: create user accounts for all our servers
  hosts: devservers
  become: True
  remote_user: devops
  vars_files:
    - secret.yml
  tasks:
    - name: Creating user from secret.yml
      user:
        name: "{{ username }}"
        password: "{{ pwhash }}"

codificamos para que reetiquete la instancia
 para comprobar correcta migración de credenciales usamos:

$ ansible-playbook --syntax-check --ask-vault-pass create_users.yml

con lo que nos pedira una contraseña de desincripción de código 
En lugar de usar --ask-vault-pass, puede usar la opción --vault-id @prompt más nueva para hacer lo mismo.


antes de crear un archivo vault tenemos que crear un archivo desencriptador
se recomienda solo usar control de salidas para aumentar la seguridad en el proceso 

$echo 'redhat' > vault-pass
$chmod 0600 vault-pass

para ahora usar solamente el siguiente comando para compilar el playbook 


$ ansible-playbook --vault-password-file=vault-pass create_users.yml


para almacenar grandes cantidades de datos en variables que funcionen con los playbooks tendriamos entonces que o usarlas como listas de datos o bien usar nuevamente archivos .yml estilo JSON

tipos de variables en el contexto de sistemas

Uso de variables mágicas
Algunas variables no son datos o no están configuradas a través del módulo setup, pero también se establecen de forma automática por Ansible. Estas variables mágicas pueden ser útiles para obtener información específica de un host administrado particular.

A continuación, se detallan cuatro de las más útiles:

hostvars
Contiene las variables para hosts administrados y puede usarse para obtener los valores para las variables de otro host administrado. No incluye los datos del host administrado si no se recopilaron aún para ese host.

group_names
Enumera todos los grupos en los que se encuentra el host administrado actual.

groups
Enumera todos los grupos y hosts en el inventario.

inventory_hostname
Contiene el nombre de host para el host administrado actual, según se configuró en el inventario. Este puede ser diferente del nombre de host informado por los datos por diversos motivos.

Ejemplo de todos los hostvars en un archivo tipo JSON en un localhost

localhost | SUCCESS => {
    "hostvars[\"localhost\"]": {
        "ansible_check_mode": false,
        "ansible_connection": "local",
        "ansible_diff_mode": false,
        "ansible_facts": {},
        "ansible_forks": 5,
        "ansible_inventory_sources": [
            "/home/student/demo/inventory"
        ],
        "ansible_playbook_python": "/usr/bin/python2",
        "ansible_python_interpreter": "/usr/bin/python2",
        "ansible_verbosity": 0,
        "ansible_version": {
            "full": "2.7.0",
            "major": 2,
            "minor": 7,
            "revision": 0,
            "string": "2.7.0"
        },
        "group_names": [],
        "groups": {
            "all": [
               "serverb.lab.example.com"
            ],
            "ungrouped": [],
            "webservers": [
                "serverb.lab.example.com"
            ]
        },
        "inventory_hostname": "localhost",
        "inventory_hostname_short": "localhost",
        "omit": "__omit_place_holder__18d132963728b2cbf7143dd49dc4bf5745fe5ec3",
        "playbook_dir": "/home/student/demo"
    }
}

Estos aunque parecienran complejas tareas para simplificar variables realmente cubren el referenciado que pueden llegar a tener distintos sistemas de linux abarcando en medida de lo posible exclusiones que puede 
automatizar

Para estandarizar los directorios usamos un directorio por ejemplo 

facts.d 
para que en el guardemos 
un archivo  .fact


Ambitos y temás de programación en ANSIBLE
Ansible tiene su propia sintaxis para hacer loop y secuencias condicionales para administrar el sistema linux entonces entramos programación orientada a objetos 
 por ejemplo la sentencia loop para emular tareas iterativas como con un for
when para condicionales pero para aspectos con servicios activos o con condicionales o con when


----------------------------------------- Manejadores | Handlers--------------------------------------- 
Manejadores de Ansible
Los módulos Ansible están diseñados para ser idempotentes. Esto implica que en una guía programada adecuadamente, la guía y sus tareas se pueden ejecutar múltiples veces sin cambiar el host administrado, a menos que necesiten realizar un cambio para llevar al host administrado a su estado deseado.

No obstante, algunas veces, cuando una tarea realiza un cambio en el sistema, se puede tener que ejecutar otra tarea. Por ejemplo, un cambio en un archivo de configuración del servicio puede luego requerir que el servicio se vuelva a cargar para que se aplique la configuración modificada.

Los manejadores son tareas que responden a una notificación desencadenada por otras tareas. Las tareas solo notifican a sus manejadores cuando la tarea cambia algo en un host administrado. Cada controlador (handler) se activa por su nombre después del bloque de tareas de la guía. Si ninguna tarea notifica al manejador por nombre, no se ejecutará el manejador. Si una o más tareas notifican al manejador, el manejador se ejecutará exactamente una vez después de que todas las demás tareas en la reproducción se hayan completado. Debido a que los manejadores son tareas, los administradores pueden usar los mismos módulos en manejadores que utilizarían para cualquier otra tarea. Generalmente, los manejadores se utilizan para reiniciar los hosts y los servicios.

Un hndler es un programador interno de tareas que a grande rasgos es un invocador de funciones pero en este caso es un invocador de 
tareas en la guía de ejecución.

Los manejadores se pueden considerar como tareas inactive que solo se desencadenan cuando se las invoca explícitamente mediante una declaración notify. El siguiente fragmento muestra cómo el servidor Apache solo se reinicia mediante el manejador restart_apache cuando un archivo de configuración se actualiza y lo notifica:

tasks:
  - name: copy demo.example.conf configuration template1
    template:
      src: /var/lib/templates/demo.example.conf.template
      dest: /etc/httpd/conf.d/demo.example.conf
    notify:2
      - restart apache3

handlers:4
  - name: restart apache5
    service:6
      name: httpd
      state: restarted


1 La tarea que notifica al manejador.

2 La declaración notify indica que la tarea debe activar un manejador.

3 El nombre del manejador que debe ejecutarse.

4 La palabra clave handlers indica el inicio de la lista de tareas del manejador.

5 El nombre del manejador invocado por las tareas.

6 El módulo que usará el manejador.

Es para mandar cambios a producción en ambientes de TI

Manejo de fallas de tareas
Objetivos
Después de completar esta sección, debe ser capaz de controlar lo que sucede cuando falla una tarea y qué condiciones hacen que esta falle.

Gestión de errores de tareas en reproducciones
Ansible evalúa los códigos de retorno de cada tarea para determinar si una tarea se realizó satisfactoriamente o con errores. Generalmente, cuando una tarea falla, Ansible cancela inmediatamente el resto de la reproducción en ese host y omite todas las tareas posteriores.

----------------------------------------------Try catch de Ansible-----------------------------------

Modulo de ignore errors al final de cada mododulo de ejecuación con un valor de "yes" En caso de que esta tarea no funcione contiua con el proceso

Forzado de tareas al principio de todo el playbook para que aunque una tarea falle el resto continue ejecutando 
- force_handlers: yes

  Especificación de las condiciones de falla de tareas
  Puede usar la palabra clave failed_when en una tarea para especificar las condiciones que indican que la tarea falló. A menudo,
  esto se usa con los módulos de comando que pueden ejecutar un comando correctamente, pero la salida del comando indica una falla.

Por ejemplo, puede ejecutar un script que presenta un mensaje de error y usar ese mensaje para definir el estado fallido para la tarea. El siguiente fragmento muestra cómo la palabra clave failed_when se puede utilizar en una tarea:

tasks:
  - name: Run user creation script
    shell: /usr/local/bin/create_users.sh
    register: command_result
    failed_when: "'Password missing' in command_result.stdout"
   El módulo fail también se puede utilizar para forzar una falla de tarea. El escenario anterior se puede escribir alternativamente como dos tareas:

tasks:
  - name: Run user creation script
    shell: /usr/local/bin/create_users.sh
    register: command_result
    ignore_errors: yes

  - name: Report script failure
    fail:
      msg: "The password is missing in the output"
    when: "'Password missing' in command_result.stdout"


Las guías de ejecución tienen su propio configurable y ejecutable 

tienen valores por defecto que seria:
  debug: 
     var: command_result.stdout

para cambios en caso de o para otros playbook como lo requieran
  changed_when: false que es instancia que se coloca cuando queremos reusar la varible

para cambios en caso de que falle alguna configuración 

En este paso, configurará una palabra clave block para que pueda experimentar con su funcionamiento.

------------------------------------------------------------------------------------------------------------------------------------

Actualice la guía anidando la primera tarea en una cláusula block. Elimine la línea que establece ignore_errors: yes. El bloque debe decir lo siguiente:

    - name: Attempt to set up a webserver
      block:
        - name: Install {{ web_package }} package
          yum:
            name: "{{ web_package }}"
            state: present
Anide la tarea que instala el paquete mariadb-server en una cláusula rescue. La tarea se ejecutará si falla la tarea que se detalla en la cláusula block. La cláusula block debe decir lo siguiente:

      rescue:
        - name: Install {{ db_package }} package
          yum:
            name: "{{ db_package }}"
            state: present
Por último, agregue una cláusula always que iniciará el servidor de base de datos una vez realizada la instalación usando el módulo service. La cláusula debe decir lo siguiente:

      always:
        - name: Start {{ db_service }} service
          service:
            name: "{{ db_service }}"
            state: started
La sección de la tarea completa debe decir lo siguiente:

  tasks:
    - name: Attempt to set up a webserver
      block:
        - name: Install {{ web_package }} package
          yum:
            name: "{{ web_package }}"
            state: present
      rescue:
        - name: Install {{ db_package }} package
          yum:
            name: "{{ db_package }}"
            state: present
      always:
        - name: Start {{ db_service }} service
          service:
            name: "{{ db_service }}"
            state: started

------ Algunos jercios de indicaciónes en las guias de trabajo---------

Asegúrese de proporcionar un nombre adecuado para la tarea. Esta tarea solo debe ejecutarse cuando el sistema remoto no cumple con los requisitos mínimos.

Los requisitos mínimos para el host remoto se enumeran a continuación:

Tiene al menos la cantidad de RAM especificada por la variable min_ram_mb. La variable min_ram_mb se define en el archivo vars.yml y tiene un valor de 256.

Ejecuta Red Hat Enterprise Linux.

La tarea completada coincide:

  tasks:
    #Fail Fast Message
    - name: Show Failed System Requirements Message
      fail:
        msg: "The {{ inventory_hostname }} did not meet minimum reqs."
      when: >
        ansible_memtotal_mb < min_ram_mb or
        ansible_distribution != "RedHat"


"Esta intrucción se encarga de compara requisitos de hardware en el host detino antes de comenzar a configurar como lo es la memoria Rma y la Distribucion de linux que tiene el sistema "

------------------------------------------------------------------------------------------------------------------------------------
Agregue una sola tarea a la guía bajo el comentario #Enable and start services para iniciar todos los servicios. Deben iniciarse y habilitarse todos los servicios especificados por la variable services que se define en el archivo vars.yml. Asegúrese de proporcionar un nombre adecuado para la tarea.

La tarea completada coincide:

    #Enable and start services
    - name: Ensure services are started and enabled
      service:
        name: "{{ item }}"
        state: started
        enabled: yes
      loop: "{{ services }}"

"Esta es una sentencia loop que permite habilitar de forma iteratica los servicios del sistema todos aquellos a los que tenga acceso y permisos el usuario"

-------------------------------------------------------------------------------------------------------------------------------------
En este capítulo, aprendió lo siguiente:

Los bucles se usan para iterar un conjunto de valores, por ejemplo, una simple lista de cadenas o una lista de hashes o diccionarios.

Los condicionales se pueden usar para ejecutar tareas o reproducciones únicamente cuando se hayan cumplido ciertas condiciones.

Los manejadores son tareas especiales que se ejecutan al final de una reproducción si otras tareas los notifican.

Los manejadores solo son notificados cuando una tarea informa que se cambió algo en un host administrado.

Las tareas se configuran para manejar condiciones de error ignorando una falla en las tareas, forzando a los manejadores a ser invocados aun cuando la tarea haya fallado, marcar una tarea como con errores cuando fue satisfactoria o reemplazar el comportamiento que hace que la tarea se marque como modificada.

Los bloques se usan para agrupar tareas como una unidad y ejecutar otras tareas dependiendo de si todas las tareas en el bloque se realizaron satisfactoriamente.
---------------------------------------------------------------------------------------------------------------------------
Completos para importar archivos a los host del inventario 

 Módulos de archivos de uso común

Nombre del módulo	Descripción del módulo
blockinfile	Inserte, actualice o elimine un bloque de texto multilínea rodeado por líneas de marcador personalizables.
copy	        Copie un archivo de la máquina local o remota a una ubicación en un host administrado. Similar al módulo file, el
                módulo copy también puede establecer atributos de archivo, incluido el contexto SELinux.
fetch	      Este módulo funciona como el módulo copy, pero a la inversa. Este módulo se utiliza para recuperar archivos de máquinas
              remotas al nodo de control y para almacenarlos en un árbol de archivos, organizado por nombre de host.
file	     Establezca atributos tales como permisos, propiedad, contextos SELinux y marcas de tiempo de archivos regulares, enlaces
             simbólicos, enlaces físicos y directorios. Este módulo también puede crear o eliminar archivos regulares, enlaces
             simbólicos, enlaces físicos y directorios. Una serie de otros módulos relacionados con archivos admiten las mismas
             opciones para establecer atributos como el módulo file, incluido el módulo copy.
lineinfile	Asegúrese de que una línea particular esté en un archivo o reemplace una línea existente usando una expresión regular
                de referencia inversa. Este módulo es principalmente útil cuando desea cambiar una sola línea en un archivo.
stat	        Recupere información de estado de un archivo, similar al comando stat de Linux.
synchronize	Un contenedor alrededor del comando rsync para hacer que las tareas comunes sean rápidas y fáciles. El módulo 
                synchronize no está destinado a proporcionar acceso a toda la potencia del comando rsync, pero hace que la
                invocaciones más comunes sean más fáciles de implementar. Es posible que aún necesite generar el comando rsync 
                directamente a través del módulo run command según su caso de uso.

Administrador de archivos usando ansible desde sus permisos, usuarios y grupos a los que pertenece la administración de archivos en uso completo en redhat linux en todos los nodos administrados 

------------------------Práctica de conección con VCenter----------------------

Lo probles pricipales que no permetian usar el modulo de community.vmware registrado en ansible galaxy el link con toda la documentación para implementar operaciónes automatizadas con ansible los primero problams que tenia era que no detectaban los modulos de ejecución para el playbook por tanto resolver estos problemas tomo demaciado tiempor pero a grandes rasgos 

Primero no aseguramos que tengamos versiónes de python soblre las que tenemos que trabajar es decir tener actualizada nuestra versión de python antes de instalar el core de Ansible a la versión 3.11 para ansible 2.14  para ello usamos 
!Con permisos de administrador¡

Upgrades ansible y python
3.6.4 a 3.11.2 --> #dnf upgrade python
2.13 a 2.14 -->    #dnf upgrade ansible

Luego para poder trabajar con el modulo de administración de vcenter homologamos nuestro entorno a el repository de galaxy 
# ansible-galaxy collection install community.vmware
para luego importar las librerias del repositorio de redhta los modulos necesarios para la administración
pero primero instalamos la correcta libreria de pip en lugar de pip3 para que nuestro entorno importe las librerias de la variable correcta a la que sta unida el interprete de python.


#  ansible_python_interpreter: /usr/bin/python3.11
#  python3.11 --version
#  python3.11 -c "import requests"
#  python3.11 -m pip install requests
#  wget https://bootstrap.pypa.io/get-pip.py
#  python3.11 get-pip.py
#  python3.11 -m pip install requests
#  python3.11 -m pip install PyVmomi


Con ello el ambiente de ejecución con vmware que corriendo y en funcionamiento para desliegue a acontinuación muestro el playbook que se lanzo para a fin de validar la comunicación con host de vcenter

Bien primero antes de implementar el playbook en mi hst d rhel se crea el ambiente para que esto sea ideal para ello proximamente veremos configuraciónes ideales en los sistemas destino playbook junto con el troubleshooting:

---------------------------------------------------------------------------------------
                                  VCenter module Ansible


---
- name: Crear VM con ISO en vCenter
  hosts: localhost
  vars:
          vcenter_hostname: 192.168.1.137 # La Ip sobre donde esta VCenter
          vcenter_username: administrator@guzdan.local.net #Usuario
          vcenter_password: Temporal123$ #Contraseña
         #vcenter_datastore: [datastore1 (1)] # no funciona por temas de
                                              # case sensitive
          vcenter_datacentername: Datacenter  # No es muy ambiguo como parece
          vcenter_networks: "VM Networks"     # Tenemos que mejorar está variable que no son detectables
          esxi_hostname: 192.168.1.130        # Hostname de IP para el ESXi host para el que va el 
                                              # el lanzamiento del playbook 
  gather_facts: False #Este parametro lo usamos para validar los cerificados de accesos VCenter 
  tasks:
     - name: Crear VM en Vcenter host
       vmware_guest:
          datacenter: "{{ vcenter_datacentername}}" #|
          hostname: "{{ vcenter_hostname }}"        #|
          username: "{{ vcenter_username }}"        #|
          password: "{{ vcenter_password }}"        #| Implementar el contenido de variables en un archivo
          validate_certs: no     # Algunos certificados son validados
          folder: /              # el folder donde se acentara la máquina virtual 
          name: test_vm_prueba   # El nombre de la máuina virtual que se creara 
          state: powered-off     # el estado inicial de la máquina virtual
          guest_id: rhel8_64Guest # la imagen base sobre donde el que se basara la máquina virtual 
          esxi_hostname: "{{ esxi_hostname }}"  # el host donde se bascaran los cambios de implementación 
          disk:                         # Disco local que tendra la máquina virtual
             - size_gb: 15
               type: thin
               datastore: Datastore2
          cdrom:                        # Almacenamiento donde se guarda la imagen de la máquina virtual 
             - type: iso
               iso_path: "[datastore1 (1)] rhel-8.5-x86_64-boot.iso"
               controller_number: 1  # Número de controlador
               unit_number: 0
               state: present
          hardware:                     # Alamcenamiento y componentes de motherboard
             memory_mb: 8192
             num_cpus: 4
          wait_for_ip_address: no       # No esperar por una autoasignación de ip por parte del host esxi
       delegate_to: localhost

----------------------------------------------------------------------------------------------------------

Comando para ver la infraestructura de del host destino 
 $ ansible hostname -m setup
Modulo para configuración de " temaplate "
Para ver que onda con los archivos usamos el modulo "stat"

Comando complejo para listar los hosts dentro de un inventario 
$ ansible 'prod,172*,*lab*' -i inventory1 --list-hosts

Se pueden importar playbooks en los hosts destino con "import_playbook" y este modulo se puede importar sobre un playbook como una función heredada y támbien permite importar tareas "import_tasks" o "include_tasks" llama a las funciónes no se tiene mucha visibilidad sobre el proceso master 

Si los desarrolladores, administradores de sistema y administradores de bases de datos gestionan los servidores de manera colectiva, cada organización puede programar su propio archivo de tareas que el administrador de sistemas luego puede revisar e integrar.

Si un servidor requiere una configuración en particular, se la puede integrar como un conjunto de tareas ejecutado en función de un condicional. En otras palabras, las tareas se incluyen solo si se cumplen criterios específicos.

Si un grupo de servidores tiene que ejecutar una tarea o un conjunto de tareas en particular, las tareas solo se pueden ejecutar en un servidor si este es parte de un grupo específico de hosts.


Arreglo de errores en Ansible

Redhat recoienda hacer una variable de sistema en los que se guarden los archivos de registro delas operaciónes  $ANSIBLE_LOG_PATH y la ruta es hacia /var/log.

El modulo de administraciónes debug para controlar lo movimientos en los sistemas

Primeros filtros de error en ansible es checando la sintaxys de un playbook
$ ansible-playbook playbook.yml --syntax-check

la bandera step permite ver que las tareas puedan consultarce 
$ ansible-playbook playbook.yml --step

Existe una bandera que nos permite ver uma tarea en especifico para ver quepudiere salir mal
#ansible-playbook playbook.yml --start-at-task="start httpd service"

Las banderas para depuración es:

-v      Aparecen los datos del resultado
-vv     Aparecen tanto los datos de entrada como los de salida
-vvv    Incluye información acerca de las conexiónes de los hosts  
        administrados
-vvvv   Incluye información adicional, como los scripts que se ejecutan en 
        cada host remoto, y el usuario que ejecuta cada script.

control de versiones ha tres cambios del archivo 
#ansible-playbook --check --diff playbook.yml

Modulos de administración para volumenens
El módulo lvol crea volúmenes lógicos, y admite el cambio de tamaño y la reducción de esos volúmenes, y de los sistemas de archivos que están encima de ellos. Este módulo también admite la creación de instantáneas para los volúmenes lógicos. La tabla siguiente enumera algunos de los parámetros para el módulo lvol.

Nombre del parámetro	Descripción
lv	   El nombre del volumen lógico.
resizefs   Redimensiona el sistema de archivos con el volumen lógico.
shrink	   Habilita la reducción de volúmenes lógicos.
size	   El tamaño del volumen lógico.
snapshot   El nombre de la instantánea del volumen lógico.
state	   Crea o elimina el volumen lógico.
vg	   El grupo de volúmenes principal para el volumen lógico.



LISTO!
----------------------------------------------------------------------------------------------------------
Apéndice B. Licencias de Ansible Lightbulb
Licencia de Ansible Lightbulb
Algunas partes de este curso se adaptaron del proyecto Ansible Lightbulb. El material original de ese proyecto está disponible en https://github.com/ansible/lightbulb con la siguiente Licencia de MIT:

Copyright 2017 Red Hat, Inc.

El aviso de copyright anterior y este aviso de permiso deberían incluirse en todas las copias o partes sustanciales del software.

Por el presente, se otorga permiso gratuito a cualquier persona que obtenga una copia de este software y los archivos de documentación asociados (el "Software") para trabajar en el software sin restricciones, que incluyen, entre otras, el derecho de usar, copiar, modificar, fusionar, publicar, distribuir, sublicenciar o vender copias del software, y permitir a personas a quienes se entrega el software para esto, sujeto a las siguiente condiciones:

El aviso de copyright anterior y este aviso de permiso deberían incluirse en todas las copias o partes sustanciales del software.

EL SOFTWARE SE PROPORCIONA "EN EL ESTADO EN EL QUE SE ENCUENTRA", SIN GARANTÍA DE NINGÚN TIPO, NI EXPRESA NI IMPLÍCITA, INCLUIDAS, ENTRE OTRAS, LAS GARANTÍAS DE COMERCIALIZACIÓN O CONVENIENCIA PARA UN PROPÓSITO PARTICULAR Y DE NO INFRACCIÓN. EN NINGÚN CASO, LOS AUTORES O LOS TITULARES DEL COPYRIGHT SERÁN RESPONSABLES POR RECLAMOS, DAÑOS U OTRA RESPONSABILIDAD, YA SEA EN UNA ACCIÓN CONTRACTUAL, EXTRACONTRACTUAL O DE OTRO MODO, QUE SURJAN DEL SOFTWARE O DE SU USO, O DE OTRAS OPERACIONES LLEVADAS A CABO CON ÉL O EN CONEXIÓN CON ESTOS.




































