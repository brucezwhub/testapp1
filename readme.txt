--enable managed identity for testapp1
--get the id
b3d6e78d-54f8-4876-a6b6-072f0270c400
----grant permission in database
CREATE USER brdemotestapp1 FROM EXTERNAL PROVIDER;
ALTER ROLE db_datareader ADD MEMBER brdemotestapp1;
ALTER ROLE db_datawriter ADD MEMBER brdemotestapp1;
ALTER ROLE db_ddladmin ADD MEMBER brdemotestapp1;

CREATE USER [brdemotestapp1] FROM EXTERNAL PROVIDER;
ALTER ROLE db_datareader ADD MEMBER [brdemotestapp1];
ALTER ROLE db_datawriter ADD MEMBER [brdemotestapp1];
ALTER ROLE db_ddladmin ADD MEMBER [brdemotestapp1];


---configure environment for testapp1
$db_server = getenv("DB_SERVER");   brdemosqltestfailover.database.windows.net
$db_name = getenv("DB_NAME");   brsqltestdb01 
$location = getenv("LOCATION");  CanadaCentral

---install sqlcmd in ubuntu
https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-setup-tools?view=sql-server-ver16#ubuntu
curl https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
curl https://packages.microsoft.com/config/ubuntu/20.04/prod.list | sudo tee /etc/apt/sources.list.d/msprod.list

sudo apt-get update 
sudo apt-get install mssql-tools unixodbc-dev
sqltestadmin
-------------------------opt in ARM beta
azurerm version is 2.76+
export ARM_THREEPOINTZERO_BETA_RESOURCES=true
-------------
sqlcmd -S brdemosqltestfailover.database.windows.net -d brsqltestdb01 -G -C -U bzhang.admin@ialab.ca -P  << EOSQL
CREATE USER brdemotestapp1 FROM EXTERNAL PROVIDER;
ALTER ROLE db_datareader ADD MEMBER brdemotestapp1;
ALTER ROLE db_datawriter ADD MEMBER brdemotestapp1;
ALTER ROLE db_ddladmin ADD MEMBER brdemotestapp1;
GO
exit
EOSQL

sqlcmd -S ${db_server}.database.windows.net -d $db -G -C -U bzhang.admin@ialab.ca -P  <<EOSQL
CREATE USER [${app}] FROM EXTERNAL PROVIDER;
ALTER ROLE db_datareader ADD MEMBER [${app}];
ALTER ROLE db_datawriter ADD MEMBER [${app}];
ALTER ROLE db_ddladmin ADD MEMBER [${app}];


----------gpg key
https://medium.com/@davidpiegza/using-pass-in-a-team-1aa7adf36592
https://blog.gruntwork.io/a-comprehensive-guide-to-managing-secrets-in-your-terraform-code-1d586955ace1
gpg --list-secret-keys

pass azure-ialab-username
pass azure-ialab-password

# Read secrets from pass and set as environment variables
export TF_VAR_azureialabusername=$(pass azure-ialab-username)
export TF_VAR_azureialabpassword=$(pass azure-ialab-password)


bruce@terraform:~/demo/az_sql_app$ gpg --default-new-key-algo rsa4096 --gen-key
gpg (GnuPG) 2.2.19; Copyright (C) 2019 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Note: Use "gpg --full-generate-key" for a full featured key generation dialog.

GnuPG needs to construct a user ID to identify your key.

Real name: brucezw@gmail.com
Email address: brucezw@gmail.com
You selected this USER-ID:
    "brucezw@gmail.com <brucezw@gmail.com>"

Change (N)ame, (E)mail, or (O)kay/(Q)uit? N
Real name: Bruce Zhang
You selected this USER-ID:
    "Bruce Zhang <brucezw@gmail.com>"

Change (N)ame, (E)mail, or (O)kay/(Q)uit? O
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: key 724498ADD0E930CA marked as ultimately trusted
gpg: directory '/home/bruce/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/home/bruce/.gnupg/openpgp-revocs.d/EC4D1B4B528C35CF2F155CB2724498ADD0E930CA.rev'
public and secret key created and signed.

Note that this key cannot be used for encryption.  You may want to use
the command "--edit-key" to generate a subkey for this purpose.
pub   rsa4096 2022-10-31 [SC] [expires: 2024-10-30]
      EC4D1B4B528C35CF2F155CB2724498ADD0E930CA
uid                      Bruce Zhang <brucezw@gmail.com>
