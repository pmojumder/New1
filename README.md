Hi Plabani,
Using the definitions below can we write a test app to generate a sha256 for the row data that allignes with what we expect in Teradata.
Any Q’s ping me directly.
Richard.


From: Andrew Carr <Andrew.X.Carr@aib.ie> 
Sent: Friday 23 May 2025 14:17
To: Richard Harnett <Richard.P.Harnett@aib.ie>
Subject: RE: Testing strategy

Hi,
Preference Centre.

Hive DDL vs EDW DDL
Challenges:
•	17 Hive fields not in EDW table ddl.
•	8 Hive fields are strings but microsecond timestamps in EDW (green highlight)
o	The Hive strings look like timestamps but the seconds precision varies from field to field so Hive side preparation has to be carefully done.
•	Hive load_date and load_time are in a different order to their mapped EDW fields srce_load_dte and srce_load_time (blue and orange highlight)
•	6 EDW fields not in Hive which are the mandatories on all EDW tables so can be ignored.

These are good use cases.

 

 

 

 
