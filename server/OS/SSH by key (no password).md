SSH by key (no password)

If you still use SSH access by password, change the habit immediately. Is it dangerous. It is better to useaccess by key.

1. Generate the key

Locally on your wheelbarrow you need to create a key:

ssh-keygen -t rsa -b 2048
# Creates a 2048 bit RSA key

2. Download the key

You need to upload the public key to the server:

ssh root@123.123.123.123 mkdir -p .ssh
cat ~/.ssh/id_rsa.pub | ssh root@123.123.123.123 'cat >> .ssh/authorized_keys'
# Folder ".ssh" can exist, that ok

3. Done

Now we can login without a password:

ssh root@123.123.123.123
