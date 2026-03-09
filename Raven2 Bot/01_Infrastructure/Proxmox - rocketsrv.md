## GPU passthrough для Win11-VM (2026-03-09)

- BIOS:
  - Включен SVM Mode.
  - Включен IOMMU (AMD IOMMU / AMD-Vi).

- IOMMU в Proxmox:
  - Хост грузится в UEFI, IOMMU активен (в `dmesg` видно строки AMD-Vi, есть IOMMU-группы в `/sys/kernel/iommu_groups`).
  - Используется GRUB, параметры ядра настроены аккуратно, без критической зависимости от `amd_iommu=on`.

- Видеокарты:
  - 01:00.0 / 01:00.1 – RTX 4090 (остаётся для хоста/других задач).
  - 05:00.0 / 05:00.1 – RTX 4090, отдана Win11-VM (IOMMU group 15).

- VFIO:
  - `/etc/modprobe.d/vfio.conf`:
    - `options vfio-pci ids=10de:2684,10de:22ba disable_vga=1`
  - `/etc/modules-load.d/vfio.conf`:
    - `vfio`
    - `vfio_pci`
    - `vfio_iommu_type1`
    - `vfio_virqfd`
  - После перезагрузки:
    - `05:00.0` и `05:00.1` используют `vfio-pci`.

- Win11-VM:
  - Machine type: `q35`.
  - BIOS: `OVMF (UEFI)` с EFI-диском.
  - CPU type: `host`.
  - В Hardware добавлено PCI-устройство `0000:05:00.0` с опцией `All Functions` (захватывает и 05:00.1).
