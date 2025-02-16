# Setup LVM Thin Pool Storage in Proxmox

This guide provides step-by-step instructions to configure a **2.5TB LVM Thin Pool** on Proxmox using an unused disk.

---

## **Step 1: Identify the Disk**

Check available disks to find the unused one:

```bash
lsblk
```

- Your disk should be `/dev/nvme1n1` (assuming no partitions exist yet).

---

## **Step 2: Create a Partition**

Use `parted` to create a **2.5TB partition**.

1. Open `parted`:
   ```bash
   parted /dev/nvme1n1
   ```
2. Inside `parted`, execute these commands:
   ```bash
   mklabel gpt
   mkpart primary 0% 2500GB
   ```
   - `mklabel gpt` → Creates a new **GPT partition table**.
   - `mkpart primary 0% 2500GB` → Creates a **2.5TB primary partition**.
3. Exit `parted`:
   ```bash
   quit
   ```
4. Verify the new partition:
   ```bash
   lsblk
   ```
   You should see `/dev/nvme1n1p1` as a **2.5TB partition**.

---

## **Step 3: Create a Physical Volume (PV)**

Now, initialize the new partition as an LVM **Physical Volume (PV)**:

```bash
pvcreate /dev/nvme1n1p1
```

Verify it:

```bash
pvs
```

---

## **Step 4: Extend the Volume Group (VG)**

Now, add this partition to your existing **VG (**``**)**:

```bash
vgcreate pve-lms /dev/nvme1n1p1
```

Check the updated volume group:

```bash
vgs
```

---

## **Step 5: Create the LVM Thin Pool**

Now, create an **LVM Thin Pool** with a **256K chunk size**:

```bash
lvcreate -L 2.5T -T -c 256K pve-lms/vmdata-lms
```

- `-L 2.5T` → Allocates **2.5TB**.
- `-T` → Specifies it's a **Thin Pool**.
- `-c 256K` → Sets the **chunk size to 256KB**.

Check the new thin pool:

```bash
lvs
```

---

## **Step 6: Add Storage to Proxmox**

Now, make this storage available in the **Proxmox Web UI**.

1. Open **Proxmox Web Interface**.
2. Navigate to **Datacenter > Storage**.
3. Click **"Add" > "LVM-Thin"**.
4. Configure:
   - **ID:** `vmdata-lms`
   - **Volume Group:** `pve-lms`
   - **Thin Pool:** `vmdata-lms`
   - **Content:** `Disk images, Containers`
5. Click **"Add"**.

---

## **Step 7: Verify Everything**

Finally, check if everything is set up correctly:

1. List **Physical Volumes**:
   ```bash
   pvs
   ```
2. List **Volume Groups**:
   ```bash
   vgs
   ```
3. List **Logical Volumes**:
   ```bash
   lvs
   ```
4. Show storage devices:
   ```bash
   lsblk
   ```

---

## **Done!** 🎉

You can now use `vmdata-lms` as **local VM storage** in Proxmox.

