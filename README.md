## LDAP OpenSSH AuthorizedKeysCommand

# What's AuthorizedKeysCommand?

One great thing about OpenSSH is the support for using an AuthorizedKeysCommand statement in sshd\_config to get the authorized_keys file for a user. This lets us store your authorized\_keys files in LDAP without actually linking OpenSSH against OpenLDAP. AuthorizedKeysCommands script should take one argument and return the authorized key(s).

# What's this utility about?

This is a small utility that I wrote in order to solve a very specific problem. I've got an OpenLDAP server which contains a user group that needs access to a linux server. Using this utility, I'm able to get the public key for each user and allow them access to the linux server.

# I wanna use it, do you assume anything about my environment?

Yes, plenty of things. You're using a LDAP server where you store the users public key, and you only have one group which needs access to the server. Your LDAP server allows for SASL GSS-API authentication. There might be more things that's kinda hard-coded, feel free to make changes and please send me a pull request.

# SASL GSS-API authentication

Yeah, great stuff. Make sure to create a kerberos keytab to use with this utility in case you want to use it with OpenSSH.

Example for creating keytab under linux:
	
	ktutil
	addent -password -p directoryagent@EXAMPLE.COM -k 1 -e aes128-cts-hmac-sha1-96
	Password for directoryagent@EXAMPLE.COM:
	********
	ktutil:  wkt /etc/directoryagent.keytab
	ktutil:  q

# Installation and configuration
You may just edit the config-template.yml and rename it to config.yml in order to test the script. But if you want it installed and configured for use with OpenSSH, I'd suggest first editing the `config-template.yml` and then run:

	cp config-template.yml /etc/ldap_ssh_authorizedkeys.yml
	chown root:root /etc/ldap_ssh_authorizedkeys.yml /usr/local/sbin/ldap_ssh_authorizedkeys
	touch /var/log/ldap_ssh_authorizedkeys.log
	chmod 600 /etc/ldap_ssh_authorizedkeys.yml /usr/local/sbin/ldap_ssh_authorizedkeys
	chmod 500 /usr/local/sbin/ldap_ssh_authorizedkeys
	
Also make sure that you create a read-only LDAP user.

# Configuration for OpenSSH
Add these two lines to your `/etc/ssh/sshd_config`

	AuthorizedKeysCommandUser root
	AuthorizedKeysCommand /usr/local/sbin/ldap_ssh_authorizedkeys

# Debug SSHD
Start sshd in debug mode and connect from a client

On server:
`/usr/local/sbin/sshd -ddd -p23`

On client:
`ssh -vvv -opubkeyauthentication=yes -p23 username@example.com`

And search for AuthorizedKeysCommand in the console output at the server.

