# apilabs

### dxtoolkit download
[dxtoolkit](https://drive.google.com/file/d/17GYvj_-nLcNDdU1iCG2rQTuARczSi_QT/view?usp=sharing) dxtoolkit for RH6


### config file
  
```  
  {
   "data" : [
      {
         "encrypted" : "false",
         "default" : "true",
         "protocol" : "http",
         "ip_address" : "10.0.X.10",
         "hostname" : "labengine",
         "timeout" : "60",
         "password" : "delphix",
         "port" : "80",
         "username" : "admin"
      }
   ]
  }
```

### lab 

1. Open a terminal
2. Log as delphix user to 10.0.X.30 where X is your lab number
 `ssh delphix@10.0.X.30` 

3. Change directory to dxtoolkit 
  `cd dxtoolkit2`
4. Set an Oracle environment - use `devdb` as ORACLE_SID
   `. oraenv`
5. Run commands from [lab](lab.txt) Labs


