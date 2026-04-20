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

Alternatively, you can get it from your local docker daemon!

```bash
enroot import dockerd://ubuntu24.04:v0.0.5-cnnvros2-runtime
```

## Creating Enroot Container Images
> Maybe the title is not the most accurate, but you based do based off a docker container
> To create your enroot "image", which then start upon it!

1. If you have docker, use it to build an image, otherwise, I will have to say good-the-luck!
2. Once built, we need to create the enroot image
   ```bash
   enroot create --name u24 cuda:12.9.0-cudnn-runtime-ubuntu24.04.sqsh
   ```

## Removing Enroot Container Images

```bash
enroot remove u24
```

## Starting Container!

```bash
enroot start \
    --rw \
    --mount /dev:/dev \
    --mount /mnt:/mnt \
    --mount /media:/media \
    --mount /tier1/jianle/docker_mount:/home/developer/docker_mount \
    --mount ${XDG_RUNTIME_DIR}/${WAYLAND_DISPLAY}:/tmp/${WAYLAND_DISPLAY} \
    --env DISPLAY=${DISPLAY} \
    --env QT_X11_NO_MITSHM=1 \
    --env XAUTHORITY=${XAUTH} \
    --env WAYLAND_DISPLAY=/tmp/${WAYLAND_DISPLAY} \
    --env HOME=/home/developer \
    u24
```

For the zsh users out there!

```bash
enroot start \
    --rw \
    --mount /dev:/dev \
    --mount /mnt:/mnt \
    --mount /media:/media \
    --mount /tier1/jianle/docker_mount:/home/developer/docker_mount \
    --mount ${XDG_RUNTIME_DIR}/${WAYLAND_DISPLAY}:/tmp/${WAYLAND_DISPLAY} \
    --env DISPLAY=${DISPLAY} \
    --env QT_X11_NO_MITSHM=1 \
    --env XAUTHORITY=${XAUTH} \
    --env WAYLAND_DISPLAY=/tmp/${WAYLAND_DISPLAY} \
    --env HOME=/home/developer \
    u24 zsh
```

## Ghostty User

For all you ghostty users out there, here's what you can do!

```bash
infocmp -x | ssh username@123.123.123.123 -- tic -x -
cp -r .terminfo/ /tier1/jianle/enroot/data/u24/home/develper
```

