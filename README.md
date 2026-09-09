# ArchInstall-HP — Guía de instalación Arch Linux seguro y moderno

Guía completa y verificada para instalar **Arch Linux** con **LUKS2 + btrfs +
UKI + Secure Boot con claves propias + desbloqueo TPM2** en un
**HP Desktop M01-F1xxx**. Escrita desde una primera instalación real, corregida
contra el sistema en funcionamiento.

> **Idioma:** español · **Probada en:** Arch 2026, kernel 7.x + linux-lts,
> systemd-boot 257+, NVIDIA GTX 1660 SUPER

## Contenido

La guía es [`ArchInstall-Guia.md`](ArchInstall-Guia.md):

| Parte | Tema |
|---|---|
| 0 | Preparación y BIOS/UEFI |
| 1 | Particionado, LUKS2, btrfs y subvolúmenes |
| 2 | Sistema base en chroot (hora, locales de/en/es, red, usuarios, sudo seguro) |
| 3 | UKI, Secure Boot con `sbctl` y TPM2 (incl. troubleshooting real de PCRs) |
| 4 | Snapshots btrfs con snapper + snap-pac |
| 5 | Swap comprimida con zram |
| Apéndices | Diferencias notas-vs-realidad, verificaciones, archivos originales |

## Uso

1. Lee la guía entera antes de tocar el disco (la Parte 1 **borra** el NVMe).
2. Ideal para validar primero en VM (libvirt) o disco USB externo.
3. Tras instalar, ejecuta los checks del Apéndice B.

## Estructura del repo

```
ArchInstall-Guia.md      # la guía (v2.3)
Readmi.md                # Parte 2 redactada original
Prompts.md               # tutorial de referencia
notas-originales-part1.md# notas crudas de la instalación real
originales/              # duplicados y stubs archivados
```

## Licencia

MIT — ver [LICENSE](LICENSE). Autor: Renato Palavecino.
