Install and setup PostgreSQL
<https://www.youtube.com/watch?v=67XGzdzv9k0&index=54&list=PLapAp_ElzBH1EE7mU1-Pk1zrL6sn3bwl6>

1. When PostgreSQL in installed, a user named "postgres" and a database "postgres" were created. 'postgres' is a super user and the admin of the 'postgres' database.
2. Switch to user "postgres": sudo su postgres
3. Log in to the database "postgres": psql postgres
4. create two users app_ro and app_rw:
    CREATE USER app_ro WITH PASSWORD '1234';
    CREATE USER app_rw WITH PASSWORD '1234';
Note: Now we should user CREATE ROLE, instead of CREATE USER
5. Change password for a user
    ALTER USER app_rw WITH PASSWORD '5678';
6. Create a database:
    CREATE DATABASE myapp;
7. Connect to the created database:
    \c myapp
    The following message will be printed
    "You are now connected to database "myapp" as user "postgres""

8. Revoke permission to myapp database from users other than admin. What are the default permissions?
    REVOKE ALL ON DATABASE myapp FROM PUBLIC;
    REVOKE ALL ON SCHEMA myapp FROM PUBLIC;    

9. List database roles(users) and their attributes(premissions):
    \du

10. Grant connection permissions:
    GRANT CONNECT ON DATABASE myapp to app_ro;
    GRANT CONNECT ON DATABASE myapp to app_rw;

11. List database, roles(users) and their attributes(premissions) again:
    \l

12. Grant some other permissions:
    GRANT SELECT ON ALL TABLES IN SCHEMA public to app_ro;
    GRANT SELECT, UPDATE, INSERT, DELETE ON ALL TABLES IN SCHEMA public to app_rw;

13. Check database, roles(users) and their attributes(premissions) again:
   \l
   **problem**
   The permission of app_ro and app_rw did not change. The reason is that their permission are related to tables, however, in the myapp database no tables has been created yet.

14. Create a table *test* in the database myapp:
   CRATE TABLE test(id serial, name varchar(40));

15. Check user's permission to the *test* table
   \c
   **Problem**
   You can see app_ro and app_rw are not granted user for this table.
   The reason is the same -- whenever you grant permission, they grant permission to table that do exist, not future tables.

16. Grant permissions:
    GRANT SELECT ON ALL TABLES IN SCHEMA public to app_ro;
    GRANT SELECT ON ALL TABLES IN SEQUENCES public to app_ro;
    GRANT USAGE ON SCHEMA public to app_ro;

    GRANT SELECT, UPDATE, INSERT , DELETE ON ALL TABLES IN SCHEMA public to app_rw;
    GRANT SELECT, UPDATE ON ALL SEQUENCES IN SCHEMA public to app_rw;
    GRANT USAGE ON SCHEMA public to app_rw;

17. Check permissions:
   \z

18. Problem: if I create a new table, I need to repeat all these, which is not efficient.

19. Example, create a new table *test_2*
   CREATE TABLE test_2(id serial, name varchar(40));
   \z # check permissions

20. permissions to tables are granted by the user who create the table

21. Solution: just change the default permission setting
    \c myapp  # connect to myapp database
    ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT SELECT ON TABLES to app_ro;
    ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT SELECT ON SEQUENCES TO app_ro;
    ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT SELECT, UPDATE, INSERT, DELETE ON TABLES TO app_rw;
    ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT SELECT, UPDATE ON SEQUENCES TO app_rw;

22. Now, if you create new tables, the users will have the previously granted permissions to the tables.

23. log in to the myapp database with app_ro user
    psql -h localhost -U app_ro myapp

24. Now I logged into the database myapp with the user name app_ro.
We can check what I can do with the table.
    SELECT * FROM test; # I can SELECT

25. switch to postgres user to add some rows to test table.
    \q # quit from psql
    exit # exit from current user
    sudo su postgres # switch user
    \c myapp   # connect to myapp database
    INSERT INTO test(name) Values('test me');
    INSERT INTO test(name) Values('test me too');
    SELECT * from test;

26.\q
    exit
    psql -h localhost -U app_ro myapp
    SELECT * FROM test;
    CREATE TABLE table_name # app_ro has no permission to create table
    insert into test(name) values ('cannot inser') # app_ro has no permission to insert values



2
