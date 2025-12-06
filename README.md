# Repositorio para la charla de la UBUParty

Este repositorio contiene los archivos necesarios para descargar y configurar un *mediastack* completo utilizando Docker.

Este proyecto sirve como material de apoyo para la charla sobre Docker en el evento tecnológico **UBUParty**. La explicación detallada de los componentes individuales del *mediastack* se abordará en una charla posterior.

---

# Configuración de Entorno Docker con WSL2 en Windows

Esta guía proporciona los pasos necesarios para configurar un entorno de desarrollo con Docker utilizando WSL2 (Subsistema de Windows para Linux) en Windows.

## Pasos Previos

Asegúrate de que tu sistema operativo Windows está actualizado a la versión 2004 o superior.

## 1. Instalación de WSL2

WSL2 es una nueva versión de la arquitectura del Subsistema de Windows para Linux que permite ejecutar un kernel de Linux completo directamente en Windows.

Para instalar WSL2, abre una terminal de comandos y ejecuta el siguiente comando:

```bash
wsl --install Debian
```

Despues de realizar la instalación, automaticamente pedirá un usuario y una clave. Este usuario y clave no tienen por que ser lo mismos que tengas en Windows.

Una vez que hayas configurado tu usuario y contraseña, es una buena práctica actualizar todos los paquetes de tu distribución de Debian. Esto asegura que tienes las últimas actualizaciones de seguridad y software.

Ejecuta los siguientes comandos en tu terminal de Debian:
```bash
sudo apt update && sudo apt upgrade -y
```

### Opcional: Configurar Red en Modo Espejo (Mirrored)

Por defecto, WSL2 utiliza una red virtual (NAT) que aisla la máquina Linux de la red local. Para que los servicios corriendo en WSL2 (y en Docker dentro de él) sean directamente accesibles desde otros dispositivos en tu red local (LAN), puedes configurar el modo de red en `mirrored`.

1.  **Crear el fichero `.wslconfig`**:
Abre una terminal de comandos o el Explorador de Archivos y navega a tu perfil de usuario. Puedes escribir `%USERPROFILE%` en la barra de direcciones del explorador o utilizar el comando `cd %USERPROFILE%` para navegar hasta tu carpeta de usuario.
    
    Crea un fichero llamado `.wslconfig` en esa ubicación (`C:\Users\<TuUsuario>\.wslconfig`).

2.  **Añadir la configuración de red**:
Abre el fichero `.wslconfig` con un editor de texto y añade el siguiente contenido:

    ```
    [wsl2]
    networkingMode=mirrored
    ```

3.  **Reiniciar WSL2**:
Para que los cambios surtan efecto, debes reiniciar completamente WSL2. Abre PowerShell o CMD y ejecuta:
    ```
    wsl --shutdown
    ```
La próxima vez que abras tu terminal de Debian, se iniciará con la nueva configuración de red.

## 2. Instalación de Docker Engine en Debian (WSL2)

Todos los comandos se realizaran dentro de la distribucion Linux instalada (Debian). Para entrar deberas abrir una linea de comandos en Windows, y ejecutar `wsl` para entrar en dicha distribucion.
En lugar de usar Docker Desktop, instalaremos Docker Engine directamente dentro de nuestra distribución de Debian en WSL2. Abre tu terminal de Debian para ejecutar los siguientes comandos.

**1. Configurar el repositorio de Docker**

Primero, actualiza el índice de paquetes e instala las dependencias necesarias para añadir un nuevo repositorio HTTPS:
```bash
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
```

Añade la clave GPG oficial de Docker:
```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

Añade el repositorio de Docker a las fuentes de APT:
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

**2. Instalar Docker Engine**

Actualiza de nuevo el índice de paquetes e instala la última versión de Docker Engine, CLI, containerd y Docker Compose:
```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**3. Gestionar Docker como un usuario no-root (Recomendado)**

Para poder ejecutar comandos de Docker sin necesidad de `sudo`, añade tu usuario al grupo `docker`:
```bash
sudo usermod -aG docker $USER
```

## 3. Puesta en Marcha de los Servicios

Este repositorio incluye un fichero `docker-compose.yml` con una selección de servicios listos para usar. La configuración se gestiona a través de variables de entorno.

**1. Configurar el entorno**

Antes de levantar los servicios, debes crear tu propio fichero de configuración a partir del ejemplo proporcionado.

Copia `.env.example` a `.env`:
```bash
cp .env.example .env
```
Abre el fichero `.env` y ajusta la zona horaria (`TZ`) y, si es necesario, los `PUID`/`PGID` según las instrucciones del fichero.

**2. Levantar los servicios**

Una vez tengas configurado tu fichero `.env`, puedes levantar todo el stack de servicios con un solo comando.

```bash
docker-compose up -d
```
La primera vez, Docker descargará todas las imágenes necesarias, por lo que puede tardar unos minutos. El flag `-d` ejecuta los contenedores en segundo plano.

### Comandos de Docker Compose

Aquí tienes los comandos más comunes para gestionar tu stack:

-   **Ver el estado de los servicios:**
    ```bash
    docker-compose ps
    ```

-   **Ver los logs de todos los servicios en tiempo real:**
    ```bash
    docker-compose logs -f
    ```

-   **Ver los logs de un servicio específico (ej. `jellyfin`):**
    ```bash
    docker-compose logs -f jellyfin
    ```

-   **Detener y eliminar los contenedores, redes y volúmenes:**
    ```bash
    docker-compose down -v
    ```

-   **Reiniciar los servicios:**
    ```bash
    docker-compose restart
    ```

## Contribuciones

Si encuentras algún error o tienes sugerencias para mejorar esta guía, no dudes in abrir un *issue* o enviar un *pull request*.
