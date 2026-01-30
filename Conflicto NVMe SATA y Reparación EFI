# Documentación de Incidencia: Instalación Limpia de Windows
## Conflicto NVMe/SATA y Reparación EFI

**Fecha:** 30 de enero de 2026  
**Contexto:** Instalación de Windows en un SSD SATA en un equipo que cuenta previamente con un SSD NVMe (con otro SO) y utilizando un USB de instalación con residuos de particiones Linux anteriores.

---

## 1. Problemas Identificados

### Error A: Conflicto de Hardware
El instalador de Windows fallaba o presentaba inestabilidad al intentar instalar en el SSD SATA mientras el SSD NVMe estaba conectado físicamente.

- **Causa:** Conflicto en la gestión de prioridades de disco del instalador de Windows ante la presencia de múltiples unidades de almacenamiento.

### Error B: Dependencia del Bootloader
Tras la instalación, el sistema operativo no arrancaba sin el USB conectado.

- **Causa:** El USB de instalación contenía una partición EFI activa (remanente de una instalación previa de MX Linux/sistema live). El instalador de Windows detectó esta partición EFI externa y la utilizó para alojar el gestor de arranque (Windows Boot Manager) en lugar de crear una partición EFI local en el disco SATA.

---

## 2. Solución Aplicada

### Fase 1: Aislamiento de Hardware
- **Acción:** Se procedió a la extracción física del SSD NVMe para dejar únicamente el SSD SATA conectado durante la instalación. Esto forzó al instalador a centrarse exclusivamente en la unidad destino.

### Fase 2: Reparación del Arranque (Bootloader)
- **Acción:** Una vez instalados los archivos de Windows, se detectó la ausencia de arranque autónomo. Se utilizó la consola de recuperación (`Shift + F10` o desde el entorno de recuperación) para generar manualmente los archivos de arranque en la partición del sistema.

---

## 3. Registro de Comandos (CLI)

El siguiente procedimiento repara la partición EFI y vincula la instalación de Windows al arranque UEFI.

### Pre-requisitos identificados:
- `C:` Letra asignada a la partición donde se instaló Windows.
- `T:` Letra temporal asignada a la partición EFI (System) del disco.

### Secuencia de Comandos:

```cmd
:: 1. Ingresar a la herramienta de gestión de discos
diskpart

:: 2. Listar volúmenes para identificar la partición de Windows y la partición EFI (aprox 100MB, FAT32)
list vol

:: 3. Seleccionar el volumen correspondiente a la partición EFI
:: (Reemplazar # por el número visto en el paso anterior)
select vol #

:: 4. Asignar una letra temporal para poder trabajar con ella
assign letter=T

:: 5. Verificar que la letra se asignó correctamente
list vol

:: 6. Salir de Diskpart
exit

:: 7. (Opcional) Cambiar al directorio T para verificar acceso
T:

:: 8. COMANDO CRÍTICO: Copiar archivos de arranque UEFI
:: Sintaxis: bcdboot <Origen_Windows> /s <Destino_EFI> /f <Firmware_Type>
bcdboot c:\Windows /s T: /f UEFI
```

---

## 4. Resultado Final

- ✅ Se creó correctamente la estructura de arranque en el SSD SATA.
- ✅ El sistema inicia de manera autónoma sin depender del USB ni del SSD NVMe.
- ✅ Se procedió a reconectar el SSD NVMe, gestionando el orden de arranque posteriormente desde la BIOS (UEFI Hard Disk priorities).

---

## Notas Adicionales

- **Prevención:** Antes de realizar instalaciones de Windows en sistemas con múltiples discos, se recomienda desconectar físicamente los discos no deseados o limpiar las particiones EFI de los medios de instalación USB.
- **Limpieza del USB:** Para evitar el Error B en futuras instalaciones, se puede limpiar completamente el USB con `diskpart` y comandos `clean` antes de crear el medio de instalación.

---

**Autor:** Sandro Gonzalo Niño Egoavil
**Licencia:** MIT 
