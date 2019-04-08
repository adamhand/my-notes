# tendermint
---
# 简介
Tendermint是一个开源的完整的区块链实现，可以用于公链或联盟链，其官方定位是面向开发者的区块链共识引擎。尽管tendermint包含了区块链的完整实现，但它却是以SDK的形式将这些核心功能提供出来，供开发者方便地定制自己的专有区块链。

Tendermint 包含了两个主要的技术组件

- Tendermint Core。共识引擎部分。包括加密算法、共识算法、区块链存储、RPC接口、P2P通信等。
- ABCI(Applacation Blockchain Interface)。应用程序区块链接口。暴露给开发人员的接口，可以调用它开发自己的区块链。语言无关。

Tendermint Core通过一个满足 ABCI 标准的 socket 协议与应用进行交流。此外，如果需要使用Tendermint开发区块链系统，还要实现client部分。所以，一个完整的基于tendermint的区块链系统包括三个部分：client部分，ABCI部分和tendermint core部分。如下图所示：

<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/tendermint.jpg">
</center>

- ABCI：位于区块链内部，ABCI Server，启动后加载 ABCI Application，为 Tendermint 提供 Socket 服务，默认服务地址：tcp://localhost:26658
- Tendermint core：位于区块链内部，ABCI Client/Tendermint Core，启动后与 ABCI 提供的 Socket 服务创建 3 条连接（Connect），同时会提供 HTTP 服务给区块链外部客户端
- Client：区块链外部，真正的客户端，通过 Tendermint 提供的服务对区块链进行读写访问


## Tendermint Core
tendermint采用的共识机制属于一种权益证明（ Proof Of Stake）算法，一组验证人（Validator）代替了矿工（Miner）的角色，依据抵押的权益比例轮流出块。

tendermint同时是拜占庭容错的（Byzantine Fault Tolerance），因此对于3f+1个验证节点组成的区块链，即使有f个节点出现拜占庭错误，也可以保证全局正确共识的达成。

下图是tendermint的状态机。

<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/tendermint_1.png">
</center>

“验证人”（validator）轮流对交易区块进行提议，并对这些区块进行投票。区块会被提交到链上，每一个块占据一个“高度”（height）。

如果提交失败，协议就会开始下一轮的提交，并且一个新的验证人会继续提交那个高度的区块。

## ABCI
ABCI 包含了 3 个主要的消息类型，它们由tendermint core 发送至应用，应用会对消息产生相应的回复。

- **DeliverTx** 消息是应用的主要部分。**链中的每笔交易都通过这个消息进行传送**。应用需要基于当前状态，应用协议，和交易的加密证书上，去验证接收到 DeliverTx 消息的每笔交易。
- **CheckTx** 消息类似于 DeliverTx，但是它**仅用于验证交易**。Tendermint Core 的内存池首先通过 CheckTx 检验一笔交易的有效性，并且只将有效交易中继到其他节点。比如，一个应用可能会检查在交易中不断增长的序列号，如果序列号过时，CheckTx 就会返回一个错误。又或者，他们可能使用一个基于容量的系统，该系统需要对每笔交易重新更新容量。 
- **Commit** 消息用于计算当前应用状态的一个加密保证（cryptographic commitment），这个加密保证会被放到下一个区块头。

一个应用可能有多个 ABCI socket 连接。Tendermint Core 给应用创建了三个 ABCI 连接：

- Mempoll Connection：用于内存池广播时的交易验证，CheckTx
- Consensus Connection：用于运行提交区块时的共识引擎，InitChain，BeginBlock，DeliverTx，EndBlock，Commit
- Info Connection。用于查询应用状态。Info，SetOption，Query

<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/tendermint_2.png">
</center>

---

[tendermint docs](https://tendermint.com/docs/)

---

# KVStore
KVStore是tendermint提供的一个例子，用来说明abci-cli client(模拟tendermint core)和abci应用服务端之间通过socket进行通信的过程。

## 源码分析
程序入口在tendermint\abci\cmd\abci-cli\main.go中的Execute()方法。这个方法主要做了三件事：

- addGlobalFlags()。
- addCommands()
- RootCmd.Execute()

### addGlobalFlags()
注册全局 Flags，主要包括：

- ABCI 应用服务端监听地址 flagAddress ，默认为：tcp://0.0.0.0:26658
- abci-cli 客户端与 ABCI 应用服务端的通信协议 flagAbci，默认为：socket
- 选择是否将命令和结果在控制行进行打印。选择的是不打印。
- 日志等级 flagLogLevel，默认为：debug

程序如下所示：
```go
func addGlobalFlags() {
	RootCmd.PersistentFlags().StringVarP(&flagAddress, "address", "", "tcp://0.0.0.0:26658", "address of application socket")
	RootCmd.PersistentFlags().StringVarP(&flagAbci, "abci", "", "socket", "either socket or grpc")
	RootCmd.PersistentFlags().BoolVarP(&flagVerbose, "verbose", "v", false, "print the command and results as if it were a console session")
	RootCmd.PersistentFlags().StringVarP(&flagLogLevel, "log_level", "", "debug", "set the logger level")
}
```

### addCommands()
添加子命令到RootCmd命令，包括kvstore 和 dummy 以及 echoCmd、infoCmd、deliverTxCmd 和 commitCmd 等客户端命令。这些命令会在后面被触发执行。
```
func addCommands() {
	RootCmd.AddCommand(batchCmd)
	RootCmd.AddCommand(consoleCmd)
	RootCmd.AddCommand(echoCmd)
	RootCmd.AddCommand(infoCmd)
	RootCmd.AddCommand(setOptionCmd)
	RootCmd.AddCommand(deliverTxCmd)
	RootCmd.AddCommand(checkTxCmd)
	RootCmd.AddCommand(commitCmd)
	RootCmd.AddCommand(versionCmd)
	RootCmd.AddCommand(testCmd)
	addQueryFlags()
	RootCmd.AddCommand(queryCmd)

	// examples
	addCounterFlags()
	RootCmd.AddCommand(counterCmd)
	addKVStoreFlags()
	RootCmd.AddCommand(kvstoreCmd)
}
```
比如KVStore命令如下：
```go
var kvstoreCmd = &cobra.Command{
	Use:   "kvstore",
	Short: "ABCI demo example",
	Long:  "ABCI demo example",
	Args:  cobra.ExactArgs(0),
	RunE: func(cmd *cobra.Command, args []string) error {
		return cmdKVStore(cmd, args)
	},
}
```
当运行`abci-cli kdstore`的时候，就会执行这个命令，触发`cmdKVStore(cmd, args)`函数。这个函数具体会在后面建立abci服务端的时候分析。

### RootCmd.Execute()

#### 启动abci-cli
这个函数的作用是执行上面添加的命令，但是在执行之前首先会执行RootCmd中的PersistentPreRunE函数。从名字可以看出，这个函数是`PreRun`的，也就是要先执行。这个函数中比较重要的操作如下：
```go
if client == nil {
	var err error
	client, err = abcicli.NewClient(flagAddress, flagAbci, false)
	if err != nil {
		return err
	}
	client.SetLogger(logger.With("module", "abci-client"))
	if err := client.Start(); err != nil {
		return err
	}
}
```
这段代码的意思是通过NewClient()函数创建abci-client。NewClient()函数有两个重要参数：

- flagAddress。客户端要连接的地址，即服务端的监听地址，前面说了，地址为tcp://0.0.0.0:26658
- flagAbci。连接方式，有socket方式和grpc方式，前面的命令配置为socket

NewClient()函数逻辑如下：

```go
func NewClient(addr, transport string, mustConnect bool) (client Client, err error) {
	switch transport {
	case "socket":
		client = NewSocketClient(addr, mustConnect)
	case "grpc":
		client = NewGRPCClient(addr, mustConnect)
	default:
		err = fmt.Errorf("Unknown abci transport %s", transport)
	}
	return
}
```
因为选择的是socket，所以会进入NewSocketClient(addr, mustConnect)函数，该函数的逻辑如下：
```
func NewSocketClient(addr string, mustConnect bool) *socketClient {
	cli := &socketClient{
		reqQueue:    make(chan *ReqRes, reqQueueSize),
		flushTimer:  cmn.NewThrottleTimer("socketClient", flushThrottleMS),
		mustConnect: mustConnect,

		addr:    addr,
		reqSent: list.New(),
		resCb:   nil,
	}
	cli.BaseService = *cmn.NewBaseService(nil, "socketClient", cli)
	return cli
}
```
函数首先创建一个socketClient的结构cli，然后将其作为参数传递给cmn.NewBaseService。先看一下socketClient。
```
type socketClient struct {
	cmn.BaseService

	reqQueue    chan *ReqRes
	flushTimer  *cmn.ThrottleTimer
	mustConnect bool

	mtx     sync.Mutex
	addr    string
	conn    net.Conn
	err     error
	reqSent *list.List
	resCb   func(*types.Request, *types.Response) // listens to all callbacks

}
```
这个结构实现了abci/client/Client的所有方法，所以它实现了abci/client/Client接口。同时它还有一个匿名字段cmn.BaseService，BaseService实现了Service接口，所以socketClient也间接实现了 libs/common/service/Service 接口。能够使用其中的Start()方法。
```
func (bs *BaseService) Start() error {
	if atomic.CompareAndSwapUint32(&bs.started, 0, 1) {
		if atomic.LoadUint32(&bs.stopped) == 1 {
			bs.Logger.Error(fmt.Sprintf("Not starting %v -- already stopped", bs.name), "impl", bs.impl)
			// revert flag
			atomic.StoreUint32(&bs.started, 0)
			return ErrAlreadyStopped
		}
		bs.Logger.Info(fmt.Sprintf("Starting %v", bs.name), "impl", bs.impl)
		err := bs.impl.OnStart()
		if err != nil {
			// revert flag
			atomic.StoreUint32(&bs.started, 0)
			return err
		}
		return nil
	}
	bs.Logger.Debug(fmt.Sprintf("Not starting %v -- already started", bs.name), "impl", bs.impl)
	return ErrAlreadyStarted
}
```
而Start()方法会调用BaseService中的OnStart()方法，这个方法中主要做的就是连接abci服务端并启动两个协程：go cli.sendRequestsRoutine(conn)、go cli.recvResponseRoutine(conn)：
```
func (cli *socketClient) OnStart() error {
	if err := cli.BaseService.OnStart(); err != nil {
		return err
	}

	var err error
	var conn net.Conn
RETRY_LOOP:
	for {
		conn, err = cmn.Connect(cli.addr)
		if err != nil {
			if cli.mustConnect {
				return err
			}
			cli.Logger.Error(fmt.Sprintf("abci.socketClient failed to connect to %v.  Retrying...", cli.addr), "err", err)
			time.Sleep(time.Second * dialRetryIntervalSeconds)
			continue RETRY_LOOP
		}
		cli.conn = conn

		go cli.sendRequestsRoutine(conn)
		go cli.recvResponseRoutine(conn)

		return nil
	}
}
```
sendRequestsRoutine(conn)用来向cli服务器发送请求，recvResponseRoutine()用来处理响应结果。sendRequestsRoutine()逻辑如下，在这个函数中reqres会等待cli.reqQueue信道传来消息，之后会先写入缓冲区中，然后在发送给cliserver。
```
func (cli *socketClient) sendRequestsRoutine(conn net.Conn) {

	w := bufio.NewWriter(conn)
	for {
		select {
		case <-cli.flushTimer.Ch:
			select {
			case cli.reqQueue <- NewReqRes(types.ToRequestFlush()):
			default:
				// Probably will fill the buffer, or retry later.
			}
		case <-cli.Quit():
			return
		case reqres := <-cli.reqQueue:
			cli.willSendReq(reqres)
			err := types.WriteMessage(reqres.Request, w)
			if err != nil {
				cli.StopForError(fmt.Errorf("Error writing msg: %v", err))
				return
			}
			// cli.Logger.Debug("Sent request", "requestType", reflect.TypeOf(reqres.Request), "request", reqres.Request)
			if _, ok := reqres.Request.Value.(*types.Request_Flush); ok {
				err = w.Flush()
				if err != nil {
					cli.StopForError(fmt.Errorf("Error flushing writer: %v", err))
					return
				}
			}
		}
	}
}
```
recvResponseRoutine处理应答的逻辑如下。先从连接中读取应答，出错的话就关闭连接，否则就调用didRecvResponse()处理应答。
```
func (cli *socketClient) recvResponseRoutine(conn net.Conn) {

	r := bufio.NewReader(conn) // Buffer reads
	for {
		var res = &types.Response{}
		err := types.ReadMessage(r, res)
		if err != nil {
			cli.StopForError(err)
			return
		}
		switch r := res.Value.(type) {
		case *types.Response_Exception:
			// XXX After setting cli.err, release waiters (e.g. reqres.Done())
			cli.StopForError(errors.New(r.Exception.Error))
			return
		default:
			// cli.Logger.Debug("Received response", "responseType", reflect.TypeOf(res), "response", res)
			err := cli.didRecvResponse(res)
			if err != nil {
				cli.StopForError(err)
				return
			}
		}
	}
}
```
```
func (cli *socketClient) didRecvResponse(res *types.Response) error {
	cli.mtx.Lock()
	defer cli.mtx.Unlock()

	// Get the first ReqRes
	next := cli.reqSent.Front()
	if next == nil {
		return fmt.Errorf("Unexpected result type %v when nothing expected", reflect.TypeOf(res.Value))
	}
	reqres := next.Value.(*ReqRes)
	if !resMatchesReq(reqres.Request, res) {
		return fmt.Errorf("Unexpected result type %v when response to %v expected",
			reflect.TypeOf(res.Value), reflect.TypeOf(reqres.Request.Value))
	}

	reqres.Response = res    // Set response
	reqres.Done()            // Release waiters
	cli.reqSent.Remove(next) // Pop first item from linked list

	// Notify reqRes listener if set
	if cb := reqres.GetCallback(); cb != nil {
		cb(res)
	}

	// Notify client listener if set
	if cli.resCb != nil {
		cli.resCb(reqres.Request, res)
	}

	return nil
}
```

这样abci-client就启动了。

#### 启动abci服务端
前面说了，当运行abci-cli kdstore的时候，会触发cmdKVStore(cmd, args)函数，这个函数的逻辑如下：
```
func cmdKVStore(cmd *cobra.Command, args []string) error {
	logger := log.NewTMLogger(log.NewSyncWriter(os.Stdout))

	// Create the application - in memory or persisted to disk
	var app types.Application
	if flagPersist == "" {
		app = kvstore.NewKVStoreApplication()
	} else {
		app = kvstore.NewPersistentKVStoreApplication(flagPersist)
		app.(*kvstore.PersistentKVStoreApplication).SetLogger(logger.With("module", "kvstore"))
	}

	// Start the listener
	srv, err := server.NewServer(flagAddress, flagAbci, app)
	if err != nil {
		return err
	}
	srv.SetLogger(logger.With("module", "abci-server"))
	if err := srv.Start(); err != nil {
		return err
	}

	// Wait forever
	cmn.TrapSignal(func() {
		// Cleanup
		srv.Stop()
	})
	return nil
}
```
这个函数主要做了两件事：

- 新建一个KVStoreApplication。app = kvstore.NewKVStoreApplication()
- 新建Server

NewKVStoreApplication()逻辑如下，主要是从内存存储 MemDB 结构中获取对应状态，app会以参数的形式传递给NewServer函数用来创建Server。
```
func NewKVStoreApplication() *KVStoreApplication {
	state := loadState(dbm.NewMemDB())
	return &KVStoreApplication{state: state}
}
```
NewServer()主要调用了NewSocketServer()，函数的逻辑如下：
```
func NewSocketServer(protoAddr string, app types.Application) cmn.Service {
	proto, addr := cmn.ProtocolAndAddress(protoAddr)
	s := &SocketServer{
		proto:    proto,
		addr:     addr,
		listener: nil,
		app:      app,
		conns:    make(map[int]net.Conn),
	}
	s.BaseService = *cmn.NewBaseService(nil, "ABCIServer", s)
	return s
}
```
关键点在最后一行：s.BaseService = *cmn.NewBaseService(nil, "ABCIServer", s)。BaseService是SocketServer中的一个匿名字段，BaseServcie接口又实现了Service接口，可以调用接口的Start()方法，而Start()方法又会调用OnStart()方法，如下：
```
func (s *SocketServer) OnStart() error {
	if err := s.BaseService.OnStart(); err != nil {
		return err
	}
	ln, err := net.Listen(s.proto, s.addr)
	if err != nil {
		return err
	}
	s.listener = ln
	go s.acceptConnectionsRoutine()
	return nil
}
```
关键地方在acceptConnectionsRoutine()这个协程，这个协程的作用是在连接中读取请求并向连接中写入应答，逻辑如下：
```
func (s *SocketServer) acceptConnectionsRoutine() {
	for {
		// Accept a connection
		s.Logger.Info("Waiting for new connection...")
		conn, err := s.listener.Accept()
		if err != nil {
			if !s.IsRunning() {
				return // Ignore error from listener closing.
			}
			s.Logger.Error("Failed to accept connection: " + err.Error())
			continue
		}

		s.Logger.Info("Accepted a new connection")

		connID := s.addConn(conn)

		closeConn := make(chan error, 2)              // Push to signal connection closed
		responses := make(chan *types.Response, 1000) // A channel to buffer responses

		// Read requests from conn and deal with them
		go s.handleRequests(closeConn, conn, responses)
		// Pull responses from 'responses' and write them to conn.
		go s.handleResponses(closeConn, conn, responses)

		// Wait until signal to close connection
		go s.waitForClose(closeConn, connID)
	}
}
```

## 总结
KVStore服务端启动以及和abci-cli交互的过程。

- abci-cli kvstore会启动abci 应用服务端，启动两个协程，一个处理请求，一个处理应答。同时会启动abci-cli client，也是两个协程，一个处理请求，一个处理应答。这里的abci-cli client是和tendermint 节点绑定在一起的，所以模拟的tendermint core的活动。
- 在命令行输入 abci-cli console 进入交互模式，创建客户端并与服务端建立了持久连接。
- 输入 deliver_tx "abc"，客户端会识别子命令，调用 cmdDeliverTx 函数，此函数在解析到 tx 后进行编码，随后会调用 cli.DeliverTxSync(txBytes)  (同步的) 函数。
- 请求消息写入连接后，服务端读取到此请求。根据请求类型，调用 KVStore 应用的 DeliverTx 函数进行处理。
- 服务端处理完毕后的应答写入连接，客户端接收到应答，前端打印显示到命令行。

---

[参考](https://www.jianshu.com/p/712f0287b60e)

---