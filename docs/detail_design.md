# SimpleCFS 详细设计

## 文档历史记录
文档版本 | 描述	| 编写人员 | 更新时间
----    | ---   | ---    | --- 
0.1	| 创建基本框架 | 李剑	| 2014-12-04
0.2	| 完成接口设计 | 李剑	| 2014-12-05
0.3 | 完成流程设计 | 李剑 | 2014-12-07

## 1.概述
Client对应用层提供接口，实现简单的文件系统功能，和MDS交互得到文件和目录的元数据，和DS交互读写文件数据内容，使用编码库对文件数据进行编码和修复，MDS和DS交互获得状态和chunk状态信息。

## 2.重要数据结构

### 2.1 文件的chunk在DS节点分布
TODO

## 3.重要类设计

### 3.1 完整类图
TODO

### 3.2 用户API类
TODO

### 3.3 编码类

### 3.4 返回值类

| RetValue  | 说明 |
|:------ |:---- |
| state | 状态值 |
| message | 状态信息 |

## 4.接口设计

### 4.1 用户API接口
Client对用户提供的接口主要包含两类，分别是目录操作和文件操作。

#### 4.1.1 创建目录(mkdir)

接口名  | mkdir(dirname)
:------ | :------------ 
参数说明 | @**dirname**：要创建的目录的路径名
返回值  | 返回类**RetValue**,包含返回状态和信息   
说明 | 目录名可以是相对路径也可以是绝对路径

#### 4.1.2 删除目录(rmdir)

接口名  | rmdir(dirname)
:------ | :------------ 
参数说明 | @**dirname**：要删除的目录
返回值  | 返回类**RetValue**
说明 | 只在目录为空时成功 

#### 4.1.3 列举目录(listdir)

接口名  | listdir(dirname)
:------ | :------------ 
参数说明 | @**dirname**：要查询的目录
返回值  | 返回一个目录的列表
说明 | 不保证顺序，不包含`.`和`..`，目录以`/`结尾

#### 4.1.4 设置当前目录(chdir)

接口名  | chdir(path)
:------ | :------------ 
参数说明 | @**path**：要到达的目录
返回值  | 返回类**RetValue**
说明 | 空

#### 4.1.5 获得当前目录(getcwd)

接口名  | getcwd()
:------ | :------------ 
参数说明 | 空
返回值  | 返回当前目录的字符串
说明 | 使用类unix的`sep`,即`/`

#### 4.1.6 目录信息(statdir)

接口名  | statdir(dirname)
:------ | :------------ 
参数说明 | @**dirname**: 查询的目录
返回值  | 返回给定目录的信息字典
说明 | 空

#### 4.1.7 写文件(putfile)

接口名  | putfile(src_path, des_path, code_info)
:------ | :------------ 
参数说明 | @**src_path**:本地的文件路径,@**des_path**:存储到系统的文件路径（包括文件名），@**code_info**:编码信息，包括编码类型和编码参数（**TODO**）
返回值  | 返回类**RetValue**
说明 | 编码信息要和博士讨论下，一个字典，包括类型和参数k,m等

#### 4.1.8 读文件(getfile)

接口名  | getfile(des_path, local_path, repair_flag)
:------ | :------------ 
参数说明 | @**des_path**:系统中文件的路径,@**local_path**:本地文件路径（包括文件名）@**repair_flag**:为True时，如果有数据损坏对文件修复，为False时不处理
返回值  | 返回类**RetValue**
说明 | 读文件返回的状态比较多，有正常读，解码读和修复读。

#### 4.1.9 删除文件(delfile)

接口名  | delfile(path)
:------ | :------------ 
参数说明 | @**path**:要删除文件的路径
返回值  | 返回类**RetValue**
说明 | 空

#### 4.1.10 文件信息(statfile)

接口名  | statfile(path)
:------ | :------------ 
参数说明 | @**path**: 查询的文件路径
返回值  | 返回给定文件的信息字典
说明 | 空

### 4.2 Client与MDS交互接口
#### 4.2.1 创建目录(make_dir)

接口名  | make_dir(dirname)
:------ | :------------ 
参数说明 | @**dirname**：要创建的目录的路径名
返回值  | 返回类**RetValue**,包含返回状态和信息   
说明 | 目录名是绝对路径

#### 4.2.2 删除目录(remove_dir)

接口名  | remove_dir(dirname)
:------ | :------------ 
参数说明 | @**dirname**：要删除的目录
返回值  | 返回类**RetValue**
说明 | 只在目录为空时成功 

#### 4.2.3 列举目录(list_dir)

接口名  | list_dir(dirname)
:------ | :------------ 
参数说明 | @**dirname**：要查询的目录
返回值  | 返回一个目录的列表
说明 | 不保证顺序，不包含`.`和`..`,目录以`/`结尾

#### 4.2.4 目录信息(status_dir)

接口名  | status_dir(dirname)
:------ | :------------ 
参数说明 | @**dirname**: 查询的目录
返回值  | 返回给定目录的信息字典
说明 | 空

#### 4.2.5 目录是否存在(valid_dir)

接口名  | valid_dir(dirname)
:------ | :------------ 
参数说明 | @**dirname**: 查询的目录
返回值  | 存在返回True，否则返回False
说明 | 空

#### 4.2.6 写文件(add_file)

接口名  | add_file(des_path,file_size,code_info)
:------ | :------------ 
参数说明 | @**des_path**:存储到系统的文件路径（包括文件名）,@**file_size**，要保存的文件大小，@**code_info**:编码信息，包括编码类型和编码参数（TODO）
返回值  | 返回object,chunk到DS的映射列表
说明 | 返回的项目是嵌套的字典，object_size由MDS定，编码方式同client定（设置默认值）

#### 4.2.8 提交写文件完成(add_file_commit)

接口名  | add_file_commit(des_path)
:------ | :------------ 
参数说明 | @**des_path**:存储到系统的文件路径（包括文件名）
返回值  | 返回类**RetValue**
说明 | 提交后，把元数据写入到数据库中

#### 4.2.9 读文件(get_file)

接口名  | get_file(des_path)
:------ | :------------ 
参数说明 | @**des_path**:系统中文件的路径
返回值  | 返回object和chunk到DS的映射列表，编码信息(code_info)
说明 | 读文件返回的状态比较多，有正常读，解码读和失败。

#### 4.2.10 修复chunk(repair_chunk)

接口名  | repair_chunk(chunk_ID)
:------ | :------------ 
参数说明 | @**chunk_ID**:要修复的chunk的ID
返回值  | 如果要修复返回新的DS信息，不用时返回空的IP
说明 | 这个是在修复读时调用

#### 4.2.11 删除文件(delete_file)

接口名  | delete_file(path)
:------ | :------------ 
参数说明 | @**path**:要删除文件的路径
返回值  | 返回类**RetValue**
说明 | 空

#### 4.2.12 文件信息(status_file)

接口名  | status_file(path)
:------ | :------------ 
参数说明 | @**path**: 查询的文件路径
返回值  | 返回给定文件的信息字典
说明 | 空

### 4.3 Client与DS交互接口

#### 4.3.1 写入chunk(add_chunk)

接口名  | add_chunk(chunk_ID, chunk_length, chunk_data)
:------ | :------------ 
参数说明 | @**chunk_ID**: ID也是存取的文件名，@**chunk_length**：数据长度,@**chunk_data**：数据内容
返回值  | 返回类**RetValue**
说明 | 空

#### 4.3.2 读取chunk(get_chunk)

接口名  | get_chunk(chunk_ID, blocks)
:------ | :------------ 
参数说明 | @**chunk_ID**: ID也是存取的文件名，@**blocks**:要取的数据分片序号列表
返回值  | 返回整合后的数据内容，失败则返回相应的状态
说明 | 空

#### 4.3.3 删除chunk(delete_chunk)

接口名  | delete_chunk(chunk_ID)
:------ | :------------ 
参数说明 | @**chunk_ID**: ID也是存取的文件名，
返回值  | 返回类**RetValue**
说明 | 空

### 4.4 编码接口
**TODO**,考虑边编码边传，还是编码完成后分别传输。目前就用内存好了。

#### 4.4.1 编码object(encode_object)

接口名  | encode_object(object_data, code_info)
:------ | :------------ 
参数说明 | @**object_data**: 要编码的数据内容，@**code_info**:编码信息，包括方式和参数
返回值  | 返回编码后的chunk列表，
说明 | 编码方式的封装在code_info中体现

#### 4.4.2 解码object(decode_object)
接口名  | encode_object(chunk_list, code_info)
:------ | :------------ 
参数说明 | @**chunk_list**: 要解码的chunk列表，@**code_info**:编码信息，包括方式和参数
返回值  | 返回解码后的object内容
说明 | 编码方式的封装在code_info中体现,chunk_list内容有原来的编号顺序

#### 4.4.3 修复某个chunk(repair_chunk)

接口名  | repair_chunk(chunk_list, chunk_gen, code_info)
:------ | :------------ 
参数说明 | @**chunk_list**: 帮助修复的chunk列表，@**chunk_gen**:要修复的chunk编号，@**code_info**:编码信息，包括方式和参数
返回值  | 返回修复后的chunk内容
说明 | 编码方式的封装在code_info中体现,chunk_list内容有原来的编号顺序

### 4.4 MDS与DS交互接口

#### 4.4.1 DS注册(add_ds)

接口名  | add_ds()
:------ | :------------ 
参数说明 | 保存ip和端口，这个在连接中可以获得
返回值  | 返回类**RetValue**
说明 | 空

#### 4.4.2 查看DS状态(check_ds)

接口名  | check_ds()
:------ | :------------ 
参数说明 | ip和端口，这个在连接中可以获得
返回值  | 返回类**RetValue**
说明 | 空

#### 4.4.3 检查chunk状态(check_chunk)

接口名  | check_chunk(chunk_ID)
:------ | :------------ 
参数说明 | @**chunk_IDs**:要检查的chunk的ID列表
返回值  | 返回状态
说明 | 空

## 5.流程设计

### 5.1 目录操作流程

#### 5.1.1 目录创建(mkdir)

![目录创建流程](https://raw.githubusercontent.com/hustlijian/simplecfs/master/docs/img/SimpleCFS Sequence.001.jpg )

#### 5.1.2 目录删除(rmdir)

![目录删除流程](https://raw.githubusercontent.com/hustlijian/simplecfs/master/docs/img/SimpleCFS Sequence.002.jpg )

#### 5.1.3 目录列举(listdir)

![目录列举流程](https://raw.githubusercontent.com/hustlijian/simplecfs/master/docs/img/SimpleCFS Sequence.003.jpg )

#### 5.1.4 目录设置(chdir)

![目录设置流程](https://raw.githubusercontent.com/hustlijian/simplecfs/master/docs/img/SimpleCFS Sequence.004.jpg )

#### 5.1.5 目录信息(statdir)

![目录信息流程](https://raw.githubusercontent.com/hustlijian/simplecfs/master/docs/img/SimpleCFS Sequence.005.jpg )

### 5.2 文件操作流程

#### 5.2.1 文件写(putfile)

![文件写入流程](https://raw.githubusercontent.com/hustlijian/simplecfs/master/docs/img/SimpleCFS Sequence.006.jpg )

#### 5.2.2 文件正常读(getfile)

![文件正常读流程](https://raw.githubusercontent.com/hustlijian/simplecfs/master/docs/img/SimpleCFS Sequence.007.jpg )

#### 5.2.3 文件解码读(getfile)

![文件解码读流程](https://raw.githubusercontent.com/hustlijian/simplecfs/master/docs/img/SimpleCFS Sequence.008.jpg )

#### 5.2.4 文件删除(delfile)

![文件删除流程](https://raw.githubusercontent.com/hustlijian/simplecfs/master/docs/img/SimpleCFS Sequence.010.jpg )

#### 5.2.5 文件信息(statfile)

![文件信息流程](https://raw.githubusercontent.com/hustlijian/simplecfs/master/docs/img/SimpleCFS Sequence.011.jpg )

### 5.3 DS操作流程

#### 5.3.1 DS注册

![DS注册流程](https://raw.githubusercontent.com/hustlijian/simplecfs/master/docs/img/SimpleCFS Sequence.012.jpg )

#### 5.3.2 DS状态

![DS状态流程](https://raw.githubusercontent.com/hustlijian/simplecfs/master/docs/img/SimpleCFS Sequence.013.jpg )

#### 5.3.3 chunk状态

![chunk状态流程](https://raw.githubusercontent.com/hustlijian/simplecfs/master/docs/img/SimpleCFS Sequence.014.jpg )
