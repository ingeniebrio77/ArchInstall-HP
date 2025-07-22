# Guía de Instalación: Arch Linux Seguro (Parte 2 - El Sistema Base)
**Versión: 1.0**

## Prerrequisitos

Este documento asume que se han completado los pasos de la **Parte 1**:
*   La BIOS/UEFI está configurada.
*   El disco ha sido particionado con una partición EFI y una partición de sistema.
*   La partición de sistema ha sido cifrada con LUKS2.
*   Se ha creado un sistema de archivos Btrfs con subvolúmenes y se han montado correctamente en `/mnt`.

---

## Fase 4: Instalación del Sistema Base con `pacstrap`

`pacstrap` es un script que instala paquetes en un nuevo directorio raíz de sistema (`/mnt`). Ahora instalaremos el "esqueleto" de Arch Linux junto con todas las herramientas que necesitaremos para la configuración posterior.

1.  **Ejecutar el comando `pacstrap`:**

    ```bash
    pacstrap -K /mnt base base-devel linux linux-firmware linux-headers \
    intel-ucode btrfs-progs neovim sudo networkmanager \
    sbctl tpm2-tools nvidia-dkms
    ```

    > **[Explicación para el Administrador Junior]:**
    >
    > *   `pacstrap -K /mnt`: Instala los paquetes en el directorio `/mnt`, inicializando primero las claves del gestor de paquetes (`pacman`).
    > *   `base`, `base-devel`: El conjunto mínimo de paquetes para que un sistema Arch funcione y para poder compilar software.
    > *   `linux`, `linux-firmware`, `linux-headers`: El Kernel de Linux, su firmware (drivers genéricos para hardware) y las cabeceras (necesarias para compilar módulos como los de NVIDIA).
    > *   `intel-ucode`: Microcódigo específico para tu CPU Intel. Es una actualización crítica de estabilidad y seguridad.
    > *   `btrfs-progs`: Herramientas para gestionar nuestro sistema de archivos Btrfs.
    > *   `neovim`, `sudo`, `networkmanager`: Un editor de texto de terminal, la herramienta para escalar privilegios y el servicio para gestionar la red.
    > *   `sbctl`, `tpm2-tools`: Las herramientas clave para gestionar nuestras propias claves de Secure Boot y para interactuar con el chip TPM 2.0.
    > *   `nvidia-dkms`: El driver de NVIDIA. La versión `dkms` (Dynamic Kernel Module Support) compila el módulo del driver específicamente contra la versión del kernel que tienes instalada. **Esto es esencial para que funcione con Secure Boot.**

---

## Fase 5: Configuración del Sistema Dentro del `chroot`

Ahora "entraremos" en nuestro nuevo sistema, que todavía no ha arrancado por sí mismo, para configurarlo desde adentro.

1.  **Generar el `fstab`:**

    El fichero `/etc/fstab` (file systems table) le dice al sistema qué particiones y subvolúmenes debe montar al arrancar.

    ```bash
    genfstab -U /mnt >> /mnt/etc/fstab
    ```
    > *   La opción `-U` usa los identificadores únicos (UUIDs) de las particiones en lugar de nombres como `/dev/nvme0n1p1`, lo que hace el sistema más robusto ante cambios en el orden de los discos.

2.  **Entrar en el entorno `chroot`:**

    `arch-chroot` cambia el directorio raíz (`/`) de nuestra sesión de terminal actual a `/mnt`. A partir de este momento, cualquier comando que ejecutemos será como si lo estuviéramos ejecutando en nuestro nuevo sistema ya instalado.

    ```bash
    arch-chroot /mnt
    ```
    > *Notarás que el prompt de tu terminal cambia, indicando que ahora estás "dentro" del nuevo sistema.*

### 5.1 Configuración de Región y Hora

1.  **Establecer la Zona Horaria:**
    ```bash
    # Sustituye 'Europe/Madrid' por tu zona horaria. Puedes listarlas con: ls /usr/share/zoneinfo/
    ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime
    ```
2.  **Sincronizar el Reloj del Hardware:**
    ```bash
    # Escribe la hora del sistema en el reloj de la placa base (RTC).
    hwclock --systohc
    ```

### 5.2 Configuración de Idioma (Localización)

1.  **Generar los `locales`:**
    ```bash
    # Edita el fichero /etc/locale.gen
    nano /etc/locale.gen

    # Descomenta (borra el # del inicio) las líneas de los idiomas que necesites.
    # Es recomendable tener en_US.UTF-8 como fallback.
    # Ejemplo:
    # en_US.UTF-8 UTF-8
    # es_ES.UTF-8 UTF-8

    # Una vez guardado el fichero, ejecuta el generador:
    locale-gen
    ```
2.  **Establecer el Idioma por Defecto:**
    ```bash
    # Este fichero establece el idioma para todo el sistema.
    echo "LANG=es_ES.UTF-8" > /etc/locale.conf
    ```
3.  **Establecer la Distribución del Teclado de la Consola:**
    ```bash
    # Esto asegura que el teclado español se cargue en las consolas virtuales (TTYs).
    echo "KEYMAP=es" > /etc/vconsole.conf
    ```

### 5.3 Configuración de Red

1.  **Establecer el Nombre del Host:**
    ```bash
    # Elige un nombre para tu máquina.
    echo "HP-Arch-Station" > /etc/hostname
    ```
2.  **Configurar el fichero `hosts`:**
    ```bash
    # Este fichero mapea nombres de host a direcciones IP. Es fundamental para la red local.
    nano /etc/hosts

    # Asegúrate de que contiene lo siguiente (reemplazando 'HP-Arch-Station' con tu hostname):
    127.0.0.1   localhost
    ::1         localhost
    127.0.1.1   HP-Arch-Station.localdomain HP-Arch-Station
    ```
3.  **Habilitar el Servicio de Red:**
    ```bash
    # Esto le dice a systemd que inicie NetworkManager automáticamente en cada arranque.
    systemctl enable NetworkManager
    ```

### 5.4 Creación de Usuarios y Contraseñas

> **[Nota de Seguridad]:** Es una mala práctica usar la cuenta de `root` para las tareas diarias. Siempre crearemos un usuario con privilegios `sudo`.

1.  **Establecer la Contraseña de `root`:**
    ```bash
    passwd
    ```
2.  **Crear tu Usuario Personal:**
    ```bash
    # Sustituye 'tu_usuario' por el nombre que desees.
    # -m : Crea el directorio /home/tu_usuario
    # -G wheel : Añade al usuario al grupo 'wheel'.
    useradd -m -G wheel -s /bin/bash tu_usuario
    ```
3.  **Establecer la Contraseña para tu Usuario:**
    ```bash
    passwd tu_usuario
    ```
4.  **Conceder Permisos de `sudo`:**
    `sudo` (superuser do) permite a un usuario ejecutar comandos como `root`. Por defecto en Arch, los miembros del grupo `wheel` están autorizados para ello, pero debemos activar esa configuración.

    ```bash
    # El comando 'visudo' es la única forma segura de editar la configuración de sudo.
    EDITOR=nano visudo

    # Se abrirá el fichero de configuración. Busca la siguiente línea:
    # # %wheel ALL=(ALL:ALL) ALL

    # Y descoméntala (borra el # del inicio):
    %wheel ALL=(ALL:ALL) ALL
    ```

---

## Próximos Pasos

Hemos instalado el sistema operativo base y lo hemos configurado con la información esencial (hora, idioma, red y usuarios). El sistema ya es funcional, pero aún no es arrancable.

En la **Parte 3** nos centraremos en el núcleo de la instalación moderna y segura:
*   Configurar el proceso de creación del Kernel (mkinitcpio).
*   Preparar el desbloqueo con TPM.
*   Generar, firmar y configurar la Imagen de Kernel Unificada (UKI).
