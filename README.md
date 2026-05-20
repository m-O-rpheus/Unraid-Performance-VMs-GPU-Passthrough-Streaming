# Unraid-Performance-VMs-GPU-Passthrough-Streaming
## by Markus Jäger

---

# Windows 11 GPU Passthrough in Unraid VM konfigurieren

Diese Anleitung beschreibt die Einrichtung einer Windows 11 Virtual Machine mit GPU Passthrough unter Unraid.

---

## 1. Erstellung der Standard-VM

Zuerst wird eine Standard-VM in Unraid erstellt. Dabei müssen folgende Einstellungen beachtet werden:

- Initial Memory und Max Memory müssen identisch sein
- Machine Type: Q35-10.2
- Migratable: ON
- Hyper-V: Enabled (Yes)

Zusätzlich sollte eine ausreichende Anzahl an CPU-Kernen sowie genügend RAM zugewiesen werden, damit das Betriebssystem stabil und performant arbeiten kann.

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
  <features>
    <acpi/>
    <apic/>
    <hyperv mode="passthrough"/>
    <kvm>
      <hidden state='on'/>
    </kvm>
  </features>
  <cpu mode='host-passthrough' check='none' migratable='on'>
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