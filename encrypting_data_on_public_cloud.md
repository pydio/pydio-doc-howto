In this how to we want to use S3FS coupled with ENCFS and Pydio to have encrypted data on an Amazon S3 server. We will see how to install and combine these features to be able to use them (directly on a shell or on Pydio).


## Install S3FS and Fuse
### Installation
Before thinking about using all these features we have to install them. First we have to install Fuse by running the following command:

    sudo apt-get install build-essential libcurl4-openssl-dev libxml2-dev libfuse-dev comerr-dev libfuse2 libidn11-dev libkadm55 libkrb5-dev libldap2-dev libselinux1-dev libsepol1-dev pkg-config fuse-utils

The command apt-get will do the job of installing the packages for us. It’s another business for S3FS. We **first** have to **download** the project by running this:

    wget http://s3fs.googlecode.com/files/s3fs-r177-source.tar.gz

Then you will have to **untar** this by running:

    tar xzvf s3fs-r177-source.tar.gz

We’ll **install** this library by running:

    cd ./s3fs

    sudo configure --prefix=/usr

    sudo make

    sudo make install

From now on you should be able to run the command s3fs and encfs (which are the most important commands we will use). Let’s now configure our environment to be able to mount and use a s3fs repository.

### Configure
If you want to allow others to access your bucket (and we will want to) you will have to uncomment the following line in /etc/fuse.conf:

    sudo nano /etc/fuse.conf

Then uncomment this line:

> #allow_others

We then have to **create**  a new file that will contain the access id key and the secret key. This file has to be located in /etc and have the following permissions “700”:

    sudo nano /etc/passwd-s3fs

Then fill it with the following information:

> accessIDKey:secretKey

Change the permissions:

    sudo chmod 700 /etc/passwd-s3fs

Ok we are now all set to install and use our features inside of Ajaxplorer.

## Mount and use an encrypted folder in a S3FS folder
### Mount the S3FS folder
This is one of the easiest thing to do. Firstly, **create** a folder, here we’ll name this folder “s3fs_test” but you can change the name if you want, that will be the **founding** of our system (better to change the owner of the folder to “www-data”):

    cd ~

    mkdir s3fs_test

Now we will **create** a folder which will be our s3fs **mounting point**:

    cd s3fs_test

    mkdir data

Now we **change** the **owner** of the folder:

    sudo chown -R www-data ../s3fs_test

You can now mount the s3fs repository on the folder data:

    sudo s3fs -o allow_other,uid=33 bucket_name data

Let me explain the options we chose:

+ **allow_other**: it will allow you to read and modify the files directly from your terminal.
+ **uid=33**: it will make www-data the owner of the mounting point (really useful if you want to use it from Ajaxplorer)
+ **bucket_name**: This one is straightforward just replace this by the name of the bucket you want to access
+ There are a lot of other options that you can use to allow caching and all, you can find them by running
    man s3fs

and

    man fuse

Now that you ran this command every file you will put on your s3fs_test/data folder will be put in your S3 server.

### Mount the ENCFS folder
Now we want to have an encrypted folder on our S3 server. We will then create and mount an ENCFS folder that will have its encrypted folder on our S3 server and the clear data on our local machine (this way we will be able to put encrypted data on our server and see them by mounting our ENCFS folder locally).

Firstly, go in our **already mounted** S3FS folder:

    cd ~/s3fs_test/data

Now run the following command that will mount our ENCFS folder (you can change the paths but be reminded that it has to be absolute):

    sudo encfs -o allow_other,uid=33 ~/s3fs_test/data/cyphered_data ~/clear_data

Once you’ve run this command ENCFS will ask you if it should create the folders, just type “y” then enter. Then it asks for a password, it’ll be the password used to encrypt the data so you should remember it. The encrypted folder is now mounted.

Everytime you will add data in the ~/clear_data folder it will encrypt it and put it in your S3 server.

### Use it on Ajaxplorer
To use this feature on an Ajaxplorer FS repository you will have to do all the preliminary work (installation and then **mounting both folders**).

Then just create one FS repository that points on ~/s3fs_test and another one that points on ~/clear_data. You will then be able to access your S3 data without using the plugin S3 (with the caching options it can be faster) and put encrypted data on it (by using the ~/clear_data FS repository).