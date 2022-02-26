---
title: 视频抖动修复 Video Stabilization or Deshake
date: 2021-11-18 21:10:20
categories:
  
tags:
---

# 视频抖动修复 Video Stabilization or Deshake

当然用Pr或者 Davinci Resolve 肯定能办到，但难免有点杀鸡焉用牛刀。在此列举几种笔者发掘的方法。急用可直接拉到底看最佳方案。

 <!-- more -->

## Online

<https://www.stabilizo.com/>，可以免费处理500M以下的文件，感觉挺快的，效果较好。无需科学上网（当然科学上网速度应该会有提升）。

## ffmpeg

<!-- 本地文件：`~/.zshfunctions/ffmpeg_functions/deshakemov` -->

### deshake

较为快速。但快速大幅度抖动时，多出的边缘会有奇怪的镜像补全。

```bash
deshakemov(){
  file=$1
  echo $file
  resultFile=${file%.*}"_deshaked.mp4"
  # Way1: strange distortion on margin
  ffmpeg -i $file -hide_banner -map_metadata 0 -movflags use_metadata_tags -vf  deshake $resultFile
  touch -r $file $resultFile
}
```

### vid.stab

参考[这篇文章](https://rainnic.altervista.org/en/how-stabilize-video-using-ffmpeg-and-vidstab?language_content_entity=en)，先分析后修复。效果较好。

```bash
deshakemov(){
  file=$1
  echo $file
  resultFile=${file%.*}"_deshaked.mp4"
  # Way2: two step
  # ref: 
  ffmpeg -i $file \
      -vf vidstabdetect=stepsize=32:shakiness=10:accuracy=10:result=transforms.trf -f null -
  ffmpeg -i $file \
      -vf vidstabtransform=input=transforms.trf:zoom=0:smoothing=10,unsharp=5:5:0.8:3:3:0.4 \
      -vcodec libx264 -tune film -preset slow  \
      $resultFile
  rm transforms.trf
  touch -r $file $resultFile
}
```



## 效果对比

具体效果可参考b站视频：

[修复抖动对比：黄腰柳莺鸣](https://www.bilibili.com/video/BV1Gf4y1K7BZ/)

[修复抖动对比·原始版本：黄腰柳莺鸣](https://www.bilibili.com/video/BV1of4y1N7xe/)

[修复抖动对比·stabilizo.com：黄腰柳莺鸣](https://www.bilibili.com/video/BV17q4y1671G/)

[修复抖动对比·ffmpeg/deshake：黄腰柳莺鸣](https://www.bilibili.com/video/BV1pL4y1i7Qj/)

[修复抖动对比·ffmpeg/vib.stab：黄腰柳莺鸣](https://www.bilibili.com/video/BV1dL411T7vp/)



顺便放一下并排组合视频的代码：

```bash
ffmpeg -i DSCN8805.MOV -i DSCN8805_stabilizo.mov -i DSCN8805_deshaked.mp4 -i DSCN8805_vidstab_macos_i5-5350U.mp4 -filter_complex \
"[0]drawtext=text='original':fontsize=50:x=(text_w)/2:y=(text_h)/2[v0];
 [1]drawtext=text='stabilizo.com':fontsize=50:x=(text_w)/2:y=(text_h)/2[v1];
 [2]drawtext=text='deshake':fontsize=50:x=(text_w)/2:y=(text_h)/2[v2];
 [3]drawtext=text='vib.stab':fontsize=50:x=(text_w)/2:y=(text_h)/2[v3];
 [v0][v1][v2][v3]xstack=inputs=4:layout=0_0|w0_0|0_h0|w0_h0[v]" \
-map "[v]" stack.mp4
# 参考： https://stackoverflow.com/questions/11552565/vertically-or-horizontally-stack-mosaic-several-videos-using-ffmpeg
```

数据对比：

|   参数   | original | stabilizo.com |       deshake        | vib.stab |
| :------: | :------: | :-----------: | :------------------: | :------: |
| 大小/MB  |   157    |      57       |          75          |    68    |
| 用时估计 |    0     |     10min     |        15min         |  20min   |
| 主观体验 | 抖动明显 |     较好      | 抖动较弱，但边缘扭曲 |   较好   |

综合来说，stabilizo.com和vib.stab是比较好的工具。后者比较耗时（机器好的话会快很多，我的5代i5实在是拉垮），但可控性较强；一般情况下若网络给力，使用stabilizo.com即可。

