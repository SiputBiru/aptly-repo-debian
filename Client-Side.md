# Adding new Repos to the client

### Trust the GPG Key
To avoid needing to add `[trusted=yes]` in your `sources.list`, you need to ensure that the public key is added to the trusted keys for APT:
1. Add the Public Key to APT:
	Copy the public key from the main apt server to the client:
		- You can scp it from the client:
	```bash
    scp -p -r youruesr@<serverIP>:directory/publicrepo.key .
    
    // Then dearmor it if the key in ASCII-armored format
    gpg --dearmor -o /etc/apt/trusted.gpg.d/myrepo.gpg ./publicrepo.key

	// If the key is already on binary format you just need to copy it to /etc/apt/trusted.gpg.d/ directory
	cp ./yourkey.gpg /etc/apt/trusted.gpg.d/
	```
	Or using apt-key(**You need to install gnupg, apt-key is Deprecated**):
	```bash
	apt-key add /path/to/repo-key.gpg
	```
	Or just wget it from the url:
	```bash
	wget http://yourserverIP/myrepo.gpg -O /etc/apt/trusted.gpg.d/myrepo.gpg
	```
2. Verify the Key:
	```
	apt-key list
	```
3. Update APT:
	```bash
	apt update
	```

### Example Format for adding new repos

where to change the source repo:
`/etc/apt/sources.list`

- repos using dlbd/dvd iso binary:
```bash
deb [trusted=yes] cdrom:[Debian GNU/Linux 12.9.0 _Bookworm_ - Official amd64 DVD Binary-1 with firmware 20250111-10:55]/ bookworm contrib main non-free-firmware
```
On using dlbd/dvd iso you need to add `[trusted=yes]` to make it work.

- for external repos:

```bash
deb http://{ServerIP/Domain}/ bookworm non-free-firmware
```
	
- If don't use gpg key, need to add `[trusted=yes]` :
```
deb [trusted=yes] http://{ServerIP/Domain}/ bookworm non-free-firmware
```
