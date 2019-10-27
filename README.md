# knox-config
## Prerequisite: 
1. ldapbind user should be created in ipa server with admin access. 
2. test the ldapsearch for the user


Step1: Setup the LDAP
```
[root@masternode ~]# ambari-server setup-ldap
Using python  /usr/bin/python
Setting up LDAP properties...
Primary URL* {host:port} (ipa-server.us-west2-a.c.dark-park-255906.internal:389): 
Secondary URL {host:port} : 
Use SSL* [true/false] (false): 
User object class* (posixAccount): 
User name attribute* (uid): 
Group object class* (posixGroup): 
Group name attribute* (cn): 
Group member attribute* (memberUid): 
Distinguished name attribute* (distinguishedName): 
Base DN* (cn=users,cn=accounts,dc=dark-park-255906,dc=internal): cn=users,cn=accounts,dc=us-west2-a,dc=c,dc=dark-park-255906,dc=internal
Referral method [follow/ignore] (ignore): 
harsha,swamy
Bind anonymously* [true/false] (false): 
Handling behavior for username collisions [convert/skip] for LDAP sync* (skip): 
Manager DN* (uid=ldapbind,cn=users,cn=accounts,dc=dark-park-255906,dc=internal): uid=ldapbind,cn=users,cn=accounts,dc=us-west2-a,dc=c,dc=dark-park-255906,dc=internal
Enter Manager Password* : 
Re-enter password: 
====================
Review Settings
====================
authentication.ldap.managerDn: uid=ldapbind,cn=users,cn=accounts,dc=us-west2-a,dc=c,dc=dark-park-255906,dc=internal
authentication.ldap.managerPassword: *****
Save settings [y/n] (y)? y
Saving...done
Ambari Server 'setup-ldap' completed successfully.
```

```
[root@masternode ~]# ambari-server restart
Using python  /usr/bin/python
Restarting ambari-serveruid=ldapbind,cn=users,cn=accounts,dc=us-west2-a,dc=c,dc=dark-park-255906,dc=internal
```

### 2. Test the user for ldapsearch to make sure it is available in ipa server 

```
[root@masternode ~]# ldapsearch -x -h ipa-server.us-west2-a.c.dark-park-255906.internal  -b cn=users,cn=accounts,dc=us-west2-a,dc=c,dc=dark-park-255906,dc=internal uid=swamy

# swamy, users, accounts, us-west2-a.c.dark-park-255906.internal
dn: uid=swamy,cn=users,cn=accounts,dc=us-west2-a,dc=c,dc=dark-park-255906,dc=i
 nternal
displayName: swamy gurram
cn: swamy gurram
objectClass: top
objectClass: person
objectClass: organizationalperson
objectClass: inetorgperson
objectClass: inetuser
objectClass: posixaccount
objectClass: krbprincipalaux
objectClass: krbticketpolicyaux
objectClass: ipaobject
objectClass: ipasshuser
objectClass: ipaSshGroupOfPubKeys
objectClass: mepOriginEntry
loginShell: /bin/sh
initials: sg
gidNumber: 321600000
gecos: swamy gurram
sn: gurram
homeDirectory: /home/swamy
uid: swamy
givenName: swamy
uidNumber: 321600003

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
[root@masternode ~]# 
```

### 3. Create the user list which are already part of ldap
```
[root@masternode ~]# cat users.txt 
harsha,swamy
[root@masternode ~]# 
```
### 4. Sync the users 
```
[root@masternode ~]# ambari-server sync-ldap --users users.txt
Using python  /usr/bin/python
Syncing with LDAP...
Enter Ambari Admin login: admin
Enter Ambari Admin password: 
Syncing specified users and groups...

Completed LDAP Sync.
Summary:
  memberships:
    removed = 0
    created = 0
  users:
    skipped = 1
    removed = 0
    updated = 0
    created = 1
  groups:
    updated = 0
    removed = 0
    created = 0
```



# KNOX - Config changes: 

### Advanced topology: 

```
uid={0},cn=users,cn=accounts,dc=us-west2-a,dc=c,dc=dark-park-255906,dc=internal

<param>
    <name>main.ldapRealm.userDnTemplate</name>
    <value>uid={0},cn=users,cn=accounts,dc=us-west2-a,dc=c,dc=dark-park-255906,dc=internal</value>
</param>

<param>
    <name>main.ldapRealm.contextFactory.url</name>
    <value>ldap://ipa-server.us-west2-a.c.dark-park-255906.internal:389</value>
</param>


<role>AMBARI</role>
   <url>http://ip-172-31-26-231.us-west-1.compute.internal:8080</url>
</service>

<service>
   <role>AMBARIUI</role>
   <url>http://ip-172-31-26-231.us-west-1.compute.internal:8080</url>
</service>

```

https://masternode.us-west2-a.c.dark-park-255906.internal:8443/gateway/knoxsso/api/v1/websso
# sso certificate
```
[root@ip-172-31-26-231 bin]# ./knoxcli.sh export-cert --type PEM
Certificate gateway-identity has been successfully exported to: /usr/hdp/3.1.4.0-315/knox/data/security/keystores/gateway-identity.pem
[root@ip-172-31-26-231 bin]#
[root@ip-172-31-26-231 bin]# cat /usr/hdp/3.1.4.0-315/knox/data/security/keystores/gateway-identity.pem
```

# Setup the Ambari SSO: 

```
[root@masternode knox]# ambari-server setup-sso
Using python  /usr/bin/python
Setting up SSO authentication properties...
Do you want to disable SSO authentication [y/n] (n)?y
Ambari Server 'setup-sso' completed successfully.
[root@masternode knox]# ambari-server setup-sso
Using python  /usr/bin/python
Setting up SSO authentication properties...
Do you want to configure SSO authentication [y/n] (y)?y
Provider URL [URL] (https://masternode.us-west2-a.c.dark-park-255906.internal:8443/gateway/knoxsso/api/v1/websso):
Public Certificate pem (stored) (empty line to finish input):

Do you want to configure advanced properties [y/n] (n) ?y
JWT Cookie name (hadoop-jwt):
JWT audiences list (comma-separated), empty for any ():
Ambari Server 'setup-sso' completed successfully.
```