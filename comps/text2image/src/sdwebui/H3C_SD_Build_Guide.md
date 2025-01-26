# H3C Stable Diffusion Image Build Guide

## 1. Build OPEA text2image Image

```shell
git clone https://github.com/opea-project/GenAIComps.git
cd GenAIComps
git checkout sdwebui
docker build -t opea/text2image-gaudi:latest --build-arg https_proxy=$https_proxy --build-arg http_proxy=$http_proxy -f comps/text2image/sdwebui/Dockerfile.intel_hpu .
```


## 2. Build Patched SD webui Image

```shell

# Enter container
docker run -it --name="opea_text2image" --rm  --runtime=habana -e HABANA_VISIBLE_DEVICES=all -e OMPI_MCA_btl_vader_single_copy_mechanism=none --cap-add=sys_nice --ipc=host     -e HF_ENDPOINT=https://hf-mirror.com -e http_proxy=$http_proxy -e https_proxy=$https_proxy     -e HF_TOKEN=$HF_TOKEN -e MODEL=stabilityai/stable-diffusion-2-1     --entrypoint /bin/bash     opea/text2image-gaudi:latest

cd /home/user/
git clone https://github.com/lkk12014402/stable-diffusion-webui.git
cd stable-diffusion-webui
python3.10 -m venv sd
source sd/bin/activate
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu
pip install -r requirements.txt
mkdir -p /home/user/stable-diffusion-webui/models/Stable-diffusion/
cd /home/user/stable-diffusion-webui/models/Stable-diffusion/
touch v1-5-pruned-emaonly.safetensors
cd /home/user/stable-diffusion-webui/
http_proxy="" /home/user/stable-diffusion-webui/sd/bin/python /home/user/stable-diffusion-webui/launch.py --skip-torch-cuda-test --share --skip-load-model-at-start --opea --opea-txt2img-url http://localhost:9379/sdapi/v1/txt2img --opea-img2img-url http://10.7.4.144:9379/sdapi/v1/img2img --server-name 0.0.0.0 --nowebui
```

After the Stable Diffusion service is started, open another terminal on the host and execute the following command to save the image

```shell
docker commit opea_text2image opea/sd-gaudi:pre
```

## 3. Build the final image 
Patch optimum-habana to fix some known issues
```shell
cd GenAIComps
docker build -t opea_sd-gaudi:1.0 --build-arg https_proxy=$https_proxy --build-arg http_proxy=$http_proxy -f comps/text2image/sdwebui/Dockerfile.sd-gaudi .
```