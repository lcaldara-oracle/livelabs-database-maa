# Provision target database

## Introduction

You must create a placeholder target database before beginning a migration to the target environment. 

The placeholder target database is overwritten during migration, but it retains the overall configuration.

Please note as of Zero Downtime Migration Release 21.3 , only Grid Infrastructure-based database services are supported as targets. For example, an LVM-based instance or an instance created in compute node without Grid Infrastructure are not supported targets.

You must use the control plane for the creation of a target placeholder database on Exadata Database Service on Dedicated Infrastructure and Exadata Cloud at Customer.

Estimated Time: 30 minutes

### Objectives

In this lab

* You will collect some information from the source database that is required for target database provisioning.

* You will prepare a database software image for the target database.

* You will provision an Oracle Base Database VM to use as the target database system.

### Prerequisites

* All previous labs have been successfully completed.

## Task 1 : Collect Source Database Details

1. Login to the source database system using the Public IP.

   Username to login : **opc** 

   Use the private SSH key generated earlier.

2. Check the Operating System version of the source database.

   Execute the below command after login in as **opc** user.
   
     ```text
     <copy>
     cat /etc/os-release
     </copy>
     ```
     Please use similar commands in case above command doesn't work for you ( in case you have selected different source database system than the one specified in Lab 2).

     You will get an output similar to the one below.

     ![Image showing output of command to check OS version ](./images/os-version.png)

3. Set the database environment to connect to your database.

   Switch user to **oracle** using below command.

   **sudo su - oracle**

   Set the environment to connect to your database using below command.

   Type **. oraenv** and press **Enter**.
    
   Enter **ORCL** when asked for **ORACLE_SID** and then press **Enter** .   --> Enter your **ORACLE\_SID** if that is different in case of an on-premises database.

4. Check the database version of the source database.

   Please check the source database version that was selected while provisioning the source database using Marketplace Image in Lab 2.

   In case you would like know the database version for your on-premises database then refer the below steps.
    
   Execute **opatch lsinventory** command as **oracle** user.

   Check for the output to determine the exact database version.

5. Check the database edition of the source database.

   Your source database is in **Enterprise Edition** since the Oracle Marketplace Image used in previous lab provisions Enterprise Edition database.

   In case you would like know the database edition for your on-premises database then refer the below steps.

   Execute the below query after connecting to database using sqlplus.
     ```console
     <copy>
     select banner from v$version;
     </copy>
      ```
     You will receive an output similar to the one below which will have the Database Edition.

     ![Image showing Database Edition of Source database](./images/database-edition.png)

6. Check database characterset.
   
   Run the below query to identify the database character set and national characterset.
     ```console
     <copy>
     select PARAMETER,VALUE from nls_database_parameters where parameter like '%NLS%CHARACTERSET';
     </copy>
      ```
     In the query output **NLS\_CHARACTERSET** is the database characterset and **NLS\_NCHAR\_CHARACTERSET** is the national characterset.

     Sample output is shown below.

     ![Image showing database and national character set in database](./images/database-characterset.png)

7. Check encryption algorithm under sqlnet.ora.

   Check the **sqlnet.ora** to identify any encryption algorithm mentioned.

8. Generate patch inventory ouput.

   Execute **opatch lsinventory** command as **oracle** user in source database server.

9. Download inventory output to the local desktop.

   We will require this file in Task 2.

## Task 2 : Prepare Database Software Image for Target Database

1. Navigate to Oracle Base Database.

   Click the **Navigation Menu** in the upper left, navigate to **Oracle Database** and then select **Oracle Base Database (VM,BM)**.

   ![Image showing navigation to Oracle Base Database](./images/navigate-to-database.png)

2. Click on **Database software images**.

   Select the appropriate compartment and then click on **Database software images** under **Resources**.

   ![Image showing compartment selection ](./images/compartment.png)

3. Click **Create Database software image**.

   Enter Display name as **DBImage-Source-DB** as below.

   ![Image showing Database Software Image Name ](./images/database-image-name.png)

4. Configure database software image.

   Select database version as **19c**   (same as the major version of your source database).

   Select PSU as **19.16.0.0** ( If you have selected a different version for the source database in the previous lab, please provide that version here).

   Upload Oracle Home patch inventory output generated in Task 1 as below.

   ![Image showing database version selected for Image ](./images/db-version-info.png)

5. Create database software image.

   Click on **Create Database software image** to create DB Image.

   Please wait for the completion of this task before proceeding to the next task.

## Task 3 : Provision Target Database

1. Navigate to Oracle Base Database in Oracle Cloud Console.

   Click the **Navigation Menu** in the upper left, navigate to **Oracle Database** and then select **Oracle Base Database (VM,BM)** as shown below.

   ![Image showing navigation to Oracle Database](./images/navigate-to-database.png)

2. Click on  **Create DB System**.
    
   ![Image showing Create DB system option](./images/createdb.png)

3. Provide name of the DB System and select compartment.

   Provide DB System name as **zdm-target-db** and ensure you have selected correct compartment for the DB system.
    
   ![Image showing the updated DB system name](./images/db-system-name.png)

4.  Modify the shape of the DB System.

   When you create the database from the console, ensure that your chosen shape can accommodate the source database, plus any future sizing requirements. A good guideline is to use a shape similar to or larger in size than source database.

   For this lab we will use **AMD Flex** with 1 OCPU.

   Click on **Change Shape** and reduce the number of OCPU per node to 1 as below.

   ![Image showing the option to reduce the OCPU](./images/ocpu.png)

   Click on **Select a Shape** , your final selection will appear as below.

   ![Image showing final selection of DB System Shape](./images/shape.png)

5. Configure storage.

   Leave this section as the default.

6. Configure database edition.

   Under **Configure the DB system** , ensure to select **Enterprise Edition** which is the same edition as our source database.

   ![Image showing the selection for Database Edition](./images/edition.png)

   
7. Upload SSH Keys.
   
   Under **Add SSH keys** , upload the SSH Public key generated earlier.

   ![Image showing option to upload SSH key](./images/ssh.png)

8. Select the appropriate License Type.

   Select appropriate license type applicable for you.

9. Specify the network information.

   Select **ZDM-VCN** as Virtual Cloud Network and **Public Subnet-ZDM-VCN** as Client subnet.

   Provide **zdm-target-db** as Hostname prefix.

   ![Image showing the Network select for DB system](./images/network.png)

10. Click Next

   Click **Next** to go to the next page.

11. Provide database name.

   If the target database is in Exadata Database Service on Dedicated Infrastructure or Exadata Cloud at Customer, then the target database **DB\_NAME** should be the same as the source database **DB\_NAME**.

   If the target database is Oracle Base Database VM , then the target **DB\_NAME** can be the same as or different from the source database **DB\_NAME**.

   Our target database is **Oracle Base Database VM** and we can specify a same or different name for **DB\_NAME**. 

   We will keep the same **DB\_NAME** as source database for this lab.

   Provide **Database** name as **ORCL** and **Database unique name suffix** as **T**.

   ![Image showing the Database Name entered](./images/dbname.png)

12. Select Database Image.

   Click on **Change Database Image** and select **Custom Database Software Images** as below.

   ![Image showing selection of Database Software Image](./images/custom.png)

   Select the appropriate compartment and select DB Image created in earlier lab as below.

   ![Image showing custom software images created earlier](./images/dbimage.png)

13. Provide SYS password.

   Enter SYS password which is same as the SYS password of the source database.

   ![Image showing the provision to enter SYS password](./images/sys-password.png)

14. Select database workload type.

   In this lab , leave it to the default.

15. Disable database backups.

   Uncheck the **Enable automatic backups** box to disable database backups.

   We don't need automatic backups until we complete the database migration.

   ![Image showing the option to disable database backups](./images/disable-backup.png)

16. Select database charactetset.

   Click on **show advanced** options.

   Ensure that you have selected same database and national characterset as the source database.

   In this lab source database has below Database and National Characterset.

   Database Characterset : AL32UTF8

   National Characterset : AL16UTF16

   Sample output is shown below.

   ![Image showing the database characterset selected](./images/db-characterset.png)

17. Start DB System provisioning.

   Click on the **Create DB System** to initiate the DB system provisioning.

   ![Image showing the option to start the provisioning](./images/prov-final.png)

   This step is going to take up to 1 hour , however you can proceed to next lab while DB System is being provisioned.

You may now **proceed to the next lab**.

## Acknowledgements
* **Author** - Amalraj Puthenchira, Cloud Data Management Modernise Specialist, EMEA Technology Cloud Engineering
* **Last Updated By/Date** - Amalraj Puthenchira, February 2023

