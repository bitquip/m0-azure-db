# Standing up a SQL Server and Database in Azure using Pulumi

## Intro

>__This course will guide you through creating a SQL Server and Database in Azure Cloud using Pulumi.__ <br>
__This guide aims to provide the required steps to correctly configure a new database in Azure for your team at Vizient.__

__Here are some resources in Confluence for additional information around many of the resources and topics discussed in this guide:__

> Resource list here <br>
> Another resource <br>
> They need links

---

## Required Cloud Resources

>#### SQL Server
>#### SQL Database
>#### DevT1 Update Perms (?)
>#### Database User(s)

__All of the listed resources above should come from Vizient's Azure Native Node.JS package:__

 >`@vizientinc/pulumi-azure-native`

__These resources have methods with names such as:__

>`DatabaseUserVzn` <br> <br>
 `DatabaseVzn`

__Notice the `Vzn` in the name. These are the newer wrapper functions the Cloud Operations team have implemented around the Azure Native Pulumi modules.__

__We will use these wrapper functions to include the necessary "under-the-hood"
configurations put in place by Cloud Operations for any of the resources we are implementing.__

---

## SQL Server

#### Module: `@vizientinc/pulumi-azure-native`
#### Method: `sql.ServerVzn`
#### Example: 
```typescript
import {Naming as VizNaming, sql} from '@vizientinc/pulumi-azure-native';

export const sqlServer = new sql.ServerVzn('pulumi-resource-name', {
    appInfo: {
      region: location,
      resourceGroupName: resourceGroup.name,
      serviceTier: serviceTier,
      tags: tags,
    },
    resourceName: VizNaming.SqlServer(app, env, location, instance),
    allowAllAzureTraffic: false,
    allowedVizientLocations: [
      LocationCode.cap,
      LocationCode.ae2,
    ],
    allowedSubnetIDs: vnetIDsSql2Allow,
    subscriptionId: subscriptionId,
    tenantId: tenantId,
  },
  {
    protect: false,
  }
});
```
---

## SQL Database
#### Module: 
`@vizientinc/pulumi-azure-native`

#### Method: 
`sql.DatabaseVzn`

#### Example: 
```typescript
export const myDatabase = new sql.DatabaseVzn(
  'pulumi-name-for-database',
  {
    createAADGroupsInSandbox: true,
    resourceName: daasApiDbName, //specify name for DB
    server: sqlServer.sqlServer,
    databaseSku: {
      name: 'S3', //read from config
    },
    maxSizeInGiB: 100,
    appInfo: {
      region: location,
      resourceGroupName: resourceGroup.name,
      tags: tags,
      serviceTier: serviceTier,
    },
  },
  {
    parent: sqlServer.sqlServer,
    dependsOn: sqlServer.sqlServer,
  }
);
```
---

## SQL User(s)

#### Module: 
`@vizientinc/pulumi-azure-native`

#### Method: 
`sql.DatabaseUserVzn`

#### Example: 

```typescript
export const devPortalServiceDaasApiDbUser = new sql.DatabaseUserVzn(
  `${app}-${daasApiDbName}-dbuser-new-server-user`,
  {
    roles: dbRoles,
    username: devPortalServiceName,
    databases: [daasApiDb.database.name],
    server: sqlServer.sqlServer.fullyQualifiedDomainName,
    sid: sid,
    permissions: executePermission,
  },
  {
    dependsOn: [
      sqlServer.sqlServer,
      daasApiDb.database,
    ],
  }
);
```

---

## Database Roles

#### Module:
 `@vizientinc/pulumi-azure-native`
#### Method: 
 `sql.ServerVzn`
#### Example: 
```typescript
//DevT1 Update Permissions
export const t1DaasApiDbUpdateGroup = new GroupMember(
  `${daasApiDbName}-t1-devs-db-update`,
  {
    groupObjectId: daasApiDb.updateGroup!.objectId,
    memberObjectId: t1Group.apply((g: { objectId: any }) => g.objectId),
  },
  {
    dependsOn: [daasApiDb.database],
  }
);

const executePermission: {
  permission: string;
  // object?: Input<string>;
}[] = [{ permission: 'EXECUTE' }];

let dbRoles = ['db_datareader', 'db_datawriter', 'db_ddladmin'];
if (env != 'prod' && env != 'preprod') {
  dbRoles = ['db_owner'];
}

// This is sid of devportal service, which is provisioned in stateless resources
const sid = output(
    getServicePrincipal({
      objectId: devPortalPid,
    })
  )
  .apply((s: { applicationId: any }) => s.applicationId);
```
---

## Things to consider

---

## Conclusion

