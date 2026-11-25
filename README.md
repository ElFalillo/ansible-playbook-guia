# Guía práctica de Ansible

Este repositorio contiene una guía sencilla para aprender **Ansible** mediante un playbook que instala y configura **Nginx** en una máquina virtual Debian.

## ¿Qué es Ansible?

**Ansible** es una herramienta de automatización de infraestructura. Permite ejecutar tareas repetibles en uno o varios servidores de forma remota, normalmente mediante SSH.

Con Ansible puedes automatizar, por ejemplo:

- Instalar paquetes.
- Crear usuarios.
- Configurar servicios.
- Desplegar aplicaciones web.
- Aplicar configuraciones de seguridad.
- Actualizar servidores.

Una ventaja importante es que Ansible es **idempotente**: si una tarea ya está aplicada, no la repite ni provoca cambios innecesarios.

## ¿Qué es un playbook?

Un **playbook** es un fichero escrito en YAML donde se describen las tareas que Ansible debe ejecutar en los servidores.

En este proyecto, el playbook realiza lo siguiente:

1. Actualiza la caché de paquetes APT.
2. Instala Nginx.
3. Crea una página web de prueba.
4. Arranca y habilita el servicio Nginx.

## Estructura del proyecto

```text
ansible-playbook-guia/
├── README.md
├── inventory.ini
└── instalar_nginx.yml
```

## Requisitos

Necesitas:

- Una máquina controladora con Ansible instalado.
- Una máquina virtual Debian como servidor gestionado.
- Conectividad SSH entre ambas máquinas.
- Permisos de administrador o `sudo` en la máquina gestionada.

## Instalar Ansible

En la máquina controladora:

```bash
sudo apt update
sudo apt install -y python3 python3-pip python3-venv
python3 -m venv ~/ansible-env
source ~/ansible-env/bin/activate
pip install ansible
ansible --version
```

## Configurar SSH

Genera una clave SSH en la máquina controladora:

```bash
ssh-keygen -t ed25519
```

Cópiala al servidor gestionado:

```bash
ssh-copy-id usuario@IP_DEL_SERVIDOR
```

Comprueba que puedes entrar sin introducir contraseña:

```bash
ssh usuario@IP_DEL_SERVIDOR
```

## Configurar el inventario

Edita `inventory.ini` y sustituye la IP y el usuario por los de tu servidor Debian:

```ini
[servidores_web]
192.168.1.50 ansible_user=tu_usuario
```

## Probar la conexión

```bash
ansible all -i inventory.ini -m ping
```

Si la conexión es correcta, deberías ver una respuesta similar a:

```text
192.168.1.50 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Ejecutar el playbook

Para comprobar qué cambios se aplicarían sin modificar el servidor:

```bash
ansible-playbook -i inventory.ini instalar_nginx.yml --check
```

Para ejecutar el playbook:

```bash
ansible-playbook -i inventory.ini instalar_nginx.yml
```

## Comprobar el resultado

Desde la máquina controladora:

```bash
curl http://IP_DEL_SERVIDOR
```

También puedes abrir en un navegador:

```text
http://IP_DEL_SERVIDOR
```

Deberías ver la página creada por Ansible.

## Explicación del playbook

| Parte | Función |
|---|---|
| `hosts: servidores_web` | Indica que las tareas se ejecutan en el grupo `servidores_web` del inventario. |
| `become: true` | Ejecuta las tareas con privilegios de administrador, normalmente mediante `sudo`. |
| `ansible.builtin.apt` | Gestiona paquetes en sistemas basados en Debian y Ubuntu. |
| `ansible.builtin.copy` | Crea o actualiza ficheros en el servidor remoto. |
| `ansible.builtin.service` | Gestiona servicios del sistema, como arrancar, parar o habilitar Nginx. |

## Siguiente paso

Cuando este playbook funcione, puedes ampliar el laboratorio añadiendo:

- Un role para Nginx.
- Configuración de MySQL o MariaDB.
- Reglas de firewall con UFW.
- Hardening de SSH.
- Uso de Ansible Vault para proteger contraseñas y claves.
