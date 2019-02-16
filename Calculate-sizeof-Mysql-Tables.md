
<h1>How to calculate the size of a MySQL database</h1>
<a target='_blank' href='https://www.codediesel.com/mysql/how-to-calculate-the-size-of-a-mysql-database/'>Author Page</a>

<pre>
SELECT  SUM(((DATA_LENGTH + INDEX_LENGTH)/1024/1024)) AS "MB"
        FROM INFORMATION_SCHEMA.TABLES
	WHERE TABLE_SCHEMA = "wordpress";
</pre>
    
    
<div>To calculate sizes for all the database on your server use the following.</div> 

<pre>
SELECT TABLE_SCHEMA as 'Database', 
       SUM(((DATA_LENGTH + INDEX_LENGTH)/1024)) AS "KB"
       FROM INFORMATION_SCHEMA.TABLES
       GROUP BY table_schema
</pre>
