# aws-lambda-pyodbc-ms-sql-layer
AWS Lambda Layer to connect to Microsoft SQL Server using pyodbc

pyodbc is the python module used to communicate with ODBC standard based databases. pyodbc in turn communicates with a driver manager which calls the database driver to talk to ODBC compliant databases. The challenge I faced was there is no single python package which does these three functions.

I used AWS EC2 instance to build the lambda layer. Spin up an EC2 instance and install the following prerequisites for pyodbc.

1. yum -y install gcc-c++
2. yum -y install python3-devel
3. yum -y install unixODBC-devel


create a folder to download these components and create lambda layer.

mkdir downloads mssql-driver-layer
cd downloads

yumdownloader unixODBC.x86_64
pip3 download pyodbc
curl packages.microsoft.com/config/rhel/7/prod.repo > /etc/yum.repos.d/mssql-release.repo
yumdownloader  msodbcsql17

cd ../mssql-driver-layer
Extract unixODBC driver manager, msodbc driver.


rpm2cpio /home/ec2-user/downloads/unixODBC-2.3.1-14.amzn2.x86_64.rpm | cpio -id
rpm2cpio /home/ec2-user/downloads/msodbcsql17-17.5.2.1-1.x86_64.rpm | cpio -id

 Install pyodbc inside mssql-driver-layer folder.

pip3 install --target /home/ec2-user/lambda/testodbc/python/lib /home/ec2-user/lambda/downloads/pyodbc-4.0.30.tar.gz



Move driver manager libraries from /usr/lib64 to /lib as lambda runtile looks for inside lib folder.

Update diver manager config file with location of msodbcsql driver.

vi etc/odbcinst.ini

[ODBC Driver 17 for SQL Server]
Description=Microsoft ODBC Driver 17 for SQL Server
Driver=/opt/opt/microsoft/msodbcsql17/lib64/libmsodbcsql-17.5.so.2.1
UsageCount=1

Package everything as zip file and the lambda layer is ready.

By default AWS lambda layers are uploaded to /opt directory and is added to python path of lambda function execution environment.

We need to tell pyodbc where to find the unixODBC libraries and config file.

ODBCSYSINI - /opt/etc
PYTHONPATH - /var/runtime:/opt/python/lib



