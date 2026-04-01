<p align="center">
  <img src="./image/logo.png" alt="logo" style="width: 80px; vertical-align: middle; ">
</p>
<h1 align="center">随机校园食物</h1>
<p align="center">
  <img src="https://img.shields.io/github/languages/code-size/xiaolizi0v0/FoodInSchool" alt="code size"/>
  <img src="https://img.shields.io/badge/Spring%20Boot-2.5.4-brightgreen" alt="Spring Boot"/>
  <img src="https://img.shields.io/github/languages/count/xiaolizi0v0/FoodInSchool" alt="languages"/>
  <img src="https://img.shields.io/badge/Java-8-blue" alt="Java"/>
  <img src="https://img.shields.io/github/last-commit/xiaolizi0v0/FoodInSchool" alt="last commit"/><br>
  <img src="https://img.shields.io/badge/Created-25.11.03-blue" alt="Created Time"/>
  <img src="https://img.shields.io/badge/Author-xiaolizi0v0-orange" alt="Author" />
</p>
<hr>

#### 工作：

1采集学校地图信息，并尝试卡通建筑建模

2调整地图建模精度，实现模型与现实场景比例缩放

3校准地图坐标，实现建筑道路导航

4实地获取食物信息，并制作食堂建筑物点击功能，食物添加功能

5添加食物信息指标

6实现食物随机功能

这三件事在 Unity 里可以按“三层流水线”实现：数据层（地图采集）→表现层（卡通建模与比例）→功能层（坐标校准与导航）。

1. 采集学校地图信息 + 卡通建筑建模  
1. 数据采集（先定标准）  
- 采集内容：建筑轮廓、道路中心线、出入口点、地标点、高程点。  
- 推荐来源：学校 CAD 总平图、无人机正射影像、手持 GPS/RTK 控制点、OpenStreetMap 补充。  
- 关键要求：所有数据统一到同一坐标系（例如 WGS84 UTM 或本地投影坐标）。

2. 数据预处理（QGIS 或 ArcGIS）  
- 清理拓扑：闭合建筑面、连通道路线、去重节点。  
- 导出格式：建筑/道路用 GeoJSON 或 Shapefile，模型用 FBX。  
- 给建筑附加属性：楼层数、用途、名称（后续做导航与标签要用）。

3. 卡通建模（Blender + Unity URP）  
- 建筑用低多边形风格：按建筑轮廓挤出，减少细碎结构，强调屋顶轮廓。  
- 材质用统一色板：少量主色 + 描边/分层阴影（Toon Shader）。  
- 在 Unity 中用 LOD Group：近景高模、远景低模，保证移动端帧率。

2. 调整地图建模精度与现实比例  
1. 统一尺度规则  
- Unity 建议 1 单位 = 1 米。  
- 选一段实测距离做比例校正，比例系数：  
  $k = \frac{D_{real}}{D_{model}}$  
- 把地图根节点整体乘以 $k$，再复测 3-5 条基准线。

2. 坐标转本地平面（用于定位和导航）  
- 选校园中心点作为原点 $(lat_0, lon_0, h_0)$。  
- 近似换算（小范围校园够用）：  
  $x = (lon-lon_0)\cdot \cos(lat_0)\cdot 111320$  
  $z = (lat-lat_0)\cdot 110540$  
  $y = h-h_0$

3. 精度验收建议  
- 建筑角点平面误差控制在 0.5-1.5 米。  
- 导航路径长度误差控制在 3% 内。  
- 如果校园很大（几公里级），加 Floating Origin 方案减少抖动。

3. 校准地图坐标，实现建筑道路导航  
1. 控制点校准  
- 采集至少 4 个地面控制点（越分散越好）。  
- 在 Unity 场景中摆对应点，解二维相似变换（平移+旋转+统一缩放）：  
  $p_{unity} = sR p_{map} + t$  
- 用残差/RMSE 检查标定质量，超阈值就重采点或剔除异常点。

2. 道路导航实现  
- 方案 A（推荐校园道路）：道路中心线建图，节点-边结构，A* 寻路。  
- 方案 B（可行走区域）：烘焙 NavMesh，用 NavMeshAgent。  
- 实战常用：室外道路用 A*，楼内短路径或动态障碍用 NavMesh。

3. 路径显示与到达逻辑  
- 起点终点先吸附到最近道路节点。  
- 路径用 LineRenderer 或样条线可视化。  
- 建筑入口做“门点”，路径最终连接到门点而不是建筑中心点。

4. 在 Unity 的最小可落地模块  
1. 地图导入模块：读取 GeoJSON/CAD 转换结果。  
2. 坐标转换模块：经纬度/投影坐标转 Unity 本地坐标。  
3. 校准模块：控制点求变换参数并持久化。  
4. 路网构建模块：从道路数据生成图结构。  
5. 寻路模块：A* 返回节点序列。  
6. 渲染模块：路径线、路标、建筑名牌。

如果你愿意，我下一步可以直接给你一套可用的 Unity 脚本骨架（坐标转换 + 控制点标定 + A* 导航 + 路径渲染），你只要把学校数据替换进去就能跑。

