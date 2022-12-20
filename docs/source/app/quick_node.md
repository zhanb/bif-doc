



# QuickNode

为更好的服务星火生态合作伙伴，加速星火生态的开放进程，星火链网制定了主链测试网准入新机制，并开发了测试网节点Quicknode的镜像包。开发者通过Docker部署节点镜像包，即可在本地并参与维护测试网络正常运行，而无需调用RPC接入测试网，实现了节点的自主可控。

![image-20221219205857431](../images/image-20221219205857431.png)

##  Quicknode节点部署

### 环境要求

```sh
部署最小硬件要求：
内存：8G
硬盘：100G
cpu：8核
```

### 1. 获取镜像版本号

用浏览器打开链接 http://test.bifcore.bitfactory.cn/hello
返回如下结果：

```json
{
	"address_prefix": "did:bid:",
	"chain_code": "",
	"chain_version": "1.9.0-8",
	"current_time": "2022-12-19 20:37:56.105204",
	"hash_type": 0,
	"ledger_version": "1004",
	"license_version": "1.1.0",
	"monitor_version": "1000",
	"network_id": 16234539267878,
	"overlay_version": "1000",
	"websocket_port": 7053
}
```

chain_version字段的值即为测试网链节点最新版本号。

### 2. 获取镜像

```shell
#docker pull 待部署镜像(前提是确保主机有docker可用，并能上外网)
docker pull caictdevelop/bif-core:v${chain_version}
#说明：其中冒号后面代表底层链的版本号，如第一步chain_version为1.9.0-8，则这一步的命令为：
#docker pull caictdevelop/bif-core:v1.9.0-8
#获取的阿拉伯数字组成越大代表版本号相较更新的版本，版本号递增更新
```

如果新机器上没有启动过docker服务会报如下错误：
<img src="../_static/images/2022-08-01-11-15-04.png" alt="2022-08-01-11-15-04.png"  />

执行命令：
```sh
service docker start
```

重新拉取镜像：
  <img src="../_static/images/2022-08-01-11-16-45.png" alt="2022-08-01-11-16-45.png"  />

### 3. 启动镜像，进入容器

- 执行docker images查看拉取的镜像`IMAGE ID`
  <img src="../_static/images/2022-08-01-11-17-49.png"/>

- 启动Quicknode服务

  ```sh
  # IMAGEID即上述查看的具体字段值
  docker run -itd -p 27002:27002 {IMAGEID} /bin/bash
  ```
  执行结果如下：
  <img src="../_static/images/2022-07-29-17-38-24.png"/>

- 查询启动的docker镜像进程信息,获取`container ID`
  <img src="../_static/images/2022-07-29-17-42-39.png"/>

- exec进入容器系统启动bif服务

  ```sh
  # 2657705f9199 即是上述查询到的container ID
  docker exec -it 2657705f9199 /bin/bash
  ```
  <img src="../_static/images/2022-07-29-17-43-41.png"/>

- 进入容器系统当前目录即是bifchain底层链目录 给可执行程序添加权限执行

  ```shell
  chmod +x bin/*
  nohup ./bin/bif &
  ```
  <img src="../_static/images/2022-07-29-17-48-27.png"/>

### 4. 查看节点进程是否启动

在镜像系统中执行上述命令后查看bif服务，如果查询不到进程，执行exit命令在宿主机再执行 **启动Quicknode服务**操作

```shell
ps aux |grep -v grep |grep bif
```

<img src="../_static/images/2022-07-29-18-00-41.png"/>

### 5. 查看节点同步高度

快速部署的节点会自动通过p2p和测试网其他节点链接，部署启动后先同步其他节点数据，跟据数据量和磁盘不同时间不同，同步完所有的数据大概需要几个小时，可以根据如下方式查询测试网以及部署后节点的区块高度，实时观察同步进度。

- 查询测试网节点区块高度url和结果如下：

  ```http
  http请求方式：GET
  http://test.bifcore.bitfactory.cn/getLedger
  ```

  响应报文：

  ```json
  {
  	error_code: 0,
  	result: {
  		header: {
  			account_tree_hash: "4c9a11f712331f5815a24dbab5fee7cff905953dee71e48549ddac00d3648372",
  			close_time: 1658995885587070,
  			consensus_value_hash: "dba4e31ce39a1297fd24478d5da58687a67a56275da01aa95cf6535f57d3e9a7",
  			fees_hash: "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
  			hash: "162beecd20a70c5dc6794e66981448466f48fc0e377a63118199cd39b52bc293",
  			previous_hash: "dd5130951b1982c55d4e52ce379d882465fe322a5f6881b91016f2272acd3d06",
  			seq: 1172057,   //seq即是测试网的区块高度值
  			tx_count: 797679,
  			validators_hash: "b8ebfb79b0aed24cd9122c4545c88b8f9c7c6c7b01d1f0f55f2d3c036064eefd",
  			version: 1003
  		},
  		ledger_length: 227
  	}
  }
  ```

- 同步查询Quicknode节点高度

  访问快速节点高度,host是主机ip(镜像映射到宿主机了)，port即部署时的`27002`

  ```http
  http请求方式：GET
  http://{host}:{port}/getLedger
  ```

  获取到响应报文，查看`seq`字段的值，高度一致即全部同步完成。