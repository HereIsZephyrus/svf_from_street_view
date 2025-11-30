# SVF from Street View

!This Doc is archived by AI
一个通过百度街景API获取街景图片，经处理后计算街道开阔度（天空可视因子，SVF）的工具。


## 项目简介

本项目旨在通过自动化流程从百度街景获取指定坐标的街景图片，经天空分割、鱼眼图合成等处理，最终计算出街道的天空可视因子（Sky View Factor, SVF），为城市规划、微气候分析等场景提供数据支持。


## 核心功能

1. **街景图片爬取**：通过百度街景REST API，根据经纬度坐标获取多个方向（0°/90°/180°/270°）的街景图片。
2. **坐标转换**：支持WGS84与百度坐标系（BD09、BD09MC）的转换，确保坐标匹配准确性。
3. **天空分割**：使用预训练的图像分割模型（SegFormer）提取街景图片中的天空区域。
4. **鱼眼图合成**：将同一坐标不同方向的天空掩码合成为鱼眼视图，模拟全景天空视角。
5. **SVF计算**：基于鱼眼图分析天空占比，计算街道开阔度（SVF）并输出结果。


## 环境配置

项目依赖通过conda管理，建议使用以下命令创建环境：

```bash
conda env create -f environment.yaml
conda activate taz
```


## 使用步骤

### 1. 准备输入数据

需准备包含目标坐标的CSV文件（默认路径：`./dir/point_coordinate_intersect.csv`），至少包含WGS84坐标系的经度（wgs_x）和纬度（wgs_y）。

CSV格式示例：
```
...,# 其他列（可选）,wgs_x,wgs_y
...,...,116.397470,39.908823
```


### 2. 运行街景爬取

执行街景图片爬取脚本，获取指定坐标的多方向街景图片：

```bash
python src/calc_svf/baiduStreetViewSpider.py
```

- 爬取的图片默认保存至 `./dir/images`，命名格式为 `{wgs_x}_{wgs_y}_{方向}_{俯仰角}.png`。
- 爬取失败的坐标会记录至 `./dir/error_road_intersection.csv`。


### 3. 天空分割与鱼眼图合成

运行天空分割脚本，提取天空掩码并合成鱼眼图：

```bash
python src/calc_svf/sky_segmentation.py
```

- 天空掩码保存至 `./sky_masks`，命名格式为 `mask_{原图片名}.png`。
- 合成的鱼眼图保存至 `./fisheye_output`，命名格式为 `fisheye_{wgs_x}_{wgs_y}.png`。


### 4. 计算SVF

执行SVF计算脚本，基于鱼眼图输出最终结果：

```bash
python src/calc_svf/fisheye_svfcalculator.py
```

- 计算结果保存至 `coordinate_svf_results.csv`，包含坐标（wgs_x, wgs_y）和对应的SVF值。


## 核心模块说明

| 模块文件 | 功能说明 |
|----------|----------|
| `baiduStreetViewSpider.py` | 街景图片爬取核心逻辑，包含坐标转换、API请求、图片保存等功能。 |
| `sky_segmentation.py` | 基于SegFormer模型的天空分割，以及多方向掩码合成鱼眼图的逻辑。 |
| `fisheye_svfcalculator.py` | 异步处理坐标数据，整合爬取、分割、合成流程，最终计算SVF并输出结果。 |
| `LCZ/definition.py` | （可选）包含本地气候区（LCZ）的定义与转换工具，可辅助分析区域特征。 |


## 注意事项

1. **API使用规范**：爬取街景图片时已添加6秒间隔以避免请求过于频繁，使用时请遵守百度地图API的使用条款。
2. 输入CSV需确保包含正确的WGS84坐标，否则可能导致街景图片获取失败。
3. 模型依赖：天空分割使用`nvidia/segformer-b0-finetuned-ade-512-512`模型，首次运行会自动下载（需网络连接）。


## 许可证

本项目基于MIT许可证开源，详见 [LICENSE](LICENSE)
