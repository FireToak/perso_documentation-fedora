# NFS Vagrant

**Message d'erreur :**

```bash
Vagrant failed to install an updated NFS exports file. This may be
due to overly restrictive permissions on your NFS exports file. Please
validate them and try again.

command: sudo mv -f /tmp/vagrant-exports /etc/exports
stdout: 
stderr: mv: cannot remove '/tmp/vagrant-exports': No such file or directory
```

Solution apporté :

```bash
sudo rm /etc/exports
sudo touch /etc/exports

vagrant halt
vagrant up
```

**Source :**

- [Vagrant error: NFS is reporting that your exports file is invalid](https://stackoverflow.com/questions/20726248/vagrant-error-nfs-is-reporting-that-your-exports-file-is-invalid)