**RBAC with Row Access Control Research**

**View with different view names**

Users can create different views for different user groups.

Cons:

DBA at customer side needs to create multiple views for each new group
of user.

It's almost the same as creating different tables for different groups
of users. Very inconvenient to customers and merely fulfilling their
requirements.

Pro:

This one does not require any new function implemented. We can achieve
this based on the current function provided by Spark or any database.

**Dynamic View with the same view name and filtering
functions(Databricks or MariaDB)**

Very similar to the above option, but this time we can create view on
the table with **where** condition based on the current user/session
information. So the view name will be the same but the content shown to
each user will be different based on the condition.

Sample SQL in MariaDB with defining rules inside a user defined
function:

<table>
<tbody>
<tr class="odd">
<td><p>SQL</p>
<p>CREATE</p>
<p>SQL SECURITY DEFINER</p>
<p>VIEW protected.important_data</p>
<p>AS</p>
<p>SELECT *</p>
<p>FROM unprotected.important_data uid</p>
<p>WHERE accesses.access_check(SESSION_USER(), uid.security_label)</p>
<p>WITH CHECK OPTION;</p></td>
</tr>
</tbody>
</table>

Sample SQL in Databricks with appending rules condition after **where**
directly:

<table>
<tbody>
<tr class="odd">
<td><p>SQL</p>
<p><strong>CREATE</strong> <strong>VIEW</strong> sales_redacted <strong>AS</strong></p>
<p><strong>SELECT</strong></p>
<p>user_id,</p>
<p>country,</p>
<p>product,</p>
<p>total</p>
<p><strong>FROM</strong> sales_raw</p>
<p><strong>WHERE</strong></p>
<p><strong>CASE</strong></p>
<p><strong>WHEN</strong> is_member('managers') <strong>THEN</strong> <strong>TRUE</strong></p>
<p><strong>ELSE</strong> total &lt;= 1000000</p>
<p><strong>END</strong>;</p></td>
</tr>
</tbody>
</table>

Cons:

Spark does not provide functions like is\_member or current\_user
function to retrieve current user/session's information. We need to
implement and externalize this to the user.

DBA still need to manually create a view for each new table they want to
apply the row access control.

Pro:

We only need to implement small amount of functions to externalize the
session information to the user.

**Build-in Rule or Permission feature**

Beside **View**, many databases also have their own integrated
permission system to control the object accessibility.

In IBM DB2, the concept is called FGAC/RCAC.

In SQL server, it's called Row Level Security.

Sample of IBM DB2:

<table>
<tbody>
<tr class="odd">
<td><p>SQL</p>
<p>CREATE or REPLACE PERMISSION check_region on db2inst1.sales for rows</p>
<p>WHERE VERIFY_ROLE_FOR_USER (USER, 'DBA') = 1</p>
<p>OR USER = 'CLIENT1' and REGION like 'Ontario%'</p>
<p>OR USER = 'CLIENT2' and REGION = 'Quebec'</p>
<p>ENFORCED FOR ALL ACCESS</p>
<p>ENABLE</p>
<p>ALTER TABLE db2inst1.sales activate row access control</p></td>
</tr>
</tbody>
</table>

Sample in SQL Server:

<table>
<tbody>
<tr class="odd">
<td><p>SQL</p>
<p>CREATE FUNCTION Security.fn_securitypredicate(@SalesRep AS sysname)</p>
<p>RETURNS TABLE WITH SCHEMABINDING AS</p>
<p>RETURN SELECT 1 AS fn_securitypredicate_resultWHERE @SalesRep = USER_NAME() OR USER_NAME() = 'Manager';</p>
<p>CREATE SECURITY POLICY SalesFilter ADD FILTER PREDICATE Security.fn_securitypredicate(SalesRep)ON dbo.Sales WITH (STATE = ON);</p></td>
</tr>
</tbody>
</table>

Overall, this approach has its own abstract level and wrap everything
into one centre place compared with the **View** approach.

Cons:

If we need to provide our own abstract level/permission system, the
workload will be huge compared with the above options.

Pro:

We can turn on or turn off the row access control more easily. We also
have a central place to configure all security/permision/filtering
rules' behaviour if necessary.

If we are introducing other permission/security rules in the future(like
column access control), we can use the same interface and syntax with
minimum changes to the customer (replacing the keyword for rows to for
columns for example). Easy for the extension.
