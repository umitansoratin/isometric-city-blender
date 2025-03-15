# Blenderでのアイソメトリックビュー設定ガイド

アイソメトリック（等角投影）ビューは、3Dオブジェクトを2Dで表現する際に奥行き感を保ちながら表示する手法です。このガイドでは、Blenderでアイソメトリックビューを正確に設定する方法を説明します。

## カメラ設定手順

### 1. 直交投影カメラの追加

1. `Shift + A` → `Camera` でカメラを追加
2. カメラのプロパティパネルで、`Perspective`を`Orthographic`に変更
3. `Orthographic Scale`を適切な値に設定（シーンのサイズによる、初期値は10程度）

### 2. アイソメトリック角度の設定

真のアイソメトリックビューでは、XYZ軸が画面上で等しく表示されます。Blenderでこれを設定するには：

1. カメラを選択した状態で、回転値を以下のように設定：
   - X: 54.736°（またはπ/6 + π/4 ラジアン）
   - Y: 0°
   - Z: 45°（またはπ/4 ラジアン）

または、以下のPythonコードをBlenderのPythonコンソールで実行して正確に設定することも可能です：

```python
import bpy
import math
import mathutils

# アクティブなカメラを取得（または新しいカメラを作成）
if 'Camera' in bpy.data.objects:
    camera = bpy.data.objects['Camera']
else:
    bpy.ops.object.camera_add()
    camera = bpy.context.active_object

# カメラを直交投影に設定
camera.data.type = 'ORTHO'

# カメラの回転をアイソメトリック角度に設定
camera.rotation_euler = mathutils.Euler(
    (math.radians(54.736), 0, math.radians(45)),
    'XYZ'
)

# カメラをアクティブに設定
bpy.context.scene.camera = camera
```

### 3. レンダリング設定

1. レンダリングプロパティで、出力解像度を設定（例：4000 x 3000 px）
2. アスペクト比を4:3や16:9など希望の比率に設定
3. 必要に応じて、`Film` → `Transparent` をオンにして背景を透明にする

## グリッドの設定

アイソメトリックモデリングをしやすくするために、グリッドの設定も調整します：

1. `Overlays` パネルで、`Grid` オプションを開く
2. `Scale` を1に設定し、`Subdivisions` を10に設定
3. XYZの軸の色を確認し、モデリング時の方向の参考にする

## 注意点

- 真のアイソメトリックでは、すべての軸が画面上で120度の角度を形成します
- モデリング時は、ワールド軸に沿って構築することで、アイソメトリックビューでの見え方が整います
- テクスチャリングの際は、マテリアルプレビューをアイソメトリックビューに設定すると作業がしやすくなります

## レンダリングのヒント

- Cycles または Eevee エンジンのどちらでもアイソメトリックレンダリングが可能です
- スタイライズされた仕上がりには、Freestyle や Toon Shaderの使用を検討します
- 影の設定は控えめにして、フラットな見た目を維持することが一般的です