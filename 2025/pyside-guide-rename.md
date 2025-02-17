---
title: "PySide6 项目开发全攻略：打造你的文件重命名神器"
date: 2025-02-08 13:18:42
draft: false
categories: ["日志"]
---


家人们，今天来给大家唠唠如何用 PySide6 打造一个超实用的文件重命名工具。这篇文章适合想搞点 GUI 开发的 Python 小白，也能帮有经验的大佬查漏补缺。话不多说，咱们开整！

### 一、开发环境搭建：魔法工具大集合

#### 1.1 开发工具

开发前，咱们得先把 “魔法工具” 准备好：

* **PyCharm 2023.1**：这可是 Python 开发的神器，智能代码补全就像你的专属小秘书，敲代码的时候自动提示，效率飞起！还集成了 Qt Designer，可视化界面设计，拖拖拽拽就搞定，简直不要太爽！
* **Python 3.10**：建议大家用虚拟环境，venv 或者 conda 都行。就好比给你的项目穿上一层 “隔离衣”，每个项目都有自己独立的 Python 环境，互不干扰，再也不用担心包冲突的问题啦！
* **PySide6 6.5.0**：Qt 官方钦点的 Python 绑定库，有了它，就能轻松调用 Qt 的各种强大功能，搭建出酷炫的 GUI 界面。

#### 1.2 项目结构：文件的秘密基地

项目结构就像一个有序的小基地，每个文件都有自己的 “小窝”：

```bash
FsPySide6Project/
├──.gitignore        # 版本控制的“小卫士”，忽略那些不需要的文件
├── batch_file_renamer.py  # 文件重命名的“大脑”，核心功能都在这儿
├── main.py           # 程序入口，就像房子的大门，从这儿开始你的旅程
├── main_window.py    # 主窗口界面，是你的“门面担当”
└── requirements.txt  # 依赖清单，记录着项目需要的各种“小帮手”
```

来看看这些文件都在干啥：

*  **.gitignore**：默默守护着项目，把__pycache__/、.idea/ 这些开发环境文件拒之门外，让你的代码仓库干干净净。
* **requirements.txt**：里面写着PySide6>=6.5.0，这是项目的 “粮草清单”，告诉别人运行这个项目需要哪些依赖。
* **main.py**：程序的启动入口，初始化 QApplication，就像给汽车点火，让整个程序跑起来。
* **main_window.py**：主界面的 “大管家”，采用模块化设计，以后想加新功能，就像搭积木一样简单。
* **batch_file_renamer.py**：文件重命名业务逻辑的 “神秘组织”，各种复杂的重命名操作都由它来搞定。

### 二、核心代码解析：揭开魔法的神秘面纱

#### 2.1 程序入口（main.py）

```python
import sys
from main_window import MainWindow
from PySide6.QtWidgets import QApplication

def main():
    # 创建Qt应用上下文，就像给程序找了个“大管家”
    app = QApplication(sys.argv)

    # 初始化主窗口，给你的程序打造一个漂亮的“门面”
    window = MainWindow()
    window.setMinimumSize(300, 250)  # 设置最小窗口尺寸，保证界面不会“挤成一团”
    window.show()

    # 进入事件循环，让程序开始“干活”
    sys.exit(app.exec())

if __name__ == '__main__':
    main()
```

这里有几个关键点：

* QApplication：管理 GUI 应用的控制流和主设置，就像一个 “幕后大 boss”，掌控着整个程序的运行。
* sys.exit(app.exec())：程序退出时，它会帮你正确释放资源，就像走的时候把房间打扫干净一样。
* 设置最小窗口尺寸：保证 UI 显示完整，不然界面太小，用户操作起来可就麻烦啦！

#### 2.2 主窗口实现（main_window.py）

```python
from PySide6.QtWidgets import QWidget, QVBoxLayout, QPushButton
from PySide6.QtGui import QFont
from batch_file_renamer import RenameFileApp

class MainWindow(QWidget):
    def __init__(self):
        super().__init__()
        self.init_ui()

    def init_ui(self):
        self.setWindowTitle("PyQt示例应用")
        self.setGeometry(100, 100, 300, 250)
        self.setStyleSheet("background-color: #F5F5F5;")  # 设置窗口背景色为淡灰色，看起来很舒服

        layout = QVBoxLayout()

        # 重命名使者，点我就可以开始重命名啦
        rename_file_btn = QPushButton("重命名使者")
        rename_file_btn.setFont(QFont('Arial', 14))
        rename_file_btn.setStyleSheet("""
            QPushButton {
                background-color: #9C27B0;
                color: white;
                border-radius: 8px;
                padding: 10px;
            }
            QPushButton:hover {
                background-color: #8E24AA;
            }
        """)
        rename_file_btn.clicked.connect(self.rename_file_btn_clicked)
        layout.addWidget(rename_file_btn)

        self.setLayout(layout)

    def rename_file_btn_clicked(self):
        print("按钮<重命名使者>被点击了")
        self.rename_file = RenameFileApp()
        self.rename_file.show()
```

主窗口里加了个 “重命名使者” 按钮，一点击就打开子窗口。要是以后想加新功能，就再添加个按钮，绑定个点击事件，简单得很！

效果图如下：

​![](https://cdn.jsdelivr.net/gh/ueYao/image-hosting@main/blog/2025/02/477a67d96b864ee0b1e9c2c50fd3a541.png)​

#### 2.3 重命名工具实现（batch_file_renamer.py）

```python
import os
from PySide6.QtWidgets import QWidget, QVBoxLayout, QHBoxLayout, QLabel, QLineEdit, QPushButton, QFileDialog
from PySide6.QtGui import QFont
from PySide6.QtWidgets import QMessageBox

class RenameFileApp(QWidget):
    def __init__(self):
        super().__init__()
        self.init_ui()

    def init_ui(self):
        self.setWindowTitle("文件名批量修改工具")

        layout = QVBoxLayout()

        # 选择文件夹相关部件
        folder_path_layout = QHBoxLayout()
        self.folder_path_label = QLabel("选择文件夹：")
        self.folder_path_label.setStyleSheet("color: #333; font-size: 14px;")
        self.folder_path_entry = QLineEdit()
        self.folder_path_entry.setFixedWidth(300)
        self.folder_path_entry.setStyleSheet("""
            QLineEdit {
                border: 1px solid #ccc;
                border-radius: 3px;
                padding: 2px 5px;
            }
        """)
        self.browse_button = QPushButton("浏览")
        self.browse_button.setStyleSheet("""
            QPushButton {
                background-color: #4CAF50;
                color: white;
                border-radius: 3px;
                padding: 5px 10px;
            }
            QPushButton:hover {
                background-color: #45a049;
            }
        """)
        self.browse_button.clicked.connect(self.browse_folder)
        folder_path_layout.addWidget(self.folder_path_label)
        folder_path_layout.addWidget(self.folder_path_entry)
        folder_path_layout.addWidget(self.browse_button)
        layout.addLayout(folder_path_layout)

        # 文件名前缀输入部件
        prefix_layout = QHBoxLayout()
        self.prefix_label = QLabel("文件名前缀：")
        self.prefix_label.setStyleSheet("color: #333; font-size: 14px;")
        self.prefix_entry = QLineEdit()
        self.prefix_entry.setStyleSheet("""
            QLineEdit {
                border: 1px solid #ccc;
                border-radius: 3px;
                padding: 2px 5px;
            }
        """)
        prefix_layout.addWidget(self.prefix_label)
        prefix_layout.addWidget(self.prefix_entry)
        layout.addLayout(prefix_layout)

        # 文件名后缀输入部件
        suffix_layout = QHBoxLayout()
        self.suffix_label = QLabel("文件名后缀：")
        self.suffix_label.setStyleSheet("color: #333; font-size: 14px;")
        self.suffix_entry = QLineEdit()
        self.suffix_entry.setStyleSheet("""
            QLineEdit {
                border: 1px solid #ccc;
                border-radius: 3px;
                padding: 2px 5px;
            }
        """)
        suffix_layout.addWidget(self.suffix_label)
        suffix_layout.addWidget(self.suffix_entry)
        layout.addLayout(suffix_layout)

        # 查找字符输入部件
        char_to_find_layout = QHBoxLayout()
        self.char_to_find_label = QLabel("查找字符：")
        self.char_to_find_label.setStyleSheet("color: #333; font-size: 14px;")
        self.char_to_find_entry = QLineEdit()
        self.char_to_find_entry.setStyleSheet("""
            QLineEdit {
                border: 1px solid #ccc;
                border-radius: 3px;
                padding: 2px 5px;
            }
        """)
        char_to_find_layout.addWidget(self.char_to_find_label)
        char_to_find_layout.addWidget(self.char_to_find_entry)
        layout.addLayout(char_to_find_layout)

        # 替换字符输入部件
        replace_char_layout = QHBoxLayout()
        self.replace_char_label = QLabel("替换字符：")
        self.replace_char_label.setStyleSheet("color: #333; font-size: 14px;")
        self.replace_char_entry = QLineEdit()
        self.replace_char_entry.setStyleSheet("""
            QLineEdit {
                border: 1px solid #ccc;
                border-radius: 3px;
                padding: 2px 5px;
            }
        """)
        replace_char_layout.addWidget(self.replace_char_label)
        replace_char_layout.addWidget(self.replace_char_entry)
        layout.addLayout(replace_char_layout)

        # 作者和Github信息文本
        author_font = QFont("楷体", 11)
        self.author_label = QLabel("Author：xueyao.me@gmail.com")
        self.author_label.setFont(author_font)
        self.author_label.setStyleSheet("color: #007BFF;")
        self.github_label = QLabel("Github：https://github.com/flowstone/FsPySide6Project")
        self.github_label.setFont(author_font)
        self.github_label.setStyleSheet("color: #007BFF;")

        info_layout = QVBoxLayout()
        info_layout.addWidget(self.author_label)
        info_layout.addWidget(self.github_label)
        layout.addLayout(info_layout)

        # 操作按钮
        button_layout = QHBoxLayout()
        self.start_button = QPushButton("开始修改")
        self.start_button.setStyleSheet("""
            QPushButton {
                background-color: #008CBA;
                color: white;
                border-radius: 3px;
                padding: 5px 15px;
            }
            QPushButton:hover {
                background-color: #007B9A;
            }
        """)
        self.start_button.clicked.connect(self.start_operation)
        self.exit_button = QPushButton("退出")
        self.exit_button.setStyleSheet("""
            QPushButton {
                background-color: #F44336;
                color: white;
                border-radius: 3px;
                padding: 5px 15px;
            }
            QPushButton:hover {
                background-color: #D32F2F;
            }
        """)
        self.exit_button.clicked.connect(self.close)

        button_layout.addWidget(self.start_button)
        button_layout.addWidget(self.exit_button)
        layout.addLayout(button_layout)

        self.setLayout(layout)

    def browse_folder(self):
        folder_path = QFileDialog.getExistingDirectory(self, "选择文件夹")
        self.folder_path_entry.setText(folder_path)

    def start_operation(self):
        folder_path = self.folder_path_entry.text()
        prefix = self.prefix_entry.text()
        suffix = self.suffix_entry.text()
        char_to_find = self.char_to_find_entry.text()
        replace_char = self.replace_char_entry.text()
        if folder_path:
            self.rename_files(folder_path, prefix, suffix, char_to_find, replace_char)
            QMessageBox.information(self, "提示", "批量文件名修改完成！")
        else:
            QMessageBox.warning(self, "警告", "请选择要修改的文件夹！")

    # 修改文件名
    def rename_files(self, folder_path, prefix, suffix, char_to_find, replace_char):
        # 遍历文件夹下的文件名
        for filename in os.listdir(folder_path):
            old_path = os.path.join(folder_path, filename)
            # 判断是否是文件
            if os.path.isfile(old_path):
                new_filename = f"{prefix}{filename}{suffix}"
                # 判断是否需要进行文件替换操作
                if char_to_find and replace_char:
                    # 替换字符
                    new_filename = new_filename.replace(char_to_find, replace_char)
                new_path = os.path.join(folder_path, new_filename)
                os.rename(old_path, new_path)
```

这个文件就是重命名的 “魔法工厂”，各种参数设置、文件遍历、重命名操作都在这里完成。选择文件夹、设置前缀后缀、查找替换字符，最后点击 “开始修改”，文件名就乖乖听话啦！

效果图如下：

​![](https://cdn.jsdelivr.net/gh/ueYao/image-hosting@main/blog/2025/02/99795fce21a64c9293c1af1c921df264.png)​

### 三、运行与部署：让你的程序跑起来

1. 安装依赖：

```
pip install -r requirements.txt
```

这一步就像给汽车加满油，把项目需要的依赖都安装好，程序才能顺利运行。安装完依赖，就可以运行main.py，启动你的文件重命名神器啦！

怎么样，家人们，是不是感觉用 PySide6 开发 GUI 应用也不难呀？赶紧动手试试，打造属于自己的文件重命名工具吧！要是在开发过程中有什么问题，欢迎在评论区留言，咱们一起交流！

要是你在开发中遇到了啥问题，或者对代码还有其他想法，都可以跟我说说，咱们一起把这个文件重命名神器变得更完美！

附：代码<[Github](https://github.com/flowstone/FsPySide6Project/tree/v0.1.0)>

‍
