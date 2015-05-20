# SimpleCFS MDS详细设计

## 文档历史记录
文档版本 | 描述	| 编写人员 | 更新时间
----    | ---   | ---    | --- 
0.1	| 开始创建 | 李剑	| 2015-03-11

## 1. MDS在整个架构中的作用

* 元数据存储：文件系统中目录结构，文件信息(数据块位置)和DS信息
* 与Clinet交互：和Client交互，提供目录和文件的操作
* 与DS交互：和DS交互，保存DS信息，检查DS,数据块状态

## 2. MDS提供的接口

### 2.1 元数据保存接口

这里使用了key-value数据库，所以元数据的保存也是使用类key-value的使用接口。

#### 2.1.1 数据保存(set)

接口名  | set(key, value)
:------ | :------------ 
参数说明 | @key：要保存的键值，@value:要保存的数据值
返回值  | 成功返回True，否则返回False
说明 | 无

#### 2.1.2 数据读取(get)

接口名  | get(key)
:------ | :------------ 
参数说明 | @key：要读取的键值
返回值  | 成功返回数据，否则返回None
说明 | 无

#### 2.1.3 数据删除(delete)

接口名  | delete(key)
:------ | :------------ 
参数说明 | @key：要删除的键值
返回值  | 成功返回True，否则返回False
说明 | 无

#### 2.1.4 判断数据是否存在(exists)

接口名  | exists(key)
:------ | :------------ 
参数说明 | @key：要本命年的键值
返回值  | 存在返回True，否则返回False
说明 | 无


### 2.2 对Client接口

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

#### 2.2.4 检查DS状态

接口名  | check_ds()
:------ | :------------ 
参数说明 | None
返回值  | 返回类**RetValue**
说明 | mds检查ds状态，可以连上就是OK的，不能连接则设置DS状态为断开

## 3. 元数据信息

	"dir:DIRECTORY_ID" = {  // 目录信息，目录名DIRECTORY以'/'结尾
        "parent_dir":           "PARENT_DIRECTORY_ID",  //父目录，根目录为'/'
        "create_time":          "2014-11-32T16:25:43.511Z",  //创建时间
    }

    "subfiles:DIRECTORY_ID" = [  // 目录子文件信息，子目录以'/'结尾
        "SUBFILE0",  //子文件或子目录名
        "SUBFILE1",
        "etc."
    ]

    "tmp:FILE_ID" = {  // 文件增加时临时保存文件的信息(设置过期时间)，在文件commit后写入正式的表中
        "filename": "/testfile", //
        "filesize":         1024,  // 文件大小(byte)
         "code" : {
            "type":       "rs/zcode/etc.", // 编码方法
            "k":                4, 
            "m":                2,
            "packet_size": 1024,
        },

        "object_num":    12, // 划分的object数目
        "object_size":      1024, // object大小(byte)
        "chunk_size":   256,  // chunk 大小(byte)
        "block_size": 256, //256的倍数
        "chunk_num":     4, // 一个stripe中chunk数目，包含数据和校验

        "objects": [
            ["ds:ip0", "ds:ip1", "..."],  // object0 信息, chunk in ds position
            ["ds:ip0", "ds:ip1", "..."],  // object1 信息, chunk in ds position
            // etc.
        ]
    }

    "file:FILENAME_ID" = {  // 文件信息
        "filename": "/testfile",
        "filesize":         "1024",  // 文件大小(byte)
        "create_time":      "2014-11-32T16:45:43.511Z",  //创建时间
        "code" : {
            "type":       "rs/zcode/etc.", // 编码方法
            "k":                "4", 
            "m":                "2",
        },
        "object_number":    12, // 划分的object数目
        "object_size":      64, // object大小(byte)
        "chunk_num":    4, // chunk num in one object
        "block_size": 256, //256的倍数

        "objects:FILENAME_ID": [  // 划分的的object ID，不用保存。
            "FILENAME_obj0",  // 文件名加"_obj[num]", num = 0 ~ (object_number-1)
            "FILENAME_obj1",
            "etc."
        ]
    }

    "object:OBJECT_ID" = {  // object 信息
        "code" : {
            "type":       "rs/zcode/etc.", // 编码方法
            "k":                "4", 
            "m":                "2",
        },
        "object_size":      "64", // object大小(byte)
        "chunk_number":     "4", // chunk数目，包含数据和校验
        "block_size": 256, //256的倍数

        "chunks:OBJECT_ID": [  // 划分后的chunk ID，不用保存。
            "OBJECT_ID_chk0", // object ID加"_chk[num]", num = 0 ~ (chunk_number-1)
            "OBJECT_ID_chk1",
            "ect."
        ]
    }

    "chunk:CHUNK_ID" = {  // chunk 信息
        "chunk_size":   256,  // chunk 大小(byte)
        "block_size": 256,    // block 大小(byte), 256的倍数
        "block_num": 1, // block num in one chunk
        "ds_id":        "DS_ID",  // 保存在的ds的ID
        "status": "ok/break/missing/damaged", // 状态：正常/断开与ds的连接/丢失/损坏，读取时检查
    }

    "ds:DS_ID" = {  // ds 信息,DS_ID是ip:port的字符串
        "ip":   "192.168.1.200",
        "port": 7000,
        "rack": 0,
        "status": "connected/break",  // 状态：连接/断开
        "space": 10240,  // M
        "chunk_num": 87, 
        "update_time": "2015-01-01 22:10:23.522",
        // etc.
    }

    "ds_alive" = [  // 记录存活的ds的ID
        "DS_ID 1",
        "DS_ID 2",
        "etc.",
    ]
    

元数据以`json`的格式表示，具体在保存到`key-value`的数据库中时，使用`key`前缀来表示数据表，如`file:/testfile`表示一个文件的`ID`，其它的类似，因为`redis`不支持嵌套的数据结构，所以所有的数据只有一层的`value`，不过这个`value`使用`python`中的字典序列化成字符串，因些可以保存不少数据，修改时也是以`value`为整体的修改方式。

## 4. 内容流程

## 5. 其它