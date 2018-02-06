![](ansible/Image_001.jpg)

<https://www.ansible.com/>

_Joaquín Salvachúa (jsalvachua@dit.upm.es)_

_Pedro Verdugo (pmverdugo@dit.upm.es)_

VVVV

## Problema

Actualizar mi servidor web

* En una máquina:

```bash
$ apt-get update; apt-get install apache2
```

* En 100 máquinas:
  ...Script?
```bash
$ ssh root@maquina[1-99]-c "apt-get update; apt-get install apache2"
```

VVVV

Scripts de conﬁguración e instalación:

* **Mantenimiento**:
  * Complicados de mantener.
  * Posibilidad de máquinas heterogéneas.
* **Reusabilidad**:
  * Reescribirlos si cambian las versiones.
  * Posibilidad de introducir errores.
* **Escalabilidad**:
  * Más complejo sincronizar 100 máquinas que 10

Al rescate...
## Sistemas de Despliegue

VVVV

## Sistemas de Despliegue Populares

|**Sistema**|**Puppet**|**Chef**|**Ansible**|
|:---:|:---:|:---:|:---:|
|*Programado en*|ruby|ruby|python|
|*Arquitectura*|Agente|Agente|Serverless|
|*SW Cliente*|Sí|Sí|sshd|
|*Lenguaje*|pp|rb|yaml|
|*Artefactos*|Módulos|Cookbooks|Playbooks|
|*Unidades*|Módulo|Receta|Tarea|
|*Dependencias*|Librarian|Berkshelf, Knife, Librarian|Galaxy, Librarian|

VVVV

## Ventajas de Ansible

* No hay nodo maestro
* No hay que instalar agentes (solo por ssh).
* Configuración en YAML (en ruby en los otros) (otro lenguaje más que aprender).
* Muy facil de aprender
VVVV
## YAML vs JSON vs XML (I)

_Extensible Markup Language_

```xml
<catalogo>
    <libro>
        <autor>Raúl González Duque</autor>
        <titulo>Python para todos</titulo>
        <genero>Computación</genero>
    </libro>
</catalogo>
```
VVVV
## YAML vs JSON vs XML (II)

_JavaScript Object Notation_

```json
{
  libro: {
    autor: Raúl González Duque,
    titulo: Python para todos,
    genero: Computación
  }
}
```
VVVV
## YAML vs JSON vs XML (III)

_Yet Another Markup Language_

```yaml
---
  libro:
    autor: Raúl González Duque
    titulo: Python para todos
    genero: Computación
```

VVVV
## Arquitectura Ansible

![](ansible/Image_002.jpg)

>>>>

## Instalando Ansible

Ubuntu:
```bash
$ apt-add-repository ppa:ansible/ansible
$ apt-get update
$ apt-get install ansible
```

OSX:
```bash
brew update
brew install ansible
```
Windows:

Plugin Vagrant

https://github.com/vovimayhem/vagrant-guest_ansible
```bash
$ vagrant plugin install vagrant-guest_ansible
```

VVVV

## Instalando Ansible (II)
Python
```bash
$ pip install ansible
```


VVVV
## Instalando Ansible (III)

... o con Docker

```DockerFile
FROM ubuntu:14.04
RUN apt-get -y update && apt-get install -y sobware-properties-common
RUN echo | apt-add-repository ppa:ansible/ansible
RUN apt-get -y update && apt-get -y install ansible
## Copy over playbook
COPY moo/container.yml /tmp/
## Run playbook
RUN ansible-playbook /tmp/container.yml
```

>>>>
## Idempotencia
_"Propiedad para realizar una acción determinada varias veces y aun así conseguir el mismo resultado que se obtendría si se realizase una sola vez."_

*Stateless*

Operaciones que pueden aplicarse múltiples veces sin cambiar el resultado ﬁnal. (Por ejemplo un HTTP GET, APIs REST)

Necesario y deseable en múltiples campos de la computación–comunicación.

>>>>
## Ansible: Conceptos Básicos
- Esta escrito en Python
(solo hace falta para escribir módulos, basta yaml)
- SSH en cliente (y python, normalmente por defecto)
- Gestión de dependencias
  - Galaxy
  - Librarian

* Inventario
  * Playbooks
    * Roles
      * Tasks


VVVV

## Inventario

Grupos de Maquinas a instalar

Descripción de qué tipos de servicio tenemos (servidor web, bases de datos, etc)

```ini
[webservers]
www1.example.com
www2.example.com

[dbservers]
db0.example.com
db1.example.com
```

VVVV

## Tareas (Tasks)
Una acción específica a realizar

Por ser idempotentes especificamos sólo el estado final.

```yaml
tasks:
  - name: ensure apache is running
    service: name:httpd state: started
```
VVVV

## Playbooks (Libretos?)

_Hosts_: Servidores sobre los que trabajamos

_Vars_: Variables para tareas y templates (de más a menos especificidad)

_Tasks_: Resultados a conseguir (estado en el que queremos que quede el sistema)

_Handlers_: Gestores de eventos (se disparan con notify)

VVVV

## Ejemplo playbook

```yaml
---
- hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  remote_user: root
  tasks:
  - name: ensure apache is at the latest version
    yum:
      name: httpd
      state: latest
  - name: write the apache config file
    template:
      src: /srv/httpd.j2
      dest: /etc/httpd.conf
    notify:
    - restart apache
  - name: ensure apache is running
    service: name: httpd state: started
  handlers:
    - name: restart apache
      service:
        name: httpd
        state: restarted
```

VVVV

## Roles
* Conjuntos de funcionalidades específicas (lista de tareas)
* Importables via Galaxy/Librarian

```bash
$ ansible-galaxy install username.rolename
```


VVVV

## Ejemplos

<https://github.com/ansible/ansible-examples>

>>>>

## Problema:
Actualizar mi servidor web con Ansible

VVVV

## 1. Definir mi host
- Nombre del host
- IP y/o DNS
- Usuario
- Password (NOOOO, usar keypairs)

_inventory.ini_
```ini
[miswebservers]
odroid ansible_host=192.168.1.2 ansible_user=ubuntu ansible_password=reverse
```

VVVV

## 2. Definir Playbook
_apache.yml_
```yaml
---
- hosts: miswebservers
  tasks:
  - name: Actualizar Apache
    apt: name=apache2 state=latest
  - name: Apache corriendo
    service: name=apache2 state=restarted
```

VVVV

## 3. Correr Playbook

```bash
$ ansible-playbook apache.yml -i inventory.ini
```
VVVV
## Proceso

- Ansible recopila datos conocidos sobre la máquina (Hechos/Facts)
- Ansible crea una secuencia temporal de modificaciones en _/tmp_
- Si la ejecución de la secuencia tiene éxito, sustituye la estructura original por la temporal

>>>>
### Próximamente...
- OpenStack
- Docker
- IaC: Chef y Pupper (again)
- Keypairs



VVVV
### Posteriormente
Orquestación a alto nivel de múltiples máquinas.

Mesos
* <http://mesos.apache.org>

Kubernetes
* <http://kubernetes.io>

VVVV

### Arquitectura Kubernetes
![](ansible/Image_005.jpg)

VVVV

### Arquitectura Mesos
![](ansible/Image_006.jpg)

VVVV

### Nuevos despliegues

Unir todas las piezas de la forma más ﬂexible

_"If a Docker application is a Lego brick, Kubernetes would be like a kit for building the Millennium Falcon and the Mesos cluster would be like a whole Star Wars universe made of Legos." ~ Solomon_
