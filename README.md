> That enroot you use on your cluster!

## Pulling Container Images!

1. Create an account at `https://org.ngc.nvidia.com/`
2. Get your NVIDIA NGC API key, and save it somewhere safe
3. Install NGC CLI, follow instructions at `https://org.ngc.nvidia.com/setup/installers/cli`
```bash
# for arm
wget --content-disposition https://api.ngc.nvidia.com/v2/resources/nvidia/ngc-apps/ngc_cli/versions/4.16.0/files/ngccli_arm64.zip -O ngccli_arm64.zip && unzip ngccli_arm64.zip
# Put it in your path, so you can use it anytime! And do start a new shell!
chmod u+x ngc-cli/ngc
echo "export PATH=\"\$PATH:$(pwd)/ngc-cli\"" >> ~/.bash_profile && source ~/.bash_profile
```
4. Login to you ngc!
```bash
ngc config set
```
5. Login to docker with that beautiful NGC API key. (For the username, enter '$oauthtoken' exactly as shown. It is a special authentication key for all users.)

```bash
docker login nvcr.io

# Username: $oauthtoken
# Password: <Your Key>
```

6. Try pulling an image

```bash
enroot import docker://'$oauthtoken':${NGC_API_KEY}@nvcr.io#nvidia/cuda:12.9.0-cudnn-runtime-ubuntu24.04
```

## Building Container Images
