## Google Cloud Storage
This post provides a quick and easy way to get started with Google Cloud Storage on a linux environment with gsutil - tool. It shows you how to:
- Install gsutils tools
- Create and delete a bucket
- Upload, delete and move objects
- List your buckets and objects
- Share objects and buckets


### 0- Install the prerequisites
To use gsutil, Python 2.6.x or 2.7.x is necessary

```sh
# Add to package source
sudo add-apt-repository ppa:fkrull/deadsnakes

# Update package source
sudo apt-get update

# Install python
sudo apt-get install python2.7
```

### 1- Install gsutils tools
```sh
# Create your home repository
mkdir gcs

# log on your home repository
cd gcs

# download gsutil as gsutil.tar.gz
wget https://storage.googleapis.com/pub/gsutil.tar.gz

# Extract the archive files
tar xfz gsutil.tar.gz -C $HOME

# Add gsutil to your PATH environment variable
export PATH=${PATH}:$HOME/gcs/gsutil

# Restart your shell or terminal program

# Updating gsutil
update gsutil
```

### Set-up access to protected data
**Credentials**
```sh
# 1- Open new command prompt instance
# 2- Run this code
gsutil config
# 3- Copy and paste the URL into a browser window
# 4- Clic the allow access button
# 5- Copy the authorization code that appears on the next page into the gsutil prompt and press Enter
# 6- Log in to the Google Developers Console to find a project ID you can specify as the default project
# 7- Copy and paste the project ID into gsutil
```

**OAuth 2.0**


