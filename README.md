# 基于 Query Routing 的智能饮食问答系统 

## 基于Kimi大模型的智能菜谱解析系统

### 🌟 新功能特性

- **🤖 AI智能解析**: 使用Kimi大模型准确提取菜谱信息
- **📊 结构化输出**: 自动生成标准化的知识图谱数据
- **🔄 批量处理**: 支持大规模菜谱目录的批量转换
- **💾 多格式导出**: 支持Neo4j和CSV两种输出格式
- **🎯 高精度**: AI模型确保食材分类、步骤解析的准确性
- **📁 智能目录识别**: 自动扫描dishes/目录，根据子目录名推断分类
- **⚡ 优化处理**: 减少AI分类工作，提高处理效率和准确性

### 🚀 快速开始

#### 1. 安装依赖

```bash
pip install -r requirements.txt
```

#### 2. 配置API密钥

方法一：设置环境变量

```bash
export KIMI_API_KEY="your_api_key_here"
```

方法二：编辑config.json

```json
{
  "kimi": {
    "api_key": "your_api_key_here"
  }
}
```

#### 3. 测试AI解析功能

```bash
python run_ai_agent.py test
```

#### 4. 批量处理菜谱

```bash
# 处理HowToCook项目（会自动扫描dishes/目录）
python run_ai_agent.py /path/to/HowToCook-master

# 处理其他菜谱目录
python run_ai_agent.py /path/to/your/recipes
```

### 📁 项目结构

```
cook-rag-example/
├── recipe_ai_agent.py         # AI解析核心引擎
├── run_ai_agent.py            # 简化运行脚本
├── batch_manager.py           # 批次管理工具
├── amount_normalizer.py       # 用量标准化工具
├── config.json                # 配置文件
├── requirements.txt           # 依赖列表
└── ai_output/                 # AI输出目录
    ├── nodes.csv              # Neo4j节点数据
    ├── relationships.csv      # Neo4j关系数据
    └── neo4j_import.cypher    # Neo4j导入脚本
```

### 📁 智能目录处理

#### 自动分类识别

系统会智能识别HowToCook项目的目录结构：

- **专门扫描**: 只处理 `dishes/` 目录下的菜谱文件
- **自动分类**: 根据子目录名自动推断菜谱分类
- **排除无关文件**: 自动跳过 `template/`、`.github/` 等目录

#### 目录分类映射

```
vegetable_dish/ → 素菜
meat_dish/     → 荤菜  
aquatic/       → 水产
breakfast/     → 早餐
staple/        → 主食
soup/          → 汤类
dessert/       → 甜品
drink/         → 饮料
condiment/     → 调料
semi-finished/ → 半成品
```

#### 处理优化

- **减少AI工作量**: 分类信息直接从目录结构获取，无需AI推理
- **提高准确性**: 避免AI分类错误，确保分类一致性
- **加快处理速度**: 减少API调用复杂度和处理时间

### 🤖 AI解析能力

#### 智能信息提取

AI系统能够从Markdown菜谱中准确提取：

1. **基本信息**
   - 菜谱名称
   - 难度等级（1-5星）
   - 菜谱分类（素菜/荤菜/水产等）
   - 菜系归属（川菜/粤菜等）

2. **食材信息** 
   - 食材名称和分类
   - 用量和单位
   - 主要食材识别

3. **烹饪步骤**
   - 步骤描述和顺序
   - 使用的烹饪方法
   - 需要的工具
   - 时间估计

4. **额外信息**
   - 准备时间/烹饪时间
   - 供应人数
   - 营养信息（当可用时）
   - 相关标签

#### 示例：AI解析结果

输入菜谱：

```markdown
# 红烧茄子的做法
预估烹饪难度：★★★★
## 必备原料和工具
- 青茄子
- 大蒜
- 酱油
- 面粉
```

AI解析输出：

```json
{
  "name": "红烧茄子",
  "difficulty": 4,
  "category": "素菜",
  "ingredients": [
    {
      "name": "青茄子",
      "category": "蔬菜",
      "is_main": true
    },
    {
      "name": "大蒜", 
      "category": "蔬菜",
      "is_main": false
    }
  ]
}
```

### 📊 知识图谱结构

#### Neo4j数据模型

**节点类型**:

- `Recipe`: 菜谱
- `Ingredient`: 食材  
- `CookingStep`: 烹饪步骤

**关系类型**:

- `has_ingredient`: 菜谱包含食材
- `has_step`: 菜谱包含步骤
- `belongs_to_category`: 属于分类
- `has_difficulty`: 具有难度

#### 示例查询

```cypher
// 查找所有包含茄子的菜谱
MATCH (recipe:Concept)-[:has_ingredient]->(ing:Concept)
WHERE ing.name CONTAINS "茄子"
RETURN recipe.name, recipe.difficulty

// 查找四星难度的素菜
MATCH (recipe:Concept)-[:belongs_to_category]->(cat:Concept)
WHERE cat.name = "素菜" AND recipe.difficulty = 4
RETURN recipe.name

// 推荐基于现有食材的菜谱
MATCH (recipe:Concept)-[:has_ingredient]->(ing:Concept)
WHERE ing.name IN ["茄子", "大蒜", "酱油"]
WITH recipe, count(ing) as matches
WHERE matches >= 2
RETURN recipe.name, matches
ORDER BY matches DESC
```

### ⚙️ 配置选项

#### config.json详细配置

```json
{
  "deepseek": {
    "api_key": "your_api_key",
    "base_url": "https://api.deepseek.com",
    "model": "deepseek-chat",
    "max_retries": 3,
    "timeout": 30
  },
  "processing": {
    "batch_size": 10,
    "delay_between_requests": 1,
    "max_concurrent_requests": 5
  },
  "output": {
    "format": "neo4j",
    "directory": "./ai_output",
    "include_nutrition": true,
    "include_tags": true
  }
}
```

### �️ 辅助工具

#### 批次管理工具 (batch_manager.py)

用于管理分批处理的进度和数据：

```bash
# 查看处理状态
python batch_manager.py status

# 继续中断的处理
python batch_manager.py continue /path/to/recipes

# 合并批次数据
python batch_manager.py merge

# 清理进度文件
python batch_manager.py clean-progress

# 清理批次数据
python batch_manager.py clean-batches

# 显示批次详情
python batch_manager.py details
```

#### 用量标准化工具 (amount_normalizer.py)

提供食材用量的标准化处理：

```python
from amount_normalizer import AmountNormalizer

normalizer = AmountNormalizer()

# 标准化用量
normalized, estimated = normalizer.normalize_amount("适量", "毫升")
# 返回: ("适量", 10.0)

# 获取可比较的数值
comparable = normalizer.get_comparable_value("一把", "")
# 返回: 50.0

# 格式化显示
display = normalizer.format_for_display("2-3个", "")
# 返回: "2-3个"
```

### �🔧 高级用法

#### 1. 自定义分类映射

```python
# 在recipe_ai_agent.py中修改
category_mapping = {
    "素菜": "710000000",
    "荤菜": "720000000", 
    "自定义分类": "999000000"
}
```

#### 2. 扩展AI提示词

```python
# 修改extract_recipe_info方法中的prompt
prompt = f"""
请分析菜谱并按以下格式提取信息：
- 添加您的自定义要求
- 特殊的分类规则
- 额外的营养信息要求
"""
```

#### 3. 批量处理优化

```python
# 调整处理参数
builder = RecipeKnowledgeGraphBuilder(ai_agent)
builder.batch_size = 20  # 增加批次大小
builder.delay = 0.5      # 减少请求间隔
```

### 📈 性能优化

#### API调用优化

1. **合理的请求频率**: 默认每秒1次请求，避免API限制
2. **错误重试机制**: 自动重试失败的请求
3. **批量处理**: 分批处理大量菜谱文件

#### 内存优化

1. **流式处理**: 逐个处理菜谱文件，避免内存溢出
2. **定期清理**: 处理完成后及时释放内存
3. **进度监控**: 实时显示处理进度

### 🔍 故障排除

#### 常见问题

1. **API密钥错误**

   ```
   错误: API调用失败: 401
   解决: 检查API密钥是否正确设置
   ```

2. **网络连接问题**

   ```
   错误: API调用超时
   解决: 检查网络连接，或增加timeout设置
   ```

3. **JSON解析错误**

   ```
   错误: JSON解析错误
   解决: AI响应格式异常，会自动使用备用解析方法
   ```

4. **菜谱格式问题**

   ```
   错误: 无法提取菜谱信息
   解决: 检查Markdown格式是否符合要求
   ```

#### 调试模式

```bash
# 启用详细日志
export DEBUG=true
python run_ai_agent.py /path/to/recipes

# 测试单个菜谱
python run_ai_agent.py test
```

### 📋 使用场景

#### 1. 菜谱网站构建

- 自动分类和标签
- 智能推荐系统
- 营养分析

#### 2. 烹饪应用开发

- 食材识别
- 步骤指导
- 工具推荐

#### 3. 营养研究

- 食材营养分析
- 膳食搭配研究
- 健康饮食推荐

#### 4. 餐饮业务

- 菜单优化
- 成本分析
- 客户偏好分析

### 🌐 扩展开发

#### 添加新的AI模型

```python
class CustomAIAgent(DeepSeekRecipeAgent):
    def __init__(self, api_key, model_name="custom-model"):
        super().__init__(api_key)
        self.model_name = model_name
    
    def call_custom_api(self, messages):
        # 实现您的自定义AI调用逻辑
        pass
```

#### 支持新的输出格式

```python
def export_to_custom_format(self, output_dir):
    """导出为自定义格式"""
    # 实现您的导出逻辑
    pass
```

### 📊 数据质量保证

#### AI解析准确性

- **多轮验证**: AI提取后进行格式验证
- **备用解析**: AI失败时使用规则解析
- **人工审核**: 提供数据审核接口

#### 数据一致性

- **标准化分类**: 统一的食材和菜谱分类
- **关系验证**: 确保图谱关系的逻辑一致性
- **重复检测**: 自动识别和处理重复数据

