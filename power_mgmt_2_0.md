Here is a concise summary of the changes we’ve made. You can save this as `framework-power-fix.md` for your records.

---

# Framework 13 AMD: Battery Drain Fix Log

## 1. The Problems Identified

* **Hardware Sleep (s2idle):** Only achieving **~53%**, meaning the CPU was staying "awake" for half the time the lid was closed.
* **Vampire Drain:** The system was drawing **1.92W** during sleep (should be **<0.6W**).
* **The Culprits:** Expansion cards (specifically the SD Reader) were keeping the **GP11, GP12, and XHC2** controllers in a high-power state ().

---

## 2. Hardware Changes: Expansion Slot Strategy

The Framework 13 AMD has specific power characteristics for its four slots.

* **Action taken:** Moved the **SD Card Expansion Card** to a **Rear Slot** (closest to the screen).
* **Reason:** The rear slots (1 & 4) are natively USB4/PCIe. They handle the "low power handshake" much more reliably than the front slots.
* **Note:** All 4 slots still support full 100W charging. Using the rear slots for charging or high-drain cards is the most efficient layout.

---

## 3. Software Changes: Force Sleep

### A. USB Autosuspend (The `udev` Rule)

**Command:**

```bash
echo 'ACTION=="add", SUBSYSTEM=="usb", TEST=="power/control", ATTR{power/control}="auto"' | sudo tee /etc/udev/rules.d/50-usb-autosuspend.rules
sudo udevadm control --reload-rules && sudo udevadm trigger

```

* **What it does:** Forces Linux to put "Idle" USB devices (like the chips inside your expansion cards) into a low-power state.
* **Context:** Without this, Linux keeps the USB bus "active" to prevent disconnects, which drains the battery.

### B. PCIe Power Management (The Kernel Flag)

**Command:**

```bash
sudo grubby --update-kernel=ALL --args="pcie_aspm=force"

```

* **What it does:** Forces Active State Power Management (ASPM). It tells the motherboard to cut power to the data lanes connecting the expansion cards when they aren't moving data.
* **Context:** This clears the "Constraint not met" errors found in the `amd-s2idle` logs, allowing the CPU to reach its deepest sleep state (C10).

---

## 4. Expected Results

* **Hardware Sleep:** Should rise from **53%** to **>90%**.
* **Battery Drain:** Should drop from **~1.9W/hr** to **~0.6W/hr**.
* **Lid-Closed Life:** Should increase from **~1.5 days** to **~4-5 days**.

---

**Next Step:** After you reboot to apply the kernel flag, run `sudo amd-s2idle test` one last time. Would you like me to analyze that final report to see if we hit the 90% goal?
