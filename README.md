# Unraid-Performance-VMs-GPU-Passthrough-Streaming
## by Markus Jäger

---

# Windows 11 GPU Passthrough in Unraid VM konfigurieren

Diese Anleitung beschreibt die Einrichtung einer Windows 11 Virtual Machine mit GPU Passthrough unter Unraid.

---

## 1. Virtuelle Maschine

**Erstellung der virtuellen Maschine:**
Zunächst wird eine Standard-VM in Unraid erstellt. Dabei sollten folgende Einstellungen vorgenommen werden:

- Initial Memory / Max Memory: Beide Werte müssen identisch sein.
- Machine Type: Q35-10.2
- Migratable: OFF
- BIOS: OVMF TPM
- Hyper-V: Enabled (Yes)

Zusätzlich müssen ausreichend CPU-Kerne und Arbeitsspeicher zugewiesen werden, damit das Betriebssystem stabil und performant arbeiten kann. Für Windows 11 sollten mindestens 10 GB RAM eingeplant werden.

**Virtuelle Festplatte:**
Um eine möglichst hohe Festplattenperformance zu erreichen, sollten für die virtuelle Festplatte folgende Einstellungen verwendet werden:

- vDisk Type: raw
- vDisk Bus: VirtIO
- Discard: Unmap (TRIM)

**Netzwerkkonfiguration:**
Für eine möglichst hohe Netzwerkperformance sollte folgendes Netzwerkmodell verwendet werden:

- Network Model: virtio

---

## 2. GPU Passthrough Konfiguration

Als Grafikkarte muss die physische GPU direkt der VM zugewiesen werden.

- Virtuelle Grafikkarten (z. B. VNC) dürfen nicht verwendet werden und müssen ersetzt werden
- Die GPU wird über PCI Passthrough eingebunden

### NVIDIA GPUs

Bei NVIDIA-Grafikkarten müssen alle zugehörigen Funktionen der PCIe-Karte durchgereicht werden:

- GPU (Grafikeinheit)
- HDMI/Display Audio
- ggf. weitere zugehörige Funktionen

---

### Intel iGPU (SR-IOV)

Bei Intel iGPUs wird in der Regel nur die Grafikeinheit selbst durchgereicht.

---

## 3. Wichtiger Hinweis zur Remote-Verbindung

Damit die VM mit der GPU gestartet werden kann, muss innerhalb der Windows-VM eine alternative Zugriffsmethode eingerichtet werden:

- Remote Desktop (RDP) oder
- ein Remote-Tool wie Sunshine, Moonlight, Parsec oder AnyDesk

Dies ist notwendig, da nach Aktivierung des GPU Passthrough keine Anzeige mehr über die virtuelle Grafikkarte verfügbar ist.

---

## 4. XML-Anpassung zur Vermeidung von Code 43

Nach dem Speichern der VM-Einstellungen muss die XML-Konfiguration angepasst werden, um den NVIDIA Code 43 Fehler (bzw. ähnliche Treibererkennungsprobleme) zu vermeiden.

Diese Anpassung funktioniert sowohl bei NVIDIA- als auch bei Intel-GPUs.

```xml
  <os>
    ...
    <smbios mode='host'/>
  </os>
  <features>
    <acpi/>
    <apic/>
    <hyperv mode="passthrough"/>
    <kvm>
      <hidden state='on'/>
    </kvm>
  </features>
  <cpu mode='host-passthrough' check='none' migratable='off'>
    <topology sockets='1' dies='1' clusters='1' cores='20' threads='1'/>
    <cache mode='passthrough'/>
  </cpu>
```

Zusätzlich muss der passende GPU-Treiber des Herstellers innerhalb der VM installiert werden.

---

## 5. Abschluss

Nach korrekter Konfiguration:

- wird die GPU vollständig an die VM durchgereicht 
- läuft Windows 11 stabil ohne Code 43 Fehler 
- kann die VM über Streaming (z. B. Moonlight / Sunshine) genutzt werden
- ist eine nahezu native Performance möglich
