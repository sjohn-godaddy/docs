## Set up local reamaze VM without conflict with dev-private

The default vm setup scripts sets the local domain to `dev-reamaze.com`. Works great, except when you want to then access the actual dev-private instance, which is also at `dev-reamaze.com`.

**NOTE**: Requires rebuilding your vm. All brands/channels/socials will reset!!



### Edit config/config.yaml
![image](https://user-images.githubusercontent.com/89797935/149369493-5c79dd13-5a47-4992-8a26-ecf4a4f8a5ad.png)

### Edit docker/app/lantirn-apache.conf
![image](https://user-images.githubusercontent.com/89797935/149369710-21409c58-502c-4ea1-b21f-8c547d7aad46.png)

### Edit site-cookbooks/lantirn-dev/recipes/web.rb (reamaze-bootstrap repo)
![image](https://user-images.githubusercontent.com/89797935/149370088-0267f1cb-f11c-4267-a5cd-9f54d4b5c362.png)

You might need to adjust the crt/key locations like so:
```
cookbook_file "/etc/ssl/certs/localhost.crt"
cookbook_file "/etc/ssl/private/localhost.key"
```

### Generate ssl certs for local-reamaze
``` bssh
brew install mkcert
mkcert \
-key-file site-cookbooks/lantirn-dev/files/default/localhost.key \
-cert-file site-cookbooks/lantirn-dev/files/default/localhost.crt \
local-reamaze.com "*.local-reamaze.com"
```

#### Destroy and rebuild vm
This is also a good time to upgrade to latest versions of VirtualBox, Vagrant, Guest Additions
```
vagrant halt
vagrant destroy
vagrant up
```

Fix vagrant error (about darwin) if it happens
```
sudo curl \
-o /opt/vagrant/embedded/gems/2.2.19/gems/vagrant-2.2.19/plugins/hosts/darwin/cap/path.rb \
https://raw.githubusercontent.com/hashicorp/vagrant/42db2569e32a69e604634462b633bb14ca20709a/plugins/hosts/darwin/cap/path.rb
```

#### Reamaze
```
https://local-reamaze.com/internal
```
