#dx_provision_vdb -d labengine -type oracle -group "DB Targets" -creategroup -sourcename "orcl"  -srcgroup "DB Sources" -targetname "dxtest"  -dbname "dxtest" -environment "Target"  -envinst "/u01/app/oracle/product/11.2.0/dbhome_1"  -envUser "delphix" -redoGroup 3 -redoSize 50 -archivelog=yes -mntpoint "/mnt/provision" -instname dxtest -uniqname dxtest

#./dx_ctl_js_container -d labengine -action create -dontrefresh -container_name "dxtest container" -template_name "orcl template" -container_owner "qa" -container_def "DB Targets,dxtest"

# EX 1

## List Containers
```
./dx_get_js_containers -d labengine
```

## Create "First Bookmark"
```
./dx_ctl_js_bookmarks -d labengine -bookmark_name "first bookmark" -action create -container_name "devdb container" -template_name "orcl template"  -bookmark_time latest
```

## List Bookmarks
```
./dx_get_js_bookmarks -d labengine
```

## Make a change in VDB
```
sqlplus / as sysdba
create table beforetest(id number);
insert into beforetest values (1);
commit;
exit
```

## Create Bookmark "Before Test" using latest timestamp and set expiration
```
./dx_ctl_js_bookmarks -d labengine -bookmark_name "before test" -action create -container_name "devdb container" -template_name "orcl template"  -bookmark_time latest -expireat "2021-08-30"

./dx_get_js_bookmarks -d labengine -container_name "devdb container"
```

## Make another change in VDB
```
sqlplus / as sysdba
create table nexttest(id number);
insert into nexttest values (1);
commit;
exit
```

## Create Bookmark "Before Test" using realtime
```
./dx_get_js_bookmarks -d labengine -bookmark_name "before test" -realtime

./dx_get_js_snapshots -d labengine
```

## Remove Bookmark
```
./dx_ctl_js_bookmarks -d labengine -bookmark_name "first bookmark" -action remove -container_name "devdb container"
```

# EX 2

## Refresh Container
```
./dx_ctl_js_container -d labengine -action refresh -container_name "devdb container" -template_name "orcl template"
```

## Verify Data from exercise 1.
```
sqlplus / as sysdba
select * from beforetest;
select * from nexttest;
exit

./dx_get_js_snapshots -d labengine
```

## Add new data
```
sqlplus / as sysdba
create table test1(id number);
insert into test1 values (1);
commit;
exit
```

## Create Bookmark "Rollback"
```
./dx_ctl_js_bookmarks -d labengine -bookmark_name "rollback" -action create -container_name "devdb container" -template_name "orcl template"  -bookmark_time latest -expireat "2021-08-30"
```

## Restore VDB to Point-In-Time (using fromtemplate option)
```
./dx_ctl_js_container -d labengine -action restore -container_name "devdb container" -template_name "orcl template" -timestamp "2021-08-17 01:31:31" -fromtemplate

sqlplus / as sysdba
select * from test1;
exit
```

## Restore VDB to Bookmark "Rollback"
```
./dx_ctl_js_container -d labengine -action restore -container_name "devdb container" -template_name "orcl template" -timestamp rollback

sqlplus / as sysdba
select * from test1;
exit
```

## Restore VDB to Point-In-Time
```
./dx_ctl_js_container -d labengine -action restore -container_name "devdb container" -template_name "orcl template" -timestamp "2021-08-17 01:05:00"

sqlplus / as sysdba
select * from test1;
exit
```

# EX3

## Create Branch
```
./dx_ctl_js_branch -d labengine -action create -container_name "devdb container" -template_name "orcl template" -branch_name "APP1" -timestamp rollback

./dx_get_js_snapshots -d labengine
```

## Add data in VDB(devdb) and create a bookmark
```
sqlplus / as sysdba
create table bug123(id number);
insert into bug123 values (1);
commit;
exit

./dx_ctl_js_bookmarks -d labengine -bookmark_name "bug123" -action create -container_name "devdb container" -template_name "orcl template"  -bookmark_time latest -expireat "2021-08-30"
```

## Share a bookmark
### This command will fail as bookmark is not shared
```
./dx_ctl_js_branch -d labengine -action create -container_name "qadb container" -template_name "orcl template" -branch_name bug123 -timestamp bug123 

./dx_ctl_js_bookmarks -d labengine -bookmark_name "bug123" -action share -container_name "devdb container"

./dx_ctl_js_branch -d labengine -action create -container_name "qadb container" -template_name "orcl template" -branch_name bug123 -timestamp bug123 
```

## Verify data in VDB(qadb)
```
. oraenv
(answer with qadb)
sqlplus / as sysdba
select * from bug123;
exit
```

## Activate original VDB(qadb) branch
```
./dx_get_js_branches -d labengine 

./dx_get_js_branches -d labengine -container_name "qadb container"

./dx_ctl_js_branch -d labengine -container_name "qadb container" -action activate -branch_name default
```
