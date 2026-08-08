# Godot Networking — 快速参考

最后验证: 2026-02-12 | 引擎: Godot 4.6

## 自 ~4.3（LLM 截止时间）以来的变更

### 4.6 变更
- **破坏性变更中的 Networking 部分**：有关 4.5→4.6 级别的具体内容，请参阅官方迁移指南

### 4.5 变更
- **无重大 networking API 破坏** — 核心 multiplayer API 保持稳定

## 当前 API 模式

### 高层 Multiplayer
```gdscript
# Server
func host_game(port: int = 9999) -> void:
    var peer := ENetMultiplayerPeer.new()
    peer.create_server(port)
    multiplayer.multiplayer_peer = peer
    multiplayer.peer_connected.connect(_on_peer_connected)
    multiplayer.peer_disconnected.connect(_on_peer_disconnected)

# Client
func join_game(address: String, port: int = 9999) -> void:
    var peer := ENetMultiplayerPeer.new()
    peer.create_client(address, port)
    multiplayer.multiplayer_peer = peer
```

### RPCs
```gdscript
# Server-authoritative 模式
@rpc("any_peer", "call_local", "reliable")
func request_action(action_data: Dictionary) -> void:
    if not multiplayer.is_server():
        return
    # 在服务器上验证，然后广播
    _execute_action.rpc(action_data)

@rpc("authority", "call_local", "reliable")
func _execute_action(action_data: Dictionary) -> void:
    # 所有 peer 执行已验证的动作
    pass
```

### MultiplayerSpawner 和 MultiplayerSynchronizer
```gdscript
# 使用 MultiplayerSpawner 进行自动节点复制
# 使用 MultiplayerSynchronizer 进行属性同步

# MultiplayerSynchronizer 设置：
# 1. 作为要同步的节点的子节点添加
# 2. 在编辑器中配置复制属性
# 3. 为 relevancy 设置可见性过滤器
```

### SceneMultiplayer 配置
```gdscript
func _ready() -> void:
    var scene_mp := multiplayer as SceneMultiplayer
    scene_mp.auth_callback = _authenticate_peer
    scene_mp.server_relay = false  # 直接 peer 连接

func _authenticate_peer(id: int, data: PackedByteArray) -> void:
    # 自定义认证逻辑
    pass
```

## 常见错误
- 客户端到服务器的 RPC 未使用 `"any_peer"`（默认仅为 authority）
- 信任客户端数据而未经服务器端验证
- 对游戏状态变更使用 `"unreliable"`（仅用于位置更新）
- 未在生成的节点上设置 multiplayer authority（`set_multiplayer_authority()`）
