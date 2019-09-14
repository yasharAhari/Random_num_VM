

## Random number generator(python) Google cloud VM instruction (Team APYZ)

This guide will focus on installing the Random number generator on google cloud VM instance.

### Description

We are using python Flask framework to run this application. we are also using the Nginx as the reverse proxy server and uwsgi as local WSGI to link the app. 

#### Requirements 

* Google cloud account access.

### Creating google cloud VM instance 

Sign in to your google cloud account and go to the console. 

* create a new project 

* Click the menu icon in the top left of the console

* Select the Compute Engine >  VM instances

* Select Create Instance

* Chose a name for your instance and preferred region and zone 

* Select your preferred Machine configuration.

  | ⚠️ **Warning**:                                               |
  | ------------------------------------------------------------ |
  | The choice of machine type will effect the cost of the operation. Although you may choose any machine configuration, our suggestion is to use ```First generation g1-small``` which will be sufficient. |
  | We are using the `Ubuntu 16.04 LTS` for this application     |

* In the `Firewall` section,  toggle the **Allow HTTP traffic** and **Allow HTTPS traffic** on.
* Save and wait to VM load up.  

### Connecting to the VM

* Click the `ssh` button under the `connection` section to connect the terminal. 

### installing required software

* Update and Upgrade your machine using 

  ```shell
  sudo apt update -y; sudo apt upgrade -y 
  ```

  

* Install python 3.7

  we are using python 3.7 and  ```pyenv``` to handle python installation. For that, clone the ```pyenv``` form its git repo.

  ```shell
  git clone https://github.com/pyenv/pyenv.git ~/.pyenv
  ```

  in order to ```pyenv``` work, it need to be configured in the machine ```.bashrc ``` file. For that do the following: 

  ```shell
  echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
  echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
  echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.bashrc
  exec "$SHELL"
  ```

  

* Install the required Python build dependencies:

  ```shell
  sudo apt-get update; sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
  ```

  

* After the build dependencies installed, use the ```pyenv``` to install the python 3.7.2 

  ```shell
  pyenv install 3.7.2
  ```

  This process might take few minutes on a small VM. In case of successful installation you will see the following:

  ```
  Downloading Python-3.7.2.tar.xz...
  -> https://www.python.org/ftp/python/3.7.2/Python-3.7.2.tar.xz
  Installing Python-3.7.2...
  ```

* Now set the system python as 7.3.2

  ```
  pyenv global 3.7.2
  ```

  

### Cloning the app 

Go to the directory that you want the application work on, or just create a new directory in the home.

we call this directory ```APYZ_Random_PVM```

```shell
cd; mkdir APYZ_Random_PVM; cd APYZ_Random_PVM
```

And now clone the GitHub repository to get the necessary files.

 ```shell
git clone https://github.com/yasharAhari/Random_num_VM.git
 ```



### Installing the dependencies

Now, install the required packages by creating and using a new virtual environment

​	***warning***, the following directory paths are based on the assumption that we put the app in home directory!

​	***warning*** : Don't forget to change ```<your_username>``` with your system username. 

*  Go to the app directory: 

  ``` shell
  cd /home/<your_username>/APYZ_Random_PVM/RandomNumberGen
  ```

  

* create the Virtual environment and call it ```env```

  ```shell
  python -m venv env
  ```

  and activate it: 

  ```shell
  source /home/<your_username>/APYZ_Random_PVM/RandomNumberGen/env/bin/activate
  ```

* Upgrade the pip

  ```shell
  pip install --upgrade pip
  ```

* And install the Python dependencies

  ```shell
  pip install Click==7.0 Flask==1.0.2 itsdangerous==1.1.0 Jinja2==2.10.1 MarkupSafe==1.1.1 uWSGI==2.0.18 Werkzeug==0.15.3
  ```

  

### Installing the NGINX

The Nginx will handle the http request. Installation process is fairly easy. 

```shell
sudo apt install nginx
```

### Configuring everything

Now you need to:

1. Configure the Random Generator app to start working on boot 
2. Configure the Nginx to serve from the .socket file 

  We configure such a way that the app started by ```uwsgi``` . The ```.socket``` file will be created by ```uwsgi``` automatically. 

#### 1st, Configuring the Systemd unit file

* use ```nano``` to create and edit the file we call ```app.service``` 

  ```shell
  sudo nano /etc/systemd/system/app.service
  ```

  Add the following: 

  ```
  [Unit]
  Description=Random Number Gen
  After=network.target
  
  [Service]
  User=<your_username>
  Group=www-data
  WorkingDirectory=/home/<your_username>/APYZ_Random_PVM/RandomNumberGen/
  Environment="PATH=/home/<your_username>/APYZ_Random_PVM/RandomNumberGen/env/bin"
  ExecStart=/home/<your_username>/APYZ_Random_PVM/RandomNumberGen/env/bin/uwsgi --ini app.ini
  
  [Install]
  WantedBy=multi-user.target
  ```

  Replace the ```<your_username>``` with your username with can be found on front of the terminal prompt

  ```shell
  username@instance_name ~$:
  ```

  Now save and exit the `nano` by pressing ```Ctrl+x``` and selecting yes by typing ```Y``` and pressing the ```Return```

    

  Now start the app service by:

  ```shell
  sudo systemctl start app
  ```

  and activate it using 

  ```shell
  sudo systemctl enable app
  ```

  #### 2nd, Configuring the NGINX
  
  Since we starting the app using ```uwsgi```, we need to configure Nginx to serve the related ```app.sock```  file that we configured to be located in the main application directory. 
  
  First, we need to make a new server block in Nginx's ```sites-available``` . 
  
  use, 
  
  ```shell
  sudo nano /etc/nginx/sites-available/app
  ```
  
  and add the following: 
  
  ```nginx
  server {
      listen 80;
      server_name <your_ip_address>;
  
      location / {
          include uwsgi_params;
          uwsgi_pass 
          unix:/home/<your_username>/APYZ_Random_PVM/RandomNumberGen/app.sock;
      }
  }
  ```
  
  **Remember** to change ```<your_username>``` with your system username. 
  
  **Warning**: if you choose to use an other directory, be sure to add the correct path. 
  
  
  
  Now, link this file to the ```sites-enabled``` using.
  
  ```shell
  sudo ln -s /etc/nginx/sites-available/app /etc/nginx/sites-enabled
  ```
  
  You can check for errors related to new file by using:
  
  ```shell
  sudo nginx -t
  ```
  
  which if successful, will print:
  
   ```shell
  nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
  nginx: configuration file /etc/nginx/nginx.conf test is successful
   ```
  
  
  
  Now, restart the Nginx by
  
  ```shell
  sudo systemctl restart nginx
  ```
  
  
  
  Congratulations, after this point, the app should be working.
  
  
  
  ### References
  
  Majority of this installation guide and code is inspired by work of  Julian Nash in [His website](https://pythonise.com/feed/flask/deploy-a-flask-app-nginx-uwsgi-virtual-machine) 
  
  https://pythonise.com/feed/flask/deploy-a-flask-app-nginx-uwsgi-virtual-machine
  
  
  
  
