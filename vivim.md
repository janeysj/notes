## 查找
* vi或者vim打开文本，直接输入/和要查找的字段按回车。Ctrl+n是查找下一个，ctrl+N是查找上一个
* 打开文件跳到第一个匹配的行
  vim +/search-term filename.txt
* 如果你输入 "/the"，你也可能找到 "there"。要找到以 "the" 结尾的单词，可以用：/the\> "\>" 是一个特殊的记号，表示只匹配单词末尾。类似地，"\<" 只匹配单词的开头。这样，要匹配一个完整的单词 "the"，只需：/\<the\>  
* :set hlsearch //高亮显示查找结果
* 当你看到一个单词，并想找它的next position,光标移动到该单词，并按下 shift+ *.
* 查看过当前文件目录：Step1： ESC模式下输入1  Step2: ctrl+G
* 显示行号
:set number
:set nonumber 不显示行号
在centos中想要永久显示行号打开/etc/virc文件，在文件最下面添加 :set number，关闭保存即可。

* 文本颜色
用命令:alias vi=vim 就OK了。 以后vi就带颜色了，这里相当于把vi变成vim命令了，但其实两者是有区别的

* 光标移动到指定位置
  shift+g 到最后一行
  ctrl+u或者gg到第一行
  ctrl+p  到前一行
  ctrl+n  到下一行
  ctrl+n+g查看当前行号、列号、在文中位置的百分比
  
## 编辑  
## 撤销与重做
* 撤销： u
* 重做： ctrl+r

### 插入：i后者shift+a

### 多页操作
:tabe file.name 再打开一个vi编辑页
gt在所有标签页之间切换
:tabc 关闭当前标签页，也可以用q退出

### 文本替换
:%s/aaa/bbb/g
全局范围内查找aaa并替换为bbb

### 鼠标模式
:set mouse=x, x取值如下, 例如:set mouse=a, 开启所有模式的mouse支持
> n 普通模式
> v 可视模式
> i 插入模式
> c 命令行模式
> h 在帮助文件里，以上所有的模式
> a 以上所有的模式
> r 跳过 |hit-enter| 提示
> A 在可视模式下自动选择

:set mouse=, =后面不要跟任何值, 可以关闭鼠标模式

### 行操作  
* 选择多行
  按v进入visual模式， 然后4j，表示选取光标位置处及以下4行的代码
  按v进入visual模式， ctrl+m 选择下一行
* 行复制
单行复制
在命令模式下，将光标移动到将要复制的行处，按“yy”进行复制；
多行复制
在命令模式下，将光标移动到将要复制的首行处，按“nyy”复制n行；其中n为1、2、3……
* 行粘贴
在命令模式下，将光标移动到将要粘贴的行处，按“p”进行粘贴

### 列操作，vim列操作
* vim中块删除：
第一步：按下组合键“CTRL+v” 进入“可视 块”模式，用上下左右选取这一列操作多少行
第二步：按下d 即可删除被选中的整块

* vim中块复制与粘贴
第一步：在当前vim中敲:tabe mac_table.txt打开文件并选取块，按下y键复制
第二步：退出该tabe, 按下p键粘贴该块

* vim中列操作插入字符
第一步：按下组合键“CTRL+v” 进入“可视 块”模式，用上下左右选取这一列操作多少行
第二步：按下shift+i，进入列编辑模式，一般会在第一列的第一行显示编辑内容
第三步：按两下esc键，退出列编辑。可以看到所有选中的列都有了编辑内容。


## 函数引用与跳转
https://blog.csdn.net/xueli1991/article/details/51598576
      使用cscope查找函数引用

