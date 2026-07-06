It can be beneficial to place ourselves in the perspective of an IT administrator when we are on an engagement. This mindset can help us remember to look for various settings that may have been misconfigured or configured in a dangerous manner by an admin. A workday in IT can be rather busy, with lots of different projects happening simultaneously and the pressure to perform with speed & accuracy being a reality in many organizations, mistakes can be easily made. It only takes one tiny misconfiguration that could compromise a critical server or service on the network. This applies to just about every network service and server role that can be configured, including MSSQL.

This is not an extensive list because there are countless ways MSSQL databases can be configured by admins based on the needs of their respective organizations. We may benefit from looking into the following:

- MSSQL clients not using encryption to connect to the MSSQL server
- The use of self-signed certificates when encryption is being used. It is possible to spoof self-signed certificates
- The use of [named pipes](https://docs.microsoft.com/en-us/sql/tools/configuration-manager/named-pipes-properties?view=sql-server-ver15)
- Weak & default `sa` credentials. Admins may forget to disable this account