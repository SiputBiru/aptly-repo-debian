There is 2 type "Repository" we can create, package repository and mirror

# Creating Package Repository:

#### 1. Mount the ISO:
- First you need to create mounting point:
```bash
mkdir -p /mnt/cdrom
```
- You can do 2 things:
	1. Create the mounting point from iso file:
		- mount the iso file to mount point directory:
			```bash
			mount -o loop /path/to/debiandlbd.iso /mnt/cdrom
			```
	2. From CD/DVD:
		- check the device name:
			```bash
			lsblk
			```
		- output:
			```bash
			NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
			sda      8:0    0  200G  0 disk
			├─sda1   8:1    0  199G  0 part     /
			├─sda2   8:2    0    1K  0 part
			└─sda5   8:5    0  975M  0 part    [SWAP]
			sr0     11:0    1 44.4G  0 rom
			```
			in this example the device name is **sr0**
		- mount the CD/DVD using the following command:
			```bash
			mount /dev/sr0 /mnt/cdrom
			```
	3. Then you can check if the mount point have any file inside:
		```bash
		ls /mnt/cdrom
		```
		output:
		```bash
		boot    dists  firmware     isolinux    pool                 README.mirrors.txt
		css     doc    install      md5sum.txt  README.html          README.source
		debian  EFI    install.amd  pics        README.mirrors.html  README.txt
		```
	
- Then you create new directory to copy all of the file from the mount point:
	```bash
	mkdir -p /var/www/debian
	cp -rf /mnt/cdrom/* /var/www/debian/
	```
- We do this because:
	- If the ISO is unmounted or the system is rebooted, the repository will become unavailable. This can lead to downtime for clients trying to access the repository.
	- Serving files directly from a mounted ISO may not be as performant as serving from a dedicated directory, especially if the ISO is on slower media (like a CD/DVD).
	- If the ISO is read-only, you cannot add or modify packages without remounting or recreating the ISO.
	- Serving a mounted directory may expose more files than intended, especially if the directory structure is not well controlled.
	- Managing backups and recovery can be more complex when serving directly from a mounted ISO, as opposed to a dedicated repository directory.
- Unmount the iso:
```bash
umount -l /mnt/cdrom
```

### 2. Installing Aptly package and all dependencies
- Just simple things:
```bash
apt install aptly bzip2 gnupg gpgv -y
```

### 3. Creating GPG Key
- First we need to create the GPG key first(You can skip this if don't want to use gpg key):
```bash
gpg --full-generate-key
```
- To see the keys you have created:
```bash
gpg --list-keys
```
- Exporting public key:
```bash
gpg --armor --export your-email@example.com > public.key
```
Replace `your-email@example.com` with the email address you used when creating the key.
- Exporting private key(Optional):
```bash
gpg --armor --export-secret-keys your-email@example.com > private.key
```
- Exporting on binary format rather than ASCII-armored format for apt key:
```bash
// Check the key ID
gpg --list-keys

// Export the public key base on key ID
gpg --export -o ./myrepo.gpg YOUR_KEY_ID
```


### 4. Aptly Configuration
- **Import Packages from the Mounted ISO**:
To use Aptly, we to create a new repository and import packages from the mounted ISO.
```bash
aptly repo create -distribution=bookworm -component=main -component=non-free-firmware my-local-repo
```
- **Add Packages to the Repository**:
You can add packages from the mounted ISO to your Aptly repository:
```bash
aptly repo add my-local-repo /var/www/debian/pool/main/*
```
- Create snapshot from the repos:
After adding packages to your repository, you can create a snapshot with the following command:
```bash
aptly snapshot create my-snapshot from repo my-local-repo
```
Replace `my-repo` with the name of your repository. and for `my-snapshot` with name of your snapshot.
- Publish the Snapshot:
You can publish the snapshot to a specific directory:
- without gpg key:
```bash
aptly publish snapshot my-snapshot
```
If you don't want to use **gpg key**, we need to change aptly configuration at `/root/.aptly.conf` and change these setting to **true**, we can also change the root directory of aptly
```json
...
	"RootDir": "/your/path/", 
...
	"gpgDisableSign": true,
	"gpgDisableVerify": true,
...
```
The final configuration file of aptly is like this:
```json
{
  "rootDir": "/var/aptly",
  "downloadConcurrency": 4,
  "downloadSpeedLimit": 0,
  "downloadRetries": 0,
  "downloader": "default",
  "databaseOpenAttempts": -1,
  "architectures": [],
  "dependencyFollowSuggests": false,
  "dependencyFollowRecommends": false,
  "dependencyFollowAllVariants": false,
  "dependencyFollowSource": false,
  "dependencyVerboseResolve": false,
  "gpgDisableSign": true,
  "gpgDisableVerify": true,
  "gpgProvider": "gpg",
  "downloadSourcePackages": false,
  "skipLegacyPool": true,
  "ppaDistributorID": "debian",
  "ppaCodename": "",
  "skipContentsPublishing": false,
  "skipBz2Publishing": false,
  "FileSystemPublishEndpoints": {},
  "S3PublishEndpoints": {},
  "SwiftPublishEndpoints": {},
  "AzurePublishEndpoints": {},
  "AsyncAPI": false,
  "enableMetricsEndpoint": false
}
```

- with gpg key:
```
aptly publish snapshot -gpg-key=<your-gpg-key-id> my-snapshot
```
Replace `<your-gpg-key-id>` with the ID of the GPG key you created earlier.

This will create a directory structure that can be served via HTTP.
output:
```
...
Local repo my-local-repo has been successfully published.
Please setup your webserver to serve directory '/var/aptly/bookworm' with autoin                                                 dexing.
Now you can add following line to apt sources:
  deb http://your-server/ bookworm non-free-firmware
Don't forget to add your GPG key to apt with apt-key.

You can also use `aptly serve` to publish your repositories over HTTP quickly.
...
```
- at this point we can serve it using `aptly serve` or use http server with autoindexing.

### 5. Configure Nginx to Serve the Repository 
1. Install the Nginx Web server:
```bash
apt install nginx -y
```
1. create new configuration file:
```bash
nano /etc/nginx/sites-available/debian-repo
```
add the following configuration:
```nginx
server {
    listen 80;
    server_name your-server-ip-or-domain;  # Replace with your server's IP or domain

    location / {
        root /var/aptly/public/;  # Path to your repository
        autoindex on;  # Enable directory listing
        index index.html;
    }

    # Optional: Add GPG key serving
    location /gpg.key {
        alias /path/to/your/gpg.key;  # Path to your GPG key
    }
}
```
2. Create Symbolic link to enable site:
```bash
ln -s /etc/nginx/sites-available/debian-repo /etc/nginx/sites-enabled/
```
3. test the nginx configuration:
```
nginx -t
```
3. if configuration test is successfull, restart Nginx:
```
systemctl restart nginx
```

## For the client configuration
[[Client-Side]]

