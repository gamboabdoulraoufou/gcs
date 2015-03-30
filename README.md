## Google Cloud Storage
This post provides a quick and easy way to get started with Google Cloud Storage on a linux environment with gsutil tool. It shows you how to:
- Install gsutils tools
- Set up access to protected data
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
tar xfz gsutil.tar.gz

# Add gsutil to your PATH environment variable
export PATH=${PATH}:$HOME/gcs/gsutil

# Restart your shell or terminal program

# Updating gsutil
update gsutil
```

### 2- Set-up access to protected data
#### 2-1- Using Credentials
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

#### 2-2- Using JSON API for Python    
#### 2-2-1- Set up environment  
```sh
virtualenv env
./env/bin/pip install --upgrade google-api-python-client httplib2 argparse
source ./env/bin/activate
```

#### 2-2-2- Create a client secrets file 
To generate a client secrets file:    
1- Go to the Google Developers Console.    
2- Select a project to which the client ID will be associated.  
3- In the left sidebar, select APIs & auth > Credentials.  
4- Click Create new Client ID.  
5- In the Create Client ID window, choose Installed application.  
6- Click Create Client ID.  
7- Click Download JSON  

#### 2-2-3- Some examples of API V1 methodes on `bucket` ans `object` 
For more details [log here](https://cloud.google.com/storage/docs/json_api/v1/)   
 
```python
# Create bucket
req = client.buckets().insert(
        project=project_id,
        body={'name': bucket_name})
resp = req.execute()
print json.dumps(resp, indent=2)
```

```python
# Delete bucket
client.buckets().delete(bucket=bucket_name).execute()
```

```python
# Create object
if reuse_metadata:
    destination_object_resource = {}
else:
    destination_object_resource = {
            'contentLanguage': 'en',
            'metadata': {'my-key': 'my-value'},
    }
req = client.objects().copy(
        sourceBucket=bucket_name,
        sourceObject=old_object,
        destinationBucket=bucket_name,
        destinationObject=new_object,
        body=destination_object_resource)
resp = req.execute()
print json.dumps(resp, indent=2)
```

```python
# List objects
fields_to_return = 'nextPageToken,items(bucket,name,metadata(my-key))'
req = client.objects().list(
        bucket=bucket_name,
        fields=fields_to_return,    # optional
        maxResults=42)              # optional

while req is not None:
    resp = req.execute()
    print json.dumps(resp, indent=2)
    req = client.objects().list_next(req, resp)
```

```python
# Get Payload Data
req = client.objects().get_media(
        bucket=bucket_name,
        object=object_name,
        generation=generation)    # optional
# The BytesIO object may be replaced with any io.Base instance.
fh = io.BytesIO()
downloader = http.MediaIoBaseDownload(fh, req, chunksize=1024*1024)
done = False
while not done:
    status, done = downloader.next_chunk()
    if status:
        print 'Download %d%%.' % int(status.progress() * 100)
    print 'Download Complete!'
print fh.getvalue()

# Delete object
client.objects().delete(
        bucket=bucket_name,
        object=object_name).execute()
```

```python
# Get Metadata
req = client.objects().get(
        bucket=bucket_name,
        object=object_name,
        fields='bucket,name,metadata(my-key)',    # optional
        generation=generation)                    # optional
resp = req.execute()
print json.dumps(resp, indent=2)
```

#### 2-2-4- Complete example: loading data
**Create sample file**
```python
"""
You can also get help on all the command-line flags the program understands
by running:
  $ python storage-sample.py --help
"""

import argparse
import httplib2
import os
import sys
import json

from apiclient import discovery
from oauth2client import file
from oauth2client import client
from oauth2client import tools

# Define sample variables.
_BUCKET_NAME = '[[INSERT_YOUR_BUCKET_NAME_HERE]]'
_API_VERSION = 'v1'

# Parser for command-line arguments.
parser = argparse.ArgumentParser(
    description=__doc__,
    formatter_class=argparse.RawDescriptionHelpFormatter,
    parents=[tools.argparser])


# CLIENT_SECRETS is name of a file containing the OAuth 2.0 information for this
# application, including client_id and client_secret. You can see the Client ID
# and Client secret on the APIs page in the Cloud Console:
# <https://console.developers.google.com/>
CLIENT_SECRETS = os.path.join(os.path.dirname(__file__), 'client_secrets.json')

# Set up a Flow object to be used for authentication.
# Add one or more of the following scopes. PLEASE ONLY ADD THE SCOPES YOU
# NEED. For more information on using scopes please see
# <https://developers.google.com/storage/docs/authentication#oauth>.
FLOW = client.flow_from_clientsecrets(CLIENT_SECRETS,
  scope=[
      'https://www.googleapis.com/auth/devstorage.full_control',
      'https://www.googleapis.com/auth/devstorage.read_only',
      'https://www.googleapis.com/auth/devstorage.read_write',
    ],
    message=tools.message_if_missing(CLIENT_SECRETS))


def main(argv):
  # Parse the command-line flags.
  flags = parser.parse_args(argv[1:])

  # If the credentials don't exist or are invalid run through the native client
  # flow. The Storage object will ensure that if successful the good
  # credentials will get written back to the file.
  storage = file.Storage('sample.dat')
  credentials = storage.get()
  if credentials is None or credentials.invalid:
    credentials = tools.run_flow(FLOW, storage, flags)

  # Create an httplib2.Http object to handle our HTTP requests and authorize it
  # with our good Credentials.
  http = httplib2.Http()
  http = credentials.authorize(http)

  # Construct the service object for the interacting with the Cloud Storage API.
  service = discovery.build('storage', _API_VERSION, http=http)

  try:
    req = service.buckets().get(bucket=_BUCKET_NAME)
    resp = req.execute()
    print json.dumps(resp, indent=2)

    fields_to_return = 'nextPageToken,items(name,size,contentType,metadata(my-key))'
    req = service.objects().list(bucket=_BUCKET_NAME, fields=fields_to_return)
    # If you have too many items to list in one request, list_next() will
    # automatically handle paging with the pageToken.
    while req is not None:
      resp = req.execute()
      print json.dumps(resp, indent=2)
      req = service.objects().list_next(req, resp)

  except client.AccessTokenRefreshError:
    print ("The credentials have been revoked or expired, please re-run"
      "the application to re-authorize")

if __name__ == '__main__':
  main(sys.argv)
```

**Run the sample**  
_1- Generate an authentication URL_  
```sh
python storage-sample.py --noauth_local_webserver
```
_2- Open the authentication URL and click Accept_  
_3- Copy the code and complete the authentication process_  
