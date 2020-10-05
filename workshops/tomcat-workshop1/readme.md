# Infrastructure setup for application deployemnt 
Apache Tomcat® is an open source Java application server. It implements the Java Servlet, JavaServer Pages, Java Expression Language and Java WebSocket technologies.
## Terraform Provider for Oracle Cloud Infrastructure
The OCI Terraform Provider is now available for automatic download through the Terraform Provider Registry. 
For more information on how to get started view the [documentation](https://www.terraform.io/docs/providers/oci/index.html) 
and [setup guide](https://www.terraform.io/docs/providers/oci/guides/version-3-upgrade.html).
* [Documentation](https://www.terraform.io/docs/providers/oci/index.html)
* [OCI forums](https://cloudcustomerconnect.oracle.com/resources/9c8fa8f96f/summary)
* [Github issues](https://github.com/terraform-providers/terraform-provider-oci/issues)
* [Troubleshooting](https://www.terraform.io/docs/providers/oci/guides/guides/troubleshooting.html)
## Clone the Module
Now, you'll want a local copy of this repo. You can make that with the commands:
git clone https://github.com/oracle-quickstart/oci-arch-tomcat-autonomous.git
    cd oci-arch-tomcat-autonomous
    ls
## Prerequisites
First off, you'll need to do some pre-deploy setup.  That's all detailed [here](https://github.com/cloud-partners/oci-prerequisites).
Secondly, create a `terraform.tfvars` file and populate with the following information:
```
# Authentication
tenancy_ocid         = "<tenancy_ocid>"
user_ocid            = "<user_ocid>"
fingerprint          = "<finger_print>"
private_key_path     = "<pem_private_key_path>"
# SSH Keys
ssh_public_key  = "<public_ssh_key_path>"
# Region
region = "<oci_region>"
# Compartment
compartment_ocid = "<compartment_ocid>"
````
Deploy:
    terraform init
    terraform plan
    terraform apply

##Add confirmation terraform screen here

## Testing your deployment
After the deployment is finished, you can test that your tomcat was deployed correctly and can access the database going to the urls:
````
http://<load balancer IP>/JDBCSample/JDBCSample_Servlet
http://<load balancer IP>/JDBCSample/UCPServlet
`````
## Destroy the Deployment
When you no longer need the deployment, you can run this command to destroy it:
terraform destroy
## Tomcat-Autonomous Database Architecture
![](./images/architecture-deploy-tomcat.png)
Although the diagram shows a private subnet for the Tomcat servers, the scripts are provisioning them on a public subnet as there is no bastion to allow access to the servers.
## Reference Architecture
- [Deploy Apache Tomcat connected to an autonomous database](https://docs.oracle.com/en/solutions/deploy-tomcat-adb)
 

Migration of application from on premise to OCI cloud

1.	Do ssh to tomcat01 vm’s (refer to OCI console for vm’s public IP)
2.	Stop the tomcat by using below cmd
systemctl stop tomcat
3.	Copied SimpleDB.war to /var/lib/tomcat/webapps/ dir
4.	Added config changes to (/usr/share/tomcat/conf)server.xml,web.xml,context.xml
 Add below tag parelle to GlobalNamingResources in Server.xml
 <GlobalNamingResources>
     <Resource name="jdbc/JDBCConnectionDS"
          global="jdbc/JDBCConnectionDS"
          auth="Container"
          type="javax.sql.DataSource"
          username="riders"
          password="Welcome#123456"
          driverClassName="oracle.jdbc.OracleDriver"
          description="RIDERS's database"
          url="jdbc:oracle:thin:@mydb_high?TNS_ADMIN=/etc/tomcat/wallet"
          maxActive="15"
          maxIdle="3"/>
  </GlobalNamingResources>
 
At last add below config to web.xml
<resource-ref>
                <description>JDBCConnection</description>
                <res-ref-name>jdbc/JDBCConnectionDS</res-ref-name>
                <res-type>javax.sql.DataSource</res-type>
                <res-auth>Container</res-auth>
        </resource-ref>
 
Add below given config at last in context.xml
<ResourceLink name="jdbc/JDBCConnectionDS"
    global="jdbc/JDBCConnectionDS"
    type="javax.sql.DataSource"/>
 
 
5.	Added lib tomcat-dbcp-7.0.76.jar to  /usr/share/java/
6.	Start Tomcat server sing below given cmd –
sudo systemctl start tomcat
7.	Test your application deployment using below given url
For vm level deployment testing
http://<IP>:<Port>/SimpleDB/
8.	Repeat the steps 1-7 for another tomcat vm as well.
9.	Once migration of SimpleDB war file done on both tomcat instance, you can validate end to end deployment using LB url
http://<Load balance IP>/SimpleDB/
