---------------------------------------------------------GSP101: Google Cloud Essential Skills--------------------------------------------------------


------------------------------------------Task 1 : Create a Compute Engine instance, add necessary firewall rules-------------------------------------

Go to Compute Engine 

----> VM Instances 

----> Create Instance.

Then Add the following data to create instance

   Name : apache

   Zone : us-central1-a

   Series : N1

   Boot Disk : Linux 9 (stretch)

   Click tick on : Allow HTTP Traffic
                   Allow HTTPS Traffic
          

------------------------------------------------------>-------- Click Create-------<--------------------------------------------------------------------

--------------------------------------------------------------------------------------------------------------------------------------------------------


------------------------------------------Task 2 :Add Apache2 HTTP Server to your instance  ------------------------------------------------------------

----------->---SSH to 'apache' instance and run---<------------

sudo su -

apt-get update

apt-get install apache2 -y

service --status-all

---------------------------------------------------------------------------------------------------------------------------------------------------------



------------------------------------------Task 3 :   Test your server------------------------------------------------------------------------------------

Refresh it and then Click on External IP of 'apache' instance.

---------------------------------------------------------------------------------------------------------------------------------------------------------


---------------------------------------------------------------------Congratulations---------------------------------------------------------------------
