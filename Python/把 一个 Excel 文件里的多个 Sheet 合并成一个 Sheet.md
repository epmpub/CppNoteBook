把 **一个 Excel 文件里的多个 Sheet 合并成一个 Sheet**。


------

## 一、需求假设（可调整）

- 输入：一个 `.xlsx` 文件
- 行结构相同（列名一致）
- 合并规则：**纵向合并（append）**
- 可选：保留来源 Sheet 名

------

## 二、安装依赖

```bash
pip install pandas openpyxl
```

------

## 三、最常用版本（推荐）✅

### 功能

- 读取所有 Sheet
- 合并成一个 DataFrame
- 新增一列 `__sheet_name__` 标记来源
- 输出为一个新 Sheet

### 脚本

```python
import pandas as pd

INPUT_FILE = "input.xlsx"
OUTPUT_FILE = "merged.xlsx"
OUTPUT_SHEET = "Merged"

def merge_excel_sheets(input_file: str, output_file: str):
    # 读取所有 sheet
    sheets = pd.read_excel(input_file, sheet_name=None)

    merged = []
    for sheet_name, df in sheets.items():
        df["__sheet_name__"] = sheet_name  # 标记来源
        merged.append(df)

    merged_df = pd.concat(merged, ignore_index=True)

    # 写回 Excel
    with pd.ExcelWriter(output_file, engine="openpyxl") as writer:
        merged_df.to_excel(writer, sheet_name=OUTPUT_SHEET, index=False)

    print(f"✔ 合并完成，共 {len(merged_df)} 行")

if __name__ == "__main__":
    merge_excel_sheets(INPUT_FILE, OUTPUT_FILE)
```

------

## 四、如果你**不想加 Sheet 名列**

把这行删掉即可：

```python
df["__sheet_name__"] = sheet_name
```

------

## 五、只合并指定的 Sheet

```python
TARGET_SHEETS = {"Sheet1", "Sheet3"}

sheets = pd.read_excel(input_file, sheet_name=None)

merged = [
    df for name, df in sheets.items()
    if name in TARGET_SHEETS
]
```

------

## 六、Sheet 列不完全一致时（自动对齐）

`pandas.concat` 默认会：

- 自动补 NaN
- 以列名为准

```python
merged_df = pd.concat(merged, ignore_index=True, sort=False)
```

------

## 七、常见坑提醒 ⚠️

| 问题         | 解决                        |
| ------------ | --------------------------- |
| Excel 很大   | 加 `dtype=str`              |
| 中文乱码     | 不会（xlsx 不存在编码问题） |
| Sheet 有空行 | `df.dropna(how="all")`      |
| Excel 被占用 | 关闭 Excel 再跑             |

------

## 八、如果你需要

我可以再给你：

- **CLI 命令行版本**（`python merge.py in.xlsx out.xlsx`）
- **按列去重**
- **横向合并（join / merge）**
- **超大 Excel（分块读取）**
- **纯 openpyxl（不依赖 pandas）**

直接说你 Excel 的结构即可。