# TODO系统-任务筛选功能
作者：SUSAN  
时间：2020-06-07

## 功能简介
- 操作界面分为三部分：左侧任务分类列表，中间任务明细，右侧任务筛选功能。点击左侧、右侧不同的选项会对中的任务明细产生影响。
- 在选择“我的一天”“全部任务”“xx清单任务”等选项（左侧）后，任务明细刷新（中间），如用户设置筛选条件（右侧），筛选结果仅在已选中范围内进行筛选。
- 筛选条件包括
    - 重要性
    - 完成状态（题外话：默认是否显示已完成任务）
    - 创建日期/截止日期：可能需要引入插件方便用户选择时间
    - 任务所属清单：仅在“全部任务”列表中出现这一选项

## 编程思路
1. （变量）获取任务列表信息：
    - 我的一天：0
    - 全部任务：NULL
    - xx清单任务：分别对应不同数字
2. （HTML表单）获取筛选条件信息:
    - 重要性：0为不重要；1为普通；2为重要
    - 完成状态：0为进行中；1为已完成；2为已逾期
    - 创建日期/截止日期：yyyymmddhhmmss（暂定，具体看插件和新增任务的时间格式）
3. 将以上两类信息POST传送至fliter.php，在此条件限制下进行数据库查询，并返回结果。

## 具体实现
- HTML form 表单
```script:HTML
name="list"
name="importance"
name="status"
name="create_date"
name="deadline"
```
- PHP 数据库查询
```script:php
$query = "SELECT * FROM "todo 信息表" WHERE 
    "类别" = $_POST['list'] and
    "重要性" = $_POST['importance'] and
    "状态" = $_POST['status'] and
    "创建日期" = $_POST['create_date'] and
    "截止日期" = $_POST['deadline']";

// $result= mysql_query($query) or die('Query failed: '.mysql_error());;
$result= mysqli_query($con,$query) or die('Query failed: '.mysql_error());;

print_r($result);
```