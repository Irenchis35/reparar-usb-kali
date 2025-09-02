#!/bin/bash

set -e

# Función para listar unidades USB
listar_unidades_usb() {
  lsblk -dpno NAME,MODEL | grep -i usb | awk '{print $1}'
}

# Crear un array para dialog (opciones: etiqueta y valor)
unidades=()
while IFS= read -r line; do
  unidades+=("$line" "$line")
done < <(listar_unidades_usb)

if [ ${#unidades[@]} -eq 0 ]; then
  dialog --msgbox "No se encontraron unidades USB conectadas." 6 40
  clear
  exit 1
fi

# Mostrar menú para seleccionar unidad
unidad=$(dialog --clear --title "Selecciona unidad USB" \
  --menu "Unidades USB disponibles:" 15 50 5 \
  "${unidades[@]}" 3>&1 1>&2 2>&3)

clear

if [ -z "$unidad" ]; then
  echo "No seleccionaste ninguna unidad. Saliendo."
  exit 1
fi

# Confirmar reparación
dialog --yesno "¿Seguro que quieres reparar la unidad $unidad?\nEsto puede borrar datos." 7 50
respuesta=$?

if [ $respuesta -ne 0 ]; then
  dialog --msgbox "Operación cancelada." 5 30
  clear
  exit 0
fi

# Desmontar particiones
sudo umount "${unidad}"* 2>/dev/null || true

# Ejecutar fsck
dialog --infobox "Ejecutando fsck para reparar $unidad..." 5 40
sudo fsck -y "$unidad"

# Preguntar si quiere formatear
dialog --yesno "¿Quieres formatear la unidad $unidad a FAT32?\nEsto borrará todos los datos." 7 50
formatear=$?

if [ $formatear -eq 0 ]; then
  dialog --infobox "Formateando $unidad a FAT32..." 5 40
  sudo mkfs.vfat -F 32 "$unidad"
  dialog --msgbox "Formateo completado." 5 30
fi

dialog --msgbox "Proceso finalizado." 5 30
clear
