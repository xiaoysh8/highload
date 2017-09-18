SSH Security Settings

If you do not want to be broken, use a few simple settings when working with ssh. These settings must be changed in the sshd_config file :

1. Disable password authorization

Instead, use authorization by key .

PasswordAuthentication no
2. Use the IP restriction

AllowUsers = *@123.123.123.123
# We will allow access only for ip 123.123.123.123

3. Only protocol version "2"

Protocol 2
4. Disable the root login

It is better to use login under specific users.

PermitRootLogin no
5. Change the ssh port

The ports will still be scanned, but they will not be detected immediately.

Port 2200
After editing, restart ssh (current sessions will not abort):

service ssh restart
