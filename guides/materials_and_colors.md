# マテリアルと配色ガイド

このプロジェクトでは、指定された4色のカラーパレットを使用して都市景観を作成します。このガイドでは、Blenderでこれらの色を効果的に使用する方法と、スタイライズされたアイソメトリック表現に適したマテリアル設定について説明します。

## カラーパレット

プロジェクトで使用する配色は以下の通りです：

- **ダークティール**: `#034C53` - RGB(3, 76, 83)
- **ミディアムティール**: `#007074` - RGB(0, 112, 116)
- **コーラル**: `#F38C79` - RGB(243, 140, 121)
- **ライトコーラル**: `#FFC1B4` - RGB(255, 193, 180)

### 配色の使用方針

1. **建物の主要部分**: ダークティールとミディアムティールを使用
2. **アクセントと特徴的な建物**: コーラルを使用
3. **ハイライトと窓の反射**: ライトコーラルを使用
4. **道路と地面**: ダークティールのバリエーション
5. **空と大気**: ティールとコーラルのグラデーション

## Blenderでのカラー設定

### 基本マテリアルの作成

```python
import bpy

# カラーパレットの定義
colors = {
    "dark_teal": (0.012, 0.298, 0.325, 1.0),    # #034C53
    "medium_teal": (0.0, 0.439, 0.455, 1.0),     # #007074
    "coral": (0.953, 0.549, 0.475, 1.0),         # #F38C79
    "light_coral": (1.0, 0.757, 0.706, 1.0)      # #FFC1B4
}

# 基本マテリアルの作成
def create_basic_material(name, color):
    mat = bpy.data.materials.new(name)
    mat.use_nodes = True
    
    # ノードへの参照を取得
    nodes = mat.node_tree.nodes
    principled = nodes.get("Principled BSDF")
    
    # ベースカラーを設定
    principled.inputs["Base Color"].default_value = color
    
    # スタイライズしたルックのためのパラメータ調整
    principled.inputs["Specular"].default_value = 0.1
    principled.inputs["Roughness"].default_value = 0.9
    
    return mat

# パレット内の各色のマテリアルを作成
for color_name, color_value in colors.items():
    create_basic_material(color_name, color_value)
```

## スタイライズされたマテリアル設定

アイソメトリックスタイルでは、一般的にフラットでシンプルなマテリアルが使われます。以下の設定を参考にしてください：

### 建物の外壁用マテリアル

1. **ノードセットアップ**:
   - Principled BSDFノードのベースに、指定されたティールカラーを設定
   - 必要に応じて「Color Ramp」ノードを使用して、階調を追加
   - 「Ambient Occlusion」ノードを使って、角や隅にわずかな陰影を追加

2. **テクスチャマッピング**:
   - 窓や建物の詳細を表現するため、単純な繰り返しパターンを使用
   - UVマッピングではなく、「Generated」または「Object」座標を使うとブロック状のモデルに適しています

### クレーン用マテリアル

1. **ベース設定**:
   - コーラルカラー（`#F38C79`）をベースに使用
   - マットな仕上がりにするために「Roughness」を高く設定
   
2. **詳細の追加**:
   - エッジをハイライトするために「Fresnel」ノードを使用
   - ライトコーラル（`#FFC1B4`）でエッジを強調

### 窓と反射

1. **窓パターン**:
   - 「Brick Texture」ノードを使って規則的な窓パターンを作成
   - カラーマッピングでライトコーラルとダークティールを組み合わせて使用

2. **反射効果**:
   - 夕日の反射を表現するために「Gradient Texture」を使用
   - 建物の上部にコーラル色の反射、下部にダークティールの影を配置

## トゥーンシェーディング（オプション）

より様式化された見た目にするために、トゥーンシェーディング技術を使用できます：

```python
import bpy

def create_toon_material(name, color, num_steps=3):
    mat = bpy.data.materials.new(name)
    mat.use_nodes = True
    
    # ノードツリーをクリア
    nodes = mat.node_tree.nodes
    links = mat.node_tree.links
    nodes.clear()
    
    # 出力ノード
    output = nodes.new(type='ShaderNodeOutputMaterial')
    output.location = (300, 0)
    
    # ディフューズBSDFノード
    diffuse = nodes.new(type='ShaderNodeBsdfDiffuse')
    diffuse.location = (-300, 0)
    diffuse.inputs["Color"].default_value = color
    
    # カラーランプ（トゥーン効果用）
    ramp = nodes.new(type='ShaderNodeValToRGB')
    ramp.location = (0, 0)
    
    # ステップ状のグラデーションを作成
    ramp.color_ramp.elements[0].position = 0.0
    ramp.color_ramp.elements[0].color = (0.0, 0.0, 0.0, 1.0)  # 影の色
    
    ramp.color_ramp.elements[1].position = 0.3
    ramp.color_ramp.elements[1].color = (1.0, 1.0, 1.0, 1.0)  # 明るい部分の色
    
    # ステップ数を調整
    for i in range(1, num_steps):
        pos = i * (1.0 / num_steps)
        element = ramp.color_ramp.elements.new(pos)
        element.color = (1.0, 1.0, 1.0, 1.0)
    
    # シェーダーノード
    shader = nodes.new(type='ShaderNodeShaderToRGB')
    shader.location = (-150, 0)
    
    # リンクを作成
    links.new(diffuse.outputs["BSDF"], shader.inputs["Shader"])
    links.new(shader.outputs["Color"], ramp.inputs["Fac"])
    links.new(ramp.outputs["Color"], output.inputs["Surface"])
    
    return mat
```

## 実装の注意点

- マテリアルは再利用可能な形で作成し、複数のオブジェクトで共有しましょう
- バリエーションを作る際は、基本色からわずかに変更するだけで統一感を保てます
- 夕暮れの雰囲気を強調するために、世界の環境照明にもティールとコーラルのグラデーションを使用しましょう
- 影の強さを控えめにして、カラーパレットが明確に表示されるようにします