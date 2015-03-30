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


**Créer un utilusateur sparkmanager**
```sh
