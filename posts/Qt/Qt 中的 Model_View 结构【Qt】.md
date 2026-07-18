---
id: qt-model-view
title: "Qt 中的 Model/View 结构"
date: 2026-07-18
tags: Qt,Model/View
excerpt: "Model/View 架构将数据模型与用户界面分离，实现高效的数据显示与编辑，适用于大型数据集处理。"
cover: linear-gradient(135deg, #43e97b 0%, #38f9d7 100%)
---

[https://muzing.top/posts/5ff61cbd/](https://muzing.top/posts/5ff61cbd/)

## 简介
<font style="color:rgb(44, 62, 80);">源数据由模型 (Model) 读取，然后在视图 (View) 组件上显示和编辑，在界面上编辑修改的数据又通过模型保存到源数据。</font>

<font style="color:rgb(44, 62, 80);">Model/View 结构将数据模型和用户界面分离开来，分别用不同的实现，是一种显示和编辑数据的有效结构，在处理大型数据时尤其明显。</font>

<!-- 这是一张图片，ocr 内容为：DATA 数据 MODEL 模型 EDITING编辑 提供 RENDERING 代理 DELEGATE VIEW 视图 RENDERING 提供 -->
![](https://cdn.nlark.com/yuque/0/2023/png/1443385/1678106378602-5762a4d6-b68c-4858-8b63-ca227f51983b.png)

Model/View基本结构

+ <font style="color:rgb(44, 62, 80);">Data（源数据）是原始数据，如数据库的一个数据表或SQL查询结果、内存中的一个字符串列表或磁盘文件结构等</font>
+ <font style="color:rgb(44, 62, 80);">Model（模型/数据模型）与源数据通信，并为视图组件提供数据接口。它从源数据提取需要的数据，用于视图组件进行显示和编辑</font>
+ <font style="color:rgb(44, 62, 80);">View（视图/视图组件）是界面控件，视图从数据模型中根据一定条件（如行号、列号等）获得模型索引（一个指向数据项的引用），然后显示在界面上</font>
+ <font style="color:rgb(44, 62, 80);">Delegate（代理）在视图与模型之间交互操作时提供临时编辑组件的功能</font>

## Delegate 代理
<font style="color:rgb(44, 62, 80);">代理在视图与模型之间交互操作时提供临时编辑组件的功能。模型向视图提供数据是单向的，一般仅用于显示。当需要在视图上编辑数据时，代理功能会为编辑数据提供一个编辑器，这个编辑器获取模型的数据、接受用户编辑的数据后又提交给模型。</font>

<font style="color:rgb(44, 62, 80);">例如在QTableView组件上双击一个单元格编辑数据时，在单元格里就会出现一个QLineEdit组件，这个编辑框就是代理提供的临时编辑器。代理的主要任务就是为视图组件提供代理编辑器。</font>

<font style="color:rgb(44, 62, 80);">对于一些特殊的数据编辑需求，例如只允许输入整型数，使用一个QSpinBox作为代理组件更合适；从列表中选择一个数据，使用一个QComboBox作为代理组件更好。这时就需要从QStyledItemDelegate继承创建自定义代理类。</font>

### 自定义代理
<!-- 这是一张图片，ocr 内容为：QABSTRACTLEEMDELEGATE QLTEMDELEGATE QSTYLEDLTEMDELEGATE QSQLRELATIONALDELEGATE -->
![](https://cdn.nlark.com/yuque/0/2023/png/1443385/1678106279786-85624060-4ee6-4254-8132-c0b71986eae8.png)

Qt_Delegate 代理类的继承关系

<font style="color:rgb(44, 62, 80);">不管从QStyledItemDelegate还是QItemDelegate继承设计自定义代理，都必须实现下面的4个方法</font>

+ <font style="color:rgb(44, 62, 80);">createEditor() 创建用于编辑模型数据的widget组件，如一个QSpinBox或一个QComboBox组件</font>
+ <font style="color:rgb(44, 62, 80);">setEditorData() 从模型获得数据，供widget组件进行编辑</font>
+ <font style="color:rgb(44, 62, 80);">setModelData() 将widget上的数据更新到数据模型</font>
+ <font style="color:rgb(44, 62, 80);">updateEditorGeometry() 用于给widget组件设置合适的大小</font>

## Qt model 类
<!-- 这是一张图片，ocr 内容为：QSTRINGLISTMODEL QABSTRACTLISTMODEL QABSTRACTLTEMMODEL QABSTRACTPROXYMODEL QSORTFILTERPROXYMODEL QSQLRELATIONAL TABLE QSGLTABLEMODEL QSQLQUERYMODEL QABSTRACTTABLEMODEL MODEL QSTANDARDLTEMMODEL QFILESYSTEMMODEL -->
![](https://cdn.nlark.com/yuque/0/2023/png/1443385/1678106343356-3875fafa-b20d-43ae-8953-d6a7eda708fa.png)

Model类的继承关系

### QFileSystemModel
<font style="color:rgb(44, 62, 80);">为本机的文件系统提供一个数据模型，可用于访问本机的文件系统。</font>

<font style="color:rgb(44, 62, 80);">比如和QTreeView视图组件结合使用，可以用目录树的形式显示本机上的文件系统，类似Windows的文件资源管理器。</font>

<font style="color:rgb(44, 62, 80);">使用QFileSystemModel提供的接口函数，可以创建目录、删除目录、重命名目录，可以获得文件名称、目录名称、文件大小等参数，可以获得文件的详细信息</font>

### QStringListModel
<font style="color:rgb(44, 62, 80);">用于处理字符串列表的数据模型，可以作为QListView的数据模型，在界面上显示和编辑字符串列表。</font>

### QStandardItemModel
<font style="color:rgb(44, 62, 80);">以项数据（item data）为基础的标准数据结构模型类，通常与QTableView配合使用，实现通用的二维数据的管理</font>

## Qt view 类
<!-- 这是一张图片，ocr 内容为：QLISTVIEW QLISTWIDGET QTABLEWIDGET QTABLE VIEW QABSTRACTLTEM VIEW QTREE VIEW QTREEWIDGET QCOLUMN VIEW QHEADERVIEW -->
![](https://cdn.nlark.com/yuque/0/2023/png/1443385/1678106343360-459ba644-4f6f-4b02-8728-fedccfce1d62.png)

View类的继承关系

<font style="color:rgb(44, 62, 80);"> </font>[QAbstractItemView](https://github.com/muziing/PyQt_practice/tree/master/46-QAbstractItemView)<font style="color:rgb(44, 62, 80);">、</font>[QListView](https://github.com/muziing/PyQt_practice/tree/master/47-QListView)<font style="color:rgb(44, 62, 80);">、</font>[QTableView](https://github.com/muziing/PyQt_practice/tree/master/48-QTableView)<font style="color:rgb(44, 62, 80);">、</font>[QTreeView](https://github.com/muziing/PyQt_practice/tree/master/49-QTreeView)

+ <font style="color:rgb(44, 62, 80);">QListView：用于显示单列的列表数据，适用于一维数据的操作</font>
+ <font style="color:rgb(44, 62, 80);">QTreeView：用于显示树状结构数据，适用于树状结构数据的操作</font>
+ <font style="color:rgb(44, 62, 80);">QTableView：用于显示表格状数据，适用于二维表格型数据的操作</font>
+ <font style="color:rgb(44, 62, 80);">QColumnView：用多个QListView显示树状层次结构，树状结构的一层用一个QListView显示</font>
+ <font style="color:rgb(44, 62, 80);">QHeaderView：提供行表头或列表头的视图组件，如QTableView的行表头和列表头</font>



[QTableWidget 和 QTableView 的主要区别是 QTableView 可以使用自定义的数据模型来显示内容，而 QTableWidget 只能使用标准的数据模型，并且其单元格数据是 QTableWidgetItem 的对象来实现的](https://blog.csdn.net/qingshuiyangfan/article/details/81063758)[1](https://blog.csdn.net/qingshuiyangfan/article/details/81063758)[2](https://www.yisu.com/zixun/190785.html)[。QTableWidget 是 QTableView 的子类，所以 QTableWidget 也有 QTableView 的大部分功能](https://blog.csdn.net/qq_41042595/article/details/106539508)



[一般来说，QTableWidget 使用起来更简单，而 QTableView 的用法相对比较复杂](http://c.biancheng.net/view/9419.html)[1](http://c.biancheng.net/view/9419.html)[。QTableWidget 是基于 QTableWidgetItem 的，而 QTableView 是基于数据模型的](https://www.csdn.net/tags/MtTaEg1sNTkxMDQ2LWJsb2cO0O0O.html)[2](https://www.csdn.net/tags/MtTaEg1sNTkxMDQ2LWJsb2cO0O0O.html)[3](https://stackoverflow.com/questions/15290932/qtablewidget-vs-qtableview)[。QTableWidget 有 QTableView 的大部分功能，但 QTableView 可以和 QSqlTableModel 绑定，而 QTableWidget 不可以](https://blog.csdn.net/qingshuiyangfan/article/details/81063758)

