# winform笔记

## 通过一个窗口打开另一个窗口

```c#
    // 新建另一个窗口    
    Form2 f2 = new Form2(this);
    //普通用户界面            
    f2.Show();
    this.Hide();
```

## 设置文件对话框

**OpenFileDialog**类用来处理此类问题，下面为示例代码：

```c#
 OpenFileDialog dlg = new OpenFileDialog();
            dlg.Filter = "STL (*.stl)|*.stl|IGES (*.igs;*.iges)|*.igs;*.iges|STEP (*.stp;*.step)|*.stp;*.step|BREP (*.brep)|*.brep|All Files(*.*)|*.*";
```

详细请看帖子：[csdn](https://blog.csdn.net/zwj_jyzl/article/details/80725056)

## 使用any cad
