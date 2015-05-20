# SimpleCFS Client详细设计

## 文档历史记录
文档版本 | 描述	| 编写人员 | 更新时间
----    | ---   | ---    | --- 
0.1	| 开始创建 | 李剑	| 2015-01-29

## 1. Client在整个架构中的作用

* 对外的通用接口：对外提供系统访问的API接口
* 与元数据服务器交互：和MDS交互，建立，读取，删除文件或目录的元信息
* 与数据存储服务器交互：和DS交互，写入，读取chunk数据
* 编码操作：对object进行编码和解码工作，必要时提供修复支持

## 2. Client提供的接口

### 2.1 对用户接口
Client对用户提供的接口主要包含两类，分别是目录操作和文件操作。

#### 2.1.1 创建目录(mkdir)

接口名  | mkdir(dirname)
:------ | :------------ 
参数说明 | @**dirname**：要创建的目录的路径名
返回值  | 返回类**RetValue**,包含返回状态和信息   
说明 | 目录名可以是相对路径也可以是绝对路径

#### 2.1.2 删除目录(rmdir)

接口名  | rmdir(dirname)
:------ | :------------ 
参数说明 | @**dirname**：要删除的目录
返回值  | 返回类**RetValue**
说明 | 只在目录为空时成功 

#### 2.1.3 列举目录(listdir)

接口名  | listdir(dirname)
:------ | :------------ 
参数说明 | @**dirname**：要查询的目录
返回值  | 返回一个目录的列表
说明 | 不保证顺序，不包含`.`和`..`，目录以`/`结尾

#### 2.1.4 设置当前目录(chdir)

接口名  | chdir(path)
:------ | :------------ 
参数说明 | @**path**：要到达的目录
返回值  | 返回类**RetValue**
说明 | 空

#### 2.1.5 获得当前目录(getcwd)

接口名  | getcwd()
:------ | :------------ 
参数说明 | 空
返回值  | 返回当前目录的字符串
说明 | 使用类unix的`sep`,即`/`

#### 2.1.6 目录信息(statdir)

接口名  | statdir(dirname)
:------ | :------------ 
参数说明 | @**dirname**: 查询的目录
返回值  | 返回给定目录的信息字典
说明 | 空

#### 2.1.7 写文件(putfile)

接口名  | putfile(src_path, des_path, code_info)
:------ | :------------ 
参数说明 | @**src_path**:本地的文件路径,@**des_path**:存储到系统的文件路径（包括文件名），@**code_info**:编码信息，包括编码类型和编码参数（**TODO**）
返回值  | 返回类**RetValue**
说明 | 编码信息要和博士讨论下，一个字典，包括类型和参数k,m等

#### 2.1.8 读文件(getfile)

接口名  | getfile(des_path, local_path, repair_flag)
:------ | :------------ 
参数说明 | @**des_path**:系统中文件的路径,@**local_path**:本地文件路径（包括文件名）@**repair_flag**:为True时，如果有数据损坏对文件修复，为False时不处理
返回值  | 返回类**RetValue**
说明 | 读文件返回的状态比较多，有正常读，解码读和修复读。

#### 2.1.9 删除文件(delfile)

接口名  | delfile(path)
:------ | :------------ 
参数说明 | @**path**:要删除文件的路径
返回值  | 返回类**RetValue**
说明 | 空

#### 2.1.10 文件信息(statfile)

接口名  | statfile(path)
:------ | :------------ 
参数说明 | @**path**: 查询的文件路径
返回值  | 返回给定文件的信息字典
说明 | 空

### 2.2 对MDS接口

#### 2.2.1 创建目录(make_dir)

接口名  | make_dir(dirname)
:------ | :------------ 
参数说明 | @**dirname**：要创建的目录的路径名
返回值  | 返回类**RetValue**,包含返回状态和信息   
说明 | 目录名是绝对路径

#### 2.2.2 删除目录(remove_dir)

接口名  | remove_dir(dirname)
:------ | :------------ 
参数说明 | @**dirname**：要删除的目录
返回值  | 返回类**RetValue**
说明 | 只在目录为空时成功 

#### 2.2.3 列举目录(list_dir)

接口名  | list_dir(dirname)
:------ | :------------ 
参数说明 | @**dirname**：要查询的目录 
返回值  | 返回一个目录的列表
说明 | 不保证顺序，不包含`.`和`..`,目录以`/`结尾

#### 2.2.4 目录信息(status_dir)

接口名  | status_dir(dirname)
:------ | :------------ 
参数说明 | @**dirname**: 查询的目录
返回值  | 返回给定目录的信息字典
说明 | 空

#### 2.2.5 目录是否存在(valid_dir)

接口名  | valid_dir(dirname)
:------ | :------------ 
参数说明 | @**dirname**: 查询的目录
返回值  | 存在返回True，否则返回False
说明 | 空

#### 2.2.6 写文件(add_file)

接口名  | add_file(des_path,file_info)
:------ | :------------ 
参数说明 | @**des_path**:存储到系统的文件路径（包括文件名）,@**file_info**:文件信息，包括编码信息，包括编码类型和编码参数。
返回值  | 返回object,chunk到DS的映射列表
说明 | 返回的项目是嵌套的字典，object_size由MDS定，编码方式同client定（设置默认值）

#### 2.2.7 提交写文件完成(add_file_commit)

接口名  | add_file_commit(des_path)
:------ | :------------ 
参数说明 | @**des_path**:存储到系统的文件路径（包括文件名）
返回值  | 返回类**RetValue**
说明 | 提交后，把元数据写入到数据库中

#### 2.2.8 文件信息(stat_file)

接口名  | stat_file(path)
:------ | :------------ 
参数说明 | @**path**: 查询的文件路径
返回值  | 返回给定文件的信息字典
说明 | 空

#### 2.2.9 删除文件(delete_file)

接口名  | delete_file(path)
:------ | :------------ 
参数说明 | @**path**:要删除文件的路径
返回值  | 返回类**RetValue**
说明 | 空

#### 2.2.10 读文件(get_file)

接口名  | get_file(des_path)
:------ | :------------ 
参数说明 | @**des_path**:系统中文件的路径
返回值  | 返回object和chunk到DS的映射列表，编码信息(code_info)
说明 | 读文件返回的状态比较多，有正常读，解码读和失败。

#### 2.2.11 读object(get_object)

接口名  | get_object(object_ID)
:------ | :------------ 
参数说明 | @**object_ID**:要读取object的ID
返回值  | 返回object和chunk到DS的映射列表，编码信息(code_info)
说明 | 读object返回的状态比较多，有正常读，解码读和失败。

#### 2.2.12 读chunk(get_chunk)

接口名  | get_chunk(chunk_ID)
:------ | :------------ 
参数说明 | @**chunk_ID**:要读取chunk的ID
返回值  | 返回chunk到DS的映射列表，编码信息(code_info)
说明 | 读chunk返回的状态比较多，有正常读，解码读和失败。

#### 2.2.13 修复chunk(repair_chunk)

接口名  | repair_chunk(chunk_ID)
:------ | :------------ 
参数说明 | @**chunk_ID**:要修复的chunk的ID
返回值  | 如果要修复返回新的DS信息，不用时返回空的IP
说明 | 这个是在修复读时调用

#### 2.2.14 修复提交完成chunk(repair_chunk_commit)

接口名  | repair_chunk_commit(chunk_ID, ds_id)
:------ | :------------ 
参数说明 | @**chunk_ID**:要修复的chunk的ID, @**ds_id**:新chunk放置的ds位置
返回值  | 返回类**RetValue**
说明 | 修复chunk相应的ds信息

### 2.3 对DS接口

#### 2.3.1 写入chunk(add_chunk)

接口名  | add_chunk(chunk_ID, chunk_length, chunk_data)
:------ | :------------ 
参数说明 | @**chunk_ID**: ID也是存取的文件名，@**chunk_length**：数据长度,@**chunk_data**：数据内容
返回值  | 返回类**RetValue**
说明 | 空

#### 2.3.2 读取chunk(get_chunk)

接口名  | get_chunk(chunk_ID, blocks)
:------ | :------------ 
参数说明 | @**chunk_ID**: ID也是存取的文件名，@**blocks**:要取的数据分片序号列表
返回值  | 返回整合后的数据内容，失败则返回相应的状态
说明 | 空

#### 2.3.3 删除chunk(delete_chunk)

接口名  | delete_chunk(chunk_ID)
:------ | :------------ 
参数说明 | @**chunk_ID**: ID也是存取的文件名，
返回值  | 返回类**RetValue**
说明 | 空

### 2.4 编码接口

#### 2.4.1 编码object(encode_object)

接口名  | encode_object(object_data, code_info)
:------ | :------------ 
参数说明 | @**object_data**: 要编码的数据内容，@**code_info**:编码信息，包括方式和参数
返回值  | 返回编码后的chunk列表，
说明 | 编码方式的封装在code_info中体现

#### 2.4.2 解码object(decode_object)
接口名  | encode_object(chunk_list, code_info)
:------ | :------------ 
参数说明 | @**chunk_list**: 要解码的chunk列表，@**code_info**:编码信息，包括方式和参数
返回值  | 返回解码后的object内容
说明 | 编码方式的封装在code_info中体现,chunk_list内容有原来的编号顺序

#### 2.4.3 修复某个chunk(repair_chunk)

接口名  | repair_chunk(chunk_list, chunk_gen, code_info)
:------ | :------------ 
参数说明 | @**chunk_list**: 帮助修复的chunk列表，@**chunk_gen**:要修复的chunk编号，@**code_info**:编码信息，包括方式和参数
返回值  | 返回修复后的chunk内容
说明 | 编码方式的封装在code_info中体现,chunk_list内容有原来的编号顺序

## 3. 关键数据结构

### TODO

## 4. 内部流程

### 4.1 启动流程

1. 命令行参数的处理，包括配置文件路径，IP和端口，开关等；
2. 配置文件的加载和修改，包括MDS的IP和端口号，目录等；
3. 根据不同的命令请求，调用不同的client API。

**TODO**，这个启动流程也不多，看到来。

### 4.2 目录流程

### 4.3 文件流程


## 5. 通信协议

协议就使用json的格式来传输数据。

### 5.1 操作码定义

#### 5.1.1 与Client端交互

读操作：

	OP_ADD_CHUNK ＝ 'ADD_CHUNK'
	OP_ADD_CHUNK_REPLY = 'ADD_CHUNK_REPLY'

	
#### 5.1.2 与MDS交互

注册：

	OP_ADD_DS = 'ADD_DS'
	OP_ADD_DS_REPLY = 'ADD_DS_REPLY'
	
	
#### 5.1.3 状态码

读相关：

	RET_READ_ERROR = 4012; //读错误	RET_CHUNK_NOT_EXIST_IN_DS = 4003;//所请求的chunk在DS上不存在

### 5.2 与DS交互协议

#### 5.2.1 读操作

Client -> DS 读chunk请求

OP_GET_CHUNK | chunk id | total blocks  | blocks list
:------ | :----------| :----------| :---------
2 bytes | 8 bytes | 8 bytes | ...

DS -> Client 读chunk回复

OP_GET_CHUNK_REPLY | chunk id | RET code | blocks num | data
:------ | :----------| :---------- | :---- | :----
2 bytes | 8 bytes | 2 bytes | 8 bytes | ...


### 5.3 与MDS交互协议

#### 5.3.1 注册

DS -> MDS 注册请求

OP_ADD_DS | rack id | ds id
:------ | :----------| :----------
2 bytes | 4 bytes | 8 bytes 

ds id :DS的id，包括hostname，IP，port

MDS -> DS 注册回复

OP_ADD_DS_REPLY | RET code 
:------ | :----------
2 bytes | 2 bytes 


## 6. 其它操作

### 6.1 本地数据chunk管理

#### 6.1.1 写入chunk

## 7. 异常情况

### 7.1 写异常



