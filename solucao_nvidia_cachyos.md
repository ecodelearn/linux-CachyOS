# Solucionando Problemas com Drivers NVIDIA no CachyOS (e outros Arch-based)

Este documento explica os passos para resolver o erro `ERROR: module not found: 'nvidia'` que pode ocorrer durante a atualização do sistema, especialmente ao usar drivers da NVIDIA via DKMS.

## O Problema

Após uma atualização de sistema (`sudo pacman -Syu`), o processo de reconstrução da imagem de inicialização (`mkinitcpio`) pode falhar com erros indicando que módulos da NVIDIA não foram encontrados:

```
==> ERROR: module not found: 'nvidia'
==> ERROR: module not found: 'nvidia_modeset'
==> ERROR: module not found: 'nvidia_uvm'
==> ERROR: module not found: 'nvidia_drm'
```

Isso acontece porque os módulos do kernel da NVIDIA não foram construídos para a nova versão do kernel que acabou de ser instalada.

## A Causa Raiz

O sistema DKMS (Dynamic Kernel Module Support) é responsável por construir automaticamente os módulos de drivers (como o da NVIDIA) para cada kernel instalado no seu sistema. Para fazer isso, o DKMS precisa dos **cabeçalhos (headers)** correspondentes a cada versão do kernel.

O erro ocorre se você tiver um kernel instalado (ex: `linux-cachyos` ou `linux`) mas não tiver o pacote de cabeçalhos correspondente (ex: `linux-cachyos-headers` ou `linux-headers`) instalado.

## A Solução Passo a Passo

A solução consiste em garantir que todos os kernels instalados tenham seus respectivos pacotes de cabeçalhos e, em seguida, forçar a reconstrução dos módulos.

### 1. Verifique os Kernels Instalados

Primeiro, liste todos os pacotes de kernel que você possui:

```bash
pacman -Q | grep linux
```

A saída pode ser algo como:
```
linux 6.18.2.arch2-1
linux-cachyos 6.18.2-2
linux-cachyos-lts 6.12.63-2
...
```

### 2. Instale os Cabeçalhos Faltantes (Headers)

Para cada kernel na lista, certifique-se de que o pacote de cabeçalhos correspondente está instalado. No nosso exemplo, precisaríamos dos pacotes:

-   `linux-headers`
-   `linux-cachyos-headers`
-   `linux-cachyos-lts-headers`

Você pode instalá-los com o `pacman`. Se não tiver certeza, é seguro tentar instalar todos eles. O `pacman` simplesmente informará se o pacote já está atualizado.

```bash
sudo pacman -S linux-headers linux-cachyos-headers linux-cachyos-lts-headers
```
*(Adapte a lista de pacotes de acordo com os kernels que você tem instalados)*

### 3. Reinstale o Pacote DKMS da NVIDIA

Para forçar a reconstrução de todos os módulos para todos os kernels, o caminho mais fácil é reinstalar o pacote DKMS da NVIDIA. Verifique o nome exato do seu pacote (`pacman -Q | grep nvidia`), mas geralmente é algo como `nvidia-dkms` ou, no nosso caso, `nvidia-580xx-dkms`.

```bash
sudo pacman -S nvidia-580xx-dkms
```
Este comando irá remover e reinstalar o driver, o que aciona o DKMS para construir os módulos para todos os kernels que possuem seus cabeçalhos instalados.

### 4. (Opcional) Regenere Manualmente o Initramfs

Na maioria dos casos, o passo anterior já aciona a reconstrução da imagem de inicialização. Se por algum motivo isso não acontecer, ou se os erros persistirem, você pode forçar a regeneração de todas as imagens de boot com o comando:

```bash
sudo mkinitcpio -P
```

Após esses passos, o problema estará resolvido e seu sistema poderá inicializar normalmente com os drivers da NVIDIA ativos.

---

# Troubleshooting NVIDIA Driver Issues on CachyOS (and other Arch-based distributions)

This document explains the steps to resolve the `ERROR: module not found: 'nvidia'` error that may occur during a system update, especially when using NVIDIA drivers via DKMS.

## The Problem

After a system update (`sudo pacman -Syu`), the initial ramdisk (`mkinitcpio`) reconstruction process may fail with errors indicating that NVIDIA modules were not found:

```
==> ERROR: module not found: 'nvidia'
==> ERROR: module not found: 'nvidia_modeset'
==> ERROR: module not found: 'nvidia_uvm'
==> ERROR: module not found: 'nvidia_drm'
```

This happens because the NVIDIA kernel modules were not built for the new kernel version that was just installed.

## The Root Cause

The DKMS (Dynamic Kernel Module Support) system is responsible for automatically building driver modules (such as NVIDIA's) for each kernel installed on your system. To do this, DKMS needs the **headers** corresponding to each kernel version.

The error occurs if you have a kernel installed (e.g., `linux-cachyos` or `linux`) but do not have the corresponding header package (e.g., `linux-cachyos-headers` or `linux-headers`) installed.

## The Step-by-Step Solution

The solution involves ensuring that all installed kernels have their respective header packages and then forcing the modules to rebuild.

### 1. Check Installed Kernels

First, list all kernel packages you have:

```bash
pacman -Q | grep linux
```

The output might look something like:
```
linux 6.18.2.arch2-1
linux-cachyos 6.18.2-2
linux-cachyos-lts 6.12.63-2
...
```

### 2. Install Missing Headers

For each kernel in the list, make sure the corresponding header package is installed. In our example, we would need the packages:

-   `linux-headers`
-   `linux-cachyos-headers`
-   `linux-cachyos-lts-headers`

You can install them with `pacman`. If you are unsure, it is safe to try installing all of them. `pacman` will simply inform you if the package is already up to date.

```bash
sudo pacman -S linux-headers linux-cachyos-headers linux-cachyos-lts-headers
```
*(Adapt the list of packages according to the kernels you have installed)*

### 3. Reinstall the NVIDIA DKMS Package

To force all modules to rebuild for all kernels, the easiest way is to reinstall the NVIDIA DKMS package. Check the exact name of your package (`pacman -Q | grep nvidia`), but it's usually something like `nvidia-dkms` or, in our case, `nvidia-580xx-dkms`.

```bash
sudo pacman -S nvidia-580xx-dkms
```
This command will remove and reinstall the driver, which triggers DKMS to build the modules for all kernels that have their headers installed.

### 4. (Optional) Manually Regenerate Initramfs

In most cases, the previous step already triggers the initial ramdisk regeneration. If for any reason this does not happen, or if the errors persist, you can force the regeneration of all boot images with the command:

```bash
sudo mkinitcpio -P
```

After these steps, the problem should be resolved, and your system should be able to boot normally with the NVIDIA drivers active.