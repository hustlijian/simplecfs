# SimpleCFS DS详细设计

## 文档历史记录
文档版本 | 描述	| 编写人员 | 更新时间
----    | ---   | ---    | --- 
0.1	| 开始创建 | 李剑	| 2015-01-06

## 1. DS在整个架构中的作用

* 数据chun存储管理
* 数据chunk读、写、删除请求处理

## 2. DS接口

### 2.1 对Client接口

#### 2.1.1 写入chunk

接口名  | add_chunk(chunk_id, chunk_length, chunk_data)
:------ | :------------ 
参数说明 | @**chunk_id**: id也是存取的文件名，@**chunk_length**：数据长度,@**chunk_data**：数据内容
返回值  | 返回类**RetValue**
说明 | 空

#### 2.1.2 读取chunk

接口名  | get_chunk(chunk_id, total_blocks, block_list)
:------ | :------------ 
参数说明 | @**chunk_id**: id也是存取的文件名，@**total_blocks**:chunk中block的总数，@**block_list**:要取的数据分片序号列表
返回值  | 返回整合后的数据内容，失败则返回相应的状态
说明 | 空

#### 2.1.3 删除chunk

接口名  | delete_chunk(chunk_id)
:------ | :------------ 
参数说明 | @**chunk_id**: id也是存取的文件名，
返回值  | 返回类**RetValue**
说明 | 空


### 2.2 对MDS接口

#### 2.2.1 DS注册

接口名  | add_ds(rack_id, ds_ip, ds_port)
:------ | :------------ 
参数说明 | @rack_id: 机柜编号，@ds_ip,@ds_port:ds监听命令的ip和端口，
返回值  | 返回类**RetValue**
说明 | ds向mds注册


#### 2.2.2 chunk状态

接口名  | check_chunk(chunk_id)
:------ | :------------ 
参数说明 | @**chunk_id**:要检查的chunk的id
返回值  | 返回状态信息
说明 | mds向ds检查chunk状态

#### 2.2.3 汇报DS状态

接口名  | report_ds(state_info)
:------ | :------------ 
参数说明 | @state_info, DS的状态，
返回值  | 返回类**RetValue**
说明 | ds向mds汇报自己的状态

### 2.3 本地存储管理接口

#### 2.3.1 增加chunk

write_chunk(chunk_id, chunk_data) -> ret_code

#### 2.3.2 删除chunk

remove_chunk(chunk_id) -> ret_code

#### 2.3.3 读取本地存储相关信息

信息包括chunk信息，也可以是存储负载，容量等信息。

info_chunk(chunk_id) -> chunk_info(list?)

#### 2.3.4 读取chunk

read_chunk(chunk_id,total_blocks, block_list)((ret_code,data_list))

## 3. 关键数据结构

### TODO

## 4. 内部流程

### 4.1 启动流程

1. 命令行参数的处理，包括配置文件路径，IP和端口，开关等；
2. 配置文件的加载和修改，包括MDS的IP和端口号，目录等；
3. 生成数据管理后台实例，进行如下操作：
    1. 全局变量的设置，
    2. 向MDS注册（报告自己的存储信息），
    3. 启动命令监听线程，
4. 循环处理。

### 4.2 写流程

写入数据到指定目录。

### 4.3 读流程

根据读取函数的要求，读取相应的数据，合并返回。

### 4.4 删除chunk

调用删除chunk函数删除数据文件

## 5. 通信协议

### 5.1 操作码定义

#### 5.1.1 与Client端交互

读操作：

	OP_ADD_CHUNK ＝ 'ADD_CHUNK'
	OP_ADD_CHUNK_REPLY = 'ADD_CHUNK_REPLY'

写操作：

	OP_GET_CHUNK = 'GET_CHUNK'
	OP_GET_CHUNK_REPLY = 'GET_CHUNK_REPLY'
	
删除操作：

	OP_DEL_CHUNK = 'DELETE_CHUNK'
	OP_DEL_CHUNK_REPLY = 'DELETE_CHUNK_REPLY'
	
#### 5.1.2 与MDS交互

注册：

	OP_ADD_DS = 'ADD_DS'
	OP_ADD_DS_REPLY = 'ADD_DS_REPLY'
	
DS状态：
	
	OP_CHECK_DS = 'CHECK_DS'
	OP_CHECK_DS_REPLY = 'CHECK_DS_REPLY'
	
chunk状态：

	OP_CHECK_CHUNK = 'CHECK_CHUNK'
	OP_CHECK_CHUNK_REPY = 'CHECK_CHUNK'
	
#### 5.1.3 状态码

读相关：

	RET_READ_ERROR = 4012; //读错误	RET_CHUNK_NOT_EXIST_IN_DS = 4003;//所请求的chunk在DS上不存在	RET_BLOCK_OR_META_FILE_NOT_EXIST = 4004; //chunk对应的chunk文件或者校验文件缺失	RET_BLOCK_OR_META_FILE_OPEN_ERROR = 4005; ///chunk对应的chunk文件或者校验文件打开失败	RET_READ_VERSION_NOT_MATCH= 4006; //所请求的chunk的数据长度和校验长度不匹配	RET_READ_CHECKSUM_NOT_MATCH = 4007; //本地的chunk文件和其校验文件校验不匹配	RET_READ_REQUEST_INVALid = 4008; //读请求不合法	RET_READ_MEMMALLOC_ERROR = 4009; //在读的时候分配内存出错	RET_READ_CRC_CHECK_ERROR = 4010; //所请求的chunnk的版本和本地存储版本不一致	RET_READ_PREAD_ERROR = 4011; //读文件出错写相关：
	RET_CHUNK_FILE_EXISTS = 4012; //所写的chunk已存在	RET_CHUNK_FILE_CREATE_ERROR = 4013; //创建chunk文件出错	RET_PIPELINE_TARGET_NUM_NOT_MATCH = 4014; //流水线的数目和packet中的数据不匹配	RET_PIPELINE_STATUS_NULL = 4015; //调用fs_interface中的接口返回NULL。	RET_PIPELINE_VERSION_NOT_MATCH= 4016; //所要写的chunk的版本不匹配	RET_PIPELINE_DATASERVER_id_NOT_MATCH= 4017; //收到的请求包中的DataServerid和本身的id不一样	RET_PIPELINE_VERSION_NOT_INVALid= 4018; //所要写的chunk的版本不合法	RET_WRITE_PACKET_CHECKSUM_ERROR= 4022; //所接受的写packet中chunk数据和其校验数据不匹配	RET_WRITE_PACKET_WRITE_FILE_ERROR= 4023; //建立pipeline失败	RET_WRITE_MEMMALLOC_ERROR = 4025; //内存池分配内存失败	RET_WRITE_MODE_INVALid = 4026; //写的模式不合法	RET_WRITE_FTRUNCATE_ERROR = 4027; //截断文件失败	RET_WRITE_COMPLETE_SUCCESS = 4028; //写提交成功	RET_WRITE_ITEM_NOT_IN_MAP = 4029; //在存储读写信息的map中没有改block对应的项	RET_OPERATION_ON_GOING = 4100;//在当前chunk上已存在其他操作

### 5.2 与Client交互协议

#### 5.2.1 读操作

Client -> DS 读chunk请求

OP_GET_CHUNK | chunk id | total blocks  | blocks list
:------ | :----------| :----------| :---------
2 bytes | 8 bytes | 8 bytes | ...

DS -> Client 读chunk回复

OP_GET_CHUNK_REPLY | chunk id | RET code | blocks num | data
:------ | :----------| :---------- | :---- | :----
2 bytes | 8 bytes | 2 bytes | 8 bytes | ...

#### 5.2.2 写操作

Client -> DS 写chunk请求

OP_ADD_CHUNK | chunk id | chunk length | checksum length | data | checksum
:------ | :----------| :----------| :---------| :---------| :-------
2 bytes | 8 bytes | 8 bytes | 8 bytes | ... | ...


DS -> Client 写chunk回复

OP_ADD_CHUNK_REPLY | chunk id | RET code
:------ | :----------| :----------
2 bytes | 8 bytes | 2 bytes 

#### 5.2.3 删除操作

Client -> DS 删chunk请求

OP_DEL_CHUNK | chunk id 
:------ | :----------
2 bytes | 8 bytes 


DS -> Client 删chunk回复

OP_DEL_CHUNK_REPLY | chunk id | RET code
:------ | :----------| :----------
2 bytes | 8 bytes | 2 bytes 

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

#### 5.3.2 DS状态

MDS -> DS 状态请求

OP_CHECK_DS | ds id
:------ | :----------
2 bytes |  8 bytes 

DS -> MDS 状态回复

OP_CHECK_DS_REPLY | RET code 
:------ | :----------
2 bytes | 2 bytes 

#### 5.3.3 chunk状态

MDS -> DS chunk状态请求

OP_CHECK_CHUNK | CHUNK id
:------ | :----------
2 bytes |  8 bytes 

DS -> MDS chunk状态回复

OP_CHECK_CHUNK_REPLY | RET code 
:------ | :----------
2 bytes | 2 bytes 

## 6. 其它操作

### 6.1 本地数据chunk管理

#### 6.1.1 写入chunk

#### 6.1.2 读取chunk

#### 6.1.3 删除chunk

## 7. 异常情况

### 7.1 写异常

### 7.2 读异常

### 7.3 数据chunk损坏

