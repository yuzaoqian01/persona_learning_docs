1. ### 设置一个请求baseUrl 例如rpc_url 参数params

2. ### 创建执行方法前置操作

```js
const chain = pm.iterationData.get("chain");

if (chain === "Ethereum") {
    pm.environment.set("rpc_url", "https://rpc.ankr.com/eth");
} else if (chain === "BSC") {
    pm.environment.set("rpc_url", "https://bsc-dataseed.binance.org");
} else if (chain === "Polygon") {
    pm.environment.set("rpc_url", "https://polygon-rpc.com");
} else if (chain === "Solana") {
    pm.environment.set("rpc_url", "https://api.mainnet-beta.solana.com");
} else if (chain === "Bitcoin") {
    pm.environment.set("rpc_url", "http://your-bitcoin-node-url");
} else if (chain === "Tron") {
    pm.environment.set("rpc_url", "https://api.trongrid.io/wallet/getaccount");
}
```

3. ### 创建一个csv文件

```css
chain,method,params
Ethereum,eth_getBalance,"[""0x742d35Cc6634C0532925a3b844Bc454e4438f44e"",""latest""]"
BSC,eth_getBalance,"[""0x0000000000000000000000000000000000001004"",""latest""]"
Polygon,eth_getBalance,"[""0x6dBd017Bb3b8bB0D289094DAbdBe5828aDfE61d5"",""latest""]"
Solana,getBalance,"{""jsonrpc"":""2.0"",""method"":""getBalance"",""params"":[""GdD2V...G2d""],""id"":1}"
Bitcoin,getreceivedbyaddress,"[""n2eMqTT929pb1RDNuqEnxdaLau1rxy3efi""]"
Tron,getaccount,"{""address"":""TQs2...ZZp"",""visible"":true}"
```

4. ### 创建postman runner 导入文件后运行就可以看到这个合集运行不同链的结果了

5. 常用语法

```js
tests["返回状态是否等于200"] = responseCode.code == 200;
tests["返回时间是否小于10毫秒"] = responseTime < 50


try {
    const response = await pm.sendRequest({
        url: "https://postman-echo.com/get",
        method: "GET"
    });

    console.log(response.json());
} catch (err) {
    console.error(err);
}

pm.variables.set("variable_key", "variable_value");

```

```shell
newman run evm_jsonrpc.postman_collection.json -e metamask.postman_environment.json -r html --reporter-html-export test.html
```

```js
let base_url = pm.environment.get('base_url');
let api_key = pm.environment.get('api_key');
if(base_url){
    pm.collectionVariables.set('baseUrl',base_url);
}
if(api_key){
    pm.collectionVariables.set('API_KEY', api_key);
}
pm.test("请求是否成功", function () {
    pm.response.to.have.status(200);
});
pm.test("返回结果是否JSON格式化", function () {
  pm.expect(function () {
    JSON.parse(pm.response.text());
  }).not.to.throw();
});

 const responseData = pm.response.json();
 
pm.expect(responseData).to.have.property('id');
pm.expect(responseData).to.have.all.keys('jsonrpc', 'id', 'result');
pm.expect(responseData).to.be.an('object');
pm.expect(responseData.id).to.exist.and.to.be.a('number').and.to.satisfy(Number.isInteger, "Value should be an integer");
pm.expect(responseData.result).to.not.be.null;
pm.expect(pm.response.responseTime).to.be.below(200);
pm.expect(pm.response.headers.get('Content-Type')).to.equal('application/json');
 pm.expect(responseData.result).to.be.a('boolean');

```

