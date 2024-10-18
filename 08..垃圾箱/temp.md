

![[Pasted image 20241018081542.png]]


```
#!/bin/bash
### BEGIN INIT INFO
# Provides:          automount-nfs
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Description:       Automatically mount NFS shares at startup.
### END INIT INFO

# Change these variables to your NFS settings
NFS_SERVER="nfs.example.com"
NFS_PATH="/path/to/share"
MOUNT_POINT="/mnt/nfs"

# Create mount point if it doesn't exist
if [ ! -d "$MOUNT_POINT" ]; then
    mkdir -p "$MOUNT_POINT"
fi

# Mount the NFS share
mount -t nfs "$NFS_SERVER:$NFS_PATH" "$MOUNT_POINT"

# Check if the mount was successful
if mount | grep "$MOUNT_POINT" > /dev/null; then
    echo "NFS share mounted successfully."
else
    echo "Failed to mount NFS share."
    exit 1
fi
```

测试![[Pasted image 20241018081725.png]]
