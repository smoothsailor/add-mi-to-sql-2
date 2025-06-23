# Composite action

## Name

Add a managed identity to a SQL Server database.

## Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `uai-name` | `true` | `N/A` | The name of the user assigned identity which needs to be added as a user to the database. |
| `uai-resource-group` | `true` | `N/A` | The resource group of the user assigned identity which needs to be added as a user to the database. |
| `sql-server-name` | `true` | `N/A` | The name of the SQL Server to which will be connected with the workflow's logged in user. |
| `sql-database-name` | `true` | `N/A` | The name of the SQL database to which will be connected with the workflow's logged in user to add the user assigned identity to. |
| `uai-subscription-name` | `false` | `N/A` | The name of the subscription of the user assigned identity which needs to be added as a user to the database. |

## Purpose

This composite action adds a managed identity to a SQL Server's database. This way you are able to use workload identity (within AKS) to let an app authenticate to a SQL database.  

In order for this composite action to work properly, make sure to:
1. Create a security group in Microsoft Entra ID
2. Add the user which is used by the workflow (via `azure/login`) to the created security group  
3. Add the security group as `Microsoft Entra admin` to the SQL Server
   > if steps 2 and 3 are not executed, you will receive message `Login failed for user '<token-identified principal>'.` ([check second bullet in this answer](https://stackoverflow.com/a/69860286/2441681))

## Verify user creation

If you would like to verify whether the user is properly created, log in to the provided database ( the value of input `sql-database-name`, not `master`).  
Execute the following query:  
```sql
select name,
       type_desc,
       authentication_type_desc,
       create_date,
       sid,
       convert(uniqueidentifier, sid) as sidAsGuid
from sys.database_principals
where name = N'<enter value of input uai-name here>';
```

The user will be visible and the value of column `sidAsGuid` will be equal to the user assigned identity's client ID.
