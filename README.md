# home-lab-ansible-runner
 This repo builds an execution environment and push it to docker hub

## Inventario Ubuntu

El inventario está en `inventories/pro/hosts.ini`, en formato INI con grupos
entre corchetes y relaciones de grupos mediante `[grupo:children]`. Contiene ocho máquinas,
distribuidas entre los grupos `containers` y `vms`, ambos hijos de `ubuntu`:

- `containers`: siete contenedores, agrupados en `dns`, `wireguard`,
  `loadbalancer` y `unifi`.
- `vms`: únicamente `ibk.perelohome.com`, dentro del grupo funcional `ibk`.

Un playbook con `hosts: ubuntu` incluye las ocho máquinas sin duplicarlas.
Usa `hosts: containers` o `hosts: vms` para seleccionar por tipo de máquina.

Antes de usarlo, confirma las IP y los nombres que aparecían truncados en la
captura de Proxmox.

Se incluyen los balanceadores aunque su nombre contenga `.okd`. Se excluyen
el host `operator.okd`, los masters/workers de OKD, los nodos Proxmox y la
plantilla `master04.k3s.pro`.

## Sincronización desde Git en AWX

Después de hacer commit y push del inventario:

1. Configura un proyecto de AWX con este repositorio Git y la rama `main`.
   Activa **Update Revision on Launch**, con **Cache Timeout** a `0`.
2. Crea un inventario y añade una fuente (**Sources**) de tipo
   **Sourced from a Project**. Selecciona el proyecto anterior y el archivo
   `inventories/pro/hosts.ini` en **Inventory file**.
3. Activa **Update on Launch** en la fuente, con **Cache Timeout** a `0`.
   Así AWX actualizará la fuente antes de cada ejecución que use el inventario
   y comprobará la revisión del proyecto antes de importar sus hosts.
4. Si este inventario será exclusivamente gestionado por Git, activa
   **Overwrite** y **Overwrite variables** para reflejar también eliminaciones
   y cambios de variables. No añadas hosts o variables manualmente que quieras
   conservar en ese inventario.
5. Ejecuta una primera sincronización de la fuente y comprueba sus ocho hosts.
   En la plantilla del job selecciona este inventario y una credencial de tipo
   **Machine** con el usuario y la clave SSH adecuados.

Las credenciales SSH y de elevación de privilegios no se guardan en Git.
Puedes dirigir los playbooks a `ubuntu` o a un grupo funcional, y restringir
una ejecución mediante **Limit** en AWX.

Documentación: [inventarios de AWX](https://docs.ansible.com/projects/awx/en/24.6.1/userguide/inventories.html)
y [proyectos de AWX](https://docs.ansible.com/projects/awx/en/24.6.1/userguide/projects.html).
