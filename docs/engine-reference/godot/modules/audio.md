# Godot Audio — 快速参考

最后验证：2026-02-12 | 引擎：Godot 4.6

## 自 ~4.3 以来的变更（LLM 知识截止点）

4.4–4.6 版本中 audio API 没有重大破坏性更改。核心 audio 系统
保持稳定。关键更新是工作流改进：

### 4.6 变更
- **此版本没有 audio 特定的破坏性更改**

### 4.5 变更
- **此版本没有 audio 特定的破坏性更改**

## 当前 API 模式

### 播放音频
```gdscript
@onready var sfx_player: AudioStreamPlayer = %SFXPlayer
@onready var music_player: AudioStreamPlayer = %MusicPlayer

func play_sfx(stream: AudioStream) -> void:
    sfx_player.stream = stream
    sfx_player.play()

func play_music(stream: AudioStream, fade_time: float = 1.0) -> void:
    var tween: Tween = create_tween()
    tween.tween_property(music_player, "volume_db", -80.0, fade_time)
    await tween.finished
    music_player.stream = stream
    music_player.volume_db = 0.0
    music_player.play()
```

### 3D 空间音频
```gdscript
@onready var audio_3d: AudioStreamPlayer3D = %AudioPlayer3D

func _ready() -> void:
    audio_3d.max_distance = 50.0
    audio_3d.attenuation_model = AudioStreamPlayer3D.ATTENUATION_INVERSE_DISTANCE
    audio_3d.unit_size = 10.0
```

### 音频总线
```gdscript
# Set bus volumes
AudioServer.set_bus_volume_db(AudioServer.get_bus_index(&"Music"), volume_db)
AudioServer.set_bus_volume_db(AudioServer.get_bus_index(&"SFX"), volume_db)

# Mute a bus
AudioServer.set_bus_mute(AudioServer.get_bus_index(&"Music"), true)
```

### 用于 SFX 的对象池
```gdscript
# Pre-create multiple AudioStreamPlayer nodes for concurrent sounds
var _sfx_pool: Array[AudioStreamPlayer] = []

func _ready() -> void:
    for i in range(8):
        var player := AudioStreamPlayer.new()
        player.bus = &"SFX"
        add_child(player)
        _sfx_pool.append(player)

func play_pooled(stream: AudioStream) -> void:
    for player in _sfx_pool:
        if not player.playing:
            player.stream = stream
            player.play()
            return
```

## 常见错误
- 在运行时创建新的 AudioStreamPlayer 节点而不是使用对象池
- 不使用音频总线来划分音量类别（Music、SFX、UI、Voice）
- 用 `_process()` 处理音频时序而不是用信号（`finished`）
