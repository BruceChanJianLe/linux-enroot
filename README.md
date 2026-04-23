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
    --root \
    --mount /dev:/dev \
    --mount /mnt:/mnt \
    --mount /media:/media \
    --mount /tier1/jianle/docker_mount:/home/$USER/docker_mount \
    --env DISPLAY=${DISPLAY} \
    --env QT_X11_NO_MITSHM=1 \
    --env XAUTHORITY=${XAUTH} \
    --env NVIDIA_VISIBLE_DEVICES=all \
    --env NVIDIA_DRIVER_CAPABILITIES=all \
    --env UV_CACHE_DIR=/uv_cache \
    --mount /tier1/jianle/uv_cache:/uv_cache \
    u24
```

For the zsh users out there!

```bash
enroot start \
    --rw \
    --mount /dev:/dev \
    --mount /mnt:/mnt \
    --mount /media:/media \
    --mount /tier1/jianle/docker_mount:/home/$USER/docker_mount \
    --env DISPLAY=${DISPLAY} \
    --env QT_X11_NO_MITSHM=1 \
    --env XAUTHORITY=${XAUTH} \
    --env NVIDIA_VISIBLE_DEVICES=all \
    --env NVIDIA_DRIVER_CAPABILITIES=all \
    --env UV_CACHE_DIR=/uv_cache \
    --mount /tier1/jianle/uv_cache:/uv_cache \
    u24 zsh
```

## Attaching to a Container!

List the pid of the container:  
```bash
enroot list -f
```

Attach to the container that you like!  
```bash
enroot exec 12345 zsh
```

## Tips

<details>
<summary>1. Mount the uv cache on a shared drive</summary>

The `--mount /tier1/jianle/uv_cache:/uv-cache` flag maps a host directory into the container. Placing the uv cache on a shared/network drive has several advantages:

- **Shared across containers**: Multiple containers (from different builds or different users) reuse the same cached wheels and source distributions, avoiding redundant downloads.
- **Persistent across container restarts**: Enroot containers started with `--rw` persist changes, but if you recreate a container from the squashfs image, the cache would be lost. A mounted cache directory survives container teardown.
- **Shared across batch jobs**: When running Slurm or other batch jobs that each spin up their own container instance, a shared cache means the first job pays the download cost and all subsequent jobs install from cache. This is especially significant for large dependencies like PyTorch or scipy.
- **Disk efficiency**: Python package caches can grow to several gigabytes. Keeping a single shared copy avoids duplicating this across every container on the system.

Adjust the host path (`/tier1/jianle/uv_cache`) to match your cluster's shared filesystem (e.g., a tier-1 NFS mount or a Lustre directory accessible from all nodes).

</details>

## Ghostty User

For all you ghostty users out there, here's what you can do!

```bash
infocmp -x | ssh username@123.123.123.123 -- tic -x -
cp -r .terminfo/ /tier1/jianle/enroot/data/u24/home/develper
```

