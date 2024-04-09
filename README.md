# openbao_vagrant_libvirt_ansible

This Vagrant setup creates a VM and configures [an OpenBao
server](https://github.com/openbao/openbao). It also creates a VM and installs
the OpenBao Agent service like mentioned in the [Hashicorp Vault
documentation](https://developer.hashicorp.com/vault/tutorials/vault-agent/agent-quick-start).

This setup is similar to the [one using the Hashicorp Vault Server and
agent](https://github.com/johanneskastl/vault_vagrant_libvirt_ansible).

Default OS is openSUSE Leap 15.5. Although that can be changed in the
Vagrantfile, please beware that this will break the Ansible provisioning.

The server is only running in development mode. This means that the server does
not need to be unsealed after the start. This also means that all configuration
is only stored in-memory aka is lost after a restart or a reboot of the VM.
Enough for playing around, but **do NOT use this in production**

The OpenBao agent is using the root token to authenticate to the OpenBao server.
This is also something that needs to be done properly in a production setup, but
is enough for a demo.

Ansible installs and configures a PostgreSQL database on the openbao-client VM and
adds instructions on how to setup the OpenBao PostgreSQL plugin to the OpenBao
Server VM. Details can be found in the section on OpenBao and PostgreSQL below.

## Vagrant

1. You need vagrant obviously. And ansible. And git...
1. Fetch the box, per default this is `opensuse/Leap-15.5.x86_64`, using
   `vagrant box add opensuse/Leap-15.5.x86_64`.
1. Make sure the git submodules are fully working by issuing `git submodule init
   && git submodule update`
1. Run `vagrant up`
1. At the end of the Ansible provisioning, Ansible prints out a message like the
   following:

   ```bash
   TASK [Output URL] *******************************************************************************
   ok: [openbao-client] => {
       "msg": "The website containing the 'secret' from OpenBao is reachable at http://192.0.2.13"
   }
   ```

1. Open the URL that Ansible displayed at the end of the run. You should see the
   secret value that Ansible wrote to OpenBao previously, fetched from OpenBao by
   the OpenBao agent and saved into Nginx's `index.html` file:

   ```bash
   $ curl http://192.0.2.13
   <!DOCTYPE html>
   <html>
   <body>

   <h1>openbao_vagrant_libvirt_ansible</h1>

   <p>The secret stored in OpenBao is: vagrant-libvirt</p>


   </body>
   </html>
   ```

1. Log in on the openbao-client VM using `vagrant ssh openbao-client`.
1. Ansible already added a line to `.bashrc` for the `root` and `vagrant` users
   to set the OpenBao address:

   ```bash
   export VAULT_ADDR='http://192.0.2.13:8200'
   ```

1. You should already be logged in and can start working with OpenBao.
1. Play around with OpenBao, e.g. get information on your token using `openbao token
   lookup`.
1. There is no WebUI currently (at alpha-20240329)
1. Party!

## OpenBao and PostgreSQL

The Ansible provisioning installs and configures a PostgreSQL database on the
OpenBao client and adds a file on the OpenBao Server at
`/root/postgresql_openbao_snippet.txt`. This file contains commands that you can
use to configure the OpenBao PostgreSQL plugin. For details, check the
[Hashicorp Vault
documentation](https://developer.hashicorp.com/vault/docs/secrets/databases/postgresql).

```bash
openbao secrets enable database

openbao write database/config/vagrant-libvirt-postgresql-database \
    plugin_name="postgresql-database-plugin" \
    allowed_roles="vagrant-libvirt-role" \
    connection_url="postgresql://{{username}}:{{password}}@192.0.2.14:5432/openbao-example-database" \
    username="openbao-user" \
    password="anothertotallysecurepassword" \
    password_authentication="scram-sha-256"

openbao write database/roles/vagrant-libvirt-role \
    db_name="vagrant-libvirt-postgresql-database" \
    creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; \
    GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\";" \
    default_ttl="1h" \
    max_ttl="24h"

openbao read database/config/vagrant-libvirt-postgresql-database
openbao read database/roles/vagrant-libvirt-role

openbao read database/creds/vagrant-libvirt-role

# replace XXX with the user from the last command
psql -h 192.0.2.14 -d openbao-example-database -U XXX
```

This will allow you to get a new set of database credentials from OpenBao via
`openbao read database/creds/vagrant-libvirt-role`, that you can use to connect to
the database from both the OpenBao server VM or the client VM.

## Cleaning up

The VMs can be torn down after playing around using `vagrant destroy`. There is
one file that needs to be removed manually. This file contains the OpenBao root
token and is located at `ansible/openbao_root_token`.

## Disabling the Ansible provisioning

In case you do not want Ansible to install OpenBao (because you want to install
it yourself), just comment out the following lines in the `Vagrantfile`:

```hcl
    node.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
          ansible.limit = "all"
      ansible.playbook = "ansible/playbook-vagrant.yml"
    end # node.vm.provision
```

You also find all of the playbooks in the `ansible` folder.

## License

BSD-3-Clause

## Author Information

I am Johannes Kastl, reachable via git@johannes-kastl.de
