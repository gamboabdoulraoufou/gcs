## Google Cloud Storage
This post provides a quick and easy way to get started with Google Cloud Storage using gsutil tools and JSON API. It shows you how to:
- Install gsutils tools
- Set up access to protected data
- Create, list and delete a bucket
- Upload, download, list and delete objects

### 0- Install the prerequisites
To use gsutil, Python 2.6.x or 2.7.x is necessary

```sh
# Add to package source
sudo add-apt-repository ppa:fkrull/deadsnakes

# Update package source
sudo apt-get update

# Install python
sudo apt-get install python2.7

# Install virtual env
sudo apt-get install python-virtualenv
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
tar xfz gsutil.tar.gz

# Add gsutil to your PATH environment variable
export PATH=${PATH}:$HOME/gcs/gsutil

# Restart your shell or terminal program

# Updating gsutil
update gsutil
```

### 2- gsutil tools
#### 2-1- Set-up access to protected data
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

#### 2-2- Manage data
```sh
# Create bucket
gsutil mb gs://<bucket-name>

# list bucket
gsutil ls

# Delete bucket
gsutil rb <bucket_url>
```

```sh
# upload object
gsutil cp <object> gs://<bucket-name>

# list object
gsutil ls gs://<bucket-name>

# download objetc
gsutil cp gs://<bucket-name>/<object> -

# Delete object
gsutil rm gs://<bucket>/<object>
```

### 3- JSON API for Python    
#### 3-1- Set up environment  
```sh
virtualenv env
./env/bin/pip install google-api-python-client 
./env/bin/pip install httplib2 
./env/bin/pip install argparse
source ./env/bin/activate

# Log to gcs repository
cd gcs
```

#### 3-2- Create a client secrets file 
To generate a client secrets file:    
1- Go to the Google Developers Console.    
2- Select a project to which the client ID will be associated.  
3- In the left sidebar, select APIs & auth > Credentials.  
4- Click Create new Client ID.  
5- In the Create Client ID window, choose Installed application.  
6- Click Create Client ID.  
7- Click Download JSON  

#### 3-3- Create file named gcs_api.py with the folowing content
For more details [log here](https://cloud.google.com/storage/docs/json_api/v1/)   
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import argparse
import httplib2
import os
import sys
import json

from apiclient import discovery
from apiclient.errors import HttpError
from apiclient.http import MediaFileUpload
from apiclient.http import MediaIoBaseDownload
from oauth2client.file import Storage as fileStorage
from oauth2client import client
from oauth2client import tools

# Define arguments
parser = argparse.ArgumentParser(
    description=__doc__,
    formatter_class=argparse.RawDescriptionHelpFormatter,
    parents=[tools.argparser])

parser.add_argument('--methode',  required=True, choices=['create', 'list', 'get_object', 'delete'])	
parser.add_argument('--objet',  required=True, choices=['bucket', 'object'], help='The operation concerns the object or backet')	
parser.add_argument('--bucket_name', required=True)
parser.add_argument('--bucket_location', default='US')
parser.add_argument('--bucket_scope',  default='https://www.googleapis.com/auth/devstorage.read_write')	
parser.add_argument('--object_name', default=None)
parser.add_argument('--object_language', default='en')
parser.add_argument('--object_md5hash', default=None)
parser.add_argument('--object_crc32c', default=None)
parser.add_argument('--object_source', default=None)
parser.add_argument('--object_dest', default=None)

# Parse arguments
args = parser.parse_args()

# Define some variables
_PROJECT_ID = 'your_project_id'
_PROJECT_NAME = 'your_project_name'
_API_VERSION = 'v1'

# CLIENT_SECRETS is name of a file containing the OAuth 2.0 information for this
# application, including client_id and client_secret. 
CLIENT_SECRETS = os.path.join(os.path.dirname('client_secrets.json'), 'client_secrets.json')

# Number of bytes to send/receive in each request.
CHUNKSIZE = 2 * 1024 * 1024

# Retry transport and file IO errors.
RETRYABLE_ERRORS = (httplib2.HttpLib2Error, IOError)

def print_with_carriage_return(s):
  sys.stdout.write('\r' + s)
  sys.stdout.flush()

def get_authenticated_service (argv):
  print 'Authenticating...'
  
  # Set up a Flow object to be used for authentication.
  FLOW = client.flow_from_clientsecrets(CLIENT_SECRETS,
	     scope = [args.bucket_scope,],
	     message = tools.message_if_missing(CLIENT_SECRETS))

  # If the credentials don't exist or are invalid run through the native client
  # flow. The Storage object will ensure that if successful the good
  # credentials will get written back to the file.
  storage = fileStorage('sample.dat')
  credentials = storage.get()
  if credentials is None or credentials.invalid:
    credentials = tools.run_flow(FLOW, storage)

  # Create an httplib2.Http object to handle our HTTP requests and authorize it
  # with our good Credentials.
  http = httplib2.Http()
  http = credentials.authorize(http)

  # Construct the service object for the interacting with the Cloud Storage API.
  service = discovery.build('storage', _API_VERSION, http=http)
  
  return service
  
 
def create (args):
  service = get_authenticated_service (args)
  if args.objet == 'bucket':
    try:
      req = service.buckets().insert(
        project=_PROJECT_ID,
        body={'name': args.bucket_name,
		      'location': args.bucket_location})
      resp = req.execute()
      print 'Created bucket:'
      print json.dumps(resp, indent=2)
    except client.AccessTokenRefreshError:
      print ("The credentials have been revoked or expired, please re-run the application to re-authorize")
	
  elif args.objet == 'object':
    #media = apiclient.http.MediaIoBaseUpload(io.BytesIO(_object_source), 'text/plain')
    print 'Building upload request...'
    media = MediaFileUpload(filename=args.object_source, chunksize=CHUNKSIZE, resumable=True)
	  
    object_resource = {
            'metadata': {'my-key': 'my-value'},
            'contentLanguage': args.object_language,
            'md5Hash': args.object_md5hash,
            'crc32c': args.object_crc32c,
    }
    req = service.objects().insert(
            bucket=args.bucket_name,
            name=args.object_name,
            body=object_resource, # optional
            media_body=media)
    #resp = req.execute()
    print 'Uploading file: %s to bucket: %s object: %s ' % (args.object_source, args.bucket_name, args.object_name)	  
    progressless_iters = 0
    response = None
    while response is None:
      error = None
      try:
        progress, response = req.next_chunk()
        if progress:
          print_with_carriage_return('Upload %d%%' % (100 * progress.progress()))
      except HttpError, err:
        error = err
        if err.resp.status < 500:
          raise
      except RETRYABLE_ERRORS, err:
        error = err

      if error:
        progressless_iters += 1
        handle_progressless_iter(error, progressless_iters)
      else:
        progressless_iters = 0

    print '\nUpload complete!'

    print 'Uploaded Object:'
    print json.dumps(response, indent=2)
  
  
def list (args):
  service = get_authenticated_service (args) 
  if args.objet == 'bucket':
    try:
      fields_to_return = 'nextPageToken,items(name,location,timeCreated)'
      req = service.buckets().list(
          project=_PROJECT_ID,
          fields=fields_to_return,  # optional
          maxResults=42)            # optional
    
      while req is not None:
        resp = req.execute()
        print json.dumps(resp, indent=2)
        req = service.buckets().list_next(req, resp)
    except client.AccessTokenRefreshError:
      print ("The credentials have been revoked or expired, please re-run the application to re-authorize") 
		
  elif args.objet == 'object':
    try:
      fields_to_return = 'nextPageToken,items(bucket,name,metadata(my-key))'
      req = service.objects().list(
          bucket=args.bucket_name,
          fields=fields_to_return,    # optional
          maxResults=42)              # optional

      while req is not None:
        resp = req.execute()
        print json.dumps(resp, indent=2)
        req = service.objects().list_next(req, resp)
    except client.AccessTokenRefreshError:
      print ("The credentials have been revoked or expired, please re-run the application to re-authorize") 
	

def get_object (args):
  service = get_authenticated_service (args)
  
  print 'Building download request...'
  f = file(args.object_dest, 'w')
  try:
    req = service.objects().get_media(
          bucket=args.bucket_name,
          object=args.object_name,
          #generation=generation # optional
    ) 
    media = MediaIoBaseDownload(f, req, chunksize=CHUNKSIZE)
    
    print 'Downloading bucket: %s object: %s to file: %s' % (args.bucket_name, args.object_name, args.object_dest)
    progressless_iters = 0
    done = False
    while not done:
      error = None
      try:
        progress, done = media.next_chunk()
        if progress:
          print_with_carriage_return(
              'Download %d%%.' % int(progress.progress() * 100))
      except HttpError, err:
        error = err
        if err.resp.status < 500:
          raise
      except RETRYABLE_ERRORS, err:
        error = err

      if error:
        progressless_iters += 1
        handle_progressless_iter(error, progressless_iters)
      else:
        progressless_iters = 0

    print '\nDownload complete!'
	
  except client.AccessTokenRefreshError:
    print ("The credentials have been revoked or expired, please re-run the application to re-authorize") 


def delete (args):
  service = get_authenticated_service (args)
  if args.objet == 'bucket':
    try:
      service.buckets().delete(bucket=args.bucket_name).execute()
      print 'Deleted bucket: %s' % (args.bucket_name)	  
    except client.AccessTokenRefreshError:
      print ("The credentials have been revoked or expired, please re-run the application to re-authorize") 		
  
  elif args.objet == 'object':
    try:
      service.objects().delete(bucket=args.bucket_name,object=args.object_name).execute()
      print 'Deleted object: %s' % (args.object_name)
    except client.AccessTokenRefreshError:
      print ("The credentials have been revoked or expired, please re-run the application to re-authorize") 
		
if __name__ == '__main__':
  if args.methode == 'create':
    create (args)
  elif args.methode == 'list':
    list (args)
  elif args.methode == 'get_object':
    get_object (args)
  elif args.methode == 'delete':
    delete (args)

```

#### 3-3-Managing CGS whith API 
_1- Generate an authentication URL_  
```sh
python gcs_api.py --noauth_local_webserver --methode 'create' --objet 'bucket' --bucket_name 'abdoul-test1'
```
_2- Open the authentication URL and click Accept_  
_3- Copy the code and complete the authentication process_  

_2- Example_
```sh
# create 2 buckets
python gcs_api.py --noauth_local_webserver --methode 'create' --objet 'bucket' --bucket_name 'abdoul-test1' # 
python gcs_api.py --noauth_local_webserver --methode 'create' --objet 'bucket' --bucket_name 'abdoul-test2' # 

# upload file to bucket 1
python gcs_api.py --noauth_local_webserver --methode 'create' --objet 'object' --bucket_name 'abdoul-test1' --object_name 'fichier1.txt' --object_source   '/home/araoufougambo_netbooster_com/gcs/download/fichier1.txt' #

# get file from bucket 1 
python gcs_api.py --noauth_local_webserver --methode 'get_object' --objet 'object' --bucket_name 'abdoul-test1' --object_name 'fichier1.txt' --object_dest '/home/araoufougambo_netbooster_com/gcs/download/fichier1_download.txt'

# del file of bucket 1
python gcs_api.py --noauth_local_webserver --methode 'delete' --objet 'object' --bucket_name 'abdoul-test1' --object_name 'fichier1.txt'

# del bucket 2
python gcs_api.py --noauth_local_webserver --methode 'delete' --objet 'bucket' --bucket_name 'abdoul-test2'
```

_3- Loading all file from given repository to google cloud storage script_
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from os import listdir
from os.path import isfile, join
import argparse

# Loading all file from given repository to google cloud storage
# Usage example: python loadin_files_togcs.py --bucket_name 'abdoul-test1' --object_source '~/gcs/download/'

# Define arguments
parser = argparse.ArgumentParser()
parser.add_argument('--bucket_name',  required=True)
parser.add_argument('--object_source',  required=True)

# Parse arguments
args = parser.parse_args()
path = args.object_source

# Loading files
for file in listdir(path):
  if isfile(join(path,file)):
    print "Processing %s file..." % (file)
    python gcs_api.py --noauth_local_webserver --methode 'create' --objet 'object' --bucket_name args.bucket_name --object_name file --object_source join(path,file))
```

