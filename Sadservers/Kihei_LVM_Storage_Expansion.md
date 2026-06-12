# Scenario: "Kihei": Surely Not Another Disk Space Scenario

**Level:** Medium  
**Tags:** `disk volumes`, `golang`, `lvm`, `storage`, `troubleshooting`  

## 📝 Problem Description
A Go binary located at `/home/admin/kihei` panics and fails on execution. Verbose logging indicates that the program attempts to dynamically allocate a single **1.5 GB** data file (`/home/admin/data/newdatafile`). 

The standard system partition (`/`) is resource-constrained, with only **576 MB** of available storage left. The system possesses two auxiliary block storage devices (`/dev/nvme1n1` and `/dev/nvme2n1`), each exactly **1 GB** in size. Because neither individual disk is large enough to contain the 1.5 GB file independently, standard partition formatting and mounting is insufficient.

## 💡 The Solution Strategy
To handle a file larger than any single available physical disk, Logical Volume Manager (LVM) must be utilized. LVM allows us to abstract physical storage hardware into an elastic pool. By initializing both disks as Physical Volumes (PVs), combining them into a single unified Volume Group (VG), and carving out a spanning Logical Volume (LV), we can create a single 2 GB filesystem pool to accommodate the application.

### The Commands
```bash
# 1. Inspect the binary constraints and current storage layout
/home/admin/kihei -v
df -h
lsblk

# 2. Initialize the raw block devices as LVM Physical Volumes
sudo pvcreate /dev/nvme1n1 /dev/nvme2n1

# 3. Aggregate both physical volumes into a single 2 GB Volume Group
sudo vgcreate vg_kihei /dev/nvme1n1 /dev/nvme2n1

# 4. Allocate 100% of the combined group space into a new Logical Volume
sudo lvcreate -l 100%FREE -n lv_kihei vg_kihei

# 5. Build an ext4 filesystem on top of the logical volume block interface
sudo mkfs.ext4 /dev/vg_kihei/lv_kihei

# 6. Build the target mount directory path
mkdir -p /home/admin/data

# 7. Mount the logical volume filesystem and delegate write permissions
sudo mount /dev/vg_kihei/lv_kihei /home/admin/data
sudo chown -R admin:admin /home/admin/data

# 8. Re-run the binary to let it allocate space and exit successfully
/home/admin/kihei
# Output returns: Done.
