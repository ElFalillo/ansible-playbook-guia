# Resolución de errores comunes en Ansible

Esta guía recoge dos errores muy habituales al **volver a abrir el laboratorio al día siguiente**: el comando `ansible` desaparece y el inventario no se carga.

> Todas las IPs, usuarios y rutas de esta guía son **ficticios**. Sustitúyelos por los de tu laboratorio.

## Escenario del laboratorio

| Máquina | Rol | IP de ejemplo | Usuario de ejemplo |
|---|---|---|---|
| `ansible-server` | Controladora, donde se ejecuta Ansible | `192.0.2.10` | `ansible` |
| `vm-gestionada` | Servidor gestionado por SSH | `192.0.2.20` | `usuario1` |

Ansible solo se instala y se ejecuta en la **controladora**. La máquina gestionada solo necesita SSH y Python.

---

## Error 1: `ansible: orden no encontrada`

### Síntoma

```text
usuario1@vm-gestionada:~$ ansible all -i inventory.ini -m ping -vvvv
-bash: ansible: orden no encontrada
```

En un sistema en inglés aparecería como `ansible: command not found`.

### Causa

Ansible se instaló dentro de un **entorno virtual de Python** (`~/ansible-env`). Ese entorno solo está activo durante la sesión en la que lo activaste. Al cerrar la terminal, reiniciar la máquina o abrir una nueva sesión SSH, se desactiva y el comando deja de estar disponible.

También ocurre si ejecutas el comando en la máquina equivocada, por ejemplo en la gestionada, donde Ansible no está instalado.

### Solución

Conéctate a la controladora y activa el entorno:

```bash
ssh ansible@192.0.2.10
source ~/ansible-env/bin/activate
```

El prompt debe mostrar `(ansible-env)` al principio:

```text
(ansible-env) ansible@ansible-server:~$
```

Comprueba la instalación:

```bash
ansible --version
```

### Si el entorno no existe

```bash
sudo apt update
sudo apt install -y python3 python3-pip python3-venv
python3 -m venv ~/ansible-env
source ~/ansible-env/bin/activate
pip install ansible
ansible --version
```

### Prevención

Activa el entorno automáticamente en cada sesión:

```bash
echo 'source ~/ansible-env/bin/activate' >> ~/.bashrc
source ~/.bashrc
```

---

## Error 2: `Unable to parse inventory.ini as an inventory source`

### Síntoma

```text
(ansible-env) ansible@ansible-server:~$ ansible all -i inventory.ini -m ping
[WARNING]: Unable to parse /home/ansible/inventory.ini as an inventory source
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
```

### Causa

Ansible no ha podido leer el inventario. Las causas más habituales son:

- Estás en una carpeta distinta a la del proyecto, por ejemplo `~`, y `inventory.ini` está en `~/ansible-lab`.
- El fichero no existe o tiene otro nombre, como `inventory.ini.txt`.
- El fichero está vacío o mal escrito.
- La IP guardada es antigua porque la VM cambió de dirección tras reiniciar.

Como no se cargó ningún host, Ansible solo ve `localhost`, y el patrón `all` no coincide con él.

### Diagnóstico

Comprueba dónde estás y qué ficheros hay:

```bash
pwd
ls -la
find ~ -name "inventory.ini" 2>/dev/null
```

Si aparece en `~/ansible-lab`, entra en esa carpeta:

```bash
cd ~/ansible-lab
cat inventory.ini
```

### Formato correcto

```ini
[servidores_web]
192.0.2.20 ansible_user=usuario1
```

### Si el fichero no existe

```bash
mkdir -p ~/ansible-lab
cd ~/ansible-lab
nano inventory.ini
```

Pega el contenido anterior con tu IP y usuario reales. Guarda con `Ctrl + O`, Enter y `Ctrl + X`.

### Comprobar la IP de la gestionada

En la VM gestionada:

```bash
ip a
```

Si usas DHCP, la IP puede cambiar tras reiniciar. Para evitarlo, reserva la IP en el router o configura una dirección fija.

### Probar de nuevo

```bash
ansible all -i inventory.ini -m ping
```

Resultado esperado:

```text
192.0.2.20 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

---

## Si ahora falla SSH

Con el inventario cargado pueden aparecer errores de conexión.

| Mensaje | Causa probable | Solución |
|---|---|---|
| `Connection timed out` | IP incorrecta, VM apagada o firewall | Revisa la IP con `ip a` y prueba `nc -zv 192.0.2.20 22` |
| `Connection refused` | SSH no está arrancado | `sudo systemctl enable --now ssh` en la gestionada |
| `Permission denied (publickey)` | Clave no copiada o usuario incorrecto | `ssh-copy-id usuario1@192.0.2.20` |
| `Host key verification failed` | Cambió la clave del host o la IP la usa otra máquina | `ssh-keygen -R 192.0.2.20` y reconectar tras verificar la máquina |

Para ver más detalle:

```bash
ansible all -i inventory.ini -m ping -vvv
```

---

## Lista de comprobación al volver al laboratorio

1. Arranca las dos VMs.
2. Confirma las IPs con `ip a`.
3. Entra por SSH en la controladora.
4. Activa el entorno: `source ~/ansible-env/bin/activate`.
5. Entra en la carpeta del proyecto: `cd ~/ansible-lab`.
6. Comprueba el inventario: `cat inventory.ini`.
7. Prueba la conexión: `ansible all -i inventory.ini -m ping`.

## Privacidad

No publiques IPs reales, usuarios, claves privadas ni contraseñas en issues o repositorios públicos. Para documentación usa rangos reservados como `192.0.2.0/24`.
