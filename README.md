# shiny-app-droplet
How to deploy any Shiny app using Digital Ocean Droplets instances

# Shiny Web App Deployment on DigitalOcean using a Droplet

This guide explains the step-by-step process I followed to set up and publish a Shiny web application on a DigitalOcean Droplet using Ubuntu. It covers the installation of R, Shiny, the required Python virtual environment, and the configuration of the Shiny Server to host the web app.

## Prerequisites

- A DigitalOcean Droplet running Ubuntu.
- SSH access to the Droplet.
- WinSCP or similar tool for file transfer (optional).

## Steps

### 1. Update and Upgrade the System
First, ensure that your system packages are up-to-date by running the following commands:

```bash
sudo apt update
sudo apt upgrade -y
```

### 2. Install Required Dependencies
Install the essential tools and libraries required for R and Shiny to run properly:

```bash
sudo apt install -y dirmngr gnupg apt-transport-https ca-certificates software-properties-common
sudo apt install -y build-essential libcurl4-gnutls-dev libxml2-dev libssl-dev
```

### 3. Install R
Add the CRAN repository to the system and install R:

```bash
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9
sudo add-apt-repository 'deb https://cloud.r-project.org/bin/linux/ubuntu focal-cran40/'
sudo apt update
sudo apt install -y r-base
```

### 4. Install Shiny and R Packages
To install Shiny and other R packages required for the web app, run the following commands:

```bash
# Remove any previous installations of Shiny-related packages if needed
sudo rm -rf /usr/local/lib/R/site-library/00LOCK*
sudo rm -rf /usr/local/lib/R/site-library/sass
sudo rm -rf /usr/local/lib/R/site-library/bslib
sudo rm -rf /usr/local/lib/R/site-library/shiny

# Install necessary R packages
sudo su - -c "R -e \"install.packages('Rcpp', repos='https://cran.rstudio.com/')\""
sudo su - -c "R -e \"install.packages('sass', repos='https://cran.rstudio.com/')\""
sudo su - -c "R -e \"install.packages('bslib', repos='https://cran.rstudio.com/')\""
sudo su - -c "R -e \"install.packages('shiny', repos='https://cran.rstudio.com/')\""

# Install additional R packages for the app
R -e "install.packages(c('reticulate', 'shinydashboard', 'shinycssloaders', 'dplyr'), repos='https://cran.rstudio.com/')"
```

### 5. Set Up a Python Virtual Environment
Some applications may require Python packages. Here we set up a virtual environment for Python:

```bash
# Create the virtual environment directory
sudo mkdir -p /opt/virtualenvs
sudo virtualenv --python=python3.8 /opt/virtualenvs/py38_umap

# Activate the virtual environment and install the required Python packages
source /opt/virtualenvs/py38_umap/bin/activate
pip install cloudpickle==1.6.0 joblib==1.4.2 llvmlite==0.37.0 numba==0.54.0 numpy==1.20.3 pandas==1.3.5 pickleshare==0.7.5 scikit-learn==1.0.2 scipy==1.6.0 umap-learn==0.5.2

# Deactivate the virtual environment
deactivate

# Set appropriate permissions
sudo chown -R shiny:shiny /opt/virtualenvs/py38_umap
sudo chmod -R 755 /opt/virtualenvs/py38_umap
```

### 6. Install and Configure Shiny Server
Download and install the Shiny Server to host the Shiny apps:

```bash
# Download the Shiny Server package
wget https://download3.rstudio.org/ubuntu-14.04/x86_64/shiny-server-1.5.20.1002-amd64.deb

# Install the gdebi package installer and install Shiny Server
sudo apt install -y gdebi-core
sudo gdebi shiny-server-1.5.20.1002-amd64.deb
```

### 7. Set Up the Shiny App
Create a directory for your Shiny app and set up the app:

```bash
# Check the status of Shiny Server
sudo systemctl status shiny-server

# Create a new directory for your Shiny app
sudo mkdir /srv/shiny-server/hello-world

# Create the Shiny app (app.R)
sudo nano /srv/shiny-server/hello-world/app.R

# Restart the Shiny Server after setting up the app
sudo systemctl restart shiny-server
```

### 8. Transfer Files with WinSCP (Optional)
If you prefer to transfer files using WinSCP or another tool, you can copy your app files to the `/srv/shiny-server` directory. After transferring, ensure the correct ownership and permissions:

```bash
# Set ownership and permissions
sudo chown -R shiny:shiny /srv/shiny-server/hello-world
sudo chmod -R 755 /srv/shiny-server/hello-world

# Restart Shiny Server
sudo systemctl restart shiny-server
```

### 9. Logs and Monitoring
- Logs are available at: `/var/log/shiny-server`
- The Shiny app directory: `/srv/shiny-server/shiny-app-test`
- Python virtual environment: `/opt/virtualenvs/py38_umap`

## Conclusion

By following these steps, you should have a fully functional Shiny web app hosted on a DigitalOcean Droplet using Shiny Server. Make sure to monitor logs for any issues and adjust configurations if necessary.
