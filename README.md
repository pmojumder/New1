Hive Query:
sql
Copy
Edit
SELECT * FROM boxever_sessions;
Teradata Query:
sql
Copy
Edit
SELECT * FROM boxever_sessions;
Comparison Query:
sql
Copy
Edit
SELECT * FROM hive.boxever_sessions 
MINUS 
SELECT * FROM teradata.boxever_sessions;
