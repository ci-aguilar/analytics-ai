# Infrastructure Configuration

## Introduction

In this lab we will build the infrastructure that we will use to run the rest of the workshop. The main three elements that we will be creating are a Virtual Cloud Network which helps you define your own data center network topology inside the Oracle Cloud by defining some of the following components (Subnets, Route Tables, Security Lists, Gateways, etc.), bastion host which is a compute instance that serves as the public entry point for accessing a private network from external networks like the internet, and we will create an Oracle Analytics Cloud instance which is embedded with machine learning, that helps organizations to discover unique insights faster with automation and intelligence. Finally, we will create a MySQL HeatWave Service instance that we will allow us to configure a heatwave cluster later. 

Estimated Time: 35 minutes

### Objectives

In this lab, you will:
- Create a Virtual Cloud Network and allow traffic through MySQL HeatWave Service port
- Create a Bastion Host compute instance 
- Connect to the Bastion Host, install MySQL Shell and download the workshop Dataset
- Create an Oracle Analytics Cloud instance
- Create an Instance of MySQL HeatWave in the Cloud

### Prerequisites

- Oracle Free Trial Account.
  
[Lab 1 Demo](youtube:W4JaHA-Fzp8)

## Task 1: Create a Virtual Cloud Network and allow traffic through MySQL HeatWave Service port

1. Log-in to your OCI tenancy. Once you have logged-in, select _**Networking >> Virtual Cloud Networks**_ from the _**menu icon**_ on the top left corner.

  ![OCI Dashboard](./images/open-vcn.png)

2. From the Compartment picker on the bottom left side, select your compartment from the list.

    > **Note:** If you have not picked a compartment, you can pick the root compartment which was created by default when you created your tenancy (ie when you registered for the trial account). It is possible to create everything in the root compartment, but Oracle recommends that you create sub-compartments to help manage your resources more efficiently.

  ![VCN Dashboard](./images/vcn-compartment.png)

3. To create a virtual cloud network, click on _**Actions, Start VCN Wizard**_.
    
  ![VCN Dashboard](./images/start-vcn-wizard.png)

4. Create _**VCN with Internet Connectivity**_ and click _**Start VCN Wizard**_.

  ![VCN Creation](./images/vcn-wizard-select.png)

5. Now you need to complete some information and set the configuration for the VCN. In the _**VCN Name**_ field enter the value 
  **`analytics_vcn_test`** (or any name at your convenience), and make sure that the selected compartment is the right one. Leave all the rest as per default, Click _**Next**_.

  ![VCN creation](./images/vcn-config-name.png)

6. Review the information showed is correct and click _**Create**_.

  ![VCN creation](./images/vcn-config-review.png)

7. Once the VCN will be created click _**View VCN**_.

  ![VCN creation](./images/vcn-complete.png)

8. Click on the **Subnet** Tab and then on the _**`Private Subnet-analytics_vcn_test`**_. 

  ![VCN creation](./images/select-private-subnet.png)

9. Earlier we set up the subnet to use the VCN's default security list, that has default rules, which are designed to make it easy to get started with Oracle Cloud Infrastructure. 
   Now we will customize the default security list of the VCN to allow traffic through MySQL Database Service ports. 

   Click on the **Security** Tab and then on _**`Security List for Private Subnet-analytics_vcn_test`**_.

  ![scurity list](./images/edit-security-list.png)

10. Click on the **Security rules** Tab and then on _**Add Ingress Rules**_.

  ![security list](./images/add-ingress-rule.png)

11. Add the necessary rule to the default security list to enable traffic through MySQL Database Service port. 

  Insert the details as below:

	```  
	Source CIDR:  <copy> 0.0.0.0/0 </copy>
	```
	```  
	Destination Port Range: <copy>3306,33060</copy>
	```
	```  
	Description:  <copy> MySQL Port </copy>
	```
	At the end click the blue button _**Add Ingress Rules**_.

  ![security list](./images/confirm-ingress-changes.png)

## Task 2: Create a Bastion Host compute instance  

1. From the main menu on the top left corner select _**Compute >> Instances**_.
    
  ![OCI Console](./images/open-instances.png)

2. In the compartment selector on the bottom left corner, select the same compartment where you created the VCN. Click on the _**Create Instance**_ blue button to create the compute instance.

  ![Compute Instance Dashboard](./images/create-instance.png)

3. In the **Name** field, insert _**mysql-analytics-test-bridge**_ (or any other name at your convenience). This name will be used also as internal FQDN. 
  	
	The _**Placement and Hardware section**_ is the section where you can change Availability Domain, Fault Domain, Image to be used, and Shape of resources. For the scope of this workshop leave everything as default. 
	

  ![Compute Instance creation](./images/select-instance-name.png)

	**Click Next 2 times**
  	
4.	You will see the **Networking** section, check that your previously created **VCN** and your Public subnet _**`Public Subnet-analytics_vcn_test(Regional)`**_ are selected.
    
  ![Compute instance creation](./images/select-instance-networking.png)

5. Scroll down and MAKE SURE TO **DOWNLOAD** the proposed key pair.
  You will use the private key to connect to the compute instance later on.

  ![Compute instance craetion](./images/download-private-key.png)

  **Click Next** and **_Create_**

6. Once the compute instance will be up and running, you will see the square icon on the left turning green. However, you can proceed to the next **Task** until the provisioning is done.
    
  ![compute instance creation](./images/instance-running.png)

## Task 3: Connect to the Bastion Host, install MySQL Shell and download the workshop Dataset

1. In order to connect to the bastion host, we will use the cloud shell, a small linux terminal embedded in the OCI interface.
  
  	To access cloud shell, click on the shell icon next to the name of the OCI region, on the top right corner of the page.

  ![Cloud shell](./images/open-cloud-shell.png)


  	Once the cloud shell is opened, you will see the command line:
    
  ![Cloud Shell](./images/view-command-line.png)

2. Drag and drop the previously saved private key into the cloud shell.   

  ![Cloud Shell Darg&Drop Private key](./images/drag-private-key-to-cloud-shell.png)

  You can verify the key file name with the following command:
    
	```
    <copy>ll</copy>
    ```
  ![Cloud Shell list files](./images/verify-filename.png)

3. Copy the _**Public IP Address**_ of the compute instance you have just created.

  	![Compute Instance Ip Address](./images/copy-public-ip.png)

  	In order to establish an ssh connection with the bastion host using the Public IP, execute the following commands:
	```
	<copy>
	chmod 400 <private-key-file-name>.key
	</copy>
	```

	```
	<copy>
	ssh -i <private-key-file-name>.key opc@<compute_instance_public_ip>
	</copy>
	```
  	![connect to compute instance](./images/connect-to-compute-instance.png)

  	If prompted to accept the finger print, type _**yes**_ and hit enter, then you will get a Warning.

	> **Note:** Warning: Permanently added '130. . . ' (ECDSA) to the list of known hosts.

  Now that you have connected to the instance you can proceed to the next Task.

4. 	You will need a MySQL client tool to connect to your new MySQL HeatWave System from the Bastion. **You might want to have MySQL Shell in the same version as your DB System**

	You can download the specific desired version and architecture at [MySQL Community Dowloads](https://dev.mysql.com/downloads/shell/)

	To Install the current latest MySQL Shell version (9.4) for x86 architecture, you can follow the instructions:

	a. Install MySQL Shell with the following command (enter y for each question)

	```bash
	<copy>wget https://dev.mysql.com/get/Downloads/MySQL-Shell/mysql-shell-9.4.0-1.el9.x86_64.rpm</copy>
	```

	```bash
	<copy>sudo yum install mysql-shell-9* -y</copy>
	```

	![mysql shell install](./images/mysql-install-shell.png "mysql shell install ")

5. Confirm that MySQL Shell is installed by printing its version.
	```
	<copy>mysqlsh --version</copy>
	```
	![MySQL version](images/check-mysql-version.png)

6. Download the airportdb sample database that we will use for this workshop using the following commands:
	```
	<copy>
	cd /home/opc
	</copy>
	```

	```
	<copy>wget -O airport-db.zip https://bit.ly/3pZ1PiW</copy>
	```

	![database sample download](./images/download-db-sample.png "database sample download")


## Task 4: Create an Oracle Analytics Cloud instance

In this task we will create an Oracle Analytics Cloud instance before we proceed to the next lab, since it may takes sometime to be provisioned, so it will be **Running** status when we will use it later in this workshop.

1. Click on the menu icon on the left. Verify that you are signed in as a **Single Sign On** (Federated user) user by selecting the **Profile** icon in the top right hand side of your screen. If your username is shown as:

	```
	<your username>
	Identity domain: OracleidentityCloudService
	```

	Then you are **connected** as a **Single Sign On** user.

  	![Federated User](./images/check-federated-user.png "Federated user")

  	If your username is shown as:

	```
	<your username>
	Identity domain: <Other>
	```

	Then you are **signed in** as an **Oracle Cloud Infrastructure** user and you may proceed to the **Task 4.2**.

  	If your user does not contain the identity provider (**oracleidentitycloudprovider**), please logout and select to authenticate using **Single Sign On**.

  	![Federated Sing In](./images/sign-in-federated.png "Federated sign in")

   	To be capable of using **Oracle Analytics Cloud** we need to be Sign-On as a **Single Sign-On** (SSO) user.

   	For more information about federated users, see **[User Provisioning for Federated Users.](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/usingscim.htm)** and **[Understand SSO Authentication and Oracle Analytics Server](https://docs.oracle.com/en/middleware/bi/analytics-server/security-oas/understand-sso-authentication-and-oracle-analytics-server.html)**

2. Go to main page click the **hamburger menu** in the upper left corner and click on _**Analytics & AI -> Analytics Cloud**_.

  ![OCI Console OAC](./images/open-oac.png "OCI Console OAC")


3. Click _**Create instance**_ and in the new window, fill out the fields as shown in the image below. Make sure to select 2 OCPUs, the Enterprise version and the _**License Included**_ button. Finally click _**Create**_ to start the provisioning of the instance.

  	![Analytics Creation](./images/oac-create.png "Analytics Creation")

  	```
    Name: <copy>OACDemo</copy>
  	```
  	```
    OCPU: <copy>2</copy>
  	```	
  	```
    License Type: <copy>License Included</copy>
  	```
	```
    Edition: <copy>Enterprise</copy>
  	```

  ![Analytics Creation](./images/oac-config-name.png "Analytics Creation name")

  	> **Note:** It takes about _**15-20 minutes**_ to create the OAC instance you can proceed to the next task until it is done.

  ![Analytics Creation](./images/oac-creating-wait.png "Analytics Creating state ")

## Task 5: Create MySQL HeatWave Database

1. From the console main menu on the left side select _**Databases >> DB Systems**_.
    
  ![OCI Console DBSystems](./images/open-db-systems.png "OCI Console DBSystems")

2. It will bring you to the DB System creation page. 
  Look at the compartment selector on the left and check that you are using the same compartment used to create the VCN and the Compute Instance. Once done, click on _**Create MySQL DB System**_.

  ![MySQL DB System creation](./images/create-mysql-db.png "MySQL DB System creation")

3. Start creating the DB System. Cross check again the compartment and assign to the DB System the name:

   	```
   	<copy>mysql-analytics-test</copy>
   	```
   	Select the HeatWave box, this will allow to create a MySQL DB System which will be HeatWave-ready. 

4. In the _**Create Administrator Credentials**_ section enter the username and choose a password of your own, but make sure to note it as you will be using it later through the workshop:

	```
	<copy>admin</copy>
	```
  	```
    <copy>**PASSWORD**</copy>
    ```

  	![MySQL DB System](./images/mysql-db-form.png "MySQL DB System")

5. Choose **Standalone**

6. In the _**Configure Networking**_ section make sure you select the same VCN, _**`analytics_vcn_test`**_ you have used to create the Compute Instance but for MySQL you will use a different subnet, the private one called _**`Private Subnet-analytics_vcn_test(Regional)`**_.

	- Leave the default availability domain and proceed to the next section.

    ![Configure hardware MySQL DB System creation](./images/mysql-db-networking.png "DB System Networking")

7. Confirm that in the _**Configure Hardware**_ section, the selected shape is **MySQL.8**, ECPU Core Count: **8**, Memory Size: **64GB**, HeatWave Cluster Shape: **HeatWave.512GB**, Nodes: **1**, Data Storage Size: **1024**.

  ![MySQL DB system hardware](./images/mysql-db-hardware.png "DB System Hardware")

8. In the _**Configure Backup**_ section leave the default backup window of **7** days. Scroll down and click on _**Show Advanced Options**_. 

  ![Advanced option MySQL DB Syatem](./images/mysql-db-advanced-options.png "DB System advanced options")

9. Go to the **Connections** tab. In the Hostname field enter (same as DB System Name):

	```
	<copy>mysql-analytics-test</copy>
	```

	Check that port configuration corresponds to the following:


	MySQL Port: **3306**
	MySQL X Protocol Port: **33060**
	Once done, click the _**Create**_ button.

	![MySQL DB System Connections Configuration](./images/mysql-db-networking.png "DB System connections configuration")


  	The MySQL DB System will have _**CREATING**_ state (as per picture below).

  	![MySQL DB Syatem Creating](./images/mysql-db-creating.png)


  As a recap we have created a VCN and added an additional Ingress rules to the Security list, and created a compute instance that serves as a bastion host and launched the cloud shell to import the private keys to connect to the compute instance, we also installed MySQL Shell and the MySQL client, and downloaded the dataset that will be used later on for benchmark analysis.
  Also, we created an Oracle Analytics Cloud instance which we will eventually use later in this workshop. Finally, created MySQL Heatwave Database instance which will be used to enable the HeatWave service later.

  Well done, you can now proceed to the next lab!

## Acknowledgements
  - **Author** - Rawan Aboukoura - Technology Product Strategy Manager, Vittorio Cioe - MySQL Solution Engineer
  - **Contributors** - Priscila Iruela - Technology Product Strategy Director, Victor Martin - Technology Product Strategy Manager 
  - **Last Updated By/Date** - Cristian Aguilar, MySQL Solution Engineering, September 2025
