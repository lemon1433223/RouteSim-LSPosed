# RouteSim 路线模拟（LSPosed + Root）v1.3.0

一个**仅用于授权测试 / 开发调试 / 教学演示**的 Android 位置与路线模拟工具。
默认只 Hook 系统框架与 Google Play services，通过 Root 授权的系统 mock 位置
让目标 App 不注入也能收到模拟结果。

> ⚠️ 严禁用于诈骗、逃避监管、考试作弊、打卡代跑、侵犯隐私等任何违法或违约用途。
> 使用本工具产生的全部后果由使用者自行承担。首次启动会强制展示并确认使用警告。

---

## 1. 主要功能

- **系统级定位模拟**：默认只注入 `android`（system_server）与
  `com.google.android.gms`，不默认注入目标 App；Root 负责 appops / mock 位置授权。
- **单点定位 / 路线模拟**：单点静止，路线沿平滑曲线移动。
- **平滑曲线**：Chaikin 平滑，路线规划与模拟轨迹都更自然。
- **闭环模式**：新增“闭环”，首尾平滑连接，循环模拟。
- **终点保持**：单次模式结束后停在终点，不自动退出，避免定位跳回真实位置。
- **速度档位**：步行 / 跑步 / 骑行 / 驾车 / 自定义。
- **随机速度与海拔**：速度在档位范围内缓慢变化；海拔支持自动联网获取模拟地区地形，
  并按范围起伏。
- **方向一致**：移动方向做平滑跟随，带轻微抖动。
- **步频采集**：`ACTIVITY_RECOGNITION` + 计步传感器采集真实步频，模拟时按分布随机化。
- **地图**：高德 / Google / Mapbox / OSM / 自定义；默认高德，无需 Key；
  Mapbox 只需填 access token；连续缩放、父/子瓦片补位、外圈预取。
- **备份恢复**：将设置、路线、步频档案导出 JSON，并可恢复。
- **调试日志**：默认关闭，开启后写入 App 外部日志目录。
- **后台保活**：前台服务 + 常驻通知，通知栏可暂停/继续/结束。
- Material 浅色界面：圆角卡片、蓝色强调色。

---

## 2. 构建

### CodeAssist

1. 将 `RouteSim/` 放到手机存储。
2. CodeAssist → Open project → 选择该文件夹。
3. 安装 Android SDK Platform 33；Build / Assemble debug。
4. 生成的 APK 可安装。

### Android Studio

需要 JDK 17、SDK 33。项目带 Gradle Wrapper 7.5.1：

```bash
./gradlew assembleDebug
```

`app/libs/api-82.jar` 是 Xposed API，`compileOnly` 不会打进 APK。

---

## 3. Root 与 LSPosed

### Root

首次进入、同意警告后会弹出 Root 检测窗：

- 点击“授权”，在 Root 管理器允许 `su`；
- 模块会自动尝试：
  - `appops set com.routesim android:mock_location allow`
  - 定位/活动识别权限
  - 打开系统定位模式
- 授权成功后写入“已验证”状态；设置页可重新检测。
- 没有 Root 可以选“受限模式”，但可能收不到持续位置更新。

### LSPosed 推荐作用域

默认只需要：

| 作用域 | 作用 |
|---|---|
| 系统框架 (`android`) | system_server 改写位置 Parcel、Location 查询 |
| Google Play services (`com.google.android.gms`) | Fused Location 兜底 |

**不要再默认勾选目标 App**。位置会由系统组件 + Root mock 位置覆盖。
只有需要“目标 App 内的步频/BLE/基站增强”时，才把该 App 加入作用域；
这会重新引入注入风险，默认不推荐。

修改作用域后强制停止目标 App，必要时重启手机。

---

## 4. 地图配置

设置 → 地图：

| 服务 | 填写 | 坐标系 |
|---|---|---|
| 高德（默认） | 无需 Key | GCJ-02 |
| Google | 无需 Key | WGS-84 |
| Mapbox | 只填 access token | WGS-84 |
| OpenStreetMap | 无需 Key | WGS-84 |
| 自定义 | 高级里填 URL（可含 `{key}`/`{s}`） | 手动选择 |

路线点内部保存 WGS-84，地图显示时按服务转换。
如果目标 App 有固定偏移，可打开“输出 GCJ-02 坐标”。

---

## 5. 使用流程

1. 同意警告 → 授权 Root → 允许定位；地图自动移动到当前位置。
2. 选择“单点定位”或“路线模拟”。
   - 单点：自动把当前位置设为单点，点击地图可更换。
   - 路线：点击地图加点，拖动调整，长按删除。
3. 选择速度档位（步行/跑步/骑行/驾车/自定义）。
4. 选择循环模式：
   - 循环：到终点后直接回到起点；
   - 往返：到终点反向；
   - 单次：到终点后保持终点；
   - 闭环：首尾平滑连接成闭合路线。
5. 可开启“平滑曲线”“自动获取模拟地区海拔”。
6. 点开始模拟。通知栏显示速度/步频/距离，并提供 **暂停/继续、结束**。
7. 目标 App 无需加入作用域；如果仍有问题，看主界面 “Hook 进程”。

### 步频采集

设置 → 运动模拟 → “采集真实步频（约 30 秒）”，授予活动识别权限，
自然行走。采集结果用于速度/步频随机分布。只有把目标 App 加入作用域时，
步频 Hook 才会作用于该 App。

### 备份与恢复

设置 → 备份与恢复：

- 备份：选择文件保存位置，写入 JSON（设置、路线、步频档案、界面偏好）。
- 恢复：选择备份 JSON，确认后重启当前页面加载。

### 调试日志

设置 → Root 与调试 → 打开“收集调试日志”。日志默认关闭，写入：

`/sdcard/Android/data/com.routesim/files/logs/`

包含服务启停、Root 配置、Hook 失败信息和定位异常，便于开发者排查。
可在设置里清空。

---

## 6. 实现架构

| 组件 | 作用 |
|---|---|
| Root | 授予 mock_location appops、定位权限、打开系统定位 |
| system_server Hook | 改写 `Location` Parcel/查询；系统 mock 位置更新对所有 App 生效 |
| GMS Hook | `LocationResult`、Fused Task、LocationCallback；覆盖 Google 系应用 |
| 目标 App Hook | **默认不注入**；仅在用户手动加入作用域时提供步频/BLE/基站增强 |
| 前台服务 | 路线推进、速度/步频/海拔、通知暂停/结束、START_STICKY |

状态通过只读 `ContentProvider` 共享；Root mock 位置即使目标 App 没有 Hook
也能收到服务生成的坐标。system_server Parcel Hook 负责清掉 mock 标志、
补齐时间戳与卫星 extras。

---

## 7. `libsensorservice.so`

用户提供的 `/storage/emulated/0/libsensorservice.so` 是 MIUI 修改过的
AOSP SensorService（AArch64 系统库），包含
`MiSensorServiceStub::sendVirtualSensorData` 等虚拟传感器注入符号。

它是 system_server 系统库，不能安全塞进 APK：同名库冲突、依赖系统 so，
且需要 MIUI 权限/SELinux 放行。当前版本没有加载它，而是用
`SensorManager` 采集真实步频 + Hook 合成步事件。详细分析见
[docs/MIUI_SENSOR.md](docs/MIUI_SENSOR.md)。

---

## 8. 已知限制

- 不能保证对抗强检测：Root/LSPosed 检测、完整性校验、传感器交叉验证仍可能发现。
- 默认不注入目标 App，因此步频/BLE/基站对普通 App 不生效；需要这些功能时
  才把目标 App 加入作用域。
- Root mock 位置仍可能被专门检测 mock 的应用识别；Parcel Hook 只能降低常见标志。
- 地图服务可用性受网络、地区、瓦片服务商限制。
- 自动海拔依赖公开接口，失败时回退到基础海拔。
- 模拟位置可能违反第三方应用服务条款。

---

## 9. 排障

| 现象 | 排查 |
|---|---|
| 一直提示 Root 失败 | Root 管理器是否允许 su；换“受限模式”看 LSPosed 是否工作 |
| 目标 App 位置不变 | LSPosed 是否启用 android/GMS；Root 是否授权 mock；强制停止目标 App |
| Hook 进程为 0 | 系统框架/GMS 未注入；检查 LSPosed 日志 |
| 地图空白 | 切换 Google/OSM；Mapbox 是否填 token；网络是否能访问 |
| 地图缩放有方块 | 已做父/子瓦片补位；首次加载某区域仍可能短暂空白 |
| 步频不生效 | 目标 App 默认未注入；需手动加入作用域（高风险） |
| 日志没有内容 | 设置里是否开启；路径 `/sdcard/Android/data/com.routesim/files/logs/` |

---

## 10. 免责声明

仅供已获得明确授权的软件测试、开发调试、安全研究与教学演示。
禁止用于诈骗、伪造位置骗取利益、逃避监管、考试作弊、打卡代跑、跟踪他人等
违法或违约行为。模拟位置可能造成账号封禁、数据异常或法律责任。
开发者不对使用者的行为及后果负责。继续使用即表示同意上述内容。

---

## License

MIT License. See [LICENSE](LICENSE).

> 本项目仅用于已授权的测试、开发调试和教学演示。请勿用于违法或违约用途。
