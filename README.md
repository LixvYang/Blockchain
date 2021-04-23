# 我用160行代码写出了个区块链...

![](https://pic.editoe.com/b1e68981168aed2b536ac06deddedc53db2fd6f38d8561e4a914247b173901c6.svg)

完成本篇教程，你将会做出一条属于自己的区块链系统

你可以在自己的浏览器中显示自己的区块链系统，类似于下图所示

![](https://pic.editoe.com/f1362f7d18873f3edaab827cc966ff75be5cdf3feb7832d0e3b5cec0ed5125ba.png)

#### 总览

很多人认识区块链是因为比特币，结实比特币也是缘于它攀升的价格，但作为技术人员，理应了解其本质。

区块链不仅仅是计算机科学，还涉及了政治经济制度，社会分工协作等等很多方面，因此我的关注点不仅在于深度，更在于其广度，更多是站在研究的角度。区块链是21世纪最具革命性的技术之一，而且这项技术还尚在发展中，仍然有很多潜力未曾展现。

在这篇文章通过160行代码使用Go语言编写自己的简单区块链，最后在web浏览器中可以打开来查看自己区块链。

在这篇文章中你可以学到

*   创建自己的区块
*   为每个区块添加哈希码
*   为区块链提供Web服务
*   ......

为了使教程简便易懂，我们使用web服务替代P2P网络，因此我们可以在浏览器中查看新添加的block。

首先，确保你下载了Go语言安装包。再下载下面的三个包：

`go get github.com/davecgh/go-spew/spew`

`go get github.com/gorilla/mux`

`go get github.com/joho/godotenv`

首先第一个包，`spew`，可以让我们在控制台查看`struct`和`slices`理论上`log`包和`fmt`包也可以查看这些信息，但spew可以使结构更加清晰化。

第二个包，`mux`用于处理web服务的包，比起`http`包`mux`可以使web服务更加简便，最近也流行`gin`框架做web服务，大家也可以去试一试。

第三个包`godotenv`，从包的名字就可以看出，go do env，可以读取同一个目录下的`env`文件，这样子我们就无需对HTTP端口之类的内容进行编码。

#### 开始

首先新建一个文件夹，在文件夹中创建一个`main.go`文件。根据第三个包`godoenv`，我们再创建一个`.env`文件。只需向此文件添加一行：

`PORT=8080`

意味着我们开启在main写入代码中的web服务监听8080端口。

好了，接下来的代码我们都将在`main.go`文件中构建。

首先的首先，我们先想一想我们想要构建一个区块链，我们到底需要一些什么？

*   [ ] **Block**
*   [ ] **Blockchain**
*   [ ] **Data**
*   [ ] **test数据**
*   [ ] **web服务**

在写代码之前进行一次构建对于整个编码过程都很重要！

所以我为以上问题做了一个图示，展示我们整个区块链架构的核心过程。

#### 总代码架构图示

![](https://pic.editoe.com/56952cc00363f1ff6aa1eb3d225fa9330d81639fb7e7730b3a74ad2aa97048da.svg)

根据函数名称，应该可以大概了解函数的主要作用

*   左侧的Block、Blockchain、Message就是我们所需要的定义组成区块链的每个块的结构。
*   `generateBlock`的作用是初始化每一个区块，`caculateHash`的作用是为每一个区块计算hash码。`isBlockValid`去校验我们构建的区块链是否正确。
*   `makeMuxRouter`启用Web服务，GET和POST我们的Data值。接下来`run`运行Web服务。

首先我们需要导入所需要的包，很简单

```go
package main

import (
	"crypto/sha256"
	"encoding/hex"
	"encoding/json"
	"io"
	"log"
	"net/http"
	"os"
	"strconv"
	"sync"
	"time"

	"github.com/davecgh/go-spew/spew"
	"github.com/gorilla/mux"
	"github.com/joho/godotenv"
)
```

#### 如何了解一个package

介绍一下如何去了解一个包吧。

很多人想要去了解一个包就先去别人的博客，自己在百度上随便找一个链接就以为自己吃透了包的内容。

但实际上，了解一个包最应该查看的资料就是官方资料，而且推荐是英文版，因为别人翻译过的资料，很可能有信息缺失，信息传递过程中有些信息就不见了，就好像吃别人吃剩的饭一样，难受死人了。

比如我们需要了解下面这个包的内容`crypto/sha256`**我们首先应该去**[**官网**](https://golang.org/pkg/)**了解，**首先打开Golang的官网，打开上方的`packages`你可以看到很多包，按`ctrl + F`去直接搜索sha256，你就找到了需要的包。

每个包都有Overview,Index和Examples，这个包里的内容是：  
Package sha256 implements the SHA224 and SHA256 hash algorithms as defined in FIPS 180-4.

翻译过来的意思是：

sha256包实现了FIPS 180-4中定义的SHA224和sha256哈希算法。

在Index内容中可以查看包内的函数，Examples中含有应用例子，比如果我们需要创建一个哈希码：

点击New案例，你就会发现有一个Run按钮，直接运行，就会运行出现应有的哈希码。

![](https://pic.editoe.com/a09ac38d16a05bb39a302920b71ce8b33961cffdc686696450ca1ccd5da24186.png)

以上就是大概了解一个包的内容，如果你想真正学会本篇的区块链教程，请你一定要自己查看每个包内的具体内容，这样再结合具体的代码实战，会让你对包的理解和使用更加流畅。  
然后，我们需要一个`Block`用来写我们的每一个区块。

我们每一个区块中含有Index,Data,Timestamp,Hash,Prehhash。

*   Index是指索引，从0开始递增。
*   Data就是我们要传递的数据，
*   Timestamp是时间戳，记录我们提交Data所用的时间，
*   Hash是表示此数据记录的sha256标识符
*   Prehash记录上个块所构建哈希码，用sha256包创建哈希码。

#### 数据模型

在main中Copy and paste以下代码

```go
type Block struct {
	Index     int
	Timestamp string
	Data      int
	Hash      string
	PrevHash  string
}
```

这样我们的每一个区块就构成了。

这里你可能有一个疑惑：哈希是如何去识别区块和区块链呢？

答案是：哈希使用散列来识别和保持块的正确顺序。通过确保它们`PrevHash`中的每个`Block`与`Hash`前面的相同，就像下图所示的一样。

![](https://pic.editoe.com/f2426510c123b18d525efed3c3126fed0664dcee0809b643a45d19f5991bfd7d.png)

这样`Block`我们知道组成链的块的正确顺序。

但我们每次直接使用Block会很麻烦，所以我们用变量`Blockchain`来构建我们的类型Block，这样在以后使用增加区块`append`时就很容易构建了。

再声明一个变量互斥锁，调用时就不需要在函数体内部再进行重新声明了。

还有需要定义一个类型是Message，用于提交Data时，便于解构代码。

```go
var Blockchain []Block
var mutex = &sync.Mutex{}

type Message struct {
	Data int
}
```

#### 生成哈希值和新区块

下面让我们写一个函数来获取我们的Block数据并创建一个SHA256哈希值。

```go
//sha256 do a hash code for every Block
func caculateHash(block Block) string {
	record_block := strconv.Itoa(block.Index) + block.Timestamp + strconv.Itoa(block.Data) + block.PrevHash
	h := sha256.New()
	h.Write([]byte(record_block))
	return hex.EncodeToString(h.Sum(nil))
}
```

这个calculateHash函数将我们提供的块作为参数的索引、时间戳、Data和PrevHash连接起来，并以字符串的形式返回SHA256散列。我们在此函数中使用了`strconv`包，这个包的作用是方便我们转换格式，比如Itoa函数的格式是将Int格式转换为String格式，虽然很简单，但还是建议去官网查看一下包的内容。

很简单的代码，4-6行代码就是我们之前在官网中所看到的代码，也就是上面图片中的代码。

![](https://pic.editoe.com/a09ac38d16a05bb39a302920b71ce8b33961cffdc686696450ca1ccd5da24186.png)

现在，我们可以用一个新的generateBlock函数生成一个新的Block，其中包含我们需要的所有元素。

```go
func generateBlock(oldBlock Block, Data int) Block {
	var newBlock Block

	nowtime := time.Now()

	newBlock.Index = oldBlock.Index + 1
	newBlock.Timestamp = nowtime.String()
	newBlock.Data = Data
	newBlock.PrevHash = oldBlock.Hash
	newBlock.Hash = caculateHash(newBlock)

	return newBlock
}
```

我们使用time包中的Now函数来表示创建`newBlock`的时间。还要注意，调用了先前的`calculateHash`函数。`PrevHash`是从前一个块的hash复制过来的。`Index`从前一个块的索引中递增。

到此为止我们已经成功声明了一个区块，并且为它计算hash值。

#### 数据核实

现在我们需要编写一些函数来确保块没有被篡改。我们通过检查`Index`来确保它们像预期的那样递增。我们还检查以确保我们的`Prehash`确实与前一个块的`Hash`相同。最后，我们希望通过在当前块上再次运行`calculateHash`函数来再次检查当前块的散列。让我们写一个`isBlockValid`函数来做所有这些事情并返回一个`bool`值,如果它通过了我们所有的检查，它将返回true。

```go
func isBlockValid(newBlock, oldBlock Block) bool {
	if oldBlock.Index+1 != newBlock.Index {
		return false
	} else if oldBlock.Hash != newBlock.PrevHash {
		return false
	} else if caculateHash(newBlock) != newBlock.Hash {
		return false
	} else {
		return true
	}
}
```

现在，我们已经完成了构建区块链的大部分工作，现在我们想要一种方便的方式来查看我们的区块链并写入它，最好是在一个web浏览器中，这样我们就可以直观的展示我们的每一个区块的内容。

#### Web服务

如果你还不了解Go语言是如何启用Web服务，你可以先去Go语言的`net/http`包的官网去先了解一下，如果不了解也没关系，我会用很直白的话讲明白。

我们使用`mux`包来帮助我们构建Web服务，去mux的github官网，可以看到一下信息，帮助我们简单地构建一下web服务。

![](https://pic.editoe.com/a3c4caabcb724d16952de00fe580158e8f8dc39cd109f6dbf5ff109987587389.png)

![](https://pic.editoe.com/4ba3f6441e60288d8f44f41040df1a3fa42e9b3bc066c5bf142fa6959c5f2f80.png)

所以我们很简单调用一下函数（之后添加），来方便我们构建GET和POST方法。

```go
//create web service
func makeMuxRouter() http.Handler {
	muxRouter := mux.NewRouter()
	muxRouter.HandleFunc("/", handleGetBlockchain).Methods("GET")
	muxRouter.HandleFunc("/", handleWriteBlockchain).Methods("POST")
	return muxRouter
}
```

这是我们的GET函数

```go
//when receive Http request we write blockchain
func handleGetBlockchain(w http.ResponseWriter, r *http.Request) {
	json, err := json.MarshalIndent(Blockchain, "", "  ")
	if err != nil {
		log.Fatal()
	}
	io.WriteString(w, string(json))
}
```

我们去json包的官网可以看到

![](https://pic.editoe.com/c89bf94f97ed22f64f1c54b2bbabe86ca41a581702059518867988cf9d9cf0ea.png)

MarshalIndent函数可以应用缩进来格式化输出。输出中的每个JSON元素将以新行开始，以前缀开头，然后根据缩进嵌套，后跟一个或多个缩进副本。

当然还有io包内的函数，需要你自己去官网中查看。

这是我们的POST函数

```go
func handleWriteBlockchain(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")

	var msg Message

	decoder := json.NewDecoder(r.Body)
	if err := decoder.Decode(&msg); err != nil {
		respondWithJSON(w, r, http.StatusBadRequest, r.Body)
		return
	}

	defer r.Body.Close()
	mutex.Lock()
	prevBlock := Blockchain[len(Blockchain)-1]
	newBlock := generateBlock(prevBlock, msg.Data)

	if isBlockValid(newBlock, prevBlock) {
		Blockchain = append(Blockchain, newBlock)
		spew.Dump(Blockchain)
	}
	mutex.Unlock()

	respondWithJSON(w, r, http.StatusCreated, newBlock)
}
```

现在你明白为什么要将Message单独作为一个结构了吗？我们使用单独的msg结构的原因是为了接收JSON POST请求的请求体，我们将使用该请求体来编写新的块。这允许我们简单地发送一个带有以下主体的POST请求，我们的处理程序将为我们填充块的其余部分。

在将请求体重新解码为var msg消息结构之后，我们通过将之前的块和新的Data传递给前面编写的generateBlock函数来创建一个新的块。这就是函数创建新块所需要的一切。我们使用前面创建的isBlockValid函数进行快速检查，以确保新块是符合要求的。

你还会发现我们使用了mutex的互斥锁内容，因为当添加新块时，是不允许其他函数进行访问的。

*   `spew.Dump`方便我们打印结构体到调试窗口
*   POST请求可以使用`curl`和`postman`工具进行调试

当我们的POST请求成功或失败时，我们希望得到相应的通知。我们使用一个小小的包装器函数respondWithJSON来让我们知道发生了什么。

```go
func respondWithJSON(w http.ResponseWriter, r *http.Request, code int, payload interface{}) {
	response, err := json.MarshalIndent(payload, "", "  ")
	if err != nil {
		w.WriteHeader(http.StatusInternalServerError)
		w.Write([]byte("HTTP 500: Internal Server Error"))
		return
	}
	w.WriteHeader(code)
	w.Write(response)
}
```

最后运行服务

```go
//run Http serve
func run() error {
	myHandler := makeMuxRouter()
	httpPort := os.Getenv("PORT")
	log.Println("HTTP Server Listening on port :", httpPort)
	s := &http.Server{
		Addr:           ":8080",
		Handler:        myHandler,
		ReadTimeout:    10 * time.Second,
		WriteTimeout:   10 * time.Second,
		MaxHeaderBytes: 1 << 20,
	}
	log.Fatal(s.ListenAndServe())

	return nil
}
```

在run函数中，也有我们需要了解的一下新知识比如http包的内容

![](https://pic.editoe.com/3cddc4924e5d831cc381988da6f8a73b8dbfd8b50718a1acd407f92508cdb497.png)

大家可以根据之前我带大家去了解新包的方式去了解`os`包的内容

#### 差不多完事了

最后是我们的main函数

让我们把所有这些不同的区块链函数、web处理程序和web服务器连接在一个简短的main函数中

```go
//main func
func main() {
	err := godotenv.Load()
	if err != nil {
		log.Fatal(err)
	}

	go func() {
		t := time.Now()
		genesisBlock := Block{}
		genesisBlock = Block{0, t.String(), 0, caculateHash(genesisBlock), ""}
		spew.Dump(genesisBlock)

		mutex.Lock()
		Blockchain = append(Blockchain, genesisBlock)
		mutex.Unlock()
	}()

	log.Fatal(run())
}
```

看看main函数做了什么。

还记得我们的.env文件吗？2-6行，就是编码.env文件，方便我们监听8080端口。

7-16行，同时我们用并行方式运行匿名函数，genesisBlock是最重要的主要功能部分。我们需要给区块链提供一个初始的块，否则一个新的块将无法与它之前的哈希值进行比较，因为之前的哈希值不存在。

18行，最后运行web服务。

最后我们的区块链就构建完成了，我数了数也就159行。

但足够你用好几天时间去消化内部的知识了。

我们来运行一下试试看:-)

在终端启用`go run main.go`

我们看到web服务器已经启动并运行

![](https://pic.editoe.com/33f5099a858433081583f11af1c4fcc2f2c5bfa7decdcfe7132bef6aa2344456.png)

打开浏览器访问`localhost:8080`我们看到了相同的创世区块。

![](https://pic.editoe.com/a876763dee220d65f05cbdc33903b4572ae3fa73446b31d563cba63acdaaf0de.png)

接下来我通过postman工具进行POST`{"Data":150}`请求

![](https://pic.editoe.com/e663ec2b846cefd3021d7565d66cbb8e6bb1d5bd066319e2724e402473182f87.png)

多试几个看看？

在浏览器中访问

![](https://pic.editoe.com/f1362f7d18873f3edaab827cc966ff75be5cdf3feb7832d0e3b5cec0ed5125ba.png)

你已经完成本篇文章的全部内容了，可能你已经看完了整篇文章不知云里雾里，但我还是希望你能够亲手将上述所有的代码全部自我实现一遍。

欢迎点赞，关注哦！