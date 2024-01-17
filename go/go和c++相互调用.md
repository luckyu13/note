# c++调用GO程序
1. 编写go语言程序，需要`import "C"`
2. c++能够调用的只有全局函数，不能调用struct的成员函数
3. 在需要被调用的函数上增加`//export FuncName`,其中`//`和`export`中间不能有空格，`FuncName`是需要向外扩展的函数名
4. 现在不支持辅助结构的数据，比如struct，只支持