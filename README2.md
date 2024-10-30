<h1 align="center">
  视频切帧和去噪超分流程
</h1>


## 一.总体项目结构
```text
/opt/drz/DN_stableSR
|-src 去噪超分目录
| |- StableSR 超分项目
| |- SCUNet-main 去噪项目
|-videos 存放视频的目录
| |- tdyjd 视频的英文首字母缩写
|   |- image 原始视频的图片目录
|   |- .MP4 原始视频
| |- xxx
|   |- image 原始视频的图片目录
|   |- .mp4 原始视频
```

## 二.视频->帧

```shell
# 进入视频目录
cd /opt/drz/DN_stableSR/videos/xxx
# 在存放视频的目录中执行
ffmpeg -i 视频.mp4 -vf bwdif=0:-1:0 images/%6d.png
```

## 三.分目录后去噪超分
1. 根据显卡数量划分目录, 在去噪或者超分的目录中执行file.py文件 (使用前需要修复里面的num_file参数, 对应文件夹数量)
```shell
# 划分文件夹数量
# --inpath 划分前的所有图片目录 --newpath 划分后的目录
# 根据数量会在后面依次加上数字如 :ymnj/image1 ymnj/image2 ....
python file.py --inpath /opt/drz/DN_stableSR/videos/ymnj/image --newpath /opt/drz/DN_stableSR/videos/ymnj/image
```

2. 去噪 
```shell
#!/bin/bash
path=$(pwd)
# 进入去噪项目目录
cd /opt/drz/DN_stableSR/src/SCUNet-main/
# 打开xxx.sh文件, 根据文件夹数量指定显卡编号, 修改对应的输入/输出路径
# 示例如下:
CUDA_VISIBLE_DEVICES=0 python main_test_scunet_real_application.py --model_name scunet_color_real_gan --testsets $path --testset_name images_1 --results $path &
CUDA_VISIBLE_DEVICES=1 python main_test_scunet_real_application.py --model_name scunet_color_real_gan --testsets $path --testset_name images_2 --results $path &
CUDA_VISIBLE_DEVICES=2 python main_test_scunet_real_application.py --model_name scunet_color_real_gan --testsets $path --testset_name images_3 --results $path &
CUDA_VISIBLE_DEVICES=3 python main_test_scunet_real_application.py --model_name scunet_color_real_gan --testsets $path --testset_name images_4 --results $path
# 最后运行sh文件
sh xxx.sh
```
3. 超分
```shell
#!/bin/bash
path=$(pwd)
# 进入超分项目目录
cd /opt/drz/DN_stableSR/src/StableSR
# 打开xxx.sh文件, 根据文件夹数量指定显卡编号, 修改对应的输入/输出路径
# 示例如下:
CUDA_VISIBLE_DEVICES=0 python scripts/test1010.py --config configs/stableSRNew/v2-finetune_text_T_768v.yaml --ckpt model/stablesr_768v_000139.ckpt --vqgan_ckpt model/face_vqgan_cfw_00011.ckpt --init-img /opt/drz/DN_stableSR/videos/ymnj/unt1 --outdir /opt/drz/DN_stableSR/videos/ymnj/sd --ddpm_steps 2 --dec_w 0.5  --upscale 3.0 --colorfix_type wavelet --f 8 --n_samples 8 &
CUDA_VISIBLE_DEVICES=1 python scripts/test1010.py --config configs/stableSRNew/v2-finetune_text_T_768v.yaml --ckpt model/stablesr_768v_000139.ckpt --vqgan_ckpt model/face_vqgan_cfw_00011.ckpt --init-img /opt/drz/DN_stableSR/videos/ymnj/unt2 --outdir /opt/drz/DN_stableSR/videos/ymnj/sd --ddpm_steps 2 --dec_w 0.5  --upscale 3.0 --colorfix_type wavelet --f 8 --n_samples 8 &
CUDA_VISIBLE_DEVICES=2 python scripts/test1010.py --config configs/stableSRNew/v2-finetune_text_T_768v.yaml --ckpt model/stablesr_768v_000139.ckpt --vqgan_ckpt model/face_vqgan_cfw_00011.ckpt --init-img /opt/drz/DN_stableSR/videos/ymnj/unt3 --outdir /opt/drz/DN_stableSR/videos/ymnj/sd --ddpm_steps 2 --dec_w 0.5  --upscale 3.0 --colorfix_type wavelet --f 8 --n_samples 8 &
CUDA_VISIBLE_DEVICES=3 python scripts/test1010.py --config configs/stableSRNew/v2-finetune_text_T_768v.yaml --ckpt model/stablesr_768v_000139.ckpt --vqgan_ckpt model/face_vqgan_cfw_00011.ckpt --init-img /opt/drz/DN_stableSR/videos/ymnj/unt4 --outdir /opt/drz/DN_stableSR/videos/ymnj/sd --ddpm_steps 2 --dec_w 0.5  --upscale 3.0 --colorfix_type wavelet --f 8 --n_samples 8 &
# 最后运行sh文件
sh xxx.sh
```
**upscale根据输入和输出分辨率和显存大小调整**

## 四.合成视频
```shell
# 使用指定格式的代码将超分后的目录合成视频
# 示例如下:
ffmpeg -framerate 25 -i $(pwd)/images/%06d_out.png \ 
  -i $(pwd)/视频.mp4 -s 1920*1080 -map 0:v -map 1:a \
  -c:v libx264 -b:v 15000K -maxrate 15000K  -pix_fmt yuv420p \
  -c:a copy $(pwd)/视频.dn.sr.xj.mp4 -y
```
