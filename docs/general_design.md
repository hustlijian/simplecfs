#SimpleCFS分布式文件系统概要设计

##文档历史记录
文档版本 | 描述	| 编写人员 | 更新时间
----    | ---   | ---    | --- 
0.1	| 创建基本需求，框架 | 李剑	| 2014-11-26
0.2 | 完成基本概要设计内容 | 李剑 | 2014-12-01

**关键字**：分布式文件系统，纠删码  

**摘要**：本文描述系统的需求分析和概要设计，作为详细设计和后序开发的准备工作。  

**缩略语列表**：

* **SimpleCFS**(Simple Coded File System): 简单编码文件系统
* **MDS**(Metadata Server): 元数据服务器，主控节点
* **DS**(Data Server): 数据存储节点服务器
* **Client**: 客户端
* **Object**: 大文件切分为多个object，每个object为固定的大小
* **Chunk**: 每个object分为多个chunk，由编码方式确定分的个数和大小，每个chunk保存到一个DS中

## 1. 概述
### 1.1 目的
本文描述需求说明和总体架构设计，方便详细设计和后序开发参考

### 1.2 系统说明
* 操作系统：ubuntu 12.04 LTS (`lsb_release -a`)  
* 硬件架构：x86_64 (`uname -m`)  
* 内核版本：3.2.0-23-generic (`cat /proc/sys/kernel/osrelease`)  
* gcc(g++)版本：4.6.3 (`gcc/g++ --version`)  
* python版本：2.7.3 (`python --version`)

## 2. 需求分析
### 2.1 系统需求
本文件系统主要是为了解决海量大文件的存储问题，使用编码方法提高数据的可靠性。本分布式文件系统应该能支撑数以千万计的大文件（百M以上）。上层应用是一个“一次写入多次读取”的文件访问模型，不支持文件修改。这一假设简化了数据一致问题。使用普通服务机，机器宕机是必然会发生的。本系统支持多种编码方法，提供通用的文件存取接口，和数据块读取支持。

基于上述需求，整理出本文件系统最基本的需求如下：  

1. 可靠性   
 数据可靠性： 使用数据编码来保证数据的可靠性，使用不同的编码方式，提供不同的可靠级别（可配置）  
 服务可靠性： 当磁盘损坏或机器宕机，不影响上层应用使用。  
2. 可扩展性  
  分布式架构，支持数据服务器扩容，增加数据容量。  
3. 数据完整性  
保证存储的数据是正确的  
4. 数据一致性  
多个客户端访问的数据是一致的  
5. 简单  
易实现和维护

### 2.2 性能
支持大数据级的存储，数据存储使用纠删码提高了数据的可靠性减小的存储开销，相关编码方法可以减少修复的带宽。  
集群大小：TODO  
存储能力：TODO  
客户端并发数：TODO  
客户端延时：

### 2.3 功能
满足大文件的读写删除，高可用性，可靠性，可扩展性，这些主要都是通过系统整体实现的。 

#### 2.3.1 MDS功能
* 元数据（metadata）管理：支持传统的层次型文件组织结构，用户可以创建、删除文件或目录，可以读写文件。     
* 元数据持久化：使用下层的内存数据库对元数据提供持久化支持     
* 负载均衡：数据结点的负载均衡  
* 与数据存储服务器交互：数据结点注册和状态监测，数据chunk检测
* 与客户端交互：提供文件，object，chunk的元数据信息

#### 2.3.2 DS功能
* 数据存储：存储数据chunk，并管理其信息
* 与元数据服务器交互：和MDS交互，提供节点信息和chunk信息
* 与客户端交互：和Client交互，提供数据chunk的写入和读取接口，并对不同编码的数据读写提供相应支持（如r>1,要对数据chunk分片）

#### 2.3.3 Client功能
* 对外的通用接口：对外提供系统访问的API接口
* 与元数据服务器交互：和MDS交互，建立，读取，删除文件或目录的元信息
* 与数据存储服务器交互：和DS交互，写入，读取chunk数据
* 编码操作：对object进行编码和解码工作，必要时提供修复支持

## 3. 系统架构
系统架构使用典型的分布式三方架构，包括：  

1. 元数据结点： MDS
2. 数据存储结点：DS
3. 客户端：Client

系统如下图：

![架构图](https://raw.githubusercontent.com/hustlijian/simplecfs/master/docs/img/SimpleCFS%20Architecture.001.jpg )

其中MDS是元数据管理结点，也是主控节点，存放DS的节点信息和文件的多级元数据信息，包括文件和目录到ID的映射，ID到信息的映射，其中文件划分为多个object，以object为单位编码到多个chunk，每个chunk相应保存到一个DS结点中。MDS使用内存数据库来做元数据的存储方式，减少了系统的复杂度，使用内存数据库提供了对数据的持久化支持，也提供了良好的访问性能。MDS中只能元数据信息流，没有数据流，相对减小了MDS的负载开销。

DS是存储实际数据的单元，存储的数据单位是chunk，读取的单位可以小于chunk，支持对chunk中给定位置的数据进行读取，由此支持多种编码方法的解码和修复要求。

Client以API的方式提供给用户使用，提供put, get, delete等几个简单的操作接口，后面的分布式和编码可以对用户透明。Client和MDS交互，可以获得文件划分成object的信息和object划分成的chunk到DS的映射关系。通过DS的信息直接和DS交互完成相应的读、写和删除操作。

## 4. 模块设计
详细模块设计如下图：

![模块设计图](https://github.com/hustlijian/simplecfs/blob/master/docs/img/SimpleCFS%20Module.001.jpg)

### 4.1 MDS模块设计
MDS维护着整个文件系统的全局元数据信息，这些信息是动态更新的，且使用了持久化机制。元数据信息主要包括目录和文件信息，文件chunk的信息和DS的信息。

MDS还提供访问和管理功能，包括：元数据操作，内存数据库使用，DS管理，Client交互。

#### 4.1.1 元数据操作
具体元数据格式参考SimpleCFS Metadata.json文件中的格式说明，元数据使用json格式保存，可以方便转化为key-value存储的数据表，或其它存储格式。

#### 4.1.2 内存数据库
元数据使用内存数据库来保存和读取，在本设计中，计划使用[redis](http://redis.io/)。redis支持相对丰富的数据类型，且性能优越，对持久化提供支持，满足本系统对元数据的存取要求。

#### 4.1.3 DS管理
DS管理中包括DS注册、DS状态检测和chunk状态检测。

DS注册功能：接收DS发送的注册信息，把DS编号，保存到DS信息列表

DS状态检测：周期性对DS信息列表中的DS检测其状态，并更新其状态，包括在线和离线

chunk状态检测：周期性对chunk对就的DS进行状态检测，并更新其状态，包括正常、离线、损坏和丢失等

#### 4.1.4 Client交互
MDS对Client提供元数据查询接口，提供元数据的增加、删除和查询，并返回结果给客户端。这些信息只是元数据信息流，没有数据流。

### 4.2 DS模块设计
DS是数据保存的位置，保存的单位是chunk，使用chunk的ID作为文件名，在设定的目录中保存chunk数据。DS启动时会向MDS注册，MDS也可以通过配置文件设置DS的信息。MDS会周期性的对DS的状态和DS保存的chunk的状态进行检查，DS会提供相应的处理接口。DS和Client之间有数据的I/O传输。

#### 4.2.1 文件存取
文件是以chunk为单位来存取的，文件名用chunk的ID。文件的写入以chunk为单位，文件的读取是以编码类型相关的单位，支持r>1的chunk读取方式，即支持chunk中指定内容的读取。chunk使用校验，提供文件状态的信息。

#### 4.2.2 与MDS交互
DS提供注册和状态和chunk状态相关功能。

DS注册功能：DS启动时，读取配置文件中的MDS地址信息，或者由命令行参数获得MDS地址信息，会向MDS发出注册命令。

DS状态：回复MDS发出的DS状态请求命令，报告当前DS的状态，及使用情况。

chunk状态：回复MDS发出的chunk状态请求命令，检查chunk文件的状态。

#### 4.2.3 与Client交互
DS提供Client chunk文件的存取功能。支持接口如：add_chunk, del_chunk, read_chunk，其中read_chunk可以支持对指定位置的文件部分进行读取。

### 4.3 Client模块设计
Client是以API的形式让用户使用，后面可以扩展加入对应的shell，方便简单的使用文件系统。Client要和MDS和DS交互且文件分块、编码、解码和修复的工作都在Client中进行。所含的模块有：

Client API:提供方便简单的类REST API接口给上层的用户使用，如put_file, get_file, delete_file，stat_file, mkdir, rmdir等。

与MDS交互：Client向MDS获得元数据信息，同时也可以增加和修改元数据信息。

与DS交互：Client向DS存取文件数据。

编码操作：Client对文件分成等大的object，然后并行根据编码类型对object编码为多个chunk,向MDS请求DS列表后，把数据chunk和编码后的校验chunk保存一不同的DS中。文件读取时，Client并行的读取不同的object，解码chunk，合并成文件后提供给用户。

## 5. 读写流程
读写文件是文件系统中的重要内容。本系统中先将大文件切割为Object(对象)，大小可配置，然后对Object编码成多个chunk(块)组成一个stripe(条带)，一个stripe中的数据分别保存到不同的DS中，部分的编码方式（R>1）可能需要对chunk分成多个逻辑block(片)。文件在Client中进行编码和解码操作。

### 5.1 写入流程
写入流程如下图：

![写入流程](https://raw.githubusercontent.com/hustlijian/simplecfs/master/docs/img/SimpleCFS%20Sequence.006.jpg )

1. 用户使用Client的API调用put_file的接口来写入文件到文件系统，在Client中修改好目标路径为绝对路径，并以此为参数向MDS发起add_file的请求；
2. MDS接收到add_file的请求，验证文件名在数据库和缓存队列中是否存在，存在则报错，Client接收到错误退出；不存在则设置文件的ID和返回相关的ID(使用一个递增值)和Object数量和大小给Client，元数据先保存到缓存队列中。
3. Client对文件分成Object，不足用0补齐，每个Object对MDS发送add_object请求，MDS处理add_object请求，验证是否存在，保存相关信息到缓存队列中，返回object编码后stripe的DS列表；
4. Client对每个Object进行编码，根据MDS返回的DS列表发送chunk信息到对应的DS；
5. 所有Object中的所有chunk都保存好了后，Client向MDS发一个add_file_commit，MDS把和ID相关的元数据从缓存队列中读取，然后写入到数据库中；
6. 中间Client和DS有问题时，Client向MDS发送一个add_file_failed命令，然后退出；MDS接收到add_file_failed命令时把缓存队列中的对应ID的元数据删除；MDS自己也会定期对缓存中的数据做清除工作。

### 5.2 正常读流程
正常读时，文件的每个相关的数据chunk都没有出错，流程图如下：

![正常读流程](https://raw.githubusercontent.com/hustlijian/simplecfs/master/docs/img/SimpleCFS%20Sequence.007.jpg )

1. 用户使用Client的API调用get_file的接口来读取文件到应用程序中，在Client设置好目标路径后，以此为参数向MDS发起get_objects请求；
2. MDS接收到get_objects的请求，验证文件ID的是否存在，不存在则报错；存在则返回objects的数目和ID；
3. Client根据返回的objects的信息，对每个object对MDS调用get_object的请求；MDS处理get_object请求，检查object_ID的状态，这里是正常读，返回数据chunk的DS列表；
4. Client从DS中读取数据chunk的数据，整合为object;所有的object整合为文件，通过文件长度去除补齐的末尾0；
5. 元数据读取和数据读取失败时返回给用户相应的错误。

### 5.3 解码读流程
解码读时，文件中的数据chunk有丢失，不过可以使用解码的方式把数据恢复出来，流程图如下：

![解码读流程](https://raw.githubusercontent.com/hustlijian/simplecfs/master/docs/img/SimpleCFS%20Sequence.008.jpg )

1. 用户使用Client的API调用get_file的接口来读取文件到应用程序中，在Client设置好目标路径后，以此为参数向MDS发起get_objects请求；
2. MDS接收到get_objects的请求，验证文件ID的是否存在，不存在则报错；存在则返回objects的数目和ID；
3. Client根据返回的objects的信息，对每个object对MDS调用get_object的请求；MDS处理get_object请求，检查object_ID的状态，这里是解码读，如果数据chunk损坏且不能解码，则返回错误，否则返回解码相关的数据chunk列表和DS列表；
4. Client从DS中读取数据chunk的数据，解码整合为object;所有的object整合为文件，通过文件长度去除补齐的末尾0；
5. 元数据读取、数据读取失败和数据解码错误时返回给用户相应的错误。

## 6. MDS流程
### 6.1 启动流程
1. 启动redis；
1. 命令行参数的处理，包括配置文件路径，IP和端口，开关等；
2. 配置文件的加载和修改，包括redis的IP和端口号，object大小，编码类型等；
3. 生成元数据管理后台实例，进行如下操作：
    1. 全局变量的设置，
    2. 连接内存数据库(redis),初始化元数据(设置root目录)，
    3. 启动命令监听线程，
    3. 启动缓存元数据队列，
    4. 启动DS检测线程，
    5. 启动chunk检测线程，
4. 循环处理。

### 6.2 元数据管理流程
元数据管理采用key-value数据库模拟目录结构，使用内存数据库redis来实现。相对使用B+树或其它实现方式，内存数据库可以提供数据存取的效率要求，也提供了持久化的支持，实现简单。元数据可以使用json格式的如下：

    "id:PATH" {  // 目录和文件名映射到一个递增的id
        "id": "1",
    }
    
    "dir:DIRECTORY_ID" = {  // 目录信息，目录名DIRECTORY以'/'结尾
        "parent_dir":           "PARENT_DIRECTORY_ID",  //父目录，根目录为'/'
        "create_time":          "2014-11-32T16:25:43.511Z",  //创建时间
        "subfiles:DIRECTORY_ID":   [  //子目录
            "SUBFILE0",  //子文件或子目录名
            "SUBFILE1",
            "etc."
        ]
    }
    
    
    "file:FILENAME_ID" = {  // 文件信息
        "parent_dir":       "PARENT_DIRECTORY_ID",  //父目录 
        "filesize":         "1024",  // 文件大小
        "create_time":      "2014-11-32T16:45:43.511Z",  //创建时间
        "object_number":    "12", // 划分的object数目
        "objects:FILENAME_ID": [  // 划分的的object ID
            "FILENAME_obj0",  // 文件名加"_obj[num]", num = 0 ~ (object_number-1)
            "FILENAME_obj1",
            "etc."
        ]
    }
    
    "object:OBJECT_ID" = {  // object 信息
        "coded_type":       "rs/zcode/etc.", // 编码方法
        "k":                "4", 
        "m":                "2",
        "object_size":      "64", // object大小(M)
        "chunk_number":     "4", // chunk数目，包含数据和校验
        "chunks:OBJECT_ID": [  // 划分后的chunk ID
            "OBJECT_ID_chk0", // object ID加"_chk[num]", num = 0 ~ (chunk_number-1)
            "OBJECT_ID_chk1",
            "ect."
        ]
    }
    
    "chunk:CHUNK_ID" = {  // chunk 信息
        "ds_id":        "DS_ID",  // 保存在的ds的ID
        "chunk_size":   "8",  // chunk 大小
        "space": "1024", // 空间M
        "status": "ok/break/missing/damaged", // 状态：正常/断开与ds的连接/丢失/损坏
    }
    
    "ds:DS_ID" = {  // ds 信息
        "ip":   "192.168.1.200",
        "port": "8779",
        "status": "connected/break",  // 状态：连接/断开
    }
    
数据库中保存文件和目录到一个ID的映射，内部使用这个ID标识文件，ID使用一个递增的唯一整数，可能不连续。文件和目录的元数据保存基本的属性，文件分成多个object，object的ID使用文件名ID加后缀(\_obj[num])，见代码；object是编码的单元，分为多个chunk，chunk的ID由object的ID加后缀(\_chk[num])，见代码；DS的信息对应一个DS的ID，这样方便对DS的动态管理；每个chunk是DS的保存单位；为了支持list功能，目录中加入子文件(subfiles)这个属性，文件或目录加入删除时会修改这个属性。文件在加入时使用二段提交，元数据信息先使用一个缓存队列，在文件成功写入到DS后，提交到内存数据库中，如果数据写入DS出错，MDS删除相关相应的缓存数据，MDS也会定期对缓存数据进行清除。    

### 6.3 与Client交互流程
命令监听线程接收Client发出的命令请求，包括add_file,add_object,delete_file等等，解析命令，然后调用相应的处理程序，然后把结果返回给Client。

### 6.4 DS管理流程
命令监听线程接收DS发出的命令请求，包括register_ds的命令，解析命令，然后存储DS信息到内存数据库，如果DS已经存在则直接返回成功，否则修改DS列表。

DS检测线程会定时检测DS列表中的项，根据DS的反馈修改DS项的状态，有返回的则标记为“连接”，多次连接没有反馈的则标记为“断开”。

### 6.5 chunk状态管理流程
chunk检测线程会定时检测chunk列表中的项，首先会检查DS的状态，如果chunk对应的DS“断开”，则chunk标记为“断开”，对DS发送check_chunk命令，根据DS的反馈修改chunk项的状态，包括“正常”，“丢失”，“损坏”。

### 6.6 异常处理
1. 文件加入时没有成功commit时，定时对队列检测，过期的项目会清除。
2. 检查DS和chunk的状态时，当没有反馈时，使用重发，多次没有反馈标记为“断开”。

## 7. DS流程
### 7.1 启动流程
1. 命令行参数的处理，包括配置文件路径，IP和端口，开关等；
2. 配置文件的加载和修改，包括MDS的IP和端口号，目录等；
3. 生成数据管理后台实例，进行如下操作：
    1. 全局变量的设置，
    2. 向MDS注册（报告自己的存储信息），
    3. 启动命令监听线程，
4. 循环处理。

### 7.2 与Client交互流程
通过命令监听线程，处理Client的文件的写/读操作。

### 7.3.与MDS交互流程
通过命令监听线程，处理MDS的状态检测和chunk信息检测命令。

### 7.4 chunk校验
文件写入时会在写入前后加chunk的校验，校验保存成一个和chunk对应的小文件，需要时用来做校验比对。

### 7.5 异常处理
1. 读操作异常，数据传输，磁盘操作等等，出错时返回错误。
2. 定操作异常，出错时返回错误。

## 8. Client流程
### 8.1 用户API
用户使用API方式调用Client的功能，后期也会做一个文件系统的简单shell，实现功能的调用。

### 8.2 与MDS交互
1. 目录操作，client直接和MDS进行一次交互，读/改元数据；
2. 文件写入使用多断提交，先获得文件ID，再获得文件的object列表，再commit元数据到数据库；
3. 文件读取时先读取文件，再获得文件object和chunk信息；
4. 文件删除时直接删除所有元数据；
5. 文件解码读时可以向MDS发repair_chunk的请求，修复chunk。

### 8.3 与DS交互
和DS进行chunk的写入操作，读取时提供编码要求的数据片读取支持。

### 8.4 编码部分
根据编码类型对object编码成多个chunk（块）组成的stripe(条带)写入到DS中，中间加入校验的选项。读取文件时，如果时解码读时，要对chunk进行解码恢复成object。

### 8.5 异常处理
1. Client读取chunk出错时，如果是网络问题，会重发读请求，如果是chunk读取失败，会直接报错。
2. Clinet写入失败，会向MDS发add_file_failed命令，元数据删除，已经写入的chunk不管，然后返回错误类型。

## 9. 测试计划
### 9.1 单元测试
系统模块要支持单元测试，使用一定的单元测试框架。

### 9.2 功能测试
系统功能要支持自动化功能测试，可以使用脚本或者相关测试框架。

### 9.3 性能测试
编写性能测试程序和自动化测试脚本。
