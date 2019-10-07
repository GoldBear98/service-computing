# service-computing
1.概述
CLI（Command Line Interface）实用程序是Linux下应用开发的基础。正确的编写命令行程序让应用与操作系统融为一体，通过shell或script使得应用获得最大的灵活性与开发效率。Linux提供了cat、ls、copy等命令与操作系统交互；go语言提供一组实用程序完成从编码、编译、库管理、产品发布全过程支持；容器服务如docker、k8s提供了大量实用程序支撑云服务的开发、部署、监控、访问等管理任务；git、npm等都是大家比较熟悉的工具。尽管操作系统与应用系统服务可视化、图形化，但在开发领域，CLI在编程、调试、运维、管理中提供了图形化程序不可替代的灵活性与效率。

2.实验要求
参照与Selpg相关的命令行程序设计 ，实现一个selpg页选择程序，满足selpg设计要求。

3.流程分析
程序按照命令输入、参数规范判断、读取文件、确认输出位置并输出得顺序执行。当发现错误时抛出错误并终止流程。

4.代码实现
（1）需要记住selpg所需要的参数有必须的开始页码-s以及结束页码-e，可选的输入文件名、自定页长-l、遇换页符换页-f和输出地址。其中自定页长和遇换页符换页两个选项不能同时使用。
*设置程序的参数结构体
*输入参数使用 github.com/spf13/pflag 包提供的pflag进行处理，pflag包于flag用法类似，但pflag相对于flag能够更好地满足 Unix 命令行规范。

（2）命令行参数获取之后，首先要进行参数检查以尽量避免参数谬误。出现错误时输出问题并正常结束程序。参数正确则将各个参数值输出到屏幕上。

首先检查开始页和结束页是否被赋值，然后检查开始页和结束页是否为正数，接下来检查开始页是否大于结束页，然后检查自定页长-l和遇换页符换页-f是否同时出现，最后判断当自定页长-l出现时args.pageLen是否小于1。遇到不合规的地方正常结束程序，全部合规则输出得到的参数。

（3）程序在检查参数后开始调用excuteCMD函数执行命令。
先检查输入：如果没有给定文件名，则从标准输入中获取；如果给出读取的文件名，就调用函数checkFileAccess检查文件是否存在。然后打开文件，使用函数checkError检查是否出现错误：如果打开出错就输出错误并抛出。最后判断是否有-d参数：如果没有那么选择的页是直接从os.Stdout标准输出中输出的；如果-d存在，就从指定的打印通道中输出。

（4）在-d参数存在时，涉及到了os/exec包的使用。

printDest是可以获取到的打印地址，将命令行的输入管道cmd.StdinPipe()获取的指针赋值给fout，再将fout返回给output2Des函数中的作为输出位置参数的输入，最后在output2Des中将需要的页输出到fout。

（5）输出函数output2Des将输入的文件，按页码要求读取并输出到fout中。


由于fout存在两种输入-io.Stdout标准输出作为输入、cmd.StdinPipe()管道作为输入。所以使用空接口interface{}作为fout的类型，借助类型断言stdOutput, ok := fout.(*os.File)和pipeOutput, ok := fout.(io.WriteCloser)来判断fout具体类型并调用相应函数。
5.程序测试
测试文件如下：

go run selpg.go -s1 -e1 input_file.txt
go run selpg.go -s1 -e1 < input_file.txt

go run selpg.go -s1 -e2 input_file.txt > output_file.txt


go run selpg.go -s1 -e4 input_file.txt 2>error_file.txt


go run selpg.go -s1 -e3 input_file.txt >output_file.txt 2>error_file.txt


go run selpg.go -s1 -e2 -f input_file.txt


go run selpg.go -s1 -e1 -dCups-PDF selpg.go

